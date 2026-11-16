# Guideline gán nhãn — v1.0 — Bộ ảnh Day 07

- Phiên bản: v1.0 — bản đề xuất để áp dụng và hiệu chỉnh trước khi chốt nhãn.
- Ngày: 16/11/2026.
- Người viết / duyệt: Vo Khac Xuan Nguyen
- Phạm vi: 40 ảnh `img_001.jpg`–`img_040.jpg` trong `data/images/`.
- Loại nhãn: bounding box chữ nhật song song với cạnh ảnh; công cụ dự kiến CVAT, xuất COCO 1.0 theo quy trình của bài

## 1. Mục tiêu

Tạo nhãn để huấn luyện và đánh giá mô hình phát hiện người và một số loại phương tiện trong ảnh đường phố. Mô hình cần xác định vị trí và lớp của từng đối tượng, kể cả khi đối tượng đứng yên, bị che một phần, bị cắt bởi mép ảnh hoặc xuất hiện trong điều kiện thiếu sáng.

Phạm vi gồm năm lớp đối tượng: `car`, `truck`, `van`, `motorcycle`, `person`; `ignore_region` là vùng loại khỏi đánh giá, không phải một loại phương tiện.




## 2. Lớp và thuộc tính

### Lớp

Mỗi đối tượng thật thuộc đúng một lớp. Khi phân biệt phương tiện, ưu tiên kết cấu thân xe nhìn thấy; không suy đoán từ màu sắc, hãng xe hoặc việc xe đang đỗ/chạy.

| Lớp | Định nghĩa (một câu) | Ví dụ trong bộ ảnh | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con chở người có thân xe dạng sedan, hatchback, wagon, SUV hoặc MPV không có kết cấu van. | Ô tô đen ở `img_023`; xe con được chở trên xe vận chuyển tại `img_002`. | Bán tải có thùng hàng riêng là `truck`; xe thân van như `img_005` là `van`. |
| `truck` | Xe tải hoặc bán tải có cabin và sàn/thùng chở hàng riêng, kể cả thùng kín hoặc thùng được lắp nắp chụp. | Xe thùng hở `img_001`, bán tải `img_012`, xe tải xanh `img_036`, bán tải có nắp cao `img_037`. | Không đổi xe tải thùng kín thành `van`; tượng bán tải `img_027` không phải xe thật. |
| `van` | Xe có thân dạng hộp, khoang phía sau liền với thân cabin, dùng chở người hoặc hàng, không có sàn/thùng tải tách biệt và không mang kết cấu xe buýt. | Xe màu bạc thân van phía sau nhóm người ở `img_005`. | Bán tải lắp nắp thùng `img_037` vẫn là `truck`; xe buýt nhỏ `img_009` không tự động thành `van`. |
| `motorcycle` | Xe máy, mô tô hoặc xe tay ga có động cơ, gồm xe mô tô gắn thùng bên hông liền bộ. | Xe máy `img_008`, `img_026`; mô tô có sidecar `img_022`. | Xe đạp, xích lô và tuk-tuk không thuộc lớp này trong phiên bản hiện tại. |
| `person` | Người thật nhìn thấy trực tiếp, gồm người đi bộ, người lái/hành khách trên xe và người hóa trang có căn cứ xác nhận. | Người qua đường `img_014`, `img_034`; người ngồi xe ở `img_005`. | Tượng `img_017`, hình người in `img_018`, ma-nơ-canh `img_033`, đồ chơi `img_038`. |
| `ignore_region` | Vùng ảnh được xác định để bỏ qua đối tượng mục tiêu quá nhỏ hoặc không thể tách/nhận diện đáng tin cậy theo R06–R07. | Cụm xe/người rất nhỏ ở hậu cảnh `img_021`, nếu đo và kiểm tra thấy đủ điều kiện. | Không dùng cho mọi vùng đông người, vật ngoài taxonomy, tượng, hình in hoặc bóng phản chiếu. |


### Thuộc tính

Ba thuộc tính dưới đây áp dụng cho năm lớp đối tượng. Với `ignore_region`, để `occluded_level=0`, `truncated=no`; các giá trị này chỉ để tương thích cấu trúc dữ liệu, không mô tả một vật. Vùng ignore còn chờ duyệt phải có `can_review=true`.

