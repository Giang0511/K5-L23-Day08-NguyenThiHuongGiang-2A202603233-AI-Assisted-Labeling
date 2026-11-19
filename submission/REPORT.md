# Báo cáo Day 08 — 2A202603233 - Nguyễn Thị Hương Giang

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

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

Mask tạo bằng `scripts/sam2_points.py` với SAM 2.1 (`sam2.1_b.pt`), click dương/âm trên ảnh gốc. "Số click" là tổng dương + âm của lần chạy cuối. Tỉ lệ "lem" là ước chừng bằng mắt, tôi không đo bằng số.

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk có người lái) | SAM 2.1 (`sam2.1_b.pt`) | 9 (3 dương + 6 âm), qua 3 lần chạy | Mask xe vẫn **lem vào người lái** (~3%); không có hàng hoá đáng kể | Lần 1 (1 dương) mask lẫn người lái; thêm click âm lên người thì giảm nhưng không hết, vì người ngồi sát thân xe. Còn vài mảng thủng nhỏ ở khung xe | `submission/masks/tuktuk_img013.jpg` |
| img_003 (xe ba gác chở hàng) | SAM 2.1 (`sam2.1_b.pt`) | 6 (3 dương + 3 âm), qua 2 lần chạy | Mask **gồm hàng hoá** (trái cây trong thùng xe); **lem vào người ngồi trong xe** (~3%) | Lần 1 (2 dương + 2 âm) đã tô hết xe nhưng lem vào người; thêm 1 dương và 1 âm vẫn lem, vì người và hàng hoá cùng nằm trong khung xe | `submission/masks/three-wheeler_img003.jpg` |
| img_017 (tuk-tuk nằm sát xe khác và người đi bộ) | SAM 2.1 (`sam2.1_b.pt`) | 4 (2 dương + 2 âm), 1 lần chạy | Không có hàng hoá; **lem vào người đi bộ** đứng trước xe (~3%) | Đạt ngay lần đầu nhờ 2 điểm âm đặt ở xe liền kề và nền; mask thân xe còn loang lổ, không đều | `submission/masks/tuktuk_img017.jpg` |

- Đối chiếu guideline mục 4.2:
  - **Giúp được:** quy tắc "người và xe là hai vật riêng" cho tôi biết phải đặt click âm lên người lái / người ngồi; quy tắc "hai vật chạm nhau thì mỗi vật một polygon, không chồng lên nhau" giúp ở img_017, nơi tôi đặt điểm âm lên chiếc xe liền kề.
  - **Chưa đủ trong guideline v1.0, và quyết định của tôi:**
    1. SAM không tách sạch ranh giữa người và xe khi họ ngồi sát nhau: cả ba mask đều còn lem khoảng 3%. Guideline chỉ nêu sai số biên ≤ ~3 px chứ chưa nói mức lem vào vật khác chấp nhận được.
    2. **Hàng hoá có thuộc mask xe không:** chốt là **có**. Hàng nằm trên xe thì tô vào mask xe (img_003: mask gồm trái cây trong thùng xe là đúng).
    3. **Người ngồi trong khung xe:** chốt khi tô mask xe thì **không tô người** ở trong khung xe (người là vật riêng, theo guideline mục 5); người đó vẫn được gán `person` riêng, và vùng người được loại khỏi mask xe bằng click âm.
  - Hai quyết định (2) và (3) đã ghi vào `calibration_decisions.csv` (CAL-02, CAL-03) và `guideline_v1.1_calibrated.md`.

## 6. Calibration (Part 6)

