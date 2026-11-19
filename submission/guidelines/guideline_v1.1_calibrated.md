# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603117 - Nguyễn Xuân Anh

Chỉ bổ sung so với guideline v1.0. Codex soạn theo yêu cầu học viên, từ kiểm tra trực quan 3 mask và 5 ảnh QC; người kiểm ghi là Codex_visual_assisted. Học viên cần đọc và giải thích được các quy tắc khi sử dụng bản nộp. Không đổi ngưỡng trong taxonomy.rules.

## 1. Câu hỏi mở được làm rõ

- Câu 3 — hàng nhô ra: CAL-03 (img_031), tách giỏ gắn cố định và hàng rời; giữ ngưỡng box khoảng 20% của v1.0, mask không tự lấy mọi hàng trong box.
- Câu 5 — kính và vùng hở: CAL-04 (img_013, img_023), kính vật lý thuộc xe; người/nền nhìn qua khoang hở không thuộc mask xe. Chưa có mask chuẩn để đo IoU.
- Câu 7 — conf: CAL-05 (img_031, img_014), không có mức conf cho phép bỏ qua kiểm ảnh.
- Các câu 1, 2, 4 và 6 chưa được giải quyết tổng quát bằng mẫu QC này. Tiếp tục dùng v1.0; nếu gặp ca không áp dụng được thì tick can_review, ghi riêng và Hold ca đó để làm rõ. Không suy diễn rằng đã giải quyết mọi trường hợp của bộ dữ liệu.

## 2. Ca khó và quy tắc bổ sung

| case_id | Ảnh ví dụ | Tình huống | Quy tắc áp dụng |
|---|---|---|---|
| CAL-01 | img_003 | Một xe ba bánh bị tách thành box thùng và box cả xe có mái. | Một xe vật lý chỉ có một box three_wheeler; box bao toàn bộ phần xe nhìn thấy gồm mái/khung gắn liền; người có box riêng. |
| CAL-02 | img_031 | YOLO11l gán person cho quần áo treo (#399, #400). | Không gán person cho quần áo treo, ma-nơ-canh hoặc vật trưng bày; kiểm dấu hiệu người thật trước khi giữ box. |
| CAL-03 | img_031 | Phân biệt giỏ xe gắn cố định với trái cây/hàng rời khi tạo box và mask. | Giỏ cố định thuộc xe. Giữ quy tắc box v1.0: hàng rời nhô hơn khoảng 20% chiều dài xe không mở rộng box theo hàng. Mask xe loại hàng rời khỏi phần xe, kể cả khi nằm trong box. |
| CAL-04 | img_013|img_023 | SAM lấy lẫn phần người lái và vùng hở trong xe ba bánh. | Ở khoang hở: phần người nhìn thấy không thuộc mask xe; vùng nền lớn xuyên qua khung không tô kín. Kính vật lý thuộc xe theo v1.0, phân biệt kính với khoang hở. |
| CAL-05 | img_031|img_014 | Có thể dùng conf để chấp nhận tự động mà bỏ qua kiểm ảnh không? | Không đặt ngưỡng conf nào thay thế việc review. Conf dùng sàng lọc và xếp thứ tự kiểm, không chứng minh đúng lớp/hình học. |

Box người phải bao cả các tay/chân nhìn thấy, dù vươn ra ngoài thân chính (img_006 #50 và img_031 #398 (ID vòng đầu), img_008 #73 (ID vòng hai)); không mở box để bao phần người bị che mà chỉ suy đoán. Ví dụ img_014 #108 cho thấy conf cao vẫn có thể có box dư nền. Các lỗi này là lỗi thực thi quy tắc hình học v1.0, không phải lý do nới dung sai.

## 3. Bằng chứng và phạm vi

- Ảnh QC có ID: ../../work/qc/review/ (thư mục work không commit); bản bằng chứng chia sẻ trong ../qc/evidence/.
- Phiếu lỗi: ../qc/qc_review.csv; các ID là ID trong release candidate hiện tại.
- Mask hiện tại nằm trong ../masks/, phiên bản trước trong ../qc/history/round1/masks/. Prompt và phép trừ mask người của vòng hai nằm trong ../automation/mask_round2/; không coi overlay là mask chuẩn đã đạt IoU.
- Theo yêu cầu mới của học viên, đã sửa 7 lỗi xác nhận của QC vòng đầu và tinh chỉnh hai mask. Sửa nhãn gồm 3 lần xóa và 5 lần chỉnh box (có img_008 ở lượt bổ sung), 468 hình khác giữ nguyên. Báo nhầm img_035 được đính chính và box khôi phục. Lỗi biên person img_008 #73 trước sửa đã được chỉnh và hiện là #69. Mẫu seed 43 sau sửa chưa ghi lỗi chắc chắn; vẫn Hold ca xe bị che img_034 #426 (CAL-06) để xác nhận lớp.

## 4. Lịch sử thay đổi

| Phiên bản | Thay đổi | Nguồn |
|---|---|---|
| v1.0 | Bản giảng viên | Guideline gốc |
| v1.1 | Làm rõ box lồng nhau, quần áo treo, giỏ/hàng rời, kính/khoang hở và giới hạn conf | CAL-01 đến CAL-05; Codex soạn từ kiểm tra trực quan theo yêu cầu học viên |

## 5. Ca cần xác nhận sau QC vòng hai

CAL-06 — img_034, release ID #426: chưa chốt truck hay three_wheeler vì phần xe bị che. Giữ Hold ca này để người kiểm/học viên xác nhận; áp dụng mục 4.3, không coi nhãn dự đoán là chuẩn. Đây là ca áp dụng lớp cần bằng chứng, không phải quy tắc cho phép đổi taxonomy.rules.
