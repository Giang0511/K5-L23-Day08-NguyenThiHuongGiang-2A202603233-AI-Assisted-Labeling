# Báo cáo Day 08 — 2A202603233 - Nguyễn Thị Hương Giang

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.
>
> **Bản nháp mục 1–4 đã điền đủ.** Đọc lại, sửa bằng lời của bạn và đối chiếu với `review_stats.md` trước khi nộp. Mục 5–8 là các Part sau.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26m | 0.1 | Ở 0.1 recall đạt 91% (≥ 90%) nhưng precision chỉ 41% (23 vật gold). Chấp nhận thừa box để không sót người, vì thừa dễ xoá hơn sót khó thấy. Run r02 (conf 0.25, 8 ảnh pilot) cho thấy nhiều box person chồng lấn, nên ngưỡng person thấp thì sẽ phải xoá bớt. Số 0.1 lấy từ `threshold_sweep` trên run r06 (YOLO26m, `--split all --conf 0.05`, 36 ảnh, 1669 box thô). |
| car | YOLO26m | 0.5 | Recall 90%, precision 69% (10 vật gold). So sánh r02 (conf 0.25) với r03 (conf 0.5) trên pilot; ngưỡng 0.5 cho `car` lấy từ `threshold_sweep` trên run r06. |
| motorbike | YOLO26m | 0.3 (mặc định) | Không ngưỡng nào ≤ 0.5 đạt recall 90% (tối đa 81%, 16 vật gold) nên dùng mặc định và nói rõ trong báo cáo. Run r03 (conf 0.5) làm mất một số xe máy ở xa ở img_007, nên không nâng ngưỡng. |
| bicycle / bus / truck | YOLO26m | 0.3 (mặc định) | Quá ít vật trong 4 ảnh gold (bicycle 2, bus < 5, truck 1) nên không chọn ngưỡng riêng. Run r03 (conf 0.5) làm mất box bicycle (0.25 ở img_015, 0.33 ở img_019) và truck 0.45 ở img_019, nên giữ ngưỡng thấp. |
| three_wheeler | YOLOE-26s, prompt: `three_wheeler` (runs r04, r07) | 0.15 | Recall 100%, precision 78% (7 vật gold). YOLO26 không có lớp này nên phải dùng model open-vocabulary. Run r04 (conf 0.25, pilot): tìm đúng tuk-tuk xanh ở img_002 (0.93), nhưng có box 0.37 nhận nhầm xe bán tải trắng thành `three_wheeler` và box 0.28 ở img_011 chỉ bao một phần xe. Ghi trong r04 là thử ngưỡng 0.4–0.5, nhưng `threshold_sweep` trên run r07 (YOLOE-26s, `--split all --conf 0.05`, 125 box) cho thấy ngưỡng 0.15 đạt recall 100% trên gold nên tôi chọn 0.15 và chấp nhận box thừa, sau đó xoá khi review. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.1,car=0.5,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`. Kết quả gộp: 786 box (person=508, motorbike=154, three_wheeler=52, car=48, bicycle=13, truck=7, bus=4).
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1 (`sam2.1_b.pt`), run r05 (box từ YOLO26, chạy CPU 4.44 giây/ảnh, 2 ảnh). Mask bám được thân, mui, bánh tuk-tuk ở img_004 và phần lớn thân ở img_013, nhưng biên bánh/mui ở img_004 chưa sát, và ở img_013 mask lẫn người lái với khung xe. Tôi chọn SAM 2.1 vì đó là phiên bản trong lệnh mẫu của hướng dẫn, và tôi **chưa so sánh với SAM 1**, nên chưa có bằng chứng 2.1 tốt hơn.
- Lỗi **không sửa được bằng đổi ngưỡng**: máy gán sai **tên lớp** chứ không phải sai điểm số. Hai kiểu gặp khi duyệt: (1) YOLO26 gọi xe ba bánh là lớp khác (img_008: vật thật là `three_wheeler` nhưng máy gán `car`); (2) YOLOE gán `three_wheeler` cho xe khác (vật thật là `truck` ở img_003, `motorbike` ở img_009, `car` ở img_013, `bus` ở img_018; r04 cũng thấy xe bán tải trắng bị nhận nhầm). Ngoài ra img_014: máy gán `bicycle` nhưng vật thật là `motorbike`. Đổi ngưỡng chỉ làm box hiện nhiều hay ít, không đổi được tên lớp, nên tôi sửa tay nhãn khi review.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): **điểm thấp mà đúng:** img_028, một box `person` có conf chỉ 0.1 nhưng đúng là người thật (nằm đúng ở ngưỡng person=0.1 nên được giữ, nếu đặt ngưỡng cao hơn sẽ mất). **Điểm cao mà sai:** img_014, box `bicycle` conf 0.88 nhưng vật thật là `motorbike`.
- Giới hạn: ngưỡng được chọn và chấm trên cùng 4 ảnh gold nên precision/recall của máy trên gold là **lạc quan**; lớp ít vật (bicycle, bus, truck) không đủ dữ liệu để chọn ngưỡng riêng.

## 2. AI error profile (từ `qc/review_stats.md`)

Ảnh tính: 24 (batch, gold). Box máy: 683. Đối tượng sau duyệt: 644. Đã **loại 22 đối tượng nằm quanh 12 bẫy** khỏi bảng này, vì đó là lỗi nhân tạo tôi chèn vào, không phản ánh chất lượng máy.

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (IoU ≥ 0.9) | 598 | 87.6% | 683 box máy |
| edited | 7 | 1.0% | 683 box máy |
| deleted | 76 | 11.1% | 683 box máy |
| added | 39 | 6.1% | 644 đối tượng sau duyệt |

- Trong deleted + added có **11 trường hợp đổi lớp** (box đúng chỗ, sai lớp; script tính là xoá + thêm). Số vật máy **sót thật** ≈ 39 − 11 = 28.
- **Lớp máy tệ nhất:**
  - `three_wheeler`: 26/39 box máy bị xoá (≈ 67%), tức máy gợi ý thừa nhiều nhất. Tôi xoá box `three_wheeler` thừa ở img_003, 008, 013, 014, 023.
  - Số `deleted` của `three_wheeler` cao một phần vì **YOLOE gán `three_wheeler` cho xe khác** (img_003, 009, 013, 018, xem mục 1). Script tính mỗi ca đổi lớp là xoá một box và thêm một box mới, nên các ca này làm tăng `added` của `truck`, `motorbike`, `car`, `bus` (cùng với số 11 ca đổi lớp ở trên).
  - `person`: 41/449 box máy bị xoá (≈ 9%), do ngưỡng 0.1. Tôi xoá person thừa ở img_003, 006, 009, 010, 018, 022, 025, 028, 005, 013.
  - Máy sót nhiều ở `motorbike`, `car`, `bicycle`, `truck` (added lần lượt 8, 15, 6, 5), ví dụ thiếu xe máy ở img_006, 013, 025 và thiếu bicycle ở img_010.
- **Box lỏng:** `edited` chỉ 7 (1.0%), khớp với số ảnh tôi ghi nhận có box lỏng (img_008, 010, 013, 017, 018, 021, 025).
- **So với gold** (4 ảnh, 59 đối tượng gold là mẫu số của recall): precision máy 53.9% (48/89, KTC 95%: 44–64%), recall 81.4% (48/59, KTC 95%: 70–89%). Máy thừa nhiều hơn sót. Con số **lạc quan** vì chọn và chấm ngưỡng trên cùng gold, và chỉ có 4 ảnh nên khoảng tin cậy rất rộng.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- **Bắt bẫy: 8/12 = 66.7%** (KTC 95%: 39–86%), dưới ngưỡng 80%. Theo loại: `missing` 1/3, `extra` 2/3, `wrong_class` 3/3, `loose` 2/3.
- **Bẫy lọt (4):** img_028 (`missing`), img_005 (`missing`), img_013 (`loose`), img_004 (`extra`).
- **Vì sao lọt:**
  - img_028 và img_005 (`missing`): tôi tập trung xoá box thừa nên vô tình bỏ qua bước tìm vật bị thiếu; thêm vào đó vật bị thiếu ở quá xa và nhỏ nên khó thấy. Theo guideline 4.1.6, vật cao dưới 20 px không vẽ từng vật nên có thể tôi đã bỏ qua mà không nhận ra đó là bẫy. Đây là hai ảnh dày xe máy, bẫy `missing` khó thấy nhất vì máy không vẽ gì để nhìn.
  - img_013 (`loose`, tức bước 4 của checklist) và img_004 (`extra`, tức bước 2): tôi quá tập trung vào các đối tượng lớn nên không kiểm kỹ các box nhỏ hơn.
- **Accepted cao (87.6%) mà bắt bẫy thấp (66.7%)** nói gì về cách duyệt: phần lớn box máy đúng nên giữ nguyên là hợp lý, nhưng những lỗi tôi để lọt nằm ở chỗ khó (ảnh dày, vật máy không vẽ). Tôi dễ tin nhãn máy và kiểm ít hơn ở đúng nơi cần kiểm nhất, đó là biểu hiện của thiên kiến tự động hóa. Ngoài ra `added` thấp (6.1%) không có nghĩa máy ít sót, vì vật nào tôi không phát hiện thì tôi cũng không thêm.
- Ghi chú duyệt: tôi bắt lỗi "thừa" (xoá) và "sai nhãn" tốt hơn lỗi "thiếu" (vẽ thêm): `wrong_class` bắt 3/3 còn `missing` chỉ 1/3.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 15 phút / 4 ảnh = **3.75 phút/ảnh**; duyệt 88 phút / 24 ảnh = **3.67 phút/ảnh**; tiết kiệm chỉ ≈ **2%** ((3.75 − 3.67) / 3.75). Thiên lệch: tôi gán gold trước nên đã quen kiểu cảnh này; gold chỉ có 4 ảnh còn batch có 6 ảnh dày xe máy, nên so sánh chỉ mang tính gần đúng.
- **Kết luận:** ở lô này pre-label **gần như không tiết kiệm thời gian** (≈ 2%), và chất lượng duyệt chưa đạt (bắt bẫy 66.7% < 80%). Máy chỉ nên dùng làm nhãn khởi đầu cho `person` và `motorbike` (accepted cao, ít sửa), không nên tin cho `three_wheeler` và `car` (thừa hoặc đổi lớp nhiều, sót nhiều). Việc duyệt đổi loại công việc (xoá và sửa thay vì vẽ) nhiều hơn là giảm thời gian, và nếu dùng máy thì phải kèm quy trình kiểm bắt buộc.
- **Ba quy tắc giảm thiên kiến của tôi:**
  1. Ở mọi ảnh dày xe máy (005, 018, 021, 024, 028, 029), ẩn nhãn máy và đếm vật theo 4 góc phần tư trước, ghi số đếm vào file ghi chú; kiểm bằng file ghi chú có đủ 6 dòng.
  2. Sau mỗi ~10 ảnh nghỉ 3–5 phút và ghi dòng `break` trong `time_log.csv`; kiểm bằng log.
  3. Mỗi ảnh có một lượt riêng chỉ để tìm vật **bị thiếu** và vật nhỏ/xa sau khi đã xử lý vật lớn (vì `missing` là loại tôi bắt kém nhất, 1/3, và tôi hay tập trung vào đối tượng lớn); kiểm bằng số bẫy `missing` bắt được ở lần review sau.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| <<img_0xx>> | <<>> | <<>> | <<>> | <<>> | <<>> |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? <<>>
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: <<>>

## 6. Calibration (Part 6)

- Số quyết định: <<>> (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): <<>>
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: <<>>
- Nguyên nhân gốc: <<guideline mơ hồ / chưa nắm quy tắc / máy>>

## 7. QC và quyết định (Part 6)

- Mẫu: <<n>> ảnh (<<m>> ảnh `can_review` + <<k>> ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: <<.../... = ...%>> · theo **ảnh**: <<.../... = ...%>> (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: <<>>
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): <<>>
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): <<>>

**Quyết định: <<Release / Hold / Rework>>**, ngày <<YYYY-MM-DD>>. Lý do (đi theo Hold → Rework → Release): <<>>
Nếu Rework / Hold, việc cần làm tiếp: <<>>

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): <<>>
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? <<không / có, như sau: ...>>
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: <<>>
- Dùng cache `data/cache/` thay vì tự chạy? <<không / có, vì ...>>
- Lỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>
