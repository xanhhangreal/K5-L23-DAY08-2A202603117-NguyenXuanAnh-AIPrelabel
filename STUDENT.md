# Thông tin học viên

- Họ và tên: Nguyễn Xuân Anh
- MSHV: 2A202603117
- Lớp: L2
- Tài khoản GitHub: xanhhangreal
- Repo bài nộp: https://github.com/xanhhangreal/K5-L23-DAY08-2A202603117-NguyenXuanAnh-AIPrelabel
- Commit chốt bài: sẽ ghi hash commit kết quả đã kiểm tra trước khi push.
- Máy: Windows, RAM khoảng 16 GB, Intel Iris Xe; model chạy CPU.
- CVAT: Docker, phiên bản 2.74.1 theo server đang chạy.

## Kết quả tóm tắt

- Quy trình kết hợp: học viên review 10 ảnh đến img_017.jpg; toolkit hỗ trợ 14 ảnh còn lại. Codex kiểm 3 mask và mẫu QC theo yêu cầu; không phải kiểm độc lập của người.
- Accepted / edited / deleted: 103/231=44.6% / 104/231=45.0% / 24/231=10.4%; added 252/459=54.9% (mẫu số theo script sau xử lý vùng bẫy, không phải toàn file 473 annotation).
- Gold: precision 28/35=80.0%; recall 28/68=41.2%, chỉ 4 ảnh và chọn ngưỡng in-sample.
- Bắt bẫy: cả quy trình 11/12=91.7%; riêng học viên 4/5=80.0%.
- Thời gian: gold 30 phút/4 ảnh=7.5 phút/ảnh; review thủ công 30 phút/10 ảnh=3 phút/ảnh; toolkit 14 ảnh có 2.626 phút tổng xử lý ảnh, không gồm nạp/xuất/kiểm nhãn.
- QC vòng hai hỗ trợ: mẫu seed 43 sau sửa thêm img_008 ghi 0/95 đối tượng lỗi chắc chắn, 0/5 ảnh lỗi chắc chắn; 1 ca phân loại CAL-06 chưa chốt. Đây là kiểm lại mẫu đã sửa, không độc lập. Tổng 3 box xóa, 5 box chỉnh; 468 hình khác giữ nguyên.
- SAM: đã tinh chỉnh hai mask còn lẫn người; img_013 dùng mask xe trừ hợp mask người, img_023 thêm điểm dương/âm. Kiểm overlay chưa thấy phần người bị lẫn rõ; chưa có IoU chuẩn.
- Calibration: 5 ca đã soạn và 1 ca Hold chưa chốt lớp trong guideline v1.1; Codex soạn theo yêu cầu, học viên cần đọc và giải thích trước khi dùng bản nộp.
- Quyết định đề xuất: **Hold** vì còn ca phân loại xe bị che img_034 #426 (CAL-06) cần xác nhận. Trạng thái chất lượng Hold; chưa Release và chưa đánh dấu Job hoàn tất.
