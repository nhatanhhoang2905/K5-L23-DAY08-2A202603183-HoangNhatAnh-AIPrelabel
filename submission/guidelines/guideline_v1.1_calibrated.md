# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603183 - Hoàng Nhật Anh

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

> Các bổ sung dưới đây được dùng cho vòng Rework. Chúng dựa trên guideline v1.0, checklist và các ca đã ghi trong phiếu calibration. Các câu hỏi mở 1–6 ở v1.0 chưa có quyết định bổ sung trong vòng này.

## 1. Câu hỏi mở (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 7 | Có mức conf đủ để giữ nguyên box? | Kiểm lớp và vị trí trên ảnh dù conf cao. Dùng conf để ưu tiên thứ tự duyệt. | img_013 | CAL-02 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_004 | Xe ba bánh có thùng hoặc mui | YOLO26n gán truck. Box khớp vị trí với nhãn three_wheeler đã duyệt (IoU 0,940). | Xác định lớp theo cấu tạo xe và số bánh nhìn được. Nếu thiếu dấu hiệu, đánh dấu can_review. | Mục 2 và 4.3 |
| img_031 | Ca bẫy người bị cắt ở mép trên còn lọt | Lượt đầu thiếu box person theo khóa bẫy. Trong lượt QC sau đó, học viên đã xem lại và không ghi nhận lỗi trên ảnh này. | Quét bốn mép ảnh trước khi hiện nhãn máy. Áp dụng ngưỡng kích thước và phần nhìn thấy như v1.0. | Mục 4.1 và checklist review |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Bổ sung cách dùng conf, cách rà lớp xe ba bánh và lượt quét mép ảnh. | CAL-01, CAL-02, CAL-03. |
