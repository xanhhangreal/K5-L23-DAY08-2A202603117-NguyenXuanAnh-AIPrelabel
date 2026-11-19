# Báo cáo Day 08 — 2A202603117 - Nguyễn Xuân Anh

Phạm vi thực hiện: học viên gán tay 4 gold và tự review 10 ảnh batch đến hết img_017.jpg; công cụ local xử lý 14 ảnh còn lại. Codex kiểm trực quan 3 mask và 5 ảnh QC, hỗ trợ viết calibration và báo cáo theo yêu cầu. Người kiểm QC là **Codex_visual_assisted**, không phải kiểm định độc lập của con người. Đã sửa các lỗi QC vòng đầu được xác minh và tinh chỉnh hai mask. QC vòng hai dùng mẫu mới seed 43, sửa thêm lỗi mới ở img_008 và kiểm lại. Hiện đề xuất **Hold** vì còn một ca chưa chốt lớp ở img_034; chưa Release.

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
- 14 ảnh bổ sung được chạy độc lập bằng YOLO11l + SAM ViT-B ở conf 0.25, imgsz 1024; gộp với YOLOE conf 0.3, lọc trùng và box cao dưới 20 px. Không dùng đáp án bẫy để xây kết quả này. Ở bước tạo nhãn 14 ảnh còn lại đã giữ nguyên 119 hình của 10 ảnh đầu. Sau đó học viên yêu cầu sửa lỗi QC, cho phép sửa đúng các đối tượng đã xác minh trong cả phần đầu. Tổng hai lượt sửa QC đã xóa 3 box và chỉnh 5 box; 468 hình khác giữ nguyên. Bằng chứng automation/qc_round2_fixes.json.
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

- Theo lớp: person có 130 added, motorbike 80 added; đây là chênh lệch theo số tuyệt đối so với bộ cuối, không bảo đảm mọi nhãn cuối đúng. Mẫu mới vòng hai từng phát hiện box person bị thiếu phần vai ở img_008; đã sửa và kiểm lại, đối tượng hiện là release ID #69.
- So với gold không đổi: TP=28, FP=7, FN=40; precision **28/35=80.0%**, recall **28/68=41.2%**. Gold có 73 annotation toàn file, mẫu số recall là 68 đối tượng theo quy tắc script. KTC 95%: precision 64–90%, recall 30–53%.
- Số đo lịch sử riêng **10 ảnh học viên làm trước sửa QC** giữ nguyên trong qc/review_stats_human_only.md: accepted 68/86=79.1%, edited 12/86=14.0%, deleted 6/86=7.0%, added 34/114=29.8%. Không gọi đây là số đo hiện tại của phần đã được công cụ sửa thêm.
- Số đo trước sửa được lưu trong qc/history/round1/review_stats.* để đối chiếu, không chỉnh tay đầu ra script.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Cả bộ kết hợp sau sửa QC: **11/12=91.7%**, gồm loose 2/3, missing 3/3, wrong_class 3/3, extra 3/3; còn lọt bẫy loose ở img_013.jpg. Khoảng tin cậy 95% tổng thể 65–99%.
- Riêng 10 ảnh học viên làm, trước sửa QC: **4/5=80.0%**, khoảng tin cậy 95% 38–96%; không coi 11/12 là kết quả cá nhân.
- Kết quả sau sửa QC không phải phép thử bẫy mù mới: đáp án đã được mở sau export vòng đầu. Không dùng số đo sau sửa để suy thiên kiến độc lập của học viên.
- Chưa có nhật ký thao tác đủ để kết luận học viên đã bỏ một bước cụ thể. Bẫy loose bị lọt cho thấy cần kiểm riêng độ ôm sát; đây là hướng cải tiến quy trình, không phải khẳng định nguyên nhân tâm lý đã được đo.
- Accepted cao không bảo đảm không có lỗi: QC vòng đầu thấy hai box cho cùng xe ba bánh ở img_003; lỗi này đã sửa. QC vòng hai từng thấy box person thiếu vai ở img_008; đã sửa theo phần nhìn thấy. Không dùng tỷ lệ accepted thay cho rà ảnh gốc.
- Đáp án bẫy chỉ được dùng sau khi đã export batch_reviewed.json. Không sửa lại phần đã làm theo đáp án để tăng số đo.

## 4. Máy có đáng dùng không?

- Gold: học viên ghi **30 phút/4 ảnh=7.5 phút/ảnh**. Review thủ công: học viên báo **30 phút/10 ảnh=3.0 phút/ảnh**. Nguồn time_log.csv; đây là hai tập ảnh khác độ khó, không phải thí nghiệm đối chứng, nên không kết luận chắc chắn tiết kiệm 60% thời gian.
- Toolkit xử lý 14 ảnh: tổng thời gian xử lý từng ảnh **2.626 phút**, khoảng **11.25 giây/ảnh**, chưa gồm nạp model, xuất CVAT và kiểm chất lượng. Không so thời gian chạy model với thời gian review thủ công như hai công việc tương đương.
- Đánh giá từ bằng chứng hiện có: dùng máy làm nhãn khởi đầu và ưu tiên review là hữu ích; chưa phù hợp để chấp nhận tự động. Recall gold 41.2% và lịch sử lỗi QC, dù mẫu đã sửa hiện ghi nhận 0/5 ảnh lỗi chắc chắn, cho thấy vẫn cần quét bỏ sót, kiểm danh tính vật và chỉnh biên.
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