| Thuộc tính | Các mức cố định | Định nghĩa và ngưỡng |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | `0`: không bị vật khác che; `1`: bị che trên 0% đến 30%; `2`: bị che trên 30% đến 70%; `3`: bị che trên 70% nhưng vẫn nhìn thấy phần đủ để nhận diện đối tượng. |
| `truncated` | `no`, `yes` | `yes`: đường biên đối tượng bị mép ảnh cắt, có phần đối tượng nằm ngoài khung; `no`: toàn bộ đối tượng nằm trong ảnh, kể cả khi bị vật khác che. Chỉ chạm mép mà chưa mất phần nào vẫn là `no`. |
| `can_review` | `false`, `true` | `true`: chưa chắc lớp, ranh giới, mức che, thật/giả hoặc điều kiện ignore; `false`: quyết định đã rõ hoặc đã được người duyệt chốt. |

## 3. Quy tắc vẽ box

### Box ôm sát phần nhìn thấy

Vẽ một hình chữ nhật nhỏ nhất bao quanh tất cả các phần nhìn thấy thuộc cùng một đối tượng. Không vẽ box theo kích thước toàn thân/toàn xe tưởng tượng. Nếu các phần nhìn thấy bị ngắt bởi vật che, một box bao cả các phần đó vẫn hợp lệ; phần nền hoặc vật che bên trong hình chữ nhật là hệ quả hình học.

- `person`: bao đầu, tóc, mũ đội, quần áo, bàn tay và chân nhìn thấy; không chủ động mở box để bao túi xách, ô, gánh hàng hoặc vật dài đang cầm.
- Phương tiện: bao thân xe, bánh, gương và bộ phận gắn cố định nhìn thấy; không mở box để bao người ngồi trên xe, bóng đổ, hàng rời hoặc vật được chở.
- Một bộ phận thuộc vật nằm trong box do vị trí hình học không có nghĩa là được gán chung lớp. Không thể cắt bỏ nền hoặc người nằm giữa các phần xe bằng một box chữ nhật.
- Box không xoay theo vật. Không thêm lề cho đẹp, không cắt bỏ phần nhỏ rõ ràng như gương hoặc bàn tay để tăng IoU.
- Đo trên ảnh gốc, không lấy số pixel của màn hình sau khi phóng to/thu nhỏ.

### Bị che và bị cắt mép

- Bị che: bao các phần nhìn thấy xác định thuộc cùng vật, gán `occluded_level` theo R03; không vẽ thêm chân/bánh bị khuất.
- Bị cắt mép: dừng box ở biên ảnh và đặt `truncated=yes`; không kéo box ra ngoài ảnh.
- Có thể đồng thời bị che và bị cắt mép; hai thuộc tính phải xét độc lập.
- Nếu chỉ thấy một phần nhưng vẫn xác định được lớp và chiều cao box đạt ngưỡng, vẫn gán. Nếu không xác định được vật hoặc không tách được cá thể, xử lý R07/R13.

###  Vật quá nhỏ

Gọi `h = ymax − ymin` là chiều cao box của phần nhìn thấy trên ảnh gốc, trước khi làm tròn số hiển thị.

- `h ≥ 20 px`: gán box nếu nhận diện được lớp và cá thể. Đúng 20 px vẫn gán.
- `0 < h < 20 px`: không tạo box đối tượng; đánh dấu vùng `ignore_region` ôm phần/cụm mục tiêu quá nhỏ.
- Nếu một cụm gồm nhiều vật đều nhỏ hơn ngưỡng và không thể tách sạch, dùng vùng ignore cho cụm; không gộp các vật ở xa nhau thành một vùng lớn xuyên qua đối tượng rõ.
- Vật ngoài taxonomy vẫn theo R02, không chuyển thành ignore chỉ vì nhỏ.
- Nếu chưa đo được vì ranh giới quá mờ, ghi vấn đề thay vì tự kết luận nhỏ hơn 20 px.

### Đám đông, bãi xe và vùng ignore

Gán riêng từng người/xe khi xác định được cá thể và ranh giới phần nhìn thấy. Đông người hoặc nhiều xe không tự động là lý do dùng ignore.

Chỉ dùng `ignore_region` khi thuộc một trong các trường hợp: mục tiêu dưới ngưỡng R06; cụm mục tiêu chồng lấp không thể tách cá thể đáng tin cậy; vùng có dấu hiệu người/xe mục tiêu nhưng nhòe, tối hoặc bị che đến mức không thể xác định box/lớp đáng tin cậy.

