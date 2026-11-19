# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 5 ảnh (0 ca khó + 5 ngẫu nhiên), 95 đối tượng.
- taxonomy v1.1

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 0/95 = 0.0% | 95 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 0/5 = 0.0% | 5 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 103/231 = 44.6% / 104/231 = 45.0% / 24/231 = 10.4% | 231 box máy |
| Máy: added | 252/459 = 54.9% | 459 đối tượng sau duyệt |
| Bắt lỗi bẫy | 11/12 = 91.7% | 12 bẫy |

## Top 3 loại lỗi
(không ghi nhận lỗi nào)

## Giới hạn thống kê
- 0 ảnh lỗi trong 5 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **43%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**HOLD**

- guideline chưa rõ (1 ca) / dữ liệu có vấn đề (0 ảnh) -> giữ lại làm rõ

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
