# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 617; đối tượng sau duyệt: 572; ngưỡng accepted: IoU >= 0.9
- Đã **loại 26 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 542 | 542/617 = 87.8% | 617 box máy |
| edited (sửa) | 11 | 11/617 = 1.8% | 617 box máy |
| deleted (xoá) | 61 | 61/617 = 9.9% | 617 box máy |
| added (vẽ thêm) | 18 | 18/572 = 3.1% | 572 đối tượng sau duyệt |

Trong số deleted + added, **10 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 8.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 315 | 2 | 14 | 4 |
| bicycle | 5 | 1 | 4 | 3 |
| motorbike | 160 | 0 | 12 | 2 |
| car | 48 | 0 | 5 | 2 |
| bus | 3 | 0 | 0 | 3 |
| truck | 2 | 0 | 3 | 2 |
| three_wheeler | 9 | 8 | 23 | 2 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 77 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 74 | 8 | 3 | 90.2% (KTC 95%: 82–95%) | 96.1% (KTC 95%: 89–99%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 9/12 = 75.0% (KTC 95%: 47–91%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 3/3
  - loose: 0/3
  - wrong_class: 3/3
  - missing: 3/3
  - lọt: img_021.jpg (loose), img_033.jpg (loose), img_028.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