Có **5 quyết định đã soạn và 1 ca Hold chưa chốt lớp (CAL-06)** trong qc/calibration_decisions.csv, đồng bộ với guidelines/guideline_v1.1_calibrated.md. Nội dung được Codex soạn theo yêu cầu, học viên cần đọc và giải thích khi dùng bản nộp.

- CAL-01, img_003: một xe ba bánh có mái chỉ là một đối tượng; rà box lồng nhau theo danh tính xe.
- CAL-02, img_031: quần áo treo không phải person; đây là lỗi máy, không nới định nghĩa lớp để hợp thức hóa dự đoán.
- CAL-03, img_031, nguồn cau_hoi_mo: phân biệt giỏ gắn cố định và hàng rời; giữ quy tắc overhang khoảng 20% cho box, không dùng box làm mask xe.
- CAL-04, img_013/img_023, nguồn cau_hoi_mo: kính thật thuộc xe, khoang hở không phải kính; phần người/nền trong khoang hở không thuộc mask xe.
- CAL-05, img_031/img_014, nguồn cau_hoi_mo: conf không thay thế kiểm tra lớp, vật thật và hình học.
- CAL-06, img_034: xe bị che ở release ID #426 chưa đủ bằng chứng để chốt truck hay three_wheeler; giữ Hold để học viên/người kiểm xác nhận. Không dùng đầu ra model làm đáp án.

Các câu hỏi chưa đủ bằng chứng để chốt tổng quát (xe điện, cụm xe sát camera, ngưỡng che người, ảnh đêm) được nêu rõ trong v1.1. Không đổi taxonomy.rules. Lỗi tay/chân bị cắt là lỗi thực hiện quy tắc sẵn có, không phải lý do đổi dung sai. Đã áp dụng sửa nhãn cho các lỗi vòng đầu được xác minh. Nhận xét về chân ở img_035 #451 vòng đầu là báo nhầm khi đối chiếu lưới tọa độ; box gốc được khôi phục, không tính là lỗi đã sửa. Đính chính trong qc/history/round1/erratum_img035.json. Lỗi mới img_008 #73 trong snapshot trước sửa cho thấy phải bao mọi phần người nhìn thấy, không chỉ khuôn mặt; đã chỉnh CVAT ID 2711, hiện là release ID #69.

## 7. QC vòng hai và quyết định

**Lịch sử:** giữ nguyên phiếu/báo cáo vòng đầu tại qc/history/round1/. Vòng đầu ghi 8/89 lỗi; kiểm tọa độ sau đó xác minh img_035 #451 là báo nhầm. Sau đính chính còn **7 lỗi xác nhận trên 4/5 ảnh** của mẫu 89 đối tượng. Box img_035 đã khôi phục chính xác về gốc; không tính là lỗi đã sửa. Đính chính có bằng chứng lưới tọa độ.

**Các sửa đổi đã thực hiện trong CVAT:** xóa 3 box (xe ba bánh trùng img_003 và 2 person giả trên quần áo img_031), chỉnh 4 box cũ (bicycle/person img_006, person img_014/img_031), rồi sửa thêm box person mới phát hiện ở img_008. Tổng **3 lần xóa + 5 lần chỉnh box**, **468 hình khác giữ nguyên** so với snapshot 476 hình trước sửa. Bộ mới có **24 ảnh, 473 annotation**. qc_fixed.json chứa 5 ảnh thực sự đã sửa và lấy từ export COCO thật; gold không đổi. Không sửa bẫy theo đáp án.

**Kiểm lại có mục tiêu:** 7 lỗi xác nhận của mẫu đầu đã sửa và được đối chiếu lại; sau đó lỗi person chỉ bao đầu ở img_008 đã được mở biên theo vai còn nhìn thấy. Đối tượng này có ID #73 ở snapshot trước sửa thêm, hiện là **#69**, CVAT ID ổn định **2711**. Không coi việc kiểm lại chính lỗi đã sửa là phép đo độc lập.

**Mẫu QC mới ban đầu chọn seed 43:** 5/24 ảnh, 0 ca khó theo can_review + 5 ngẫu nhiên, **95 đối tượng**: img_004, img_008, img_017, img_024, img_034. Sau khi mẫu này phát hiện lỗi img_008, đã sửa rồi kiểm lại cùng 5 ảnh. Đây là **kiểm hồi quy trên mẫu đã sửa**, không phải mẫu thứ ba mới lấy sau khi sửa. Người kiểm **Codex_visual_assisted_round2_after_fix**, không độc lập với người/công cụ sửa.