Mỗi vùng ignore phải có ghi chú cấp ảnh gồm vị trí, lý do và Rule ID. Trước khi chốt, người duyệt kiểm tra vùng này không bao phủ đối tượng rõ cần đánh giá. Ưu tiên chia thành các vùng nhỏ; nếu box chữ nhật không thể tách phần khó khỏi đối tượng rõ, giữ ca ở trạng thái review và chưa nghiệm thu ảnh. Không dùng một vùng lớn che toàn bộ bãi xe hoặc đám đông.

### Người và phương tiện là các đối tượng riêng

- Một xe máy có hai người: một box `motorcycle` và hai box `person`, nếu mỗi vật đều đủ điều kiện; không vẽ một box bao cả nhóm.
- Người điều khiển xe đạp/xích lô/tuk-tuk: gán `person`, bỏ nhãn phương tiện theo R02.
- Người nhìn rõ qua kính xe: gán riêng `person` theo ngưỡng kích thước; không suy đoán có người chỉ vì xe đang chạy.
- Không gộp hai người sát nhau. Không tách đầu và thân của cùng một người thành hai box.

### Vật nhiều phần, xe chở xe và hàng hóa

- Cabin và thùng tải của cùng xe là một box `truck`. Với xe vận chuyển ô tô tại `img_002`, gộp đầu kéo và giàn chở xe nối liền thành một đối tượng vận chuyển trong phạm vi bài này.
- Mỗi xe con thật được chở trên giàn có box `car` riêng, nếu đủ điều kiện. Box xe vận chuyển có thể chồng lấp box xe con.
- Mô tô và sidecar gắn liền là một box `motorcycle`; người ngồi trong đó, nếu nhìn thấy, là box `person` riêng.
- Hàng rời, bao tải, gánh hàng, thanh kim loại dài hoặc tượng trên thùng xe không làm thay đổi lớp và không được dùng để kéo rộng box phương tiện/người.
- Bán tải có nắp thùng cao vẫn là `truck` khi nhận ra kết cấu thùng riêng; không chuyển sang `van` chỉ vì thùng được đóng kín.

### Phản chiếu, hình in, tượng, đồ chơi và hóa trang

- Không gán đối tượng chỉ xuất hiện qua gương hoặc mặt phản chiếu; không tạo box cho bóng đổ hay vệt phản sáng trên đường.
- Không gán người/xe in trên bảng quảng cáo, tranh, màn hình, bao bì.
- Không gán tượng, ma-nơ-canh hoặc mô hình/đồ chơi thành `person` hay phương tiện thật, kể cả hình dáng rất giống.
- Không đặt ignore lên các vật giả đã xác định rõ; chúng là ví dụ nền âm tính cần giữ.
- Người hóa trang vẫn là `person` nếu có căn cứ là người thật; không mặc định mọi nhân vật hình người đều là tượng.
- Xe thật được trưng bày hoặc trang trí vẫn được gán. Với vật khó phân biệt xe thật và mô hình, dùng R13; không kết luận chỉ vì nó nằm trong bảo tàng hoặc có hình dáng lạ.

### Xe chủ (ego vehicle)

Không gán phần thân, nắp ca-pô, tay lái hoặc gương thuộc chính xe mang camera, khi xác định được từ góc chụp. Ở `img_020`, cụm gương/cửa sát camera và hình phản chiếu trong gương không tạo box mục tiêu. Nếu không chắc vật có thuộc xe chủ hay là xe khác, ghi review; không áp dụng quy tắc ego cho mọi xe bị cắt mép.

### Nhòe chuyển động, thiếu sáng và ngược sáng

Vẫn gán khi xác định được lớp và ranh giới phần nhìn thấy. Với vật nhòe, đặt cạnh box ở biên ngoài còn nhận ra cấu trúc vật; không bao vệt sáng/vệt mờ kéo dài không xác định được thuộc vật. Nếu không tìm được ranh giới đáng tin cậy, chuyển R07/R13. Không tăng `occluded_level` chỉ vì ảnh tối hoặc nhòe.

### Ca chưa chốt

Nếu đã có box tạm hợp lý, đặt `can_review=true` và ghi `tên ảnh — vị trí/object_id — Rule ID — điều chưa rõ — phương án đề xuất`. Nếu chưa thể đặt box hoặc chọn lớp, ghi vấn đề cấp ảnh; không tạo lớp giả hoặc box tùy ý để lấp chỗ trống. Người duyệt chốt quyết định và lý do trước khi bỏ cờ review. Một ca chưa chốt không được xem là nhãn chuẩn để đánh giá người khác.

