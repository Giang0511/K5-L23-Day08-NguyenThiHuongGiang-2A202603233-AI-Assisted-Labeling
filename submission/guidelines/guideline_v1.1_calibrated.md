# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603233 - Nguyễn Thị Hương Giang

> **Chỉ viết phần thay đổi so với v1.0.** Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao) và thêm quy tắc rút ra từ lỗi của máy và của tôi. Mỗi ca khó có ảnh ví dụ và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`? | Gán `motorbike` (`bicycle` chỉ là xe đạp đạp bằng chân) | img_025 | CAL-06 |
| 2 | Khối xe máy đỗ dày đặc gần camera: từng xe hay `ignore_region`? | Cụm xe sát nhau mà box từng xe cao dưới 20 px thì gộp thành một `ignore_region` (img_024: vùng 429.8x79.5 px ở đáy ảnh). Xe còn tách được và từ 20 px trở lên thì vẽ từng xe | img_024 | CAL-07 |
| 3 | Hàng nhô ra: box xe có gồm hàng không? Mask thì sao? | **Mask:** hàng nằm trên xe thì tô vào mask xe; phần hàng nhô ra quá ~20% chiều dài xe thì không tô vào mask xe. **Box:** theo 4.1.5 của v1.0 (hàng nhô quá ~20% chiều dài xe thì không tính vào box) | img_003 | CAL-02 |
| 4 | Người thấp thoáng trong xe ba bánh có mui: ngưỡng "còn thấy ≥ 20%" có hợp lý không? | **Chưa chốt:** chưa có ca đo được trong 24 ảnh batch | — | — |
| 5 | Kính chắn gió / cửa sổ xe có thuộc mask xe không? | Thuộc mask xe (theo 4.2.3). Chưa gặp ca nào trong 24 ảnh batch nên chưa có ảnh ví dụ | — | CAL-09 |
| 6 | Box máy chồng nhau quá nhiều (ảnh đêm): gộp hay giữ? | Xác định được biên từng vật thì giữ từng box; sát khít đến mức không xác định được biên thì gộp thành `ignore_region` | img_029 | CAL-08 |
| 7 | Mức conf tối thiểu để nhận box máy không cần sửa? | **Không đặt.** `conf` chỉ xếp thứ tự duyệt; mọi box đều phải nhìn ảnh. Lý do: img_014 box `bicycle` conf 0.88 nhưng thật là `motorbike`; img_028 box `person` conf 0.1 lại đúng. | img_014, img_028 | CAL-04 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_003, img_013, img_017 | Người ngồi trong khung xe (hoặc đứng sát xe) | SAM 2.1 (`sam2.1_b.pt`) tô lem vào người, khoảng 3% mask | Khi tô mask xe thì không tô người trong khung xe; người vẫn là `person` riêng; loại vùng người bằng click âm | Mục 4.2: vùng người trong khung xe không thuộc mask xe (khác "lỗ thủng" ở 4.2.2, vì đây là vật khác chứ không phải khoảng trống) (CAL-03) |
| img_003 | Hàng hoá nằm trên xe ba gác | SAM 2.1 tô cả trái cây trong thùng xe vào mask xe | Giữ: hàng nằm trên xe thuộc mask xe | Mục 4.2: hàng hoá nằm trên xe thuộc mask xe (CAL-02) |
| img_003, img_009, img_013, img_018 | Xe khác bị gán `three_wheeler` | YOLOE-26s gán `three_wheeler` cho xe tải, xe máy, ô tô, xe buýt (thật) | Không đổi guideline; sửa tay, đếm số bánh theo 4.3 | Không đổi (mục 2 và 4.3 đã đủ) (CAL-01) |
| img_025 | Xe máy điện / xe đạp điện nhỏ | Không áp dụng: đây là câu hỏi mở của guideline, không phải lỗi máy | Gán `motorbike` | Mục 2: xe máy điện và xe đạp điện thuộc `motorbike` (CAL-06) |
| img_024 | Cụm xe máy sát nhau, từng xe cao dưới 20 px | YOLO26m vẽ rất nhiều box chồng nhau | Gộp thành `ignore_region` | Mục 4.1.6 và mục 5 (CAL-07) |
| img_029 | Ảnh đêm, xe máy sát khít | YOLO26m vẽ nhiều box chồng nhau | Giữ từng box nếu xác định được biên; sát khít quá thì gộp thành `ignore_region` | Mục 4.1.6 (CAL-08) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu hỏi mở số 1, 2, 3, 5, 6, 7 (câu 4 chưa chốt: chưa có ca đo được); thêm quy tắc mask: không tô người trong khung xe vào mask xe, hàng nhô quá 20% không tô vào mask | CAL-02, CAL-03, CAL-04, CAL-06, CAL-07, CAL-08, CAL-09 |