- Hiện phiếu ghi **0/95 đối tượng lỗi chắc chắn** và **0/5 ảnh có lỗi chắc chắn** sau sửa, kèm **1 ca phân loại chưa chốt** CAL-06 ở img_034 #426. Không diễn giải 0 là không còn lỗi trong toàn lô, hoặc ca chưa chốt đã được xác nhận đúng.
- Riêng img_024: crop xác nhận #256 là xe van, #259 là ô tô trắng khác phía sau; rút nhận xét sơ bộ rằng chúng trùng, giữ nguyên hai box. Với img_034, hai người áo đỏ trên một xe là hai cá thể; không xóa box do chồng nhau. Xe bị che ở #426 vẫn cần người đối chiếu thêm về lớp.
- Cận trên 95% Wilson cho tỉ lệ ảnh lỗi khi 0/5 là **43%** theo qc_report_auto.md; đây còn là mẫu đã tham gia sửa nên số đo lạc quan. Không chứng minh chất lượng ≤2% trên toàn bộ 24 ảnh.
- Bắt bẫy sau sửa **11/12=91.7%** không đổi; đây không phải phép thử mù mới vì key đã được mở sau export vòng đầu.

Chạy `python scripts/qc_report.py --guideline-gap 1` để phản ánh **1 ca áp dụng lớp chưa được chốt**, không sửa đầu ra báo cáo tự động bằng tay. Con số này không có nghĩa toàn bộ các câu hỏi mở khác đã được giải quyết tổng quát.

**Quyết định đề xuất: Hold**, ngày **2026-11-19**. Các lỗi đã xác minh được sửa và mẫu sau sửa không ghi nhận thêm lỗi chắc chắn, nhưng chưa chốt ca CAL-06; cần học viên/người kiểm xác nhận xe #426 img_034 là truck hay three_wheeler hoặc có bằng chứng khác trước khi quyết định tiếp. Sau khi chốt, cập nhật nhãn/thuộc tính nếu cần và lấy mẫu QC mới độc lập hơn. Chưa Release, chưa đánh dấu Job hoàn tất.

Nguồn: qc/qc_review.csv, qc/qc_sample.csv, qc/qc_report_auto.md, qc/evidence/, automation/qc_round2_fixes.json và automation/qc_visual_review.json. Script kiểm cấu trúc/ghép box/thống kê; không tự chứng minh đúng vật, đúng mask hoặc mọi vật bị sót.

## 8. Ghi chú cá nhân và nguồn hỗ trợ

- Công cụ: Codex hỗ trợ lệnh, chuẩn hóa CSV, chạy model local, quét ngưỡng, tạo/gộp pre-label, xuất CVAT, kiểm trực quan 3 mask/5 ảnh, soạn calibration và báo cáo theo yêu cầu. Học viên đã cung cấp nhận xét r03/r04 và thời gian 30 phút/10 ảnh. Không trình bày phần công cụ thực hiện là học viên tự làm. Ở vòng hai, Codex sửa đúng các box được xác minh theo yêu cầu mới của học viên và tinh chỉnh mask.
- Gold: học viên xác nhận đã hoàn thành gán tay 4 ảnh trước khi yêu cầu chạy tiếp. Các lượt all do Codex chạy sau xác nhận đó. Không có xác minh độc lập toàn bộ lịch sử xem ảnh/model trước phiên trao đổi, nên không khẳng định vượt quá thông tin học viên cung cấp.
- Máy: Windows, RAM khoảng 16 GB, Intel Iris Xe, chạy model CPU. CVAT Docker báo phiên bản **2.74.1**, khác 2.81 trong tài liệu bài; dùng API của bản đang chạy để export COCO thật.
- Môi trường Python có thư viện là toolkits/day08/.venv; .venv bên trong repo thiếu numpy ở lần kiểm tra. YOLOE đã tự cài CLIP khi chạy. Không dùng data/cache cho hai lượt all và 14 ảnh toolkit trong phiên hỗ trợ.
- Vướng mắc tích hợp: API PATCH delete yêu cầu đầy đủ trường shape; lần gửi chỉ ID bị từ chối, chưa xóa dữ liệu. API có thể đổi thứ tự trả shapes, nên kiểm bảo toàn theo ID và toàn bộ trường, không theo thứ tự mảng. Đường dẫn dataset/download được thêm vào bridge giới hạn Task 27. Không sửa script chấm hay taxonomy.rules.
- Chưa đo riêng thời gian QC/viết báo cáo; không thêm số phút giả. Không có benchmark tốc độ đồng nhất giữa các lượt model hoặc phép đo IoU mask chuẩn.
- Đính chính QC: đã rút lỗi báo nhầm img_035 #451 sau khi kiểm lưới tọa độ, khôi phục box gốc. Nhận xét sơ bộ box trùng img_024 cũng được rút sau khi crop cho thấy hai xe khác nhau. Chỉ đưa lỗi đã xác minh vào phiếu cuối; không lấy suy đoán từ box chồng nhau làm lỗi.
- Hướng cải thiện cho lần sau: đo thời gian từng giai đoạn ngay khi làm; dùng checklist và QC độc lập sớm hơn, không đợi hết lô mới phát hiện lỗi person giả hoặc box lồng nhau.
