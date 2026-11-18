# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202604058 - Trần Nguyễn Phi Long

> Chỉ phần thay đổi so với v1.0. Mỗi quyết định khớp một dòng trong `qc/calibration_decisions.csv`.
> Các quy tắc dưới đây đã được tôi trực tiếp áp dụng và kiểm tra trong `labels/gold_manual.json` và `labels/batch_reviewed.json`.
> YOLO26, YOLOE và SAM được sử dụng để hỗ trợ pre-label và segmentation. Tôi trực tiếp gán nhãn gold, review, chỉnh sửa annotation, thực hiện calibration và cập nhật guideline

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện: bicycle hay motorbike? | **Chưa chốt**: không gặp ca nào rõ ràng trong 28 ảnh đã gán | — | — |
| 2 | Khối xe máy đỗ dày gần camera: từng xe hay `ignore_region`? | Tiêu chí là **chiều cao từng vật + tách được bằng mắt**, không phải mật độ. Xe gần tách được → gán từng xe. Cụm ≥ 5 vật cao < 20 px, hoặc vùng nhoè chuyển động không tách được → một `ignore_region` | img_026, img_028 (từng xe); img_036, img_024 (ignore) | CAL-02 |
| 3 | Hàng nhô ra: box xe có gồm hàng không? | Box xe ôm thân xe + giỏ/khay **gắn sát**; hàng hoặc khay nhô quá ~20% chiều dài xe **không** vào box (mask cũng vậy); tick `can_review` | img_009 (xe máy chở bưởi), img_031 (xe đạp có khay phía trước) | CAL-01 |
| 4–7 | Người trong xe có mui, kính xe, box chồng ban đêm, conf tối thiểu | **Chưa chốt** ở v1.1 (không đủ ca trong lô này để quyết) | — | — |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_001, img_002 | Bán tải / SUV | YOLO26 gọi `truck` (0.73–0.88) | `car` | Không đổi chữ (mục 2 đã ghi bán tải = car); thêm ví dụ — CAL-03 |
| img_010, img_027, img_035 | Xích lô | YOLO26: bánh sau → `bicycle`, cả xe → `motorbike`; YOLOE tách 1 xe thành 2 box | 1 box `three_wheeler` ôm mui + 3 bánh; người đạp là `person` riêng | Mục 2, lớp `three_wheeler`, cột "Không gồm": *không tách bánh / khung xích lô thành `bicycle`* — CAL-04 |
| img_018, img_024, img_034 | Xe van chở khách | YOLOE gọi minibus là `three_wheeler`; YOLO26 gọi van là `truck` | Minibus/van chở khách cỡ lớn → `bus`; minivan/van nhỏ → `car`; không chắc → `can_review` | Mục 2: lớp `bus` thêm *minibus / van chở khách cỡ lớn*; lớp `car` thêm *van nhỏ* — CAL-05 |
| img_008 | Xe đẩy trái cây dùng bánh xe đạp | YOLOE gọi `three_wheeler` (0.25–0.90), một box gộp 2 xe | Tạm `three_wheeler` + `can_review` | **Câu hỏi mở mới số 8** (chưa chốt): xe đẩy tay có phải phương tiện không — CAL-06 |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu 2 (cụm dày) và câu 3 (hàng nhô); thêm ranh giới bus/car cho van; xích lô không tách bicycle; thêm câu hỏi mở số 8 (xe đẩy) | CAL-01, CAL-02, CAL-04, CAL-05, CAL-06; CAL-03 và CAL-07 không đổi chữ guideline |
