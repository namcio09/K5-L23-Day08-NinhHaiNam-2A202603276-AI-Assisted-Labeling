# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603276

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Cụm xe máy đỗ dày gần camera: từng xe hay `ignore_region`? | Vẽ riêng từng xe nếu phân biệt được và box cao ít nhất 20 px. Với cụm từ 5 xe trở lên, mỗi xe cao dưới 20 px, sát nhau và khó tách, dùng một `ignore_region` bao cụm; không gộp các xe nhìn rõ chỉ vì ảnh đông. | `img_018.jpg` | CAL-03 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_032.jpg` | Cyclo bị nhầm với xe đạp thường | YOLO26 gán `bicycle`; cyclo thuộc lớp `three_wheeler` | Xác định lớp theo loại xe nhìn thấy; cyclo là `three_wheeler` | Mục 2: phân biệt `bicycle` với `three_wheeler` theo loại xe |
| `img_021.jpg` | Bẫy box motorbike lỏng vẫn lọt trong kết quả cuối | Bẫy tạo box lỏng; kết quả cuối chưa sửa đủ | Kiểm tra đủ bốn cạnh trước khi chấp nhận, sửa box lỏng dù nhãn đúng | Mục 4.1.1: kiểm tra bốn cạnh khi review |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Làm rõ cách xử lý cụm xe máy dày gần camera; phân biệt cyclo với xe đạp; thêm nhắc kiểm tra bốn cạnh box khi review | CAL-01, CAL-02, CAL-03 |
