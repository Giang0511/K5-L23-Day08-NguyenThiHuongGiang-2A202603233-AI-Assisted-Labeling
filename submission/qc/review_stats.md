# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 683; đối tượng sau duyệt: 644; ngưỡng accepted: IoU >= 0.9
- Đã **loại 22 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 598 | 598/683 = 87.6% | 683 box máy |
| edited (sửa) | 7 | 7/683 = 1.0% | 683 box máy |
| deleted (xoá) | 76 | 76/683 = 11.1% | 683 box máy |
| added (vẽ thêm) | 39 | 39/644 = 6.1% | 644 đối tượng sau duyệt |

Trong số deleted + added, **11 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 28.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 406 | 2 | 41 | 3 |
| bicycle | 6 | 0 | 2 | 6 |
| motorbike | 132 | 3 | 4 | 8 |
| car | 37 | 0 | 1 | 15 |
| bus | 2 | 0 | 1 | 1 |
| truck | 4 | 0 | 1 | 5 |
| three_wheeler | 11 | 2 | 26 | 1 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 59 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 48 | 41 | 11 | 53.9% (KTC 95%: 44–64%) | 81.4% (KTC 95%: 70–89%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 8/12 = 66.7% (KTC 95%: 39–86%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 1/3
  - extra: 2/3
  - wrong_class: 3/3
  - loose: 2/3
  - lọt: img_028.jpg (missing), img_005.jpg (missing), img_013.jpg (loose), img_004.jpg (extra)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
