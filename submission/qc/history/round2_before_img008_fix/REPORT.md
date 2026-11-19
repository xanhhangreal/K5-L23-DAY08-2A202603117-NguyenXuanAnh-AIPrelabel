# Báo cáo Day 08 — 2A202603117 - Nguyễn Xuân Anh

Phạm vi thực hiện: học viên gán tay 4 gold và tự review 10 ảnh batch đến hết img_017.jpg; công cụ local xử lý 14 ảnh còn lại. Codex kiểm trực quan 3 mask và 5 ảnh QC, hỗ trợ viết calibration và báo cáo theo yêu cầu. Người kiểm QC là **Codex_visual_assisted**, không phải kiểm định độc lập của con người. Đã sửa các lỗi QC vòng đầu được xác minh và tinh chỉnh hai mask. QC vòng hai dùng mẫu mới seed 43, vẫn đề xuất **Rework** vì phát hiện một lỗi mới; chưa phải bộ nhãn đạt Release.

## 1. Model và recipe (Part 2–3)

| Lớp | Model ban đầu | Ngưỡng dùng khi gộp | Cơ sở |
|---|---|---|---|
| person, bicycle, motorbike, car, bus, truck | YOLO26-n, CPU, imgsz 640 | 0.3 cho từng lớp | r01/r02 thử conf 0.25/0.5; r05 chạy 36 ảnh ở 0.05; threshold_sweep không chọn được ngưỡng riêng |
| three_wheeler | YOLOE-26s-seg; prompt tuk-tuk, cyclo, cargo tricycle | 0.3 | r03: học viên quan sát nhận được tuk-tuk và xích lô; r06 là nguồn quét ngưỡng |

Lệnh gộp đã chạy:

```text
python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
```

YOLOE được ưu tiên trước YOLO26 khi box chồng nhau. Sau đó inject_traps.py dùng MSHV 2A202603117 tạo 12 bẫy. File nạp CVAT là pre_traps_import.coco.json, chỉ chứa batch.

- Quét ngưỡng: person có 29 vật gold, bicycle 5, motorbike 18, car 8, truck 1, three_wheeler 7, bus 0. Các lớp đủ mẫu không đạt mục tiêu recall 90% tại ngưỡng được phép; bus/truck quá ít mẫu. Giữ mặc định 0.3 theo script, không ép ra ngưỡng đẹp. Nguồn: qc/threshold_sweep.md.
- SAM đã dùng: **SAM 1 MobileSAM** trong r04 để thử box→mask trên CPU; **SAM 2.1 base** trong r08 cho 3 ví dụ có box và điểm dương. Toolkit bổ sung r07 dùng **SAM 1 ViT-B** cùng YOLO11l cho 14 ảnh sau img_017.jpg. Đây là các phiên bản và mục đích khác nhau.
- Lỗi sai taxonomy không thể sửa chỉ bằng đổi conf: học viên ghi nhận r04 chưa nhận ra tuk-tuk ở img_004 và img_013. Nguồn YOLOE bổ sung lớp three_wheeler; vẫn phải kiểm bằng ảnh.
- Conf không bảo đảm chất lượng: img_014 box person #108 conf 0.83 vẫn dư nền; img_031 #399 conf 0.4759 là quần áo treo. Ngược lại img_035 #468 conf 0.3898 ứng với người đàn ông đang ngồi nhìn thấy rõ. Các ID lỗi ở đoạn này là ID release candidate vòng đầu, được giữ trong qc/history/round1; không suy trực tiếp sang ID hiện tại.
- 14 ảnh bổ sung được chạy độc lập bằng YOLO11l + SAM ViT-B ở conf 0.25, imgsz 1024; gộp với YOLOE conf 0.3, lọc trùng và box cao dưới 20 px. Không dùng đáp án bẫy để xây kết quả này. Ở bước tạo nhãn 14 ảnh còn lại đã giữ nguyên 119 hình của 10 ảnh đầu. Sau đó học viên yêu cầu sửa lỗi QC, cho phép sửa đúng các đối tượng đã xác minh trong cả phần đầu. Vòng sửa cuối xóa 3 box và chỉnh 4 box; 469 hình khác giữ nguyên. Bằng chứng automation/qc_round2_fixes.json.
- Giới hạn: chọn ngưỡng và đo trên cùng 4 gold làm kết quả trên gold lạc quan; ngưỡng chỉ là ước lượng của mẫu nhỏ. Không có mask chuẩn để đo IoU của SAM.

