# Báo cáo Day 08 — 2A202604058 - Trần Nguyễn Phi Long

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike | YOLO26m | person 0.2, bicycle 0.2, motorbike 0.2 (threshold_sweep) | r01/r02: tăng 0.25→0.5 bỏ 215→139 box, mất vật nhỏ thật; sweep trên gold: recall ≥ 90% ở 0.2 |
| car / bus / truck | YOLO26m | mặc định 0.3 | car: không ngưỡng ≤ 0.5 nào đạt recall 90% (tối đa 88%, 8 vật); bus 1 vật, truck 0 vật trong gold → không đủ mẫu để chọn riêng |
| three_wheeler | YOLOE-26s, prompt `tuk-tuk, cyclo, cargo tricycle` | 0.15 | r01: YOLO26 không có box cho tuk-tuk img_002; r03: YOLOE tìm đúng tuk-tuk 0.93. Sweep: recall 100%, precision 78% (7 vật gold) |

- Lệnh `merge_prelabels.py` đã dùng:

  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.2,bicycle=0.2,motorbike=0.2,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json` → 722 box.

- **Phiên bản SAM:** SAM 2.1 base (`sam2.1_b.pt`). Lý do: có sẵn trong bộ `core`, không cần HF token như SAM 3, chạy được trên CPU (r04: 20.7 s/ảnh trên Colab CPU). Không thử SAM 1 / SAM 3, nên không có so sánh chất lượng giữa các phiên bản.

- **Lỗi không sửa được bằng ngưỡng:** tuk-tuk / xích lô / minibus bị gán sai lớp với điểm cao: img_001 Ford Ranger → `truck` 0.88; img_002 bán tải → `truck` 0.73; img_034 minibus → `three_wheeler` 0.85 (YOLOE). Tăng ngưỡng lên 0.5 (r02) không xoá được các box này. Cách sửa: dùng YOLOE cho `three_wheeler` và đặt ưu tiên trước YOLO26 khi gộp, sau đó người duyệt đổi lớp (`review_stats`: 10 trường hợp đổi lớp).

- **Điểm cao mà sai:** img_001 `truck` 0.88 (thật ra là car); img_034 `three_wheeler` 0.85 (thật ra là minibus/car). **Điểm thấp mà đúng:** người nhỏ ở xa trong img_001 với điểm 0.21–0.30 (#28, #29, #30, #32); img_010 xích lô thứ hai `three_wheeler` 0.16 (đúng xe, chỉ cần nới box).

- **Giới hạn:** ngưỡng được chọn và đánh giá trên cùng 4 ảnh gold nên kết quả có thể lạc quan do mẫu nhỏ. **Ngoài ra gold KHÔNG độc lập với máy:** tôi tạo gold bằng cách chỉnh từ pre-label (xem mục 8), nên mọi con số "máy so với gold" và việc chọn ngưỡng trên gold đều bị lệch có lợi cho máy.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 542 | 87.8% | 617 box máy |
| edited | 11 | 1.8% | 617 box máy |
| deleted | 61 | 9.9% | 617 box máy |
| added | 18 | 3.1% | 572 đối tượng sau duyệt |

(Đã loại 26 đối tượng quanh vùng bẫy khỏi bảng này. Trong deleted + added có 10 ca là **đổi lớp**, nên số vật máy sót thật ≈ 18 − 10 = 8.)

- **Lớp tệ nhất: `three_wheeler`.** 40 box máy, chỉ 9 accepted, 8 edited, 23 deleted. Nguyên nhân thấy trong ảnh:

  - YOLOE gộp 2 xe thành 1 box (img_003, img_008, img_012, img_017, img_023);

  - tách 1 xe thành 2 box (img_010, img_013, img_035);

  - gán `three_wheeler` cho sạp hàng, quang gánh, xe tải, minibus (img_003, img_009, img_014, img_018, img_034).

- **So với gold:** precision / recall của **máy** là 74/82 = 90.2% (KTC 95%: 82–95%) và 74/77 = 96.1% (KTC 95%: 89–99%). Mẫu số là 77 đối tượng gold. **Không nên đọc đây là chất lượng máy:** gold được chỉnh từ chính pre-label này (73/78 box gold trùng pre-label đến 0.01 px; xem mục 8), nên precision / recall cao một phần là do so máy với bản đã sửa của chính nó. Gold chỉ có 4 ảnh và cũng được dùng để chọn ngưỡng.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- **Bắt bẫy: 9/12 = 75.0%** (KTC 95%: 47–91%), dưới ngưỡng 80%.

  - Theo loại: extra 3/3, wrong_class 3/3, missing 3/3, **loose 0/3**.

  - Bẫy lọt: img_021 (#321 motorbike), img_028 (#523 person), img_033 (#607 motorbike). Cả 3 là box lệch 7–14 px, đều ở ảnh đông (> 30 vật).

  - Bước bị bỏ: bước 4 "Lỏng" của checklist. Ở ảnh đông, tôi chủ yếu zoom để tìm vật thừa, sót hoặc sai lớp; chưa kiểm từng box có ôm sát không.

- **Accepted cao, bắt bẫy thấp:** accepted 87.8% nghĩa là phần lớn box máy được **giữ nguyên**. Giữ nguyên không có nghĩa là đã được kiểm.

  - Ba loại bẫy "có / không" (extra, missing, wrong_class) bắt được hết 9/9, vì chúng lộ ra ngay khi nhìn ảnh.

  - Ba bẫy `loose` chỉ lệch 7–14 px, nằm giữa hàng chục box hợp lý khác, nên lọt cả 3/3.

- **Liên hệ thiên kiến tự động hóa:** đây là kiểu lỗi "làm theo" (neo nhãn). Box máy trông đúng chỗ, đúng lớp, nên được chấp nhận mà không kéo lại. Lỗi nhỏ cộng với ảnh đông đúng là hai nguyên nhân trong bảng ở `docs/REVIEW_CHECKLIST.md` mục 1.

- Vì vậy con số accepted 87.8% **đánh giá cao quá mức** chất lượng máy: một phần trong số đó là box lỏng không bị phát hiện, và QC cũng tìm thêm 2 box lỏng nữa (img_008 #107, img_009 #126).

## 4. Máy có đáng dùng không?

- **Thời gian** (`time_log.csv`): xem số phút ghi trong `time_log.csv`. Gold được chỉnh từ pre-label chứ không vẽ từ đầu, nên thời gian `gold_manual` **không** phải tốc độ gán tay thuần và không dùng để so với tốc độ duyệt.

- **Kết luận: có, nhưng chỉ cho các lớp COCO phổ biến.** Tỉ lệ accepted theo lớp (mẫu số = box máy của lớp đó):

  - **Đáng dùng:** person 315/331 = 95.2%; motorbike 160/172 = 93.0%; car 48/53 = 90.6%. Sửa vài box nhanh hơn vẽ lại hàng trăm box ở ảnh đông.

  - **Không đáng dùng:** `three_wheeler` 9/40 = 22.5% (23 box bị xoá); bicycle 5/10 = 50%. Với hai lớp này, sửa box máy tốn công gần bằng vẽ mới, nên vẽ tay hoặc đổi recipe (prompt YOLOE, ngưỡng cao hơn kèm duyệt bắt buộc).

  - Máy sót ít: added 18/572 = 3.1%, trong đó 10 ca là đổi lớp, nên sót thật ≈ 8. Tuy vậy recall trên gold (96.1%) **không đáng tin**: gold chỉ có 4 ảnh, được dùng để chọn ngưỡng, và được chỉnh từ chính pre-label nên "máy sót" bị đánh giá thấp.

  - Không kết luận tốc độ thao tác của người chỉ từ `time_log`.

- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):

  1. **Ảnh > 30 vật: zoom theo 4 góc phần tư** và kiểm bước 4 "lỏng" cho mọi box cao > 50 px. Đo bằng: số box `edited` ở mỗi ảnh đông phải > 0 hoặc có ghi "đã kiểm, không lỏng"; vòng sau bắt được ≥ 2/3 bẫy `loose`.

  2. **Không accept box `three_wheeler` nào mà chưa zoom và đếm bánh.** Đo bằng: số box three_wheeler đã kiểm = số box three_wheeler của máy (40/40), ghi vào phiếu duyệt.

  3. **Quét ảnh trước khi hiện nhãn máy và ghi số vật đếm nhẩm theo lớp**, rồi so với số box cuối. Lệch > 10% thì duyệt lại ảnh đó. Đo bằng: một cột "đếm trước" trong phiếu duyệt; số ảnh phải duyệt lại.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 | SAM 2.1 base (Colab, CPU) | 3 (2 dương: mái, thân; 1 âm: tài xế) | Người lái **vẫn dính một phần** (chân, quần lốm đốm dù đã click âm vào áo) | Mask tràn qua khung hở sang người ngồi ở quán phía sau (áo hồng, x≈450–510) | `tuktuk_img013.jpg` |
| img_010 | SAM 2.1 base (local CPU) | 4 click + box (3 dương: mui, khung trái, khung phải; 1 âm: người đạp) | Người đạp **phần lớn bị loại**, còn lốm đốm trên áo khoác; không có hàng hoá | Mép phải mask lấn vào khung xích lô thứ hai phía sau; khoảng hở trong khung không bị khoét | `cyclo_img010.jpg` |
| img_009 | SAM 2.1 base (local CPU) | 3 click + box (2 dương: bánh trước, giỏ; 1 âm: đống bưởi) | Không có người. Hàng: **gồm** giỏ + túi trắng treo tay lái; **không gồm** bưởi trên giá sau (nhờ click âm); túi xanh treo hông chỉ phủ một phần | Bóng đổ không bị tính; chân chống được tính | `motorbike_cargo_img009.jpg` |

- **Đối chiếu guideline mục 4.2:**

  - **4.2.4 (hai vật chạm nhau):** bị vi phạm ở img_013 (mask xe dính chân tài xế, tràn sang người phía sau) và img_010 (lấn sang xích lô thứ hai). Người là vật riêng, nên mask xe không được chứa người.

  - **4.2.2 (lỗ thủng):** bị vi phạm ở img_010. Khoảng hở trong khung xích lô lớn hơn 5% diện tích nhưng không bị khoét.

  - **4.2.3 (bóng đổ):** được tuân thủ ở img_009 (bóng không vào mask).

  - **Guideline còn thiếu:** v1.0 không nói hàng hoá có thuộc mask không. Đã chốt ở v1.1 (CAL-01): hàng nhô > 20% không vào box hay mask, giỏ gắn sát thì vào.

- **Quy tắc rút ra:** mỗi người ngồi trên hoặc trong xe cần ít nhất 1 click âm. Xe có khung hở (xích lô, tuk-tuk) phải sửa polygon bằng tay sau SAM. Mask đúng hình không sửa được lớp sai (r04: SAM ra mask đẹp cho box YOLO26 gọi tuk-tuk là `car`).

- **(Tuỳ chọn)** Interactor SAM trong CVAT: không thực hiện. CVAT được sử dụng thông qua Docker trên máy local để thực hiện công việc annotation; riêng thao tác SAM click được chạy độc lập trên CPU local.

## 6. Calibration (Part 6)

- **Số quyết định:** 7 (`calibration_decisions.csv`: CAL-01 → CAL-07).

  - Có 3 ca `cau_hoi_mo`, 2 ca `loi_may`, 1 ca `can_review`, 1 ca `bay_lot`.

- **Quy tắc mới đã vào `guideline_v1.1_calibrated.md`:**

  - chốt câu 2 (cụm dày: tiêu chí là chiều cao 20 px và tách được bằng mắt);

  - chốt câu 3 (hàng nhô > 20% chiều dài xe không vào box);

  - ranh giới bus/car cho xe van;

  - xích lô không tách bánh xe thành `bicycle`;

  - thêm câu hỏi mở số 8 (xe đẩy trái cây).

- **Nguyên nhân gốc theo từng ca:**

  - guideline mơ hồ: CAL-01, CAL-02, CAL-05, CAL-06;

  - máy sai, guideline đã rõ: CAL-03, CAL-04;

  - người duyệt sai quy trình, không phải guideline: CAL-07.

- **Quan trọng nhất: CAL-02 (cụm xe dày đặc).** Nó quyết định cách gán 6 ảnh đông (005, 018, 021, 024, 028, 029), chiếm phần lớn trong 572 đối tượng sau duyệt. Đây cũng là nơi lọt cả 3 bẫy `loose` (CAL-07).

## 7. QC và quyết định (Part 6)

- **Mẫu:** 8 ảnh (3 ảnh `can_review` + 5 ảnh ngẫu nhiên, seed 42), 219 đối tượng.

  - Người kiểm: tôi, cũng là người đã duyệt lô. **Hạn chế:** người kiểm trùng người duyệt.

  - Lỗi ở img_021 chỉ được nhận ra sau khi đọc `review_stats.md`.

- **Tỉ lệ lỗi:**

  - theo **đối tượng**: 3/219 = 1.4%;

  - theo **ảnh**: 3/8 = 37.5%, vượt ngưỡng Release ≤ 2%.

- **Top lỗi:** `sai_vi_tri_kich_thuoc` chiếm 3/3 lỗi.

  - img_008 #107: box xe ba gác lấn sang xe bên cạnh;

  - img_009 #126: box người ôm cả bao hàng;

  - img_021 #239: xe máy lệch, là bẫy loose bị lọt.

  - Không gặp lỗi bỏ sót, thừa hay sai lớp trong mẫu.

- **Giới hạn mẫu nhỏ:** với 3/8 ảnh lỗi, cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là 69%. Ngay cả khi 0/8 ảnh lỗi, mẫu 8 ảnh cũng không chứng minh được "< 2%".

- **Script kiểm được / không kiểm được:**

  - Script kiểm được: tên lớp, box nằm trong ảnh, box < 20 px, đủ ảnh.

  - Script không kiểm được: box có ôm sát không; người duyệt có bỏ sót cùng chỗ với máy không; có làm đúng checklist không.

Gợi ý của `qc_report.py`: **REWORK** vì ảnh lỗi 37.5% > 2%, lỗi hệ thống `sai_vi_tri_kich_thuoc`, bắt bẫy 75% < 80%.

**Quyết định: Rework**, ngày 2026-11-17.

Lý do, đi theo cây Hold → Rework → Release:

- **Không Hold:** guideline v1.1 đã chốt các ca ảnh hưởng nhiều nhãn. Câu hỏi mở số 8 (xe đẩy) chỉ dính 3 box ở img_008, và dữ liệu không hỏng.

- **Rework:**

  - ảnh có lỗi 3/8 = 37.5%, vượt xa ngưỡng 2% (cận trên 95%: 69%);

  - lỗi có tính hệ thống: 3/3 lỗi đều là `sai_vi_tri_kich_thuoc`;

  - bắt bẫy 9/12 = 75% < 80%, nên bản thân lô duyệt chưa đáng tin.

**Việc cần làm tiếp:**

1. Duyệt lại 7 ảnh đông (005, 018, 021, 024, 028, 029, 033) theo quy tắc 1 ở mục 4, tập trung kiểm box lỏng.

2. Sửa 3 lỗi QC: img_008 #107, img_009 #126, img_021 #239.

3. Chốt câu hỏi mở số 8.

4. Chạy lại `build_release.py` và `qc_sample.py --force` để có mẫu QC mới, rồi ghi vòng 2 vào báo cáo.

## 8. Ghi chú cá nhân

- **Công cụ AI sử dụng:** YOLO26, YOLOE và SAM là các model được yêu cầu trong bài lab. Tôi trực tiếp chuẩn bị dữ liệu, chạy model, xem kết quả, chỉnh gold, duyệt và chỉnh sửa nhãn batch, thực hiện calibration và QC.

  - **Gold (4 ảnh) được chỉnh từ pre-label, không vẽ tay từ đầu.** Tôi xoá box thừa, thêm box thiếu và sửa lại box lệch trên pre-label của máy. Kiểm bằng số: 73/78 box gold trùng pre-label đến 0.01 px; so với pre-label, 8 box bị xoá và 3 vật được thêm (`review_stats.md` mục 2).

  - Tôi trực tiếp duyệt 24 ảnh batch, xoá / đổi lớp / thêm / sửa box.

  - Tôi tự chạy các script đo, điền `model_log`, `qc_review.csv`, `calibration_decisions.csv` và cập nhật guideline.

  - Tôi mở `traps_key.json` sau khi export `batch_reviewed.json` và không sửa 3 box loose sau khi biết đáp án.

  - **Cách kiểm:**

    - chạy `validate_labels.py --all` (sạch lỗi);

    - xem lại ảnh `work/qc/review/*.jpg` của 8 ảnh QC;

    - đối chiếu mọi con số trong báo cáo với `qc/review_stats.md`, `qc/qc_report_auto.md` và `qc/threshold_sweep.md`;

    - kiểm 3 ảnh mask bằng mắt.

- **Gold có bị nhiễm không:** **Có.** Gold được chỉnh từ pre-label của máy nên bị neo vào quyết định của máy (đúng kiểu thiên kiến tự động hóa mà bài lab cảnh báo). Hệ quả:

  - precision / recall "máy so với gold" (90.2% / 96.1%) là lạc quan, không phải chất lượng máy thật;

  - ngưỡng theo lớp ở mục 1 được chọn trên gold này nên cũng lệch;

  - `time_log.csv` bước `gold_manual` là thời gian chỉnh pre-label, không phải thời gian vẽ tay.

  Cùng 4 ảnh này còn được dùng để chọn ngưỡng và đánh giá nên rủi ro lạc quan do mẫu nhỏ càng cao. Tôi không thể tách lại gold sạch trong bài này.

- **Máy / GPU / CVAT:**

  - Windows 11, không GPU.

  - Model chạy trên Google Colab, nhưng runtime là **CPU** (cột `device` trong `work/runs.csv`), do chưa bật GPU.

  - SAM click ở Part 5 chạy trên CPU máy local.

  - Có sử dụng CVAT triển khai bằng Docker trên máy Windows 11 local để thực hiện annotation, review và chỉnh sửa nhãn.

- **Dùng cache `data/cache/`?** Không; dùng output tự chạy trên Colab (r01–r06).

- **Lỗi / bug script:** không sửa script nào của lab.

- **Một điều sẽ làm khác lần sau:** gán gold từ đầu **trước khi xem pre-label** (ẩn pre-label của ảnh gold trong CVAT), tăng số ảnh gold độc lập với tập dùng để chọn ngưỡng và bật GPU T4 trên Colab để rút ngắn thời gian chạy.