- Số quyết định: 9 (`calibration_decisions.csv`): 2 `loi_may` (CAL-01, CAL-03), 5 `cau_hoi_mo` (CAL-02, 04, 07, 08, 09), 1 `bay_lot` (CAL-05), 1 `can_review` (CAL-06). Trong đó 7/9 thành quy tắc ở `guideline_v1.1_calibrated.md` (CAL-02, 03, 04, 06, 07, 08, 09; CAL-09 chỉ xác nhận lại mục 4.2.3 của v1.0, chưa có ca thực tế trong batch) và 2/9 không đổi guideline (CAL-01: lỗi máy; CAL-05: lỗi quy trình của tôi). Câu hỏi mở số 4 (người thấp thoáng trong xe ba bánh có mui, ngưỡng 20%) chưa chốt vì chưa có ca đo được trong 24 ảnh batch.
- Quyết định quan trọng nhất (dẫn `CAL-xx`): **CAL-01.** YOLOE gán `three_wheeler` cho 4 loại xe khác nhau (`truck` ở img_003, `motorbike` ở img_009, `car` ở img_013, `bus` ở img_018), còn YOLO26 thì gọi xe ba bánh là `car` (img_008). Tức là với lớp `three_wheeler`, máy phần lớn hoặc không bắt được, hoặc bắt nhầm sang lớp khác. Đây cũng là lỗi máy ảnh hưởng nhiều nhất tới công sửa: `three_wheeler` có 26/39 box máy bị xoá (≈ 67%, mục 2), cao nhất trong các lớp. Vì lỗi nằm ở máy chứ không ở định nghĩa lớp (mục 2 và 4.3 của v1.0 đã đủ), tôi không đổi guideline mà sửa tay khi duyệt, bằng cách đếm số bánh nhìn thấy.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: xe máy điện / xe đạp điện gán `motorbike` (CAL-06, img_025); cụm xe sát nhau có box từng xe cao dưới 20 px thì một `ignore_region` (CAL-07, img_024); ảnh đêm xe sát khít thì giữ từng box nếu xác định được biên, nếu không thì gộp `ignore_region` (CAL-08, img_029); mask: hàng nằm trên xe thuộc mask xe, hàng nhô quá ~20% chiều dài xe thì không (CAL-02, img_003); mask: không tô người trong khung xe vào mask xe (CAL-03, img_003/013/017); không đặt ngưỡng `conf` tối thiểu, mọi box đều phải nhìn ảnh (CAL-04, img_014/028).
- Nguyên nhân gốc (theo cột `nguon` và `quyet_dinh` trong `calibration_decisions.csv`):
  - **Guideline mơ hồ (v1.0 chưa có quy tắc):** CAL-02 (hàng hoá trên xe có thuộc mask không), CAL-04 (có đặt ngưỡng `conf` tối thiểu không), CAL-06 (xe máy điện / xe đạp điện), CAL-07 (cụm xe đỗ dày gần camera), CAL-08 (box chồng nhau trong ảnh đêm), CAL-09 (kính xe thuộc mask không). Đây là các câu hỏi mở của v1.0 nên phải thêm quy tắc vào v1.1.
  - **Máy:** CAL-01 (YOLOE gán `three_wheeler` cho xe khác, YOLO26 gọi xe ba bánh là `car`): lớp đã định nghĩa đủ ở mục 2 và 4.3 của v1.0 nên không đổi guideline, chỉ sửa tay khi duyệt. CAL-03 (SAM lem khoảng 3% vào người ngồi trong khung xe): lỗi do máy, nhưng v1.0 cũng chưa nói người trong khung xe có thuộc mask xe không, nên tôi thêm quy tắc vào v1.1.
  - **Quy trình của tôi:** CAL-05 (4 bẫy lọt: 2 `missing`, 1 `loose`, 1 `extra`): guideline đã đủ, lỗi nằm ở cách tôi duyệt nên sửa bằng quy trình (ba quy tắc ở mục 4), không đổi guideline.

## 7. QC và quyết định (Part 6)

- Mẫu: 8 ảnh (3 ảnh `can_review` + 5 ngẫu nhiên, seed 42), 232 đối tượng; người kiểm: tôi (**hạn chế**: tôi vừa là người duyệt nên có thể bị neo vào nhãn đã duyệt).
- Lỗi theo **đối tượng**: 10/232 = 4.3% · theo **ảnh**: 7/8 = 87.5% (ngưỡng Release ≤ 2%). Nguồn: `qc/qc_report_auto.md`.
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask): thừa 6 (img_003, img_004, img_008); sai vị trí/kích thước 2 (img_003, img_008); sai lớp 2 (img_013, img_018); bỏ sót, sai thuộc tính, sai mask: 0 lỗi ghi nhận.
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): với 7/8 ảnh lỗi, cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là 98%. Mẫu 8 ảnh quá nhỏ để chứng minh ngưỡng 2%: ngay cả khi 0/8 ảnh lỗi thì cận trên vẫn là 32% (kết quả của lần chạy `qc_report.py` khi phiếu còn trống), nên kết quả này chỉ cho biết lô chưa đạt, không đo được chính xác tỉ lệ lỗi thật.
- Script kiểm được: tên file, lớp có trong taxonomy, box nằm trong ảnh, đủ ảnh được giao, các số đo accepted/edited/deleted/added và độ lệch giữa hai bộ nhãn. Không kiểm được: box có ôm sát vật thật không, lớp có đúng thực tế không, vật mà cả tôi lẫn máy cùng bỏ sót, và tôi có theo checklist thật hay không. Vì vậy số lỗi QC phụ thuộc vào mắt của một người kiểm (là tôi).

