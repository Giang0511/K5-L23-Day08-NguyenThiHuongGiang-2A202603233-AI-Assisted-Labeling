# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 8 ảnh (3 ca khó + 5 ngẫu nhiên), 232 đối tượng.
- taxonomy v1.1

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 10/232 = 4.3% | 232 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 7/8 = 87.5% | 8 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 598/683 = 87.6% / 7/683 = 1.0% / 76/683 = 11.1% | 683 box máy |
| Máy: added | 39/644 = 6.1% | 644 đối tượng sau duyệt |
| Bắt lỗi bẫy | 8/12 = 66.7% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| thua | 6 | img_003.jpg, img_004.jpg, img_008.jpg |
| sai_vi_tri_kich_thuoc | 2 | img_003.jpg, img_008.jpg |
| sai_lop | 2 | img_013.jpg, img_018.jpg |

**Lỗi hệ thống**: 'thua' lặp lại -> sửa quy tắc / recipe, không chỉ sửa từng ảnh.

## Giới hạn thống kê
- 7 ảnh lỗi trong 8 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **98%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**HOLD**

- guideline chưa rõ (1 ca) / dữ liệu có vấn đề (0 ảnh) -> giữ lại làm rõ

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
