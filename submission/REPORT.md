# Báo cáo Day 08 — 2A202603276 - Ninh Hai Nam

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Lý do |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | Mặc định 0.3; riêng motorbike 0.25 | r05 là nguồn pre-label toàn bộ ảnh. Sweep trên 4 ảnh gold không đạt recall mục tiêu cho person/car; bicycle/bus/truck quá ít mẫu. Motorbike đạt recall 92% ở 0.25. |
| three_wheeler | YOLOE-26s, prompt tuk-tuk / cyclo / cargo tricycle | 0.15 | YOLO26 không có lớp riêng; sweep gợi ý 0.15, recall 100% trên 7 vật gold (ước lượng thô). Xem threshold_sweep.md và r06. |

- Lệnh gộp: `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "motorbike=0.25,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`.
- Dùng SAM 2.1 base (`sam2.1_b.pt`) tạo mask offline từ click. Mask phác vùng đối tượng được nhưng còn lẫn/rò tại người, hàng hóa hoặc vùng bị che; cần kiểm tra biên.
- YOLO26 nhận cyclo thành `bicycle` ở `img_032.jpg`. Đổi confidence chỉ giữ/bỏ box, không sửa lớp; dùng YOLOE với prompt `three_wheeler` và kiểm tra theo loại xe thật.
- Pilot YOLO26 có 215 box ở conf 0.25 (r01) và 139 box ở 0.5 (r02) trên cùng 8 ảnh. Log không lưu confidence từng box nên không gán ví dụ cụ thể là điểm cao/thấp; lỗi lớp cyclo là ví dụ định tính.
- Ngưỡng được chọn trên chính 4 ảnh gold nên số đo gold lạc quan; các lớp có ít vật có độ bất định lớn.

## 2. AI error profile

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---:|---:|---|
| accepted | 401 | 75.1% | 534 box máy |
| edited | 55 | 10.3% | 534 box máy |
| deleted | 69 | 12.9% | 534 box máy |
| added | 81 | 15.1% | 537 đối tượng sau duyệt |

`three_wheeler` có tỉ lệ giữ nguyên thấp nhất: 11/42 (26.2%), 9 edited, 22 deleted và 6 added. So với gold: precision 48/68 = 70.6% (KTC 95%: 59–80%); recall 48/59 = 81.4% (KTC 95%: 70–89%), chỉ trên 4 ảnh.

## 3. Thiên kiến tự động hóa và ảnh bẫy

Bắt được 10/12 bẫy = 83.3% (KTC 95%: 55–95%), đạt ngưỡng 80%. Theo loại: missing 3/3, wrong_class 2/3, loose 2/3, extra 3/3. Bẫy lọt: loose ở `img_021.jpg`, wrong_class ở `img_028.jpg`. Accepted cao không chứng minh không còn vật bỏ sót; hai bẫy nhắc kiểm đủ bốn cạnh box và xác minh lớp theo vật thật.

## 4. Máy có đáng dùng không?

Theo `time_log.csv`: gán tay 30/4 = 7.5 phút/ảnh; review 45/24 = 1.875 phút/ảnh (xấp xỉ 1.9). Review nhanh hơn nhưng có thể thiên lệch vì đã quen dạng cảnh sau khi gán gold. Kết luận: dùng model làm pre-label để người gán kiểm tra, không tự động chấp nhận. Có 401/534 box được giữ; thêm 81 đối tượng, trong đó 10 là đổi lớp và khoảng 71 là vật máy bỏ sót thật. Precision/recall gold vẫn là ước lượng lạc quan từ 4 ảnh.

Ba quy tắc giảm thiên kiến:
1. Quét ảnh ít nhất 15 giây trước khi hiện box; áp dụng 24/24 ảnh.
2. Rà bỏ sót → box thừa → sai lớp → box lỏng trên từng ảnh.
3. Lưu CVAT và nghỉ 3–5 phút sau mỗi khoảng 10 ảnh.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | Phiên bản | Click | Mask gồm người lái / hàng hóa? | Lỗi quan sát | Tệp |
|---|---|---:|---|---|---|
| img_013.jpg tuk-tuk | SAM 2.1 base | 3 | Lẫn một phần người lái/hàng hóa | Mask phủ rộng vào người và hàng; click âm chưa tách sạch | `masks/tuktuk_img013.jpg` |
| img_032.jpg cyclo | SAM 2.1 base | 3 | Tập trung phần xe/hàng, không lấy người đẩy | Rò nhẹ ở biên dưới | `masks/cyclo_img032.jpg` |
| img_005.jpg motorbike bị che | SAM 2.1 base | 3 | Chủ yếu phần xe; loại phần lớn người đi xe | Biên đuôi/vùng che chưa gọn | `masks/occluded_motorbike_img005.jpg` |

Điểm dương trên vật và điểm âm gần vùng cần loại giúp tách mask; một điểm âm chưa chắc loại hết người/hàng nên phải kiểm vùng tiếp xúc. Không ghi nhận đã thử SAM Interactor trong CVAT; mask được tạo offline bằng `sam2_points.py`.

## 6. Calibration (Part 6)

Có 3 quyết định trong `calibration_decisions.csv`. CAL-03 làm rõ cụm xe dày ở `img_018.jpg`: vẽ riêng xe phân biệt được và cao ít nhất 20 px; dùng một `ignore_region` cho cụm từ 5 xe trở lên khi từng xe dưới 20 px, sát nhau và khó tách. Guideline v1.1 cũng bổ sung kiểm bốn cạnh box và quy tắc gán cyclo thành `three_wheeler`. Nguyên nhân: nhầm lớp địa phương ở `img_032.jpg`, quy tắc cụm dày chưa rõ, và box lỏng lọt ở `img_021.jpg`.

## 7. QC và quyết định

Mẫu QC 5/24 ảnh batch, seed 42; 106 đối tượng. AI hội thoại hỗ trợ xem ảnh mẫu và ghi nhận lỗi theo yêu cầu; người học quyết định cách xử lý. Có 3/106 đối tượng lỗi (2.8%) và 2/5 ảnh lỗi (40%): hai box trùng ở `img_003.jpg` (#18, #19), một box person nhầm mũ trưng bày ở `img_031.jpg` (#481). Cận trên Wilson 95% của tỉ lệ ảnh lỗi là 77%; mẫu 5 ảnh không chứng minh được mức lỗi thật ≤2%.

**Quyết định: Rework**, ngày 2026-11-17. Tỉ lệ ảnh lỗi 2/5 vượt ngưỡng Release ≤2%; script cũng khuyến nghị Rework do lỗi thừa lặp lại. Theo quyết định của người học, ba lỗi chưa được sửa; release hiện tại chưa được xác nhận đạt QC. Muốn xét Release cần sửa nhãn, tạo qc_fixed.json, chạy lại build_release.py và lấy mẫu QC mới.

## 8. Ghi chú cá nhân

- AI hội thoại hỗ trợ giải thích quy trình và xem ảnh QC theo yêu cầu; người học quyết định tiếp tục mà chưa sửa lỗi QC.
- Inference toàn bộ đã được chạy trước khi hoàn tất gold; kết quả dùng `--no-viz` và không nhập vào task gold. Gold được gán thủ công. Hãy xác nhận câu này khớp thao tác thực tế trước khi nộp.
- Dùng Windows laptop cho CVAT/tác vụ nhẹ; inference trên Google Colab Tesla T4 (`cuda`). Phiên bản CVAT chưa ghi nhận.
- Không dùng `data/cache/`; model chạy trên Colab.
- Vướng mắc: thiếu numpy ở Python local; Colab disconnect/reset thư mục làm việc. Lần sau hoàn tất gold trước inference toàn bộ, sao lưu output Colab sau từng giai đoạn và xử lý QC trước khi build release.