## 2. AI error profile sau sửa QC

Nguồn hiện tại: qc/review_stats.json và .md, được chạy lại trên COCO export thật từ CVAT sau sửa. Quy trình gồm phần học viên và công cụ, không phải toàn bộ do học viên tự review.

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 103 | 44.6% | 231 box máy |
| edited | 104 | 45.0% | 231 box máy |
| deleted | 24 | 10.4% | 231 box máy |
| added | 252 | 54.9% | 459 đối tượng sau duyệt được tính |

File hiện tại có **24 ảnh, 473 đối tượng** (giảm 3 box thừa từ 476). Script xử lý loại 27 đối tượng quanh bẫy khỏi so sánh, nên mẫu số 231/459 khác số annotation toàn file. Có 5 cặp đổi lớp trong deleted+added; added−đổi lớp = 247 chỉ là ước lượng theo thuật toán ghép, không phải số đếm ground truth.

- Theo lớp: person có 130 added, motorbike 80 added; đây là chênh lệch theo số tuyệt đối so với bộ cuối, không bảo đảm mọi nhãn cuối đúng. QC vòng hai vẫn phát hiện box person bị thiếu phần vai ở img_008.
- So với gold không đổi: TP=28, FP=7, FN=40; precision **28/35=80.0%**, recall **28/68=41.2%**. Gold có 73 annotation toàn file, mẫu số recall là 68 đối tượng theo quy tắc script. KTC 95%: precision 64–90%, recall 30–53%.
- Số đo lịch sử riêng **10 ảnh học viên làm trước sửa QC** giữ nguyên trong qc/review_stats_human_only.md: accepted 68/86=79.1%, edited 12/86=14.0%, deleted 6/86=7.0%, added 34/114=29.8%. Không gọi đây là số đo hiện tại của phần đã được công cụ sửa thêm.
- Số đo trước sửa được lưu trong qc/history/round1/review_stats.* để đối chiếu, không chỉnh tay đầu ra script.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Cả bộ kết hợp sau sửa QC: **11/12=91.7%**, gồm loose 2/3, missing 3/3, wrong_class 3/3, extra 3/3; còn lọt bẫy loose ở img_013.jpg. Khoảng tin cậy 95% tổng thể 65–99%.
- Riêng 10 ảnh học viên làm, trước sửa QC: **4/5=80.0%**, khoảng tin cậy 95% 38–96%; không coi 11/12 là kết quả cá nhân.
- Kết quả sau sửa QC không phải phép thử bẫy mù mới: đáp án đã được mở sau export vòng đầu. Không dùng số đo sau sửa để suy thiên kiến độc lập của học viên.
- Chưa có nhật ký thao tác đủ để kết luận học viên đã bỏ một bước cụ thể. Bẫy loose bị lọt cho thấy cần kiểm riêng độ ôm sát; đây là hướng cải tiến quy trình, không phải khẳng định nguyên nhân tâm lý đã được đo.
- Accepted cao không bảo đảm không có lỗi: QC vòng đầu thấy hai box cho cùng xe ba bánh ở img_003; lỗi này đã sửa. QC vòng hai thấy box person thiếu phần vai ở img_008. Không dùng tỷ lệ accepted thay cho rà ảnh gốc.
- Đáp án bẫy chỉ được dùng sau khi đã export batch_reviewed.json. Không sửa lại phần đã làm theo đáp án để tăng số đo.

## 4. Máy có đáng dùng không?

