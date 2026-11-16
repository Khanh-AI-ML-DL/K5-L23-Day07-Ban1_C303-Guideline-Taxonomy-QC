# GUIDELINE GÁN NHÃN ẢNH ĐƯỜNG PHỐ

**Phiên bản:** Bản nháp v1.0\
**Trạng thái:** Chờ nhóm review và thống nhất\
**Công cụ:** CVAT\
**Kiểu gán nhãn:** Bounding Box\
**Định dạng xuất:** COCO 1.0

> Đây là bản tổng hợp đề xuất từ 4 guideline cá nhân. Các mục còn cần
> nhóm xác nhận được đánh dấu **\[CẦN THỐNG NHẤT\]**. File chỉ dùng để
> review, chưa được đẩy lên GitHub.

## 1. Mục đích và nguyên tắc chung

Tài liệu thống nhất cách gán nhãn người và phương tiện trong bộ ảnh của
nhóm, nhằm giảm bỏ sót, gán sai lớp và vẽ box không nhất quán.

Nguyên tắc: - Chỉ gán nhãn khi có đủ bằng chứng trực quan. - Không suy
đoán phần đối tượng bị che khuất. - Mỗi đối tượng độc lập có một
bounding box riêng nếu có thể tách. - Nếu không chắc chắn, đánh dấu để
review thay vì tự đoán.

## 2. Danh sách lớp (taxonomy)

Chỉ sử dụng đúng 6 lớp trong `taxonomy.json`, không tự ý thêm lớp mới.

  -----------------------------------------------------------------------
  Lớp                                 Định nghĩa
  ----------------------------------- -----------------------------------
  `car`                               Ô tô chở người như sedan,
                                      hatchback, SUV, MPV.

  `truck`                             Xe tải hoặc phương tiện có
                                      thùng/khoang chở hàng riêng, nhận
                                      diện được là xe tải.

  `van`                               Xe thùng kín cỡ nhỏ, khoang hàng
                                      liền với cabin.

  `motorcycle`                        Xe máy, mô tô, xe tay ga.

  `person`                            Người thật xuất hiện trong ảnh, dù
                                      đang đi bộ, đứng hoặc ngồi trên
                                      phương tiện.

  `ignore_region`                     Vùng có đối tượng thuộc phạm vi
                                      quan tâm nhưng không thể gán nhãn
                                      đáng tin cậy. Đây là vùng bỏ qua,
                                      không phải một loại vật thể.
  -----------------------------------------------------------------------

### 2.1. Đối tượng không có lớp riêng

Xe buýt, xe đạp, xe ba bánh/tuk-tuk, xe kéo và phương tiện không thuộc 5
lớp vật thể trên không được ép vào lớp gần giống. Không tự tạo lớp mới.
Nếu trường hợp khó quyết định hoặc ảnh hưởng đến đánh giá, ghi chú để
nhóm review.

### 2.2. Vật thể không phải đối tượng thật

Không gán `person` hoặc lớp phương tiện cho tượng, ma-nơ-canh, hình in
trên biển quảng cáo/áp phích/màn hình, hình phản chiếu hoặc mô hình đồ
chơi. Không dùng `ignore_region` cho vật thể rõ ràng không phải đối
tượng thật thuộc phạm vi gán nhãn.

## 3. Thuộc tính (attributes)

Sử dụng đúng các thuộc tính được khai báo trong taxonomy:

  -----------------------------------------------------------------------
  Thuộc tính              Giá trị                 Ý nghĩa
  ----------------------- ----------------------- -----------------------
  `occluded_level`        `0`                     Không bị che khuất hoặc
                                                  gần như nhìn thấy đầy
                                                  đủ.

  `occluded_level`        `1`                     Bị che khuất nhẹ.

  `occluded_level`        `2`                     Bị che khuất ở mức
                                                  trung bình.

  `occluded_level`        `3`                     Bị che khuất nhiều
                                                  nhưng vẫn đủ dấu hiệu
                                                  nhận diện.

  `truncated`             `no`                    Không bị mép ảnh cắt
                                                  mất.

  `truncated`             `yes`                   Một phần đối tượng nằm
                                                  ngoài khung ảnh do mép
                                                  ảnh cắt.

  `can_review`            `true`                  Cần người khác kiểm tra
                                                  lại.

  `can_review`            `false`                 Người gán nhãn tự tin
                                                  với kết quả.
  -----------------------------------------------------------------------

