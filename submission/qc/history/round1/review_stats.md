# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 231; đối tượng sau duyệt: 462; ngưỡng accepted: IoU >= 0.9
- Đã **loại 27 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 108 | 108/231 = 46.8% | 231 box máy |
| edited (sửa) | 101 | 101/231 = 43.7% | 231 box máy |
| deleted (xoá) | 22 | 22/231 = 9.5% | 231 box máy |
| added (vẽ thêm) | 253 | 253/462 = 54.8% | 462 đối tượng sau duyệt |

Trong số deleted + added, **5 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 248.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 51 | 55 | 6 | 131 |
| bicycle | 2 | 2 | 1 | 4 |
| motorbike | 21 | 34 | 8 | 80 |
| car | 17 | 7 | 3 | 30 |
| bus | 0 | 1 | 1 | 2 |
| truck | 0 | 0 | 1 | 0 |
| three_wheeler | 17 | 2 | 2 | 6 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 68 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 28 | 7 | 40 | 80.0% (KTC 95%: 64–90%) | 41.2% (KTC 95%: 30–53%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 11/12 = 91.7% (KTC 95%: 65–99%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 2/3
  - missing: 3/3
  - wrong_class: 3/3
  - extra: 3/3
  - lọt: img_013.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
