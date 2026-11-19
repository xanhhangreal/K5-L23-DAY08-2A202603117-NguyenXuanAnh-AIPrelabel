# Số đo review (det)

- Ảnh tính: 10 (batch,gold); box máy: 86; đối tượng sau duyệt: 114; ngưỡng accepted: IoU >= 0.9
- Đã **loại 10 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 68 | 68/86 = 79.1% | 86 box máy |
| edited (sửa) | 12 | 12/86 = 14.0% | 86 box máy |
| deleted (xoá) | 6 | 6/86 = 7.0% | 86 box máy |
| added (vẽ thêm) | 34 | 34/114 = 29.8% | 114 đối tượng sau duyệt |

Trong số deleted + added, **1 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 33.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 39 | 7 | 2 | 20 |
| bicycle | 1 | 1 | 0 | 2 |
| motorbike | 14 | 2 | 1 | 4 |
| car | 3 | 0 | 0 | 1 |
| bus | 0 | 0 | 0 | 1 |
| truck | 0 | 0 | 1 | 0 |
| three_wheeler | 11 | 2 | 2 | 6 |

## 3. Ảnh bẫy
- Bắt được 4/5 = 80.0% (KTC 95%: 38–96%) (ngưỡng đạt: 80%). Chỉ 5 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 1/2
  - extra: 2/2
  - missing: 1/1
  - lọt: img_013.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