- Gold: học viên ghi **30 phút/4 ảnh=7.5 phút/ảnh**. Review thủ công: học viên báo **30 phút/10 ảnh=3.0 phút/ảnh**. Nguồn time_log.csv; đây là hai tập ảnh khác độ khó, không phải thí nghiệm đối chứng, nên không kết luận chắc chắn tiết kiệm 60% thời gian.
- Toolkit xử lý 14 ảnh: tổng thời gian xử lý từng ảnh **2.626 phút**, khoảng **11.25 giây/ảnh**, chưa gồm nạp model, xuất CVAT và kiểm chất lượng. Không so thời gian chạy model với thời gian review thủ công như hai công việc tương đương.
- Đánh giá từ bằng chứng hiện có: dùng máy làm nhãn khởi đầu và ưu tiên review là hữu ích; chưa phù hợp để chấp nhận tự động. Recall gold 41.2% và lỗi QC vòng hai 1/5 ảnh cho thấy vẫn cần quét bỏ sót, kiểm danh tính vật và chỉnh biên.
- Ba quy tắc quy trình đề xuất để học viên áp dụng:
  1. Mỗi ảnh ẩn box trước, tự quét người/xe rồi hiện nhãn; hoàn thành đủ 24/24 ảnh theo checklist.
  2. Mỗi box kiểm lần lượt vật thật, đúng lớp và biên; không có mức conf nào được bỏ qua bước này. Box lồng nhau phải đối chiếu cùng vật hay hai vật.
  3. Ghi ca mơ hồ theo tên ảnh và can_review, nghỉ sau khoảng 8–10 ảnh; chỉ đọc đáp án bẫy sau export và giữ số đo vòng đầu để tránh sửa kết quả hồi tố.

## 5. Mask SAM sau tinh chỉnh (Part 5)

Ví dụ gốc được lưu trong qc/history/round1/masks; file hiện tại ở masks/. Codex đã đối chiếu lại cả hai kết quả tinh chỉnh với ảnh gốc, không có mask chuẩn để đo IoU. Không trình bày các điểm do công cụ chọn là click của học viên.

| Ảnh / file hiện tại | Phương pháp và prompt cuối được giữ | Diện tích | Kết quả kiểm lại |
|---|---|---|---|
| img_013 / sam21_candidate_img_013_1.jpg | SAM 2.1: mask xe trừ hợp 4 mask người; 5 nhóm box-prompt, tổng 14 điểm dương và 9 điểm âm ở các nhóm khác nhau | 283547 px; gốc 292427 px | Loại phần tay/chân người lái và phần người phía sau từng bị lẫn; giữ mái, thân và bánh xe chính. Chưa thấy lẫn người rõ ở tỷ lệ kiểm; chi tiết mảnh/khoang hở vẫn cần thận trọng |
| img_023 / sam21_candidate_img_023_1.jpg | Giữ kết quả gốc: 1 box, 1 điểm dương, 0 điểm âm | 35818 px | Chưa thấy lỗi lớn ở tỷ lệ toàn ảnh; không có IoU chuẩn |
| img_023 / sam21_candidate_img_023_2.jpg | Đổi điểm dương vào phần xe: 4 điểm dương và 5 điểm âm trên người/nền, kèm box | 44427 px; gốc 56641 px | Phần tay/khoang người được loại rõ hơn; vẫn giữ mái, thân và bánh xe, chưa thấy lấy lẫn người rõ ở tỷ lệ kiểm |

Ở img_013, thử thêm nhiều điểm âm trực tiếp làm mất một phần thân xe nên không chọn bản đó. Bản cuối dùng phép boolean **vehicle_mask AND NOT union(person_masks)** với các mask đều do SAM sinh; không tô/xóa pixel bằng tay. Đây là xử lý mask sau SAM, không phải một lần suy luận SAM đơn lẻ. Ở img_023, đổi điểm dương ra khỏi vùng giáp người và thêm điểm âm cho kết quả tốt hơn.

Binary mask và prompt nằm trong automation/mask_round2/; metadata hiện tại ở automation/sam_point_examples.json. Diện tích giảm không tự chứng minh độ chính xác tăng: việc giữ/loại phần người được kiểm bằng overlay. Không có phép đo IoU, không khẳng định sai số biên đã ≤3 px trên toàn mask.

CAL-04 tiếp tục áp dụng: kính vật lý thuộc xe, người/nền nhìn qua khoang hở không thuộc mask xe. Interactor trực tiếp CVAT chưa được benchmark với pipeline offline này.

## 6. Calibration (Part 6)

Có **5 quyết định** trong qc/calibration_decisions.csv, đồng bộ với guidelines/guideline_v1.1_calibrated.md. Nội dung được Codex soạn theo yêu cầu, học viên cần đọc và giải thích khi dùng bản nộp.

