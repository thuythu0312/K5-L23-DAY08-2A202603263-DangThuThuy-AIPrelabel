# Báo cáo Day 08 — 2A202603263 - Đặng Thu Thúy

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn run_id) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | person 0.25; motorbike 0.10; các lớp còn lại 0.30 | r01–r02 cho thấy ngưỡng cao loại nhiều box; quét Gold chọn riêng person và motorbike. Bicycle và car không đạt mục tiêu recall, bus và truck không có mẫu Gold. |
| three_wheeler | YOLOE-26s (tuk-tuk, cyclo, cargo tricycle) | 0.15 | r03 cho thấy tìm được tuk-tuk bằng prompt; trên Gold đạt recall 6/6. |

- Lệnh đã dùng: `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.25,motorbike=0.1,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`.
- SAM 2.1 Base chạy được trên CPU và nhận điểm dương, điểm âm để thử biên mask.
- YOLO26 gọi tuk-tuk là car ở img_013.jpg. Cần dùng YOLOE hoặc sửa lớp khi review.
- img_002.jpg có dự đoán three_wheeler 0.37 chồng lên vùng xe bán tải trắng, cần kiểm lại dù qua ngưỡng.
- Cùng 4 ảnh được dùng để chọn và đo ngưỡng, nên kết quả Gold có thể lạc quan.

## 2. AI error profile (từ qc/review_stats.md)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---:|---:|---|
| accepted | 642 | 99,2% | 647 box máy |
| edited | 1 | 0,2% | 647 box máy |
| deleted | 4 | 0,6% | 647 box máy |
| added | 2 | 0,3% | 645 đối tượng sau duyệt |

- Trong các lớp có chỉnh sửa, three_wheeler có 3/42 box được sửa hoặc xóa; img_014.jpg vẫn có box nhầm xe đẩy tay.
- Gold: precision 47/95 = 49,5%; recall 47/54 = 87,0%.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt được 0/12 bẫy. Cả bốn loại (lệch, thừa, sót, sai lớp) đều 0/3. Bản COCO không cho biết người duyệt đã bỏ bước nào.
- 642/647 box ngoài vùng bẫy được giữ, trong khi không bẫy nào được sửa. Tỉ lệ giữ cao không đủ để kết luận nhãn đúng.

## 4. Máy có đáng dùng không?

- Gold 16:30–19:00, 150 phút/4 ảnh = 37,5 phút/ảnh. Batch 19:00–20:00, 60 phút/24 ảnh = 2,5 phút/ảnh (tính theo hai phần liên tục). Hai tập khác độ khó và Batch cần làm lại, nên chênh lệch này chưa đo được mức tiết kiệm thời gian ở cùng chất lượng.
- Có thể dùng pre-label làm bản nháp. Vẫn cần duyệt vì precision Gold chỉ 47/95 và kết quả bắt bẫy 0/12.
- **Ba quy tắc giảm thiên kiến**
  1. Ẩn box và quét toàn ảnh trước để tìm vật thiếu.
  2. Kiểm lần lượt box thừa, sai lớp và sai biên trước khi lưu mỗi ảnh.
  3. Ghi ca chưa chắc và kiểm tra một mẫu ngẫu nhiên sau khi xuất.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013.jpg | SAM 2.1 Base | 3 (2 dương, 1 âm) | Lấn vào người lái | Mask phủ thân, mui và bánh nhưng còn dính một phần người lái cạnh cabin. | tuktuk_img013.jpg |
| img_010.jpg | SAM 2.1 Base | 3 (2 dương, 1 âm), thêm 1 box | Loại phần lớn thân người lái | Mask tách được mui và phần lớn khung trước, nhưng biên sát người lái vẫn thiếu đều. | xichlo_img010.jpg |
| img_003.jpg | SAM 2.1 Base | 3 (2 dương, 1 âm) | Có lẫn hàng hóa và người ở sát thùng | Mask bám mui và thùng, nhưng chưa tách rõ hàng hóa khỏi mép thùng. | xe_chua_hang_img003.jpg |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? Mục 4.2 yêu cầu bám biên phần nhìn thấy, bỏ khoảng trống lớn và tách các vật chạm nhau. Hai ca xe có người và hàng sát khung cho thấy cần ví dụ rõ hơn về ranh giới giữa khung xe, người lái và hàng rời.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: chưa thử trong CVAT. Ba ảnh được tạo bằng script offline.

## 6. Calibration (Part 6)

- Có 3 quyết định trong `qc/calibration_decisions.csv`. CAL-02 làm rõ cách xử lý người bị mui xe che gần ngưỡng 20%.
- Guideline v1.1 bổ sung dấu hiệu nhận diện xe ba bánh có mui, cách ước lượng phần người nhìn thấy và cách tách hàng rời khỏi mask xe.
- Nguồn ca khó gồm lỗi phân lớp YOLO26, mask SAM lấn hàng rời và câu hỏi mở của v1.0.

## 7. QC và quyết định (Part 6)

- Mẫu gồm 5/24 ảnh ngẫu nhiên (seed 42), 129 đối tượng. Codex đối chiếu ảnh có box trên bản COCO cập nhật để thực hiện lượt QC sơ bộ.
- Ghi nhận 5/129 đối tượng có lỗi (3,9%) và 5/5 ảnh có lỗi (100%), vượt ngưỡng 2% theo ảnh. Đây là số lỗi đã ghi nhận trong lượt kiểm sơ bộ.
- Có 3/5 lỗi là box thừa, ví dụ box car trên khoảng tối dưới mái ở img_006.jpg. Có 2/5 lỗi là sai vị trí hoặc kích thước, ví dụ box bicycle thiếu bánh sau ở img_031.jpg.
- Cận trên Wilson 95% là 100% với 5/5 ảnh có lỗi. Mẫu 5 ảnh không đủ để chứng minh tỷ lệ lỗi toàn lô dưới 2%.
- Script đếm lỗi theo phiếu QC. Độ sát biên, nhãn đúng và vật bị bỏ sót cần đối chiếu với ảnh.

**Quyết định: Rework**, ngày 2026-11-17. Bản xuất cập nhật từ 24anhsua.zip có 5/5 ảnh mẫu có lỗi và 0/12 bẫy được sửa.
Duyệt lại 24 ảnh trong CVAT, ưu tiên phát hiện box thừa và kiểm tra biên. Sau khi lưu và xuất COCO mới, chạy lại thống kê review, tạo bản dữ liệu ứng viên và kiểm tra QC lần nữa.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Codex hỗ trợ chuẩn bị CVAT, chạy pre-label, thống kê, tạo ảnh SAM và thực hiện lượt QC sơ bộ trên 5 ảnh. Kết quả được đối chiếu với file COCO, ảnh mask và phiếu QC.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Trong phiên làm việc, dự đoán AI trên 4 ảnh Gold được chạy sau khi nhập bản COCO Gold. Học viên cần xác nhận thao tác ngoài phiên.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: CVAT local chạy bằng Docker, các lượt mô hình chạy CPU trong .venv.
- Dùng cache `data/cache/` thay vì tự chạy? Không. Các mô hình được chạy trực tiếp trên máy.
- Lỗi / bug script đã gặp (nếu có): YOLOE tự cài CLIP khi chạy lần đầu và sau đó hoàn tất.
- Điều cần thay đổi ở vòng sau: quét ảnh khi ẩn nhãn và ghi thời gian riêng cho từng phần.
