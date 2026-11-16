# Guideline gán nhãn cá nhân — v1.0 (bản đề xuất)

- Người viết: cá nhân người gán nhãn
- Công cụ: CVAT
- Định dạng xuất: COCO 1.0
- Phạm vi: 40 ảnh đường phố của Day 07
- Trạng thái: bản cá nhân đề xuất để thảo luận ở Pha 2; chưa phải guideline nhóm đã khóa.

## 1. Mục tiêu
Tạo bounding box nhất quán cho người và phương tiện thật nhìn thấy trong ảnh đường phố. Ưu tiên không bỏ sót đối tượng hợp lệ, không gán nhầm vật không phải người/phương tiện thật, và để reviewer nhận ra các ca chưa chắc chắn.

## 2. Lớp và thuộc tính

| Lớp | Gán khi | Không gán nhầm với |
|---|---|---|
| `car` | Ô tô chở người, gồm sedan, hatchback, SUV và pickup dùng như xe con. | `truck` nếu có thùng hàng chuyên dụng rõ ràng; `van` nếu là xe thùng kín cỡ nhỏ. |
| `truck` | Xe tải có cabin và thùng/sàn chở hàng hoặc kết cấu chuyên dụng rõ ràng. | `car`/SUV; xe van thùng kín cỡ nhỏ. |
| `van` | Xe chở người/hàng cỡ nhỏ, thân hộp và khoang sau kín liền thân. | Xe tải có thùng hàng riêng; ô tô con/SUV. |
| `motorcycle` | Xe hai bánh có động cơ, gồm xe máy và mô tô. | Xe đạp không thuộc taxonomy. |
| `person` | Người thật, gồm người đi bộ, người lái và người ngồi trên phương tiện khi nhìn thấy đủ để xác định là người. | Tượng, hình in, hình phản chiếu, ma-nơ-canh và đồ chơi. |
| `ignore_region` | Vùng chứa vật/đám đông quá nhỏ, quá dày hoặc không thể tách từng đối tượng theo quy tắc dưới đây. Đây là vùng bỏ qua, không phải một vật thể. | Không dùng để che một đối tượng bình thường chỉ vì khó vẽ box. |

**Quy tắc ngoài taxonomy:** không tự tạo class mới. Tuk-tuk/xe ba bánh không khớp rõ một trong các lớp trên thì không gán class phương tiện; nếu vùng đó gây mơ hồ cho việc đánh giá, vẽ `ignore_region` bao vùng mơ hồ và bật `can_review` nếu cần. Xe đạp, xe buýt, tàu, vật thể tưởng tượng và phương tiện không thuộc các lớp trên cũng xử lý tương tự. Không gán tượng, ma-nơ-canh, hình in, ảnh trên biển quảng cáo hoặc phản chiếu thành `person`/phương tiện.

### Thuộc tính
| Thuộc tính | Giá trị | Quy tắc chọn |
|---|---|---|
| `occluded_level` | `0` | Không bị che hoặc phần bị che không đáng kể (ước lượng dưới 10%). |
| | `1` | Bị che nhẹ, khoảng 10–35% diện tích toàn vật. |
| | `2` | Bị che rõ, khoảng trên 35% đến 70%, nhưng vẫn nhận diện được lớp. |
| | `3` | Bị che trên 70% hoặc không đủ căn cứ xác định chắc lớp; nếu không xác định được vật thể thì không tạo box đoán mò, dùng `ignore_region`/ghi issue. |
| `truncated` | `no` | Vật nằm trọn trong khung ảnh, dù bị vật khác che. |
| | `yes` | Vật bị đường biên ảnh cắt mất một phần. |
| `can_review` | tick | Chỉ tick khi còn nghi ngờ về class, số box, ranh giới box hoặc cách xử lý ca ngoài taxonomy. |

Các tỷ lệ che là ước lượng nhất quán bằng mắt, không cần đo pixel. Nếu phân vân giữa hai mức, chọn mức cao hơn và tick `can_review`.

## 3. Quy tắc vẽ box trên CVAT