**\[CẦN THỐNG NHẤT\] Ngưỡng phần trăm che khuất:** Các guideline cá nhân
chưa thống nhất hoàn toàn về ranh giới phần trăm giữa các mức. Bản nháp
tạm mô tả theo mức độ định tính, chưa đặt ngưỡng phần trăm cứng. Nhóm
cần chốt một bộ ngưỡng duy nhất trước khi gán nhãn chính thức.

## 4. Quy tắc vẽ bounding box

### 4.1. Nguyên tắc chung

1.  Vẽ hình chữ nhật bao sát phần đối tượng nhìn thấy.
2.  Không lấy quá nhiều nền xung quanh.
3.  Không ước lượng hoặc vẽ thêm phần bị che khuất.
4.  Mỗi đối tượng độc lập có một box riêng.
5.  Chọn đúng lớp và điền thuộc tính tương ứng.
6.  Nếu không đủ căn cứ xác định lớp hoặc vị trí box, đánh dấu
    `can_review` và ghi chú.

### 4.2. Người ngồi trên xe máy

-   Gán một box `motorcycle` cho xe máy.
-   Gán một box `person` cho từng người nhìn thấy trên xe.
-   Không gộp người và xe máy vào cùng một box.
-   Nếu có người lái và người ngồi sau, mỗi người là một box `person`
    riêng khi có thể phân biệt.

### 4.3. Đối tượng bị che khuất (occlusion)

-   Nếu vẫn nhận diện được đối tượng, vẽ box theo phần nhìn thấy.
-   Không vẽ phần bị vật khác che khuất.
-   Chọn `occluded_level` theo mức độ che khuất.
-   Nếu còn quá ít dấu hiệu để xác định đáng tin cậy, không đoán lớp;
    đánh dấu review hoặc dùng `ignore_region` nếu phù hợp quy tắc nhóm.

### 4.4. Đối tượng bị cắt bởi mép ảnh (truncation)

-   Nếu đối tượng vẫn nhận diện được, vẽ box đến sát mép ảnh.
-   Không mở rộng box ra ngoài khung hình.
-   Đặt `truncated = yes` nếu bị mép ảnh cắt; ngược lại đặt `no`.

### 4.5. Đối tượng nhỏ

Theo cấu hình taxonomy, chiều cao box tối thiểu là **20 px**. - Đo chiều
cao trên ảnh gốc. - Nếu đối tượng nhận diện được và box cao từ 20 px trở
lên, gán nhãn bình thường. - Nếu chiều cao nhỏ hơn 20 px, không tạo box
vật thể thông thường. - Chỉ dùng `ignore_region` khi vùng đó thực sự cần
được loại khỏi đánh giá theo quy định của nhóm; không tạo ignore region
tràn lan.

### 4.6. Đám đông và đối tượng chồng lấn

-   Tách từng người/phương tiện thành box riêng nếu có thể xác định ranh
    giới.
-   Không gộp nhiều đối tượng rõ ràng vào một box.
-   Nếu vùng chồng lấn khiến không thể tách hoặc nhận diện đáng tin cậy,
    đánh dấu review hoặc dùng `ignore_region` cho đúng vùng không thể
    gán nhãn.
-   Không để `ignore_region` che phủ đối tượng khác vẫn có thể gán nhãn
    rõ ràng.

### 4.7. Xe chở phương tiện khác

**\[CẦN THỐNG NHẤT\] Đề xuất:** Gán box `truck` cho xe vận chuyển. Nếu
xe được chở là `car` và nhìn thấy đủ rõ, gán thêm box `car` riêng cho
từng xe. Không gán box cho phần xe bị che hoàn toàn hoặc không thể nhận
diện. Nếu nhóm chọn không gán riêng các xe được vận chuyển, cần ghi rõ
quy tắc thay thế và áp dụng thống nhất.

### 4.8. Xe bán tải (pickup)

**\[CẦN THỐNG NHẤT\] Đề xuất:** Nếu phương tiện có thùng chở hàng riêng
biệt và nhận diện rõ đặc trưng xe chở hàng, cân nhắc gán `truck`. Nếu
nhóm xác định pickup thuộc nhóm ô tô con trong phạm vi dataset, gán
`car`. Nhóm phải chốt một quy tắc duy nhất; không để mỗi thành viên tự
chọn khác nhau.

### 4.9. Người mang hoặc đứng cạnh đồ vật

-   Box `person` chỉ bao phần cơ thể người nhìn thấy.
-   Không mở rộng box người để bao túi xách, hàng hóa hoặc đồ vật đang
    mang.
-   Đồ vật không thuộc taxonomy thì không tạo lớp mới.

## 5. Khi nào sử dụng `ignore_region`?

Có thể cân nhắc dùng khi đối tượng thuộc phạm vi cần quan tâm nhưng quá
nhỏ, quá mờ hoặc chồng lấn đến mức không thể gán nhãn tin cậy.

Không dùng `ignore_region` khi: - Đối tượng rõ ràng không thuộc
taxonomy. - Đó là hình in, hình phản chiếu, tượng hoặc đồ chơi. - Đối
tượng vẫn có thể gán nhãn bình thường. - Chỉ vì người gán nhãn muốn bỏ
qua một trường hợp khó mà chưa kiểm tra hoặc ghi chú.

## 6. Quy trình gán nhãn trên CVAT

1.  Mở đúng task/job và ảnh được phân công.
2.  Kiểm tra ảnh ở kích thước phù hợp; phóng to khi cần.
3.  Xác định các đối tượng thuộc taxonomy.
4.  Vẽ bounding box theo phần nhìn thấy.
5.  Chọn đúng class.
6.  Điền `occluded_level`, `truncated` và `can_review`.
7.  Kiểm tra lại đối tượng nhỏ, bị che khuất, sát mép ảnh và chồng lấn.
8.  Lưu kết quả.
9.  Đánh dấu hoặc ghi chú các trường hợp chưa chắc chắn để người review
    xử lý.

## 7. Kiểm tra chất lượng (QC)

### 7.1. Checklist kiểm tra từng ảnh

-   Có bỏ sót đối tượng thuộc taxonomy không?
-   Có gán nhầm lớp không?
-   Box có bao sát phần nhìn thấy và không chứa quá nhiều nền không?
-   Có tách riêng người và xe máy không?
-   Thuộc tính occlusion/truncation có hợp lý không?
-   `ignore_region` có được dùng đúng và không che mất đối tượng rõ ràng
    không?
-   Các trường hợp `can_review` đã được xem xét chưa?

### 7.2. Ngưỡng QC theo taxonomy

  -----------------------------------------------------------------------
  Quy tắc                                                          Ngưỡng
  ------------------------------ ----------------------------------------
  Chiều cao box tối thiểu                                           20 px

  Box lớn (chiều cao từ 50 px)                                 IoU ≥ 0.85

  Box nhỏ (chiều cao dưới 50 px,                               IoU ≥ 0.70
  nhưng đạt kích thước tối       
  thiểu)                         

  Ngưỡng IoU để ghép cặp khi                                         0.50
  tính IAA                       

  Mean IAA IoU                                                     ≥ 0.85

  Class agreement                                            ≥ 0.95 (95%)

  Tỷ lệ ảnh có lỗi tối đa                                     ≤ 0.02 (2%)
  -----------------------------------------------------------------------

Lưu ý: box dưới 20 px không đạt ngưỡng kích thước tối thiểu để tạo nhãn
vật thể thông thường. Ngưỡng box lớn/nhỏ được phân chia tại 50 px theo
cấu hình taxonomy.

