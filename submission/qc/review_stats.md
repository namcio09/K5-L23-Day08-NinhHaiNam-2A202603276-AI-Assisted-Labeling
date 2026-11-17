# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 534; đối tượng sau duyệt: 537; ngưỡng accepted: IoU >= 0.9
- Đã **loại 24 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 401 | 401/534 = 75.1% | 534 box máy |
| edited (sửa) | 55 | 55/534 = 10.3% | 534 box máy |
| deleted (xoá) | 69 | 69/534 = 12.9% | 534 box máy |
| added (vẽ thêm) | 81 | 81/537 = 15.1% | 537 đối tượng sau duyệt |

Trong số deleted + added, **10 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 71.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 229 | 19 | 22 | 30 |
| bicycle | 5 | 0 | 1 | 3 |
| motorbike | 108 | 23 | 14 | 20 |
| car | 43 | 4 | 6 | 15 |
| bus | 2 | 0 | 1 | 3 |
| truck | 3 | 0 | 3 | 4 |
| three_wheeler | 11 | 9 | 22 | 6 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 59 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 48 | 20 | 11 | 70.6% (KTC 95%: 59–80%) | 81.4% (KTC 95%: 70–89%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 3/3
  - wrong_class: 2/3
  - loose: 2/3
  - extra: 3/3
  - lọt: img_021.jpg (loose), img_028.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
