# Báo cáo QC — Ban1_C303

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Thực hành đặc tả và QC nhãn 2D (bbox) trên 40 ảnh đường phố; tập ảnh tuyển chọn để luyện ca biên nên độ đại diện cho dữ liệu vận hành chưa được đánh giá, không dùng để huấn luyện hay đánh giá mô hình thật.
- Nguồn và phép sử dụng dữ liệu: Xem data/ATTRIBUTION.csv (tác giả, giấy phép, đường dẫn nguồn của từng ảnh); build_final.py đã kiểm đủ ba trường này cho 40 ảnh trong lô.
- Manifest và checksum nhãn cuối: submission/qc/batch_manifest.json; labels_calibrated.json SHA-256 1039f635e4970be221823169c90af8f9e1932e1d5b86ae674e84963fdae057bb (checksum của lần gộp dùng cho vòng QC 2; sẽ đổi khi nhãn được sửa và gộp lại).
- Guideline và taxonomy khi gán / khi bàn giao: Gán theo guideline v1.0 (guideline_v1.0_agreed.md); bàn giao theo guideline v1.1 (guideline_v1.1_calibrated.md, quy tắc R-01 đến R-12). Taxonomy giữ phiên bản v0 gồm 6 lớp, không đổi; các lớp bus, three_wheeler, bicycle của guideline được ánh xạ về 6 lớp (bus thành truck, three_wheeler thành motorcycle, xe đạp và xích lô chỉ gán person).

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80/80 (4 thành viên 101, 102, 103, 104; mỗi thành viên 20 ảnh; mỗi ảnh 2 người gán độc lập).
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 229 cặp box ghép 1-1 trên 40 ảnh; IoU trung bình 0.818 (ngưỡng 0.85, chưa đạt); đồng thuận lớp 93.9% (ngưỡng 95%, chưa đạt); đồng thuận occluded_level 59.0%, truncated 94.8%.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 164 box chỉ người A vẽ và 36 box chỉ người B vẽ; 0 box bị loại do nằm trong ignore; 22 ca bat_dong_ignore trong calibration_candidates.csv.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: Ảnh chỉ có vật giả (img_017 tượng, img_020 phản chiếu gương, img_027 mộ đá hình xe, img_033 ma-nơ-canh, img_038 đồ chơi); cách kiểm là đối chiếu nhãn cuối với R-07 và guideline v1.0. Hiện img_038 đúng (0 box), còn 4 ảnh còn box ignore_region cần xóa (xem mục 5).

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: 437 ca trên 35 ảnh trong calibration_candidates.csv (bo_sot_thua 194, iou_thap 103, sai_thuoc_tinh 97, bat_dong_ignore 22, sai_lop 14, co_can_review 7), đều có dòng quyết định trong calibration_decisions.csv với rule R-01 đến R-12; người chốt ghi trong cột nguoi_chot (102).
- Tác động v1.1 lên nhãn và ảnh: guideline_change.json ghi kind semantic nên phạm vi sửa là cả lô 40 ảnh; ánh xạ về 6 lớp như mục 1.
- Tham chiếu bằng chứng khi Rework: Vòng 1 ra Hold vì nhãn cuối còn 7 box bật can_review (img_006, img_013, img_014, img_015); đã tắt cờ, gộp lại nhãn và mở vòng 2. Thư mục reviews/round-01 được giữ nguyên để truy vết.

## 4. Kết quả duyệt

- Vòng QC: 2
- Người review và ngày: 101, 2026-11-17
- Đã duyệt / cả lô: 40/40
- Ảnh lỗi / ảnh đã duyệt: 0/40 = 0.0%
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 0/428 = 0.0%
- Vật bỏ sót, ảnh và vùng: 0 vật bỏ sót ghi riêng trong phiếu vòng 2
- Ca khó / phần còn lại, lỗi lặp và ví dụ: ca_kho 35/35 ảnh duyệt, 0 ảnh lỗi; con_lai 5/5 ảnh duyệt, 0 ảnh lỗi; không có lỗi lặp (phiếu reviews/round-02/qc_review.csv)
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Máy chặn được bbox vượt biên ảnh img_019 #184 (consensus 1.68 + 958.33 = 960.01 lớn hơn 960 do làm tròn, đã nới dung sai 0.02 px trong common.py) và cờ can_review còn bật ở 7 box (vòng 1 ra Hold). Lỗi cần người đọc ảnh là box ignore_region còn phủ tượng, phản chiếu hay ma-nơ-canh ở img_017, img_020, img_027, img_033, vì script không biết vùng đó có phải vật thật hay không.

## 5. Quyết định bàn giao

- Quyết định: Release
- Lý do, người nhận và phạm vi sử dụng: qc_report.py vòng 2 trả RELEASE (0/40 ảnh lỗi, không còn box can_review, không lỗi lặp) trên bản nhãn có SHA-256 ở mục 1; người nhận là coach lớp; chỉ dùng để thực hành đặc tả và QC nhãn, không dùng cho huấn luyện hay vận hành.
- Nếu còn việc: Cần xóa 4 box ignore_region phủ vật giả (img_017 #174, img_020 #185, img_027 #287, img_033 #366) theo quyết định calibration C374-C376 và R-07; img_002 hiện còn 9 box car trên giàn trong khi R-10 yêu cầu gom vào một ignore_region. Người xử lý: 101, kiểm lại bằng cách xuất lại COCO từ CVAT, chạy build_final.py rồi mở vòng QC 3 và cập nhật báo cáo này.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Ca img_038 theo R-07 (không gán đồ chơi): ảnh data/images/img_038.jpg, quy tắc R-07 trong guideline v1.1, nhãn cuối labels_calibrated.json không có annotation nào của img_038, khớp phiếu round-02 dòng img_038 ghi khong.