### 7.3. Mã loại lỗi

  -----------------------------------------------------------------------
  Mã lỗi                              Ý nghĩa
  ----------------------------------- -----------------------------------
  `bo_sot`                            Bỏ sót đối tượng cần gán nhãn.

  `thua`                              Gán nhãn thừa đối tượng không thuộc
                                      phạm vi hoặc không tồn tại.

  `sai_lop`                           Chọn sai class.

  `sai_vi_tri_kich_thuoc`             Box sai vị trí hoặc kích thước.

  `sai_thuoc_tinh`                    Gán sai attribute.

  `khong`                             Không phát hiện lỗi.
  -----------------------------------------------------------------------

Khi ghi nhận lỗi, lưu tối thiểu: mã ảnh, vị trí/đối tượng, mã lỗi, mô tả
ngắn và cách sửa.

### 7.4. Lưu ý về tỷ lệ lỗi ảnh

Nếu bộ dữ liệu có 40 ảnh, một ảnh lỗi tương ứng 2.5%, lớn hơn ngưỡng 2%.
Vì vậy nhóm cần kiểm soát chặt và thống nhất cách tính tỷ lệ theo hướng
dẫn QC của bài tập.

## 8. Các trường hợp cần nhóm chốt

  -----------------------------------------------------------------------
  Nội dung                Đề xuất trong bản nháp  Trạng thái
  ----------------------- ----------------------- -----------------------
  Ngưỡng phần trăm cho    Nhóm chốt ngưỡng số cụ  Chưa chốt
  `occluded_level` 1/2/3  thể                     

  Xe bán tải pickup       `truck` nếu có thùng    Chưa chốt
                          hàng riêng; cần xác     
                          nhận phạm vi            

  Ô tô được chở trên xe   Gán `car` riêng nếu đủ  Chưa chốt
  tải                     rõ                      

  Đối tượng ngoài         Không ép vào lớp khác,  Theo taxonomy
  taxonomy                không tự thêm class     

  Box nhỏ                 Chiều cao tối thiểu 20  Theo taxonomy
                          px                      

  Box bị che khuất        Chỉ vẽ phần nhìn thấy   Đề xuất thống nhất
  -----------------------------------------------------------------------

## 9. Quản lý phiên bản

-   `v0`: guideline mẫu/khởi tạo.
-   `v1.0`: bản tổng hợp từ 4 guideline cá nhân, chờ nhóm review.
-   `v1.1`: bản cập nhật sau khi nhóm thống nhất các trường hợp còn
    tranh luận.
-   Mọi thay đổi ảnh hưởng đến class, box, attribute hoặc QC phải được
    ghi lại.
-   Không tự ý thay đổi quy tắc giữa chừng khi các thành viên đang gán
    nhãn.

## 10. Checklist nhanh cho người gán nhãn

-   [ ] Đã rà soát toàn bộ ảnh để tránh bỏ sót.
-   [ ] Chỉ sử dụng đúng 6 class trong taxonomy.
-   [ ] Mỗi đối tượng có box riêng khi có thể tách.
-   [ ] Box bám sát phần nhìn thấy, không đoán phần bị che.
-   [ ] Người ngồi trên xe máy được gán riêng với xe.
-   [ ] Đối tượng bị cắt mép ảnh có `truncated = yes`.
-   [ ] Đã kiểm tra ngưỡng chiều cao box 20 px.
-   [ ] Đã điền thuộc tính phù hợp.
-   [ ] Trường hợp không chắc chắn đã đánh dấu `can_review`.
-   [ ] `ignore_region` chỉ dùng cho vùng thực sự không thể gán nhãn tin
    cậy.

------------------------------------------------------------------------

**Ghi chú review:** Đây là bản nháp để các thành viên góp ý. Sau khi
chốt các mục ở Mục 8, cần cập nhật nội dung và phát hành phiên bản chính
thức. Không sử dụng bản nháp này như quy định cuối cùng khi các điểm còn
tranh luận chưa được nhóm xác nhận.