- CAL-01, img_003: một xe ba bánh có mái chỉ là một đối tượng; rà box lồng nhau theo danh tính xe.
- CAL-02, img_031: quần áo treo không phải person; đây là lỗi máy, không nới định nghĩa lớp để hợp thức hóa dự đoán.
- CAL-03, img_031, nguồn cau_hoi_mo: phân biệt giỏ gắn cố định và hàng rời; giữ quy tắc overhang khoảng 20% cho box, không dùng box làm mask xe.
- CAL-04, img_013/img_023, nguồn cau_hoi_mo: kính thật thuộc xe, khoang hở không phải kính; phần người/nền trong khoang hở không thuộc mask xe.
- CAL-05, img_031/img_014, nguồn cau_hoi_mo: conf không thay thế kiểm tra lớp, vật thật và hình học.

Các câu hỏi chưa đủ bằng chứng để chốt tổng quát (xe điện, cụm xe sát camera, ngưỡng che người, ảnh đêm) được nêu rõ trong v1.1. Không đổi taxonomy.rules. Lỗi tay/chân bị cắt là lỗi thực hiện quy tắc sẵn có, không phải lý do đổi dung sai. Đã áp dụng sửa nhãn cho các lỗi vòng đầu được xác minh. Nhận xét về chân ở img_035 #451 vòng đầu là báo nhầm khi đối chiếu lưới tọa độ; box gốc được khôi phục, không tính là lỗi đã sửa. Đính chính trong qc/history/round1/erratum_img035.json. Lỗi mới img_008 #73 tiếp tục cho thấy phải bao mọi phần người nhìn thấy, không chỉ khuôn mặt.

## 7. QC vòng hai và quyết định

**Lưu lịch sử:** phiếu và báo cáo vòng đầu được giữ nguyên ở qc/history/round1/. Vòng đầu ghi 8/89 lỗi, nhưng kiểm tọa độ chi tiết xác minh lỗi img_035 #451 là báo nhầm. Sau đính chính còn **7/89 lỗi xác nhận trên 4/5 ảnh**. Không sửa số đo lịch sử để che sai sót; đính chính đi kèm nêu rõ lý do.

**Sửa nhãn:** xóa 3 box (một box xe ba bánh trùng ở img_003 và hai person giả trên quần áo ở img_031), chỉnh 4 box (bicycle/person img_006, person img_014 và img_031). Box img_035 được khôi phục chính xác về gốc. Kiểm bằng API cho thấy **469 hình khác giữ nguyên**, tổng hiện tại **473 annotation**. qc_fixed.json chứa 4 ảnh đã sửa, lấy từ export COCO thật của CVAT. Gold không đổi. Không sửa lại bẫy theo đáp án.

**Kiểm lại các mục vừa sửa:** 7 lỗi xác nhận của vòng đầu đã được sửa và đối chiếu lại; danh sách theo CVAT ID lưu trong automation/round1_confirmed_fixes_recheck.json. Đây là kiểm định có mục tiêu trên lỗi đã biết, không phải 7 đối tượng ngẫu nhiên độc lập.

**Mẫu mới:** seed **43**, 5/24 ảnh, 0 ca khó theo can_review + 5 ngẫu nhiên, **95 đối tượng**: img_004, img_008, img_017, img_024, img_034. Người kiểm là **Codex_visual_assisted_round2**. Mẫu này khác seed 42/89 đối tượng ở vòng đầu; không trừ hai tỉ lệ để tính mức cải thiện.

- Lỗi chắc chắn: **1/95=1.1% đối tượng**, **1/5=20.0% ảnh có lỗi**. Box person **#73 ở img_008** chỉ bao đầu, thiếu vai/cánh tay nhìn thấy ngoài box. ID theo release candidate hiện tại. Chưa sửa lỗi mới này trong vòng sửa các mục đã phát hiện trước đó; giữ trong phiếu vòng hai để xử lý tiếp.
- Bốn ảnh còn lại không có lỗi chắc chắn được ghi ở tỷ lệ đã kiểm, không có nghĩa đã chứng minh hoàn toàn sạch. Riêng img_024: phóng to xác nhận #256 là xe van, #259 là ô tô trắng khác ở phía sau; rút nhận xét sơ bộ rằng chúng trùng nhau, giữ nguyên cả hai box. Với img_034, hai người áo đỏ trên một xe là hai cá thể, không xóa box chỉ vì chồng nhau. Lớp vehicle #426 còn cần đối chiếu thêm; chưa khẳng định phân loại khi phần xe bị che.
- Loại lỗi ghi nhận vòng hai: sai_vi_tri_kich_thuoc, 1 trường hợp; không tạo thêm loại lỗi khi chưa đủ bằng chứng.
- Cận trên 95% Wilson của tỉ lệ ảnh lỗi: **62%** theo qc_report_auto.md. Mẫu nhỏ và cùng trợ lý tham gia sửa/kiểm không thay thế QC độc lập của người, không chứng minh ngưỡng lỗi ≤2% trên toàn lô.
- Bắt bẫy sau sửa: 11/12=91.7%, chỉ là mô tả sau khi đã biết key; không phải thí nghiệm mù mới.