| Điều kiện trong cây quyết định | Số liệu của tôi | Thoả điều kiện? |
|---|---|---|
| 1. Guideline còn câu hỏi chưa chốt / dữ liệu hỏng | 1 câu chưa chốt (câu 4); 0 ảnh hỏng | có |
| 2. Ảnh lỗi > 2% | 7/8 = 87.5% | có |
| 2. Một loại lỗi lặp lại (≥ 3 lỗi và ≥ 50%) | `thua` 6/10 = 60% | có |
| 2. Bắt bẫy < 80% | 8/12 = 66.7% | có |

Script `qc_report.py` (chạy với `--guideline-gap 1`) gợi ý **HOLD**; đó chỉ là gợi ý, quyết định do tôi xác nhận.

**Quyết định: Hold**, ngày 2026-11-19. Lý do (đi theo Hold → Rework → Release):
Câu 1 (Hold): có. Guideline v1.1 còn 1 câu hỏi mở chưa chốt (câu 4, ngưỡng "còn thấy ≥ 20%" của người trong xe ba bánh có mui) vì trong 24 ảnh batch không có ca nào đo được. Câu hỏi này ảnh hưởng tới việc gán `person`, nên chưa thể coi nhãn là ổn định. Vì cây dừng ở câu đầu tiên đúng nên quyết định là Hold. Ghi thêm: nếu câu 4 được chốt thì các điều kiện của Rework (ảnh lỗi 7/8 = 87.5% > 2%, lỗi `thua` 6/10 = 60%, bắt bẫy 8/12 = 66.7% < 80%) vẫn thỏa, nên lô sẽ chuyển sang Rework chứ không thể Release.
Việc cần làm tiếp: (1) tìm hoặc chụp thêm ca người trong xe ba bánh có mui rồi chốt câu 4; (2) sửa các lỗi đã tìm thấy ở QC; (3) lấy mẫu QC mới và đo lại.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào):
- Claude (Anthropic, claude.ai): hướng dẫn quy trình và lệnh theo tài liệu của repo; giải thích lỗi môi trường; góp ý chỗ chưa chặt trong bản nháp guideline v1.1 và `taxonomy.json`; hỗ trợ cấu trúc báo cáo. 
- Công cụ khác (soạn nháp guideline v1.1 và báo cáo mục 1–4, nếu có): ChatGPT hỗ trợ chỉnh chính tả và hoàn thiện báo cáo.
- Máy / GPU / CVAT dùng: Windows 11 Pro, Intel Core i7-11850H, RAM 64 GB, NVIDIA T1200 Laptop GPU 4 GB; các lần chạy model trong `work/runs.csv` ghi thiết bị `cpu`. CVAT v2.74.1 chạy bằng Docker (khác bản 2.81 trong tài liệu của lab). Vướng mắc cài đặt: `download_models.py` báo `No module named 'ultralytics'` vì venv dùng chung với bài Day09 chưa cài `requirements.txt`; `threshold_sweep.py` báo thiếu file `pre.coco.json` vì tôi chưa chạy bước `--split all --conf 0.05` ở cuối Part 2. Cả hai đều do thứ tự thao tác, đã xử lý bằng cách cài thư viện và chạy lại các lệnh còn thiếu.
- Dùng cache `data/cache/` thay vì tự chạy? Không: tôi tự chạy r06 (YOLO26m) và r07 (YOLOE-26s) trên cả 36 ảnh ở ngưỡng 0.05.
- Lỗi / bug script đã gặp (nếu có): không sửa script nào; các lỗi gặp đều nằm ở môi trường hoặc thứ tự thao tác như nêu ở trên.
s