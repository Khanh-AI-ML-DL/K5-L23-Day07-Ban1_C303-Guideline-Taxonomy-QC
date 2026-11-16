# GUIDELINE GÁN NHÃN — v0.1 — CÁ NHÂN

- **Phiên bản:** v0.1
- **Ngày:** 16/11/2026
- **Người viết:** 2A202603083

## 1. Mục tiêu

Xây dựng bộ dữ liệu ảnh đường phố phục vụ huấn luyện mô hình phát hiện người và phương tiện bằng Bounding Box.

Mục tiêu là gán nhãn chính xác, nhất quán, không bỏ sót đối tượng đủ điều kiện và đảm bảo các annotator hiểu giống nhau.

## 2. Lớp và thuộc tính

### 2.1. Các lớp nhãn

| Lớp | Định nghĩa | Ví dụ | Không nhầm với |
|---|---|---|---|
| `car` | Ô tô con dùng để chở người. | Sedan, SUV | Xe tải, xe van |
| `truck` | Phương tiện vận tải hàng hóa có kích thước lớn. | Xe tải thùng | Ô tô con, xe van |
| `van` | Xe chở người hoặc hàng hóa dạng thùng kín cỡ nhỏ. | Xe van | Ô tô con, xe tải |
| `motorcycle` | Phương tiện hai bánh có động cơ. | Xe máy, mô tô | Xe đạp |
| `person` | Con người xuất hiện trong ảnh. | Người đi bộ, người lái xe | Tượng, hình in |
| `ignore_region` | Vùng không thể xác định hoặc phân tách đối tượng để gán nhãn riêng một cách đáng tin cậy. | Đám đông quá dày, vùng hình ảnh không rõ | Đối tượng bình thường có thể nhận diện |

**Nguyên tắc:** Mỗi đối tượng chỉ thuộc một lớp. Đối tượng không thuộc các lớp trên không được tự ý tạo lớp mới.

### 2.2. Thuộc tính

| Thuộc tính | Giá trị | Quy tắc |
|---|---|---|
| `occluded_level` | `0` | Không bị che khuất. |
| | `1` | Bị che khuất không quá 25%. |
| | `2` | Bị che khuất trên 25% đến 50%. |
| | `3` | Bị che khuất trên 50% nhưng vẫn nhận diện được. |
| `truncated` | `no` | Đối tượng nằm hoàn toàn trong ảnh. |
| | `yes` | Đối tượng bị cắt bởi cạnh ảnh. |
| `can_review` | Tick / Không tick | Tick khi đối tượng hoặc quyết định gán nhãn chưa chắc chắn, cần reviewer kiểm tra. |

*Tỷ lệ che khuất được ước lượng trên toàn bộ hình dạng dự kiến của đối tượng.*

## 3. Quy tắc vẽ Bounding Box

**R-01 – Phạm vi gán nhãn:** Gán nhãn tất cả người và phương tiện thuộc các lớp quy định, nếu có thể nhận diện được.

**R-02 – Kích thước Box:** Box phải ôm sát phần đối tượng nhìn thấy, không bao gồm nền hoặc vật thể bên cạnh.

**R-03 – Che khuất:** Chỉ vẽ phần nhìn thấy của đối tượng. Không tự suy đoán phần bị che.

**R-04 – Cắt mép ảnh:** Vẽ box đến sát mép ảnh và đặt `truncated=yes`.

**R-05 – Người và xe máy:** Người điều khiển xe máy và xe máy là hai đối tượng riêng biệt, phải vẽ hai box nếu cả hai đều nhìn thấy.

**R-06 – Đối tượng chồng lấn:** Mỗi đối tượng có box riêng, không gộp nhiều đối tượng thành một box.

**R-07 – Vật thể không phải đối tượng thật:** Không gán nhãn cho tượng, hình ảnh trên biển quảng cáo, ảnh in, đồ chơi hoặc hình phản chiếu.

**R-08 – Phương tiện chở phương tiện:** Gán nhãn riêng cho từng phương tiện nếu có thể nhận diện và phân biệt rõ ràng.

**R-09 – Đối tượng quá nhỏ hoặc mờ:** Chỉ gán nhãn khi có đủ thông tin để xác định lớp và vị trí. Nếu không thể xác định đáng tin cậy, ghi nhận vấn đề để reviewer kiểm tra.

**R-10 – Vùng không thể phân tách:** Sử dụng `ignore_region` khi có vùng chứa đối tượng nhưng không thể gán nhãn từng đối tượng một cách đáng tin cậy.

## 4. Các trường hợp khó (Case Study)

| Tình huống | Rule ID | Quyết định gán nhãn | Lý do |
|---|---|---|---|
| Người ngồi trên xe máy | R-05 | Tạo hai Bounding Box riêng cho `person` và `motorcycle`. | Người và xe máy là hai đối tượng độc lập. |
| Đối tượng bị che khuất một phần | R-03 | Vẽ box theo phần nhìn thấy và xác định `occluded_level` phù hợp. | Không suy đoán phần bị che khuất. |
| Phương tiện bị cắt bởi cạnh ảnh | R-04 | Vẽ box đến sát mép ảnh và đặt `truncated=yes`. | Đối tượng không xuất hiện đầy đủ trong ảnh. |
| Nhiều phương tiện chồng lấn | R-06 | Vẽ Bounding Box riêng cho từng phương tiện có thể nhận diện. | Tránh gộp nhiều đối tượng thành một box. |
| Người xuất hiện trên biển quảng cáo hoặc tượng | R-07 | Không gán nhãn `person`. | Đây không phải người thật. |
| Đám đông hoặc đối tượng quá mờ | R-09, R-10 | Không tự suy đoán; đánh dấu `can_review` hoặc sử dụng `ignore_region` khi phù hợp. | Đảm bảo tính chính xác và tránh gán nhãn sai. |

*Lưu ý: Thay `img_0xx` bằng ID ảnh thực tế trong bộ 40 ảnh sau khi khảo sát; điều chỉnh quyết định nếu ảnh thực tế khác mô tả.*

## 5. Kiểm soát chất lượng (QC) và phiên bản

- Kiểm tra mỗi box về vị trí, kích thước, lớp và thuộc tính.
- Đảm bảo không bỏ sót đối tượng đủ điều kiện hoặc gán nhãn trùng lặp.
- Kiểm tra các trường hợp `occluded_level`, `truncated` và `can_review`.
- Đối chiếu nhãn xuất ra với ảnh gốc và đảm bảo dữ liệu COCO 1.0 hợp lệ.
- Những trường hợp chưa thống nhất phải được ghi nhận và chuyển reviewer quyết định.
- Mọi thay đổi ảnh hưởng đến ý nghĩa nhãn phải cập nhật phiên bản guideline và taxonomy tương ứng.

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v0.1 | Hoàn thiện quy tắc gán nhãn, thuộc tính, Bounding Box và xử lý ca khó | Bổ sung các nội dung còn thiếu của guideline v0 |