**Quyết định đề xuất: Rework**, ngày **2026-11-19**: đã sửa lỗi cũ nhưng mẫu mới có 1/5 ảnh lỗi=20%, vượt ngưỡng Release 2%. Không đánh dấu Job hoàn tất. Việc tiếp theo là sửa box #73 img_008 theo phần người nhìn thấy, đối chiếu ca vehicle bị che ở img_034 và ảnh đêm nhòe, rồi lấy mẫu/kiểm tiếp. Học viên cần đọc, đối chiếu các nhận xét hỗ trợ này trước khi chốt bản nộp.

Nguồn: qc/qc_review.csv, qc/qc_sample.csv, qc/qc_report_auto.md, qc/evidence/ và automation/qc_visual_review.json. Script kiểm cấu trúc/ghép box/thống kê; không tự xác minh đúng vật, đúng mask hoặc mọi vật bị sót.

## 8. Ghi chú cá nhân và nguồn hỗ trợ

- Công cụ: Codex hỗ trợ lệnh, chuẩn hóa CSV, chạy model local, quét ngưỡng, tạo/gộp pre-label, xuất CVAT, kiểm trực quan 3 mask/5 ảnh, soạn calibration và báo cáo theo yêu cầu. Học viên đã cung cấp nhận xét r03/r04 và thời gian 30 phút/10 ảnh. Không trình bày phần công cụ thực hiện là học viên tự làm. Ở vòng hai, Codex sửa đúng các box được xác minh theo yêu cầu mới của học viên và tinh chỉnh mask.
- Gold: học viên xác nhận đã hoàn thành gán tay 4 ảnh trước khi yêu cầu chạy tiếp. Các lượt all do Codex chạy sau xác nhận đó. Không có xác minh độc lập toàn bộ lịch sử xem ảnh/model trước phiên trao đổi, nên không khẳng định vượt quá thông tin học viên cung cấp.
- Máy: Windows, RAM khoảng 16 GB, Intel Iris Xe, chạy model CPU. CVAT Docker báo phiên bản **2.74.1**, khác 2.81 trong tài liệu bài; dùng API của bản đang chạy để export COCO thật.
- Môi trường Python có thư viện là toolkits/day08/.venv; .venv bên trong repo thiếu numpy ở lần kiểm tra. YOLOE đã tự cài CLIP khi chạy. Không dùng data/cache cho hai lượt all và 14 ảnh toolkit trong phiên hỗ trợ.
- Vướng mắc tích hợp: API PATCH delete yêu cầu đầy đủ trường shape; lần gửi chỉ ID bị từ chối, chưa xóa dữ liệu. API có thể đổi thứ tự trả shapes, nên kiểm bảo toàn theo ID và toàn bộ trường, không theo thứ tự mảng. Đường dẫn dataset/download được thêm vào bridge giới hạn Task 27. Không sửa script chấm hay taxonomy.rules.
- Chưa đo riêng thời gian QC/viết báo cáo; không thêm số phút giả. Không có benchmark tốc độ đồng nhất giữa các lượt model hoặc phép đo IoU mask chuẩn.
- Đính chính QC: đã rút lỗi báo nhầm img_035 #451 sau khi kiểm lưới tọa độ, khôi phục box gốc. Nhận xét sơ bộ box trùng img_024 cũng được rút sau khi crop cho thấy hai xe khác nhau. Chỉ đưa lỗi đã xác minh vào phiếu cuối; không lấy suy đoán từ box chồng nhau làm lỗi.
- Hướng cải thiện cho lần sau: đo thời gian từng giai đoạn ngay khi làm; dùng checklist và QC độc lập sớm hơn, không đợi hết lô mới phát hiện lỗi person giả hoặc box lồng nhau.
