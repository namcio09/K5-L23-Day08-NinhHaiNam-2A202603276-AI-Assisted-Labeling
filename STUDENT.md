# Thông tin học viên

* Họ và tên: Ninh Hai Nam
* MSHV: 2A202603276
* Lớp (L2 / L3 / khác): L2
* Tài khoản GitHub: namcio09
* Repo bài nộp (URL): https://github.com/namcio09/K5-L23-Day08-NinhHaiNam-2A202603276-AI-Assisted-Labeling
* Commit chốt bài (hash): Chưa chốt; ghi hash commit sau khi hoàn tất và commit hồ sơ
* Máy: Windows laptop (GPU local chưa xác nhận); suy luận dùng Google Colab Tesla T4, CUDA
* CVAT (Docker): Chưa ghi nhận phiên bản

## Kết quả tóm tắt

* Máy: accepted 401 / edited 55 / deleted 69 (mẫu số: 534 box máy); added 81 (mẫu số: 537 đối tượng sau duyệt)
* So với gold: precision 70.6% (48/68); recall 81.4% (48/59), trên 4 ảnh gold
* Bắt bẫy: 10/12 = 83.3%
* Thời gian: gán tay 7.5 phút/ảnh; duyệt có pre-label 1.875 phút/ảnh
* Lỗi QC: 40% theo ảnh (2/5); 2.8% theo đối tượng (3/106)
* **Quyết định:** Rework — mẫu QC có lỗi ở 2/5 ảnh, vượt ngưỡng Release; lỗi chưa sửa.