1. Dùng Rectangle, mỗi đối tượng hợp lệ một box; box sát **phần nhìn thấy** của đối tượng, không ước lượng phần bị che khuất.
2. Người đi xe máy: vẽ một box `motorcycle` quanh phần xe nhìn thấy và một box `person` cho từng người nhìn thấy. Không gộp người và xe vào một box. Người ngồi sau cũng là một `person` riêng nếu phân biệt được.
3. Xe chở xe: gán box cho xe chở (thường `truck` nếu có kết cấu xe tải rõ) và từng xe được chở nếu chúng nhìn thấy, đủ lớn và thuộc taxonomy. Không gộp cả cụm thành một box.
4. Vật bị che: vẫn gán nếu phần nhìn thấy đủ nhận diện; box chỉ ôm phần nhìn thấy, đặt `occluded_level` theo tỷ lệ ước lượng.
5. Vật chạm/cắt mép ảnh: vẽ box đến đúng biên ảnh, đặt `truncated=yes`. Không tự kéo box ra ngoài ảnh.
6. Vật quá nhỏ: nếu chiều cao box dưới 20 px, không tạo box đối tượng; dùng `ignore_region` cho vùng liên quan theo quy tắc lab.
7. Đám đông: vẽ từng `person` khi từng người tách biệt được. Nếu chồng lấn dày đến mức không thể tách box đáng tin cậy, dùng một `ignore_region` bao vùng đám đông thay vì box người tùy tiện.
8. Xe/đối tượng phản chiếu, ảnh in, tượng, mô hình hoặc đồ chơi: không gán là vật thể thật. Chỉ dùng `ignore_region` nếu vùng đó là nguồn mơ hồ đáng kể cho reviewer.
9. Không vẽ box cho vùng đường, bóng đổ, cây cối, biển báo hoặc nền.
10. Ảnh không có đối tượng hợp lệ vẫn giữ trong Task và COCO; không tạo box giả.

## 4. Ca khó và cách ghi nhận

| Tình huống | Quyết định cá nhân | Lý do |
|---|---|---|
| Người lái/ngồi trên xe máy | Hai loại box riêng: `person` và `motorcycle`. | Hai đối tượng mục tiêu độc lập. |
| Xe ba bánh/tuk-tuk | Không ép vào `car`/ `truck`/ `van`; dùng `ignore_region` nếu cần đánh dấu vùng mơ hồ. | Taxonomy hiện không có class phù hợp. |
| Xe tải chở xe | Gán xe tải và từng xe được chở đủ điều kiện thành các box riêng. | Mỗi vật thể thuộc lớp mục tiêu có box riêng. |
| Tượng/ma-nơ-canh/hình in/phản chiếu | Không gán `person`. | Không phải người thật trong cảnh. |
| Vật cao dưới 20 px | `ignore_region`, không tạo box đối tượng. | Theo ngưỡng vật nhỏ của lab. |
| Người/xe bị cắt ở biên | Vẽ phần nhìn thấy, `truncated=yes`. | Giữ thông tin đối tượng hiện diện trong ảnh. |
| Đối tượng bị che quá nhiều, không xác định lớp | Không đoán class; đánh dấu vùng và ghi issue/`can_review`. | Tránh nhãn suy diễn. |

Khi áp dụng trên 40 ảnh, ghi mã ảnh chính xác (ví dụ `img_0xx`) vào bảng này cùng mô tả quan sát thực tế. Không tự điền ID ảnh khi chưa xác minh ảnh.

## 5. QC và phiên bản

- Kiểm tra đủ 40 ảnh, đúng class, mỗi vật một box, box sát phần nhìn thấy, thuộc tính đầy đủ và không gán nhầm tượng/phản chiếu.
- Ngưỡng QC của lab: box cao từ 50 px có IoU 0.85; box nhỏ hơn 50 px có IoU 0.70; IAA IoU trung bình tối thiểu 0.85 và đồng thuận class tối thiểu 95% trên các cặp box ghép hợp lệ. Ngưỡng lỗi ảnh tối đa 2%; lô 40 ảnh cần không có ảnh lỗi để đạt ngưỡng này.
- `can_review` không thay thế việc ghi `image_issues.csv` khi không thể tạo box.
- Mọi thay đổi làm đổi class, số box, phạm vi box hoặc thuộc tính phải tăng phiên bản; sửa câu chữ không đổi ý nghĩa có thể ghi là editorial.
- Chỉ người được nhóm phân công chốt ca calibration; mọi quyết định phải có ảnh, lý do, rule ID và người kiểm lại.

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v1.0 cá nhân | Bổ sung định nghĩa class, quy tắc người đi xe máy, vật ngoài taxonomy, box/che khuất/cắt mép/vật nhỏ và QC. | Các điểm v0 chưa quy định đủ để hai người gán nhất quán. |
