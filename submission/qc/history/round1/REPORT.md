# Báo cáo Day 08 — 2A202603117 - Nguyễn Xuân Anh

Phạm vi thực hiện: học viên gán tay 4 gold và tự review 10 ảnh batch đến hết img_017.jpg; công cụ local xử lý 14 ảnh còn lại. Codex kiểm trực quan 3 mask và 5 ảnh QC, hỗ trợ viết calibration và báo cáo theo yêu cầu. Người kiểm QC là **Codex_visual_assisted**, không phải kiểm định độc lập của con người. Bản này ghi kết quả thực tế và kết luận **Rework**, chưa phải bộ nhãn đạt Release.

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
- Conf không bảo đảm chất lượng: img_014 box person #108 conf 0.83 vẫn dư nền; img_031 #399 conf 0.4759 là quần áo treo. Ngược lại img_035 #468 conf 0.3898 ứng với người đàn ông đang ngồi nhìn thấy rõ. Các ID ở đây là ID release candidate.
- 14 ảnh bổ sung được chạy độc lập bằng YOLO11l + SAM ViT-B ở conf 0.25, imgsz 1024; gộp với YOLOE conf 0.3, lọc trùng và box cao dưới 20 px. Không dùng đáp án bẫy để xây kết quả này. Giữ nguyên cả ID, tọa độ, nhãn và thuộc tính của 119 hình trên 10 ảnh đầu; bằng chứng automation/day08_continuation.json.
- Giới hạn: chọn ngưỡng và đo trên cùng 4 gold làm kết quả trên gold lạc quan; ngưỡng chỉ là ước lượng của mẫu nhỏ. Không có mask chuẩn để đo IoU của SAM.

## 2. AI error profile

Nguồn: qc/review_stats.json và .md. Đây là **quy trình kết hợp người và công cụ** trên 24 ảnh, không phải toàn bộ do học viên tự review.

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 108 | 46.8% | 231 box máy |
| edited | 101 | 43.7% | 231 box máy |
| deleted | 22 | 9.5% | 231 box máy |
| added | 253 | 54.8% | 462 đối tượng sau duyệt được tính |

File nhãn có 476 đối tượng, nhưng script loại 27 đối tượng quanh bẫy khỏi so sánh; các mẫu số 231/462 không bằng số đối tượng toàn file. Có 5 cặp đổi lớp trong deleted+added, không nên coi mọi added là máy bỏ sót thật. Con số added−đổi lớp = 248 là ước lượng theo thuật toán ghép, không phải ground truth đếm bằng mắt.

- Theo lớp, person có 131 added và motorbike 80 added, là hai số lớn nhất theo số tuyệt đối. Điều này phản ánh độ lệch so với nhãn cuối, chưa chứng minh mọi nhãn cuối đều đúng. QC còn thấy nhãn person thừa trên quần áo ở img_031 và biên person sai ở img_006/img_035.
- So với gold: TP=28, FP=7, FN=40; precision **28/35=80.0%**, recall **28/68=41.2%**. Gold có 73 annotation toàn file; mẫu số recall là 68 đối tượng được script tính sau quy tắc ignore. Khoảng tin cậy 95%: precision 64–90%, recall 30–53%.
- Riêng 10 ảnh học viên làm: accepted **68/86=79.1%**, edited **12/86=14.0%**, deleted **6/86=7.0%**, added **34/114=29.8%**. Nguồn: qc/review_stats_human_only.md. Không gộp các tỉ lệ này với phần công cụ để suy hiệu quả cá nhân.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Cả bộ kết hợp: **11/12=91.7%**, gồm loose 2/3, missing 3/3, wrong_class 3/3, extra 3/3; còn lọt bẫy loose ở img_013.jpg. Khoảng tin cậy 95% tổng thể 65–99%.
- Riêng 10 ảnh học viên làm: **4/5=80.0%**, khoảng tin cậy 95% 38–96%; không coi 11/12 là kết quả cá nhân.
- Chưa có nhật ký thao tác đủ để kết luận học viên đã bỏ một bước cụ thể. Bẫy loose bị lọt cho thấy cần kiểm riêng độ ôm sát; đây là hướng cải tiến quy trình, không phải khẳng định nguyên nhân tâm lý đã được đo.
- Accepted cao không bảo đảm không có lỗi: ảnh img_003 vẫn có hai box cho cùng xe ba bánh, và kiểm QC phát hiện lỗi hình học trên cả nhãn người lẫn xe. Không dùng tỷ lệ accepted thay cho rà ảnh gốc.
- Đáp án bẫy chỉ được dùng sau khi đã export batch_reviewed.json. Không sửa lại phần đã làm theo đáp án để tăng số đo.

