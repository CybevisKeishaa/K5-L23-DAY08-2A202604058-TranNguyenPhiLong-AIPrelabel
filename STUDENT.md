# Day 08 Lab — 2A202604058 - Trần Nguyễn Phi Long

## Thông tin học viên

- **Họ và tên:** Trần Nguyễn Phi Long
- **MSHV:** 2A202604058
- **Lớp:** L2
- **GitHub:** CybevisKeishaa
- **Repository:** https://github.com/CybevisKeishaa/K5-L23-DAY08-2A202604058-TranNguyenPhiLong-AIPrelabel
- **Commit chốt bài:** `162723d13e989d038a2c7fb8fc2056e41e9a3858` (bài nộp; commit sau đó chỉ cập nhật dòng này)

## Môi trường thực hiện

- **Hệ điều hành:** Windows 11.
- **Máy cá nhân:** Không có GPU rời.
- **Môi trường chạy model:** Google Colab.
- **CVAT:** Có cài đặt và sử dụng CVAT thông qua Docker trên máy local.
- **Quy trình:** Sử dụng các model AI để tạo pre-label, đưa kết quả vào CVAT, sau đó trực tiếp kiểm tra, chỉnh sửa và hoàn thiện annotation.

## Model đã sử dụng

Bài thực hành sử dụng đầy đủ 3 model theo yêu cầu:

| Model | Vai trò |
|---|---|
| YOLO26 | Object Detection, tạo bounding box cho các đối tượng thuộc danh mục cố định |
| SAM | Phân đoạn đối tượng (Segmentation) thông qua tương tác click |
| YOLOE | Open-vocabulary detection, tìm kiếm đối tượng bằng văn bản và tạo pre-label |

## Quy trình thực hiện

1. Chuẩn bị môi trường và dữ liệu theo yêu cầu của bài lab.
2. Chỉnh 4 ảnh gold từ pre-label (không vẽ tay từ đầu).
3. Sử dụng YOLO26, SAM và YOLOE để thực hiện các tác vụ pre-label và segmentation.
4. Sử dụng Google Colab để chạy model do máy cá nhân không có GPU.
5. Sử dụng CVAT chạy bằng Docker để quản lý dữ liệu annotation.
6. Import kết quả pre-label vào CVAT.
7. Trực tiếp review và chỉnh sửa bounding box, class và annotation.
8. Thực hiện calibration guideline, kiểm tra chất lượng nhãn và chuẩn bị dữ liệu đầu ra.

## Kết quả thực hành

Số liệu lấy từ `submission/qc/review_stats.md`, `qc_report_auto.md` và `release/release_manifest.json`.

- Tổng số ảnh đã xử lý: 28 (24 ảnh batch + 4 ảnh gold).
- Số lượng annotation sau review: 587 đối tượng trên 24 ảnh batch (gồm 3 `ignore_region`); gold: 78 đối tượng trên 4 ảnh.
- Số lượng box được giữ nguyên: 542/617 box máy = 87.8% (đã loại 26 đối tượng quanh vùng bẫy).
- Số lượng box được chỉnh sửa: 11/617 = 1.8%.
- Số lượng box bị xóa: 61/617 = 9.9%.
- Số lượng đối tượng được bổ sung: 18/572 = 3.1% (10 ca là đổi lớp, sót thật ≈ 8).
- Kết quả đánh giá trên gold: precision 74/82 = 90.2%, recall 74/77 = 96.1%. **Con số lạc quan, không phản ánh chất lượng máy thật:** gold được chỉnh từ pre-label (73/78 box trùng pre-label), chỉ 4 ảnh và cũng dùng để chọn ngưỡng (xem `REPORT.md` mục 8).
- Kết quả kiểm tra ảnh bẫy: bắt 9/12 = 75.0% (dưới ngưỡng 80%); lọt 3/3 bẫy `loose` (img_021, img_028, img_033).
- Kết quả QC: mẫu 8 ảnh / 219 đối tượng; lỗi theo đối tượng 3/219 = 1.4%, theo ảnh 3/8 = 37.5% (ngưỡng Release ≤ 2%).
- Quyết định Release / Hold / Rework: **Rework** (2026-11-17).

## Ghi chú

Tôi trực tiếp thực hiện bài thực hành, sử dụng ba model YOLO26, SAM và YOLOE theo yêu cầu. CVAT được triển khai bằng Docker và sử dụng làm công cụ quản lý, review và chỉnh sửa annotation.

**Lưu ý về gold:** 4 ảnh gold được chỉnh từ pre-label của máy, không vẽ tay từ đầu (nên gold không độc lập với máy). Chi tiết và hệ quả trong `submission/REPORT.md` mục 8.
