# Báo cáo Day 08 — 2A202603183 - Hoàng Nhật Anh

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26n (`r05`) | 0,30 (mặc định) | Nguồn box detector đóng cho sáu lớp. Lượt `r01` và `r02` so sánh ngưỡng trên pilot, `r05` cung cấp dự đoán cho toàn bộ ảnh. |
| three_wheeler | YOLOE-26s (`r06`), prompt `tuk-tuk`, `cyclo`, `cargo tricycle` | 0,30 (mặc định) | Nguồn box xe ba bánh. `r03` sinh 5 box trên 8 ảnh pilot, `r06` là đầu vào quét ngưỡng trên Gold. |

- Tham số gộp đã dùng: `scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --conf 0.30 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`.
- **Phiên bản SAM đã chạy:** MobileSAM (SAM 1, `mobile_sam.pt`) trên CPU. Trọng số MobileSAM có sẵn và chạy được trên CPU. Đã kiểm lại bằng tọa độ box Batch và vùng mask trả về, với giới hạn nêu ở mục 5.
- Ở img_013, YOLO26n gán `truck` với conf 0,7837 cho box trùng vùng nhãn `three_wheeler` đã duyệt (IoU 0,940). Thay ngưỡng chỉ giữ hoặc loại box. Lớp đúng cần nguồn YOLOE hoặc sửa khi duyệt.
- Box điểm cao khác nhãn Batch đã duyệt: ví dụ img_013 ở dòng trên.
- Dữ liệu quét ngưỡng: 4 ảnh Gold, mục tiêu recall 90%. `bus` có 1 vật Gold và `truck` có 0 vật Gold. Năm lớp còn lại đều có recall tối đa dưới 90% trong dải xét, nên lệnh gộp dùng 0,30 cho cả 7 lớp. Bốn ảnh đã được dùng để khảo sát ngưỡng, nên điểm trên tập này chỉ mô tả mẫu Gold, chưa xác nhận khả năng tổng quát hóa trên ảnh mới.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 153 | 153/231 = 66,2% | 231 box máy |
| edited | 58 | 58/231 = 25,1% | 231 box máy |
| deleted | 16 | 16/231 = 6,9% | 231 box máy |
| added | 207 | 207/418 = 49,5% | 418 đối tượng sau duyệt |

- Theo `qc/threshold_sweep.md`, `bicycle` khớp 0/5 vật Gold ở mọi ngưỡng được quét. Theo `qc/review_stats.md`, `person` có số box vẽ thêm lớn nhất trên Batch (113). Nguyên nhân cụ thể theo từng ảnh cần đối chiếu vật thật với box.
- Theo `qc/review_stats.md`, precision của máy trên Gold là 24/35 = 68,6% và recall là 24/50 = 48,0%. Bảng Batch loại 24 đối tượng quanh vùng bẫy theo script.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Lượt đầu bắt 7/12 = 58,3%. Theo loại: bỏ sót 1/3, sai lớp 1/3, box lỏng 2/3, box thừa 3/3. Bẫy lọt ở img_031 và img_005 (bỏ sót), img_024 và img_017 (sai lớp), img_034 (box lỏng). Không có dữ liệu thao tác từng ảnh để xác định chính xác bước nào đã bị bỏ qua.
- Đối chiếu để học viên phân tích: giữ nguyên 153/231 box máy (66,2%) và bắt 7/12 bẫy (58,3%). Hai số đo dùng mẫu số khác nhau.

## 4. Máy có đáng dùng không?

- Theo `time_log.csv`, gán tay 15 phút / 4 ảnh = 3 phút 45 giây/ảnh. Duyệt Batch 50 phút / 24 ảnh = 2 phút 05 giây/ảnh. Gold và Batch khác về ảnh, số lượng ảnh và thứ tự thao tác. Chênh lệch thời gian giữa hai lượt chưa tách được riêng tác động của pre-label.
- Đánh giá kỹ thuật: pre-label có thể làm điểm xuất phát. Tốc độ duyệt quan sát được là 2 phút 05 giây/ảnh, nhưng 207/418 đối tượng phải vẽ thêm và 5/12 bẫy lọt. Đặc biệt cần quét độc lập lớp person và motorbike vì hai lớp này có nhiều đối tượng vẽ thêm.
- **Ba bước cho vòng duyệt lại, theo checklist của bài:**
  1. Ẩn toàn bộ box và quét bốn mép ảnh trước khi hiện nhãn máy.
  2. Kiểm lớp từng box xe ba bánh theo guideline, kể cả box có conf cao.
  3. Trước khi sang ảnh kế tiếp, làm riêng một lượt tìm vật sót rồi rà viền các box bị che hoặc cắt mép.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013.jpg | MobileSAM (SAM 1) | 2 dương, 1 âm | 24.844/245.541 pixel mask nằm trong box person #109 | 245.541 pixel, 2 polygon. Cần xem biên tiếp giáp xe và người. | `tuktuk_img013_sam1.jpg` |
| img_004.jpg | MobileSAM (SAM 1) | 2 dương, 1 âm | 3.741/93.702 pixel mask nằm trong box person #23 | 93.702 pixel, 1 polygon. Một phần nhỏ vượt box xe mục tiêu. | `tuktuk_img004_sam1.jpg` |
| img_017.jpg | MobileSAM (SAM 1) | 2 dương, 1 âm | 0/33.889 pixel mask nằm trong các box person đã duyệt | 33.889 pixel, 2 polygon. Toàn bộ mask nằm trong box xe mục tiêu. | `tuktuk_img017_sam1.jpg` |