## 4. Máy có đáng dùng không?

- Gold: học viên ghi **30 phút/4 ảnh=7.5 phút/ảnh**. Review thủ công: học viên báo **30 phút/10 ảnh=3.0 phút/ảnh**. Nguồn time_log.csv; đây là hai tập ảnh khác độ khó, không phải thí nghiệm đối chứng, nên không kết luận chắc chắn tiết kiệm 60% thời gian.
- Toolkit xử lý 14 ảnh: tổng thời gian xử lý từng ảnh **2.626 phút**, khoảng **11.25 giây/ảnh**, chưa gồm nạp model, xuất CVAT và kiểm chất lượng. Không so thời gian chạy model với thời gian review thủ công như hai công việc tương đương.
- Đánh giá từ bằng chứng hiện có: dùng máy làm nhãn khởi đầu và ưu tiên review là hữu ích; chưa phù hợp để chấp nhận tự động. Recall gold 41.2% và lỗi QC 5/5 ảnh cho thấy vẫn cần quét bỏ sót, kiểm danh tính vật và chỉnh biên.
- Ba quy tắc quy trình đề xuất để học viên áp dụng:
  1. Mỗi ảnh ẩn box trước, tự quét người/xe rồi hiện nhãn; hoàn thành đủ 24/24 ảnh theo checklist.
  2. Mỗi box kiểm lần lượt vật thật, đúng lớp và biên; không có mức conf nào được bỏ qua bước này. Box lồng nhau phải đối chiếu cùng vật hay hai vật.
  3. Ghi ca mơ hồ theo tên ảnh và can_review, nghỉ sau khoảng 8–10 ảnh; chỉ đọc đáp án bẫy sau export và giữ số đo vòng đầu để tránh sửa kết quả hồi tố.

## 5. Mask SAM cho 3 vật (Part 5)

Ba ví dụ dùng **SAM 2.1 base**, mỗi ví dụ một box YOLOE và **1 điểm dương tự động**, **0 điểm âm**. Đây không phải 3 lần click do học viên thực hiện. Nguồn tham số/diện tích: automation/sam_point_examples.json. Codex đã đối chiếu overlay theo yêu cầu, chưa có ground truth mask.

| Ảnh / file trong masks | Người lái / hàng hóa | Quan sát trực quan | Xử lý tiếp |
|---|---|---|---|
| img_013 / sam21_candidate_img_013_1.jpg | Còn lẫn một phần tay/chân người lái | Bám mái, thân và bánh chính; biên quanh người/khoang hở còn lỗi; diện tích 292427 px | Rework: thêm điểm âm trên phần người/nền bị lấy thừa, rà lại toàn biên |
| img_023 / sam21_candidate_img_023_1.jpg | Chưa thấy người bị lấy lẫn rõ ở tỷ lệ toàn ảnh | Bám mái và thân tuk-tuk phía sau; diện tích 35818 px. Chưa có số đo sai số biên/kính | Chưa thấy lỗi lớn; vẫn cần học viên kiểm chi tiết, không coi là đã đạt IoU |
| img_023 / sam21_candidate_img_023_2.jpg | Còn lấy một phần tay người lái ở khoang hở | Bám xe giữa ảnh; diện tích 56641 px | Rework: thêm điểm âm trên tay người, kiểm lại quanh tay lái và giỏ |