## 4. Ca khó

Các dòng dưới đây là quyết định hướng dẫn dựa trên ảnh đã xem, không phải danh sách box đã gán hay kết quả đo thuộc tính. Mọi ngưỡng pixel và mức che phải được kiểm tra trên đối tượng cụ thể khi gán nhãn.

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
|
| `img_015` | Hai ô tô tiền cảnh bị cắt ở mép trái và phải. | R05, R10 | Gán từng `car` phần nhìn thấy, `truncated=yes`; bỏ phản chiếu dưới đường. | Cắt mép không đồng nghĩa bỏ nhãn. |
| `img_016`, `img_035`, `img_040` | Tuk-tuk/xe ba bánh xen với người, ô tô và xe máy. | R02, R08 | Tuk-tuk ngoài phạm vi; giữ người, ô tô và xe máy thuộc taxonomy. | Không ép mọi xe có động cơ vào `motorcycle`. |
| `img_017` | Tượng người cưỡi ngựa. | R10 | Không gán tượng thành `person`. | Đây là vật thể mô phỏng, không phải người thật. |
| `img_018` | Hình người trên biển quảng cáo, xe thật sau hàng rào. | R03, R05, R10 | Bỏ hình người in; gán các xe thật nhận diện được, xét phần hàng rào che xe. | Hình in và vật thật phía sau phải xử lý khác nhau. |
| `img_019` | Xe diễu hành vàng, người thật, nhân vật hoạt hình và đám đông. | R07, R08, R10, R13 | Xe thật gán `car`, người thật gán riêng. Nhân vật đầu chuột không tự động gán `person`; ghi review nếu chưa phân biệt người mặc trang phục với mô hình. | Xe trang trí vẫn là xe; nhân vật hình người cần căn cứ riêng. |
| `img_020` | Gương chiếu hậu và hình phản chiếu. | R10, R11 | Không gán các hình trong gương hoặc phần xe chủ; kiểm tra phần cảnh nhìn trực tiếp nếu có. | Không coi ảnh phản chiếu là đối tượng thật thứ hai. |
| `img_021` | Dòng xe máy rất đông, nhỏ dần về phía xa. | R06, R07, R08 | Tách người/xe tiền cảnh; chỉ ignore các cụm đạt điều kiện nhỏ hoặc không tách được. | Mật độ cao không đủ để ignore toàn ảnh. |
| `img_022` | Mô tô có thùng bên hông. | R09 | Một box `motorcycle` bao cả mô tô và sidecar. | Hai phần thuộc một tổ hợp xe gắn liền. |
| `img_023` | Người đạp xích lô gần ô tô. | R02, R08 | Người là `person`, ô tô là `car`; xích lô không có nhãn phương tiện. | Taxonomy không có xích lô. |
| `img_024` | Xe trưng bày và vật hình xe nhỏ màu vàng trên nóc. | R10, R13 | Gán các ô tô thật. Với vật màu vàng, ghi review để xác nhận xe thật/mô hình; nếu là mô hình thì bỏ, nếu là xe thật thì gán riêng. | Vị trí trên nóc và tỷ lệ nhỏ chưa đủ để kết luận chắc chắn. |
| `img_025`, `img_039` | Người và xe nhỏ ở hậu cảnh. | R06, R07 | Đo chiều cao từng ứng viên trên ảnh gốc; từ 20 px thì gán nếu nhận diện được, dưới ngưỡng thì ignore theo cụm phù hợp. | Không dùng cảm giác “ở xa” thay cho ngưỡng pixel. |
| `img_026` | Người đi xe máy trước tranh tường. | R08, R10, R12 | Gán người và xe thật; bỏ hình vẽ phía sau. | Tách đối tượng thật khỏi hình minh họa. |
| `img_027` | Công trình tưởng niệm hình bán tải trên bệ. | R10 | Không gán `truck` cho vật hình xe này. | Hình ảnh và tên gốc trong bảng ghi công xác định đây là tượng/công trình tưởng niệm. |
| `img_028` | Bán tải thật chở tượng ngựa, ô tô đỏ ở phía sau. | R01, R09, R10 | Gán `truck` cho bán tải và `car` cho xe đỏ; không mở box tải lên theo tượng ngựa. | Tượng/hàng chở không thuộc thân xe. |
| `img_029` | Người và xe cạnh các thanh kim loại dài. | R02, R04, R09 | Gán người và xe máy; bỏ xe đạp khỏi nhãn phương tiện, không kéo box theo thanh kim loại. | Hàng rời không quyết định kích thước người/xe. |

