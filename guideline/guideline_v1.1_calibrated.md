# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603263 - Đặng Thu Thủy

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 4 | Người trong xe có mui bị che | Giữ mốc 20%; dựa trên phần đầu, thân và chi phân biệt được; nếu sát ngưỡng, ghi ca cần xem lại. | img_013.jpg | CAL-02 |

Các câu hỏi mở còn lại chưa được chốt trong vòng này vì chưa chọn đủ ảnh đối chiếu.

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013.jpg | Tuk-tuk có mui | YOLO26 gợi ý car | Gán three_wheeler | Mục 2. Dựa vào cấu trúc xe và số bánh nhìn thấy, kể cả xe có mui; nếu chưa chắc thì ghi ca cần xem lại. (CAL-01) |
| img_003.jpg | Hàng rời trên thùng xe | SAM 2.1 tô lẫn hàng | Tách hàng rời khỏi mask xe | Mục 4.2. Giữ mui và thùng gắn liền; bỏ phần hàng rời nhìn thấy, bám mép thùng nơi hàng che. (CAL-03) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Làm rõ xe có mui, ngưỡng nhìn thấy của người và mép mask ở thùng hàng | CAL-01, CAL-02, CAL-03 |