Guideline 4.2 về phần nhìn thấy, loại bóng đổ và lỗ lớn giúp kiểm mask. CAL-04 bổ sung cách phân biệt khoang hở với kính vật lý và tách phần người khỏi mask xe. Giữ cả kết quả chưa tốt làm bằng chứng, không báo rằng cả ba mask đạt chất lượng. Interactor trực tiếp CVAT là tùy chọn; các ví dụ này chạy bằng scripts/sam2_points.py, không có phép so sánh trực tiếp để kết luận khác biệt công cụ.

## 6. Calibration (Part 6)

Có **5 quyết định** trong qc/calibration_decisions.csv, đồng bộ với guidelines/guideline_v1.1_calibrated.md. Nội dung được Codex soạn theo yêu cầu, học viên cần đọc và giải thích khi dùng bản nộp.

- CAL-01, img_003: một xe ba bánh có mái chỉ là một đối tượng; rà box lồng nhau theo danh tính xe.
- CAL-02, img_031: quần áo treo không phải person; đây là lỗi máy, không nới định nghĩa lớp để hợp thức hóa dự đoán.
- CAL-03, img_031, nguồn cau_hoi_mo: phân biệt giỏ gắn cố định và hàng rời; giữ quy tắc overhang khoảng 20% cho box, không dùng box làm mask xe.
- CAL-04, img_013/img_023, nguồn cau_hoi_mo: kính thật thuộc xe, khoang hở không phải kính; phần người/nền trong khoang hở không thuộc mask xe.
- CAL-05, img_031/img_014, nguồn cau_hoi_mo: conf không thay thế kiểm tra lớp, vật thật và hình học.

Các câu hỏi chưa đủ bằng chứng để chốt tổng quát (xe điện, cụm xe sát camera, ngưỡng che người, ảnh đêm) được nêu rõ trong v1.1. Không đổi taxonomy.rules. Những lỗi tay/chân bị cắt là lỗi thực hiện quy tắc sẵn có, không phải lý do đổi dung sai. Calibration chưa được áp dụng bằng cách sửa nhãn; bộ hiện tại dùng để ghi nhận vòng QC đầu và cần Rework.

## 7. QC và quyết định (Part 6)

- Mẫu cố định seed 42: **5/24 ảnh**, 0 ca khó được chọn theo can_review + 5 ngẫu nhiên; **89 đối tượng**. Ảnh: img_003, img_006, img_014, img_031, img_035.
- Người kiểm: **Codex_visual_assisted**, đối chiếu ảnh gốc, ảnh có ID và crop. Đây là kiểm hỗ trợ bằng AI; cùng trợ lý đã chuẩn bị một phần nhãn, nên không độc lập và không thay thế QC con người.
- Phiếu ghi **8/89=9.0% đối tượng lỗi**, **5/5=100.0% ảnh có lỗi**. Đây là 8 lỗi chắc chắn đã ghi nhận, không bảo đảm đã tìm hết mọi lỗi hoặc vật bị sót. Không suy tỉ lệ 100% cho toàn bộ 24 ảnh.
- Loại lỗi: **5/8 lỗi hình học**, ví dụ img_006 #48/#50, img_014 #108, img_031 #398, img_035 #451; **3/8 box thừa**, img_003 #19 và img_031 #399/#400. Không ép ra loại lỗi thứ ba khi chưa có bằng chứng. ID này thuộc release candidate, khác ID nội bộ CVAT.
- Ví dụ và ghi chú chi tiết: qc/qc_review.csv; ảnh có ID trong qc/evidence/; metadata kiểm tra automation/qc_visual_review.json.
- Nguồn tính: qc/qc_report_auto.md, chạy qc_report.py từ phiếu đã điền. Lỗi hình học 5/8 lặp lại được script đánh dấu lỗi hệ thống. Ngưỡng Release theo ảnh là ≤2%; mẫu hiện tại không đạt.
- Mẫu chỉ có 5 ảnh: cận trên 95% Wilson cho tỉ lệ ảnh lỗi là 100%; không thể dùng mẫu nhỏ này chứng minh chất lượng ≤2%. Không có self-check mù hay người kiểm độc lập.
- Script kiểm được cấu trúc COCO, số ảnh, lớp, tọa độ, số đo ghép box và tỷ lệ từ phiếu. Script không xác minh box ôm đúng vật, mask đúng biên hoặc vật cả người và máy cùng bỏ sót; các điểm đó cần kiểm trực quan.

