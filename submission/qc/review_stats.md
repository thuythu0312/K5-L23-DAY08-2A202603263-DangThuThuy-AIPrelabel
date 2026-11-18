# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 647; đối tượng sau duyệt: 645; ngưỡng accepted: IoU >= 0.9
- Đã **loại 22 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 642 | 642/647 = 99.2% | 647 box máy |
| edited (sửa) | 1 | 1/647 = 0.2% | 647 box máy |
| deleted (xoá) | 4 | 4/647 = 0.6% | 647 box máy |
| added (vẽ thêm) | 2 | 2/645 = 0.3% | 645 đối tượng sau duyệt |

Trong số deleted + added, **0 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 2.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 294 | 0 | 0 | 0 |
| bicycle | 8 | 0 | 0 | 0 |
| motorbike | 239 | 0 | 2 | 2 |
| car | 54 | 0 | 0 | 0 |
| bus | 2 | 0 | 0 | 0 |
| truck | 6 | 0 | 0 | 0 |
| three_wheeler | 39 | 1 | 2 | 0 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 54 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 47 | 48 | 7 | 49.5% (KTC 95%: 40–59%) | 87.0% (KTC 95%: 76–94%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 0/12 = 0.0% (KTC 95%: 0–24%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 0/3
  - extra: 0/3
  - missing: 0/3
  - wrong_class: 0/3
  - lọt: img_003.jpg (loose), img_006.jpg (extra), img_008.jpg (missing), img_004.jpg (extra), img_017.jpg (wrong_class), img_021.jpg (missing), img_024.jpg (wrong_class), img_029.jpg (extra), img_031.jpg (loose), img_028.jpg (missing), img_034.jpg (wrong_class), img_010.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
