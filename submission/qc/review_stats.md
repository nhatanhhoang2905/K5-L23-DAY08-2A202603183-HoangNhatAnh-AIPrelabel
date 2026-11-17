# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 231; đối tượng sau duyệt: 418; ngưỡng accepted: IoU >= 0.9
- Đã **loại 24 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 153 | 153/231 = 66.2% | 231 box máy |
| edited (sửa) | 58 | 58/231 = 25.1% | 231 box máy |
| deleted (xoá) | 16 | 16/231 = 6.9% | 231 box máy |
| added (vẽ thêm) | 207 | 207/418 = 49.5% | 418 đối tượng sau duyệt |

Trong số deleted + added, **1 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 206.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 77 | 30 | 6 | 113 |
| bicycle | 2 | 2 | 1 | 5 |
| motorbike | 35 | 24 | 5 | 53 |
| car | 23 | 0 | 0 | 33 |
| bus | 2 | 0 | 0 | 0 |
| truck | 0 | 0 | 1 | 0 |
| three_wheeler | 14 | 2 | 3 | 3 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 50 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 24 | 11 | 26 | 68.6% (KTC 95%: 52–81%) | 48.0% (KTC 95%: 35–61%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 7/12 = 58.3% (KTC 95%: 32–81%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 1/3
  - wrong_class: 1/3
  - loose: 2/3
  - extra: 3/3
  - lọt: img_031.jpg (missing), img_024.jpg (wrong_class), img_005.jpg (missing), img_034.jpg (loose), img_017.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