**Quyết định đề xuất: Rework**, ngày **2026-11-19**. Trong các ca đã kiểm, không có ảnh hỏng hoặc ca đang làm dừng việc áp dụng guideline để buộc Hold toàn lô; có lỗi thực tế 5/5 ảnh và lỗi hình học lặp lại, nên không Release. Các ca mới ngoài phạm vi quy tắc v1.1 vẫn phải Hold riêng để làm rõ.

Việc tiếp theo: sửa các ID đã ghi; rà person giả do quần áo và biên tay/chân trên 14 ảnh công cụ tạo; tinh chỉnh hai mask còn lẫn người; kiểm lại vòng mới, giữ phiếu và số đo vòng đầu. Vì có yêu cầu giữ nguyên phần học viên đã làm, phiên QC này **không sửa nhãn hay ghi đè CVAT**, kể cả các lỗi phát hiện ở 10 ảnh đầu. Chỉ tạo qc_fixed.json sau khi thực sự sửa, rồi build_release và lấy mẫu QC vòng sau. Chưa đánh dấu Job hoàn tất. Chốt cuối cùng do học viên sau khi đọc bằng chứng.

## 8. Ghi chú cá nhân và nguồn hỗ trợ

- Công cụ: Codex hỗ trợ lệnh, chuẩn hóa CSV, chạy model local, quét ngưỡng, tạo/gộp pre-label, xuất CVAT, kiểm trực quan 3 mask/5 ảnh, soạn calibration và báo cáo theo yêu cầu. Học viên đã cung cấp nhận xét r03/r04 và thời gian 30 phút/10 ảnh. Không trình bày phần công cụ thực hiện là học viên tự làm.
- Gold: học viên xác nhận đã hoàn thành gán tay 4 ảnh trước khi yêu cầu chạy tiếp. Các lượt all do Codex chạy sau xác nhận đó. Không có xác minh độc lập toàn bộ lịch sử xem ảnh/model trước phiên trao đổi, nên không khẳng định vượt quá thông tin học viên cung cấp.
- Máy: Windows, RAM khoảng 16 GB, Intel Iris Xe, chạy model CPU. CVAT Docker báo phiên bản **2.74.1**, khác 2.81 trong tài liệu bài; dùng API của bản đang chạy để export COCO thật.
- Môi trường Python có thư viện là toolkits/day08/.venv; .venv bên trong repo thiếu numpy ở lần kiểm tra. YOLOE đã tự cài CLIP khi chạy. Không dùng data/cache cho hai lượt all và 14 ảnh toolkit trong phiên hỗ trợ.
- Vướng mắc tích hợp: API PATCH delete yêu cầu đầy đủ trường shape; lần gửi chỉ ID bị từ chối, chưa xóa dữ liệu. API có thể đổi thứ tự trả shapes, nên kiểm bảo toàn theo ID và toàn bộ trường, không theo thứ tự mảng. Đường dẫn dataset/download được thêm vào bridge giới hạn Task 27. Không sửa script chấm hay taxonomy.rules.
- Chưa đo riêng thời gian QC/viết báo cáo; không thêm số phút giả. Không có benchmark tốc độ đồng nhất giữa các lượt model hoặc phép đo IoU mask chuẩn.
- Hướng cải thiện cho lần sau: đo thời gian từng giai đoạn ngay khi làm; dùng checklist và QC độc lập sớm hơn, không đợi hết lô mới phát hiện lỗi person giả hoặc box lồng nhau.