## 5. QC và phiên bản

### Ngưỡng QC áp dụng

| Hạng mục | Cách kiểm tra | Điều kiện đạt |
|---|---|---|
| Kích thước nhỏ nhất | Chiều cao box phần nhìn thấy trên ảnh gốc. | `h ≥ 20 px` để gán đối tượng; nhỏ hơn xử lý R06. |
| Vị trí/kích thước box lớn | IoU = diện tích giao / diện tích hợp với box tham chiếu đã duyệt; xác định nhóm kích thước theo chiều cao box tham chiếu. | `h ≥ 50 px`: IoU ≥ 0,85. |
| Vị trí/kích thước box nhỏ | Như trên; đúng 50 px thuộc nhóm lớn. | `20 ≤ h < 50 px`: IoU ≥ 0,70. |
| Ghép box đo IAA | Ghép một–một theo IoU, không yêu cầu cùng lớp; một box không ghép với nhiều box. | IoU ghép ≥ 0,50. |
| IAA vị trí | IoU trung bình trên các cặp đã ghép. | ≥ 0,85. |
| IAA lớp | Số cặp ghép có cùng lớp / tổng cặp ghép. | ≥ 95%. |
| Độ phủ phân công | Số lượt gán hợp lệ / tổng lượt được phân công. | 100% trước khi đo IAA. |
| Lỗi theo ảnh | Số ảnh có ít nhất một lỗi / số ảnh đã duyệt. | ≤ 2%; phải duyệt toàn bộ lô để Release. |
| Lỗi theo đối tượng | Số annotation đối tượng có lỗi / số annotation đối tượng đã duyệt. | Báo kèm tử số và mẫu số; không thay thế chỉ số lỗi theo ảnh. |

Với lô 40 ảnh, 1 ảnh lỗi tương đương 2,5%, đã vượt ngưỡng. Vì vậy, phiên bản bàn giao cuối phải còn 0/40 ảnh lỗi đã xác nhận. Không suy luận rằng lô đạt chỉ từ một mẫu nhỏ sạch lỗi.

IAA chỉ phản ánh các cặp đã ghép; box không ghép được phải thống kê riêng để phát hiện bỏ sót/thừa. Khi không có cặp ghép, ghi `N/A`, không ghi 100%. Với ba người gán, mỗi ảnh có ba cặp so sánh người. Nếu chỉ một người đặt ignore phủ một đối tượng, đưa ca đó vào buổi đối chiếu để thống nhất quy tắc, không tự coi là đã đồng thuận.

### Trình tự kiểm tra

1. **Tự kiểm từng ảnh:** quét trái sang phải, trên xuống dưới; kiểm tra lớp, box, thuộc tính, đối tượng nhỏ, người trên xe và các ca âm tính. Ảnh không có đối tượng hợp lệ vẫn phải được xác nhận đã xem.
2. **Kiểm tra cấu trúc:** tên ảnh đúng; lớp và giá trị thuộc tính đúng tập cho phép; box có chiều rộng/chiều cao dương, nằm trong ảnh; không trùng box cho cùng vật; dùng cùng một phiên bản guideline.
3. **So sánh độc lập:** đo IAA khi đủ lượt phân công; lưu kết quả trước khi thảo luận/chỉnh sửa. Không thay kết quả độc lập bằng kết quả sau khi hai người đã cùng sửa.
4. **Duyệt ca khó:** giải quyết toàn bộ `can_review=true`, vấn đề cấp ảnh và vùng ignore. Người duyệt được nhóm/giảng viên phân công là người quyết định cuối; chưa có tên người duyệt được xác nhận trong bản này.
5. **Sửa và kiểm lại:** ghi lỗi, Rule ID, người sửa, quyết định và kết quả kiểm lại. Nếu thay quy tắc, rà soát tất cả ảnh cùng loại ca, không chỉ ảnh vừa phát hiện.
6. **Chốt lô:** chỉ Release sau khi duyệt đủ 40 ảnh, đủ bằng chứng QC và không còn ca chưa chốt. Thiếu bằng chứng hoặc chỉ duyệt mẫu thì Hold. Vượt ngưỡng lỗi hoặc phát hiện lỗi hệ thống lặp lại thì Rework; giữ lịch sử lỗi trước sửa.


### 5. QC va phien ban

Phien Baản: V1.0