- Theo mục 4.2, mask bám viền phần nhìn thấy. Cả ba điểm âm nằm ngoài mask. Phần mask chồng box person chỉ khoanh vùng cần kiểm vì box cũng chứa nền hoặc thân xe. Các số trên chưa xác nhận mask dính người lái hay hàng hóa theo đường biên pixel.
- Interactor SAM trong CVAT là bước tuỳ chọn. Ba ảnh trên được tạo bằng script cục bộ.

## 6. Calibration (Part 6)

- Có 3 ca calibration trong `qc/calibration_decisions.csv`, mã CAL-01 đến CAL-03. Các quy tắc cho vòng duyệt lại bám mục 2, 3 và 4 của v1.0 và checklist review.
- `guideline_v1.1_calibrated.md` ghi cách phân biệt xe ba bánh, cách dùng conf và bước quét mép ảnh.
- CAL-01 xuất phát từ khác biệt giữa lớp máy và lớp sau duyệt. CAL-02 là câu hỏi mở về conf. CAL-03 xuất phát từ chênh lệch giữa điểm bẫy và lần kiểm ảnh QC.
- Ca có dữ liệu để kiểm: `img_013.jpg` có box `truck` của YOLO26n khớp vị trí với box `three_wheeler` đã duyệt (IoU 0,940). `img_031.jpg` có bẫy bỏ sót `person` còn lọt ở lượt đầu. Trong QC, học viên không ghi nhận lỗi trên img_031.

## 7. QC và quyết định (Part 6)

- Mẫu theo `qc/qc_sample.csv`: 5/24 ảnh Batch, 78 đối tượng. Gồm 0 ảnh `can_review` và 5 ảnh ngẫu nhiên với seed 42: `img_003`, `img_006`, `img_014`, `img_031`, `img_035`. Người kiểm là học viên đã duyệt Batch, do đó đây là phép tự kiểm trên một mẫu nhỏ.
- Học viên đã kiểm cả 5/5 ảnh trong mẫu và không ghi nhận lỗi.
- Theo `qc/qc_report_auto.md`, lỗi theo **đối tượng** là 0/78 = 0,0% và theo **ảnh** là 0/5 = 0,0%. Ngưỡng Release theo ảnh là ≤ 2%.
- Phiếu QC không ghi nhận lỗi trong 5 ảnh mẫu. Kết quả bẫy của lượt duyệt đầu được ghi riêng ở mục 3.
- **Giới hạn mẫu nhỏ:** với 0/5 ảnh lỗi, cận trên của khoảng tin cậy Wilson 95% là 43%. Mẫu này chưa đủ để chứng minh tỷ lệ ảnh lỗi của cả lô ≤ 2%.
- Script tổng hợp phiếu QC và số liệu bẫy. Việc đánh giá viền, lớp và vật bị cả máy lẫn người bỏ sót cần đối chiếu ảnh.

- **Gợi ý tự động:** Rework vì lượt đầu bắt 7/12 bẫy (58,3%), dưới ngưỡng 80%.

**Quyết định kỹ thuật: Rework (17/11/2026).** Dựa trên ngưỡng của bài và báo cáo QC tự động. Mẫu QC ghi 0/5 ảnh lỗi, còn lượt review Batch bắt 7/12 bẫy (58,3%), dưới ngưỡng 80%.
Đã sửa lỗi kích thước qua `labels/qc_fixed.json`: bỏ box car #308 cao 17,1 px trong img_027 theo ngưỡng 20 px. Bản ứng viên sau sửa có 24 ảnh, 432 đối tượng. img_027 nằm ngoài mẫu QC nên mẫu 5 ảnh và 78 đối tượng giữ nguyên. Giữ số đo bẫy lượt đầu làm mốc và rà lại các ca lọt trong vòng Rework.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** Codex hỗ trợ đọc hướng dẫn, chạy script cục bộ, đối chiếu đầu ra và chép số liệu. Ở Part 6, chạy YOLOE cục bộ trên 5 ảnh QC và dùng hai box của img_003 làm prompt độc lập cho MobileSAM. Hai phép kiểm tạo ghi chú trong qc_review.csv. Học viên đã xác nhận không phát hiện lỗi ở cả 5 ảnh QC. Codex chuẩn hóa bản ghi calibration theo guideline và checklist có sẵn, tính số pixel mask chồng với box và tạo bản sửa QC theo ngưỡng kích thước. Quyết định Rework được ghi theo tiêu chí tự động của bài; nhận định trực tiếp của học viên là kết quả kiểm 5 ảnh QC.
- **Gold có tiếp xúc với tư vấn AI:** ảnh xe máy hỏi Codex về Occluded trước khi báo hoàn thành Gold được đối chiếu cục bộ và khớp img_001.jpg thuộc Gold. Phản hồi đề cập cả lớp motorbike và phạm vi box. Vì vậy không coi Gold hoàn toàn độc lập với AI. Các số đo trên Gold chỉ mang tính tham khảo.
- Windows 11 Home Single Language, PyTorch CPU, Docker CVAT v2.81.0.
- Các đầu vào Part 3–4 được tạo từ các lượt chạy thật `r05` và `r06` trong repo.
- Lượt YOLOE chạy xong và có cảnh báo thư viện về `torch.jit.load`. Cảnh báo box 17,1 px còn trong file Batch gốc để bảo toàn lượt đầu. Bản ứng viên 432 đối tượng đã qua kiểm tra kích thước.
- Ghi chú cá nhân cho lần sau: học viên bổ sung bằng trải nghiệm duyệt của mình.
