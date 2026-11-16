# Guideline gán nhãn — v1.1 (calibrated)

- Phiên bản: v1.1 (calibration sau IAA, dựa trên v1.0 đã thống nhất)
- Loại nhãn: bounding box (hộp chữ nhật) trên ảnh đường phố
- Công cụ: CVAT, xuất COCO 1.0
- Thay đổi so với v0: thêm lớp `bus`, `three_wheeler`, `bicycle`; định nghĩa lại `car/van/truck`; thêm ngưỡng kích thước, định nghĩa số cho `occluded_level`, quy tắc đám đông, quy tắc "vật không phải thật", và 30 ca khó có ảnh dẫn chứng.

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện **người và phương tiện giao thông thật** trên ảnh đường phố (camera giao thông, xe tự lái). Bối cảnh chính của bộ ảnh: đường phố Đông Nam Á (Việt Nam, Thái Lan), mật độ xe máy rất cao; ngoài ra có ảnh Âu–Mỹ, ảnh đen trắng, ảnh ban đêm, ảnh mưa.

**Nguyên tắc gốc** (khi gặp ca không có trong guideline, quay lại đây):
1. Chỉ gán vật **có thật, đang là người/phương tiện thật** (không gán tượng, mô hình, hình in, hình phản chiếu).
2. Chỉ gán khi **nhận ra được lớp và đủ lớn** để một người khác cũng nhận ra như vậy.
3. Mơ hồ thì chọn quy tắc ở mục 4, rồi tick `can_review`. **Không tự bịa quy tắc riêng.**

## 2. Lớp và thuộc tính

### 2.1 Lớp (mỗi vật thuộc đúng một lớp)

| Lớp | Định nghĩa | Ví dụ trong bộ ảnh |
|---|---|---|
| `car` | Ô tô con chở người, thân liền, **không có thùng hàng riêng**: sedan, hatchback, SUV, MPV, taxi 4 chỗ. | 002 (xe đỏ/trắng trên xe chở xe), 015, 023, 031, 040 |
| `van` | Xe thùng kín liền cabin cỡ nhỏ hoặc xe khách nhỏ dạng minivan (Toyota Hiace, Ford Transit…), ≤ ~16 chỗ. | 005 (xe 51LD-70.65), 016 |
| `truck` | Mọi xe **có thùng hàng riêng phía sau cabin**: xe tải, xe bán tải (pickup), xe tải nhỏ, xe thùng bạt, đầu kéo, xe chở xe. | 001, 002, 007, 012, 036, 037 |
| `bus` | Xe buýt, xe khách cỡ lớn/vừa (> ~16 chỗ, có cửa hành khách dọc thân). | 009 (xe County), 031, 037 (nền), 040 (nền) |
| `motorcycle` | Xe máy, xe tay ga, mô tô, kể cả có thùng xe bên hông (sidecar). | 005, 006, 008, 021, 022, 026 |
| `three_wheeler` | Xe 3 bánh có động cơ: tuk-tuk, xe lôi, auto-rickshaw, xe ba gác máy. | 004, 016, 035, 040 |
| `bicycle` | Xe đạp, xe đạp chở hàng, xích lô/cyclo đạp (xe đạp 3 bánh). | 003, 006, 010, 023, 029 |
| `person` | Người thật: đi bộ, đứng, ngồi, người lái hoặc ngồi trên xe hở. | 003, 014, 034 |
| `ignore_region` | Vùng không gán từng vật (đám đông, hàng xe đỗ dày, vật quá nhỏ). Xem mục 3.6. | 021, 030, 032 |

> **Vì sao thêm 3 lớp so với v0:** v0 chỉ có 6 lớp nhưng trong 40 ảnh có ít nhất 4 ảnh có tuk-tuk (004, 016, 035, 040), 5 ảnh có xe buýt/khách (009, 031, 037, 040, 016) và 5 ảnh có xe đạp/xích lô (003, 006, 023, 029, 010). Nếu không có lớp riêng, mỗi người gán một kiểu (tuk-tuk → `car` hay `motorcycle`?) → nhãn không nhất quán.
> **Nếu nhóm chỉ được dùng đúng 6 lớp của v0**, ánh xạ: `bus`→`truck`, `three_wheeler`→`motorcycle`, `bicycle`→(không gán, chỉ gán `person`) và ghi rõ trong mục 5.

### 2.2 Ranh giới giữa các lớp xe (hay nhầm)

Hỏi lần lượt, dừng ở câu đầu tiên trả lời "có":

1. Có đúng 3 bánh và có động cơ? → `three_wheeler`.
2. Có 2 bánh và có động cơ? → `motorcycle`. Có 2 bánh không động cơ (đạp)? → `bicycle`.
3. Xe chở nhiều hành khách, kích thước lớn, cửa hành khách dọc thân? → `bus`.
4. Phía sau cabin có **thùng hàng tách biệt** (thùng hở, thùng kín, thùng bạt, giàn chở xe)? → `truck`. *(pickup dân dụng như Ford Ranger/Hilux cũng là `truck` — 012, 007, 037.)*
5. Thân xe liền khối, thùng kín, dạng hộp chở người hoặc hàng nhẹ (Hiace)? → `van`.
6. Còn lại là xe con chở người? → `car`.

### 2.3 Thuộc tính

| Thuộc tính | Giá trị | Định nghĩa số (để mọi người chấm giống nhau) |
|---|---|---|
| `occluded_level` | `0` | Thấy ≥ 90% vật. |
| | `1` | Bị che 10–40%. Vẫn nhận ra lớp ngay. |
| | `2` | Bị che 40–80%. Phải nhìn kỹ mới nhận ra. |
| | `3` | Chỉ thấy < 20% hoặc chỉ đoán được từ ngữ cảnh. Chỉ gán nếu vẫn nhận ra chắc chắn (ví dụ thấy bánh + đèn + biển số). Nếu không chắc → không gán. |
| `truncated` | `no`, `yes` | `yes` khi vật bị **cắt bởi mép ảnh**. Vật bị che bởi vật khác thì **không** phải truncated, mà là `occluded_level`. |
| `can_review` | tick / không | Tick khi không chắc lớp, không chắc có gán không, hoặc ca khó thuộc mục 4. |

*Chú thích:* Với `ignore_region` không điền `occluded_level`/`truncated`.

## 3. Quy tắc vẽ box

### 3.1 Cái gì được gán

1. Gán box cho mọi **người và phương tiện thật** thỏa ngưỡng kích thước (3.2) trong ảnh, kể cả xe đang đỗ.
2. Chỉ gán nếu nhận ra được lớp. Mờ do chuyển động (014, 029): gán được nếu vẫn thấy rõ hình dạng; nếu chỉ còn vệt mờ → không gán, cân nhắc `ignore_region`.

### 3.2 Ngưỡng kích thước tối thiểu

- Cạnh dài của box (theo ảnh gốc) **< 20 px** → không gán từng vật.
- Cạnh dài **20–40 px** → chỉ gán nếu nhận ra lớp chắc chắn; nếu không, gộp vào `ignore_region`.
- Ví dụ: người đi bộ xa ở 013, 025, 039 và xe xa ở 031, 034: gán nếu ≥ 20 px, còn lại bỏ.

### 3.3 Cách vẽ box

1. **Vật bị vật khác che (xe che xe, người che người, cột/cây che): vẽ box theo kích thước ĐẦY ĐỦ ước lượng của vật (box "amodal"), gồm cả phần bị che**, miễn là ước lượng được. Mức che ghi bằng `occluded_level`, không ghi bằng cách thu nhỏ box.
   - Ước lượng được khi: thấy ≥ ~40% vật **và** đoán được phần còn lại từ hình dạng (đối xứng, bánh/đèn/mui còn thấy, cùng loại xe ở cạnh). Khi đó kéo box ra hết phần đoán được.
   - Không ước lượng chắc (che nhiều, `occluded_level` 2–3, thấy ít, hoặc không đoán được phần khuất kéo dài tới đâu): **chỉ box phần nhìn thấy** và tick `can_review`.
   - Không kéo box ra quá phần hợp lý: tối đa bằng kích thước điển hình của lớp đó trong ảnh (ví dụ xe máy phía sau bị che nửa thân → kéo bằng đúng một thân xe máy cùng cỡ ở cạnh).
   - Vật bị che bởi **vật mỏng** (hàng rào lưới, dây điện — xe sau hàng rào ở 018): luôn box ôm cả vật.
   - **Phần bị cắt bởi mép ảnh thì KHÔNG kéo ra ngoài ảnh**: box dừng ở mép ảnh, `truncated = yes` (xem điểm 3).
2. Bắt đầu từ mép ngoài cùng của vật, không chừa viền trống > 2 px.
3. Vật bị cắt mép ảnh: box dừng đúng ở mép ảnh và tick `truncated = yes` (015: hai xe bị cắt hai bên).
4. Gương chiếu hậu, cần ăng-ten, tay lái, chắn bùn: **tính vào** box xe.
5. **Hàng hóa xếp trên xe** (bao hàng trên tuk-tuk 035, thùng/túi trên thùng xe 012, 001) và **hàng nằm trong khối xe**: tính vào box xe nếu nằm sát thân xe, vượt không quá khoảng nửa chiều dài xe. Vật dài nhô ra xa (ống nhựa dài vắt qua xe máy 029): box chỉ ôm xe, bỏ phần ống nhô ra.
6. Người/vật mà người đang **mang** (gánh hàng, giỏ — 003, túi xách — 014): box `person` ôm **cơ thể**, bỏ phần gánh/giỏ nhô ra ngoài.

### 3.4 Người và phương tiện chồng nhau

- **Xe máy / xe đạp / xích lô / tuk-tuk có người lái và người ngồi sau:** gán **riêng** `person` cho từng người nhìn thấy + một box cho phương tiện. Hai box được phép chồng lên nhau (005, 006, 008, 010, 026).
- Box `motorcycle` **không** cần ôm phần thân người ở trên, nhưng phải ôm cả bánh, thân xe và chân chống. Gán box vừa khít phần xe nhìn thấy.
- **Người ngồi bên trong xe kín** (ô tô, van, xe tải, buýt — nhìn qua kính): **không gán** `person`.
- **Người ngồi ở chỗ hở** (tuk-tuk 004, sidecar 022, xích lô 023, thùng xe tải): có gán `person` nếu thấy ≥ 30% cơ thể.
- Trẻ em ngồi trước/sau trên xe máy (005): gán `person` như người lớn.

### 3.5 Vật không phải người/phương tiện thật → KHÔNG gán

| Loại | Ví dụ trong bộ ảnh | Cách xử lý |
|---|---|---|
| Tượng, điêu khắc | 017 (tượng người cưỡi ngựa), 011 (tượng thú + xe điêu khắc từ kim loại), 028 (tượng ngựa trên thùng xe) | Không gán tượng. Xe thật chở tượng (028) **vẫn gán** `truck`. |
| Mô hình, đồ chơi, xe tí hon | 038 (tủ xe đồ chơi/vịt cao su), 024 (xe vàng mô hình trên nóc xe) | Không gán. |
| Mộ/đài kỷ niệm hình xe | 027 (mộ đá hình xe bán tải) | Không gán. |
| Ma-nơ-canh | 033 | Không gán `person`. |
| Hình in/áp phích/pano/tranh tường | 018 (pano có ảnh người), 030 (áp phích người), 026 (tranh tường tàu, cảng) | Không gán, kể cả hình xe/người trong đó. |
| Hình phản chiếu (gương, kính, mặt nước) | 020 (gương chiếu hậu), 033 (phản chiếu trên kính) | Không gán vật chỉ thấy qua phản chiếu. |
| Linh vật/nhân vật hoạt hình | 019 (chuột trắng trong xe diễu hành) | Không gán linh vật. Người lái thật vẫn là `person` nếu ở chỗ hở. |

### 3.6 Đám đông, hàng xe dày → `ignore_region`

Dùng khi **không thể** gán từng vật một cách đáng tin.

1. Vẽ **một polygon/box** bao vùng, nhãn `ignore_region`. Không đặt chồng lên vật đã gán riêng.
2. Quy tắc nhóm:
   - **Hàng xe đỗ/xe máy kẹt đường dày đặc** (021 phía xa; 030 hàng xe đỗ phía sau; 008 hàng xe máy đỗ): gán riêng những xe ở **hàng đầu** (nhìn rõ ≥ 50% và ≥ 40 px), phần phía sau gộp thành `ignore_region`.
   - **Đám đông người ≥ 10 người sát nhau** (032: nhóm chờ xe buýt, 019: khán giả sau hàng rào): gán riêng người ở gần nhất, nhìn rõ ≥ 50%; phần còn lại gom vào `ignore_region`.
   - **Xe/người ở xa dưới ngưỡng 3.2:** gom thành `ignore_region` nếu có ≥ 3 vật liền kề, còn lại bỏ qua.
3. Ca đặc biệt — **xe chở xe** (002): gán một box `truck` cho xe chở; vùng các xe trên giàn gán một `ignore_region` (xe đang được chở, không tham gia giao thông).

## 4. Ca khó (tra cứu nhanh)

| # | Tình huống | Ảnh | Quyết định | `can_review` |
|---|---|---|---|---|
| 1 | Xe bán tải có thùng hở/thùng hàng | 001, 007, 012, 037 | `truck` | Không |
| 2 | Xe tải nhỏ (mini truck) thùng bạt | 036 | `truck` | Không |
| 3 | Minivan Hiace chở khách | 005, 016 | `van` | Không |
| 4 | Xe khách cỡ vừa (Hyundai County) | 009 | `bus` | Có (ranh giới van/bus) |
| 5 | Tuk-tuk Thái chở khách/xe lôi chở hàng | 004, 016, 035, 040 | `three_wheeler`; người lái ngồi ở chỗ hở gán `person` | Không |
| 6 | Xích lô đạp (cyclo) có người chở | 023 | `bicycle` (box ôm cả khung xe và ghế khách), người đạp là `person` | Có |
| 7 | Xe máy kèm sidecar | 022 | `motorcycle`, một box ôm cả xe + sidecar | Có |
| 8 | Xe máy chở 2–3 người | 005, 006, 026 | 1 `motorcycle` + từng `person` nhìn thấy | Không |
| 9 | Xe máy đỗ sát nhau thành hàng | 008, 030 | Gán hàng đầu, còn lại `ignore_region` | Có nếu không chắc ranh giới |
| 10 | Kẹt xe dày đặc, rất nhiều xe máy và người | 021 | Gán các xe/người gần; xa → `ignore_region` | Có |
| 11 | Người đi bộ gánh hàng, giỏ | 003 | `person` ôm cơ thể, bỏ phần gánh nhô ra | Không |
| 12 | Hàng quá khổ trên xe 3 bánh | 035 | Box xe gồm bao hàng nếu nằm trên xe | Có |
| 13 | Vật dài vắt ngang/nhô ra khỏi xe máy | 029 | Box xe bỏ phần ống nhô ra | Có |
| 14 | Xe bị che bởi hàng rào lưới | 018 | Gán, `occluded_level = 1`, box ôm cả xe | Có |
| 14b | Xe/người bị vật khác che một phần (xe máy sau xe máy, người sau xe) | 005, 008, 021, 030 | Ước lượng được → box đầy đủ gồm cả phần khuất; không ước lượng chắc → chỉ box phần thấy | Có nếu phải đoán nhiều |
| 15 | Hai xe bị cắt ở mép ảnh | 015 | Gán, `truncated = yes` | Không |
| 16 | Mờ do chuyển động (người, xe máy nền) | 014 | Gán người ở rõ nét; người/xe nền mờ không rõ hình → không gán | Có |
| 17 | Phản chiếu trên gương chiếu hậu/kính | 020, 033 | Không gán | Không |
| 18 | Tượng, mô hình, ma-nơ-canh, linh vật | 011, 017, 019, 024, 027, 028, 033, 038 | Không gán đối tượng giả; xe thật đi kèm vẫn gán | Có nếu thấy giống thật |
| 19 | Hình người/xe trên pano, áp phích, tranh tường | 018, 026, 030 | Không gán | Không |
| 20 | Xe diễu hành dạng "xe hoạt hình" lái được | 019 | `car` (là xe thật có biển số); người lái thấy rõ ở chỗ hở → `person` | Có |
| 21 | Ảnh đen trắng/đơn sắc | 022, 029 | Gán bình thường theo hình dạng | Không |
| 22 | Ảnh ban đêm, thiếu sáng, phản chiếu mặt đường ướt | 004, 007, 008, 013, 031 | Gán nếu nhận ra lớp; không chắc → bỏ và/hoặc tick | Có nếu mờ |
| 23 | Người ngồi quán cà phê sau xe tải đỗ | 001 | Gán `person` nhìn thấy ≥ 30%, `occluded_level` theo mức che | Không |
| 24 | Người nổi ở tiền cảnh bị cắt (mũ bảo hiểm, nửa thân) | 005, 016, 040 | Gán, `truncated = yes` nếu bị cắt mép ảnh | Không |
| 25 | Người cầm hàng/chợ vỉa hè cạnh xe máy | 010 | `person` + `motorcycle` + `bicycle` (xe bán hàng) | Không |
| 26 | Xe đỗ phía xa, nhỏ | 013, 025, 031, 039 | Áp dụng ngưỡng 3.2 | Không |
| 27 | Xe buýt ở nền, bị che một phần | 037, 040 | Gán nếu nhận ra; `occluded_level` theo mức che | Có |
| 28 | Cảnh không có vật hợp lệ (chỉ tượng/đồ chơi) | 017, 038 | **Không vẽ box nào**; ảnh vẫn hợp lệ và được giữ (ảnh "âm") | Không |
| 29 | Ô tô bị cắt chỉ còn góc | 015, 040 | Gán nếu ≥ 20 px và nhận ra được là xe | Có |
| 30 | Vật vừa giống thật vừa giống đồ chơi (xe độ, xe nghệ thuật) | 011 | Không gán nếu rõ là điêu khắc/trưng bày; không chắc → tick | Có |

## 5. QC và phiên bản

### 5.1 Tự kiểm trước khi nộp mỗi ảnh
- [ ] Mọi người/xe thật ≥ 20 px đã có box, kể cả xe đỗ?
- [ ] Có box nào đang gán vật giả (tượng, hình in, phản chiếu, mô hình)?
- [ ] Mỗi xe máy/xe đạp có người ngồi: đã gán `person` riêng chưa?
- [ ] Lớp xe đã đi qua đúng 6 câu hỏi ở 2.2?
- [ ] Vật bị che: đã box đầy đủ phần ước lượng được (hoặc chỉ phần thấy + `can_review` nếu không chắc)? Box không kéo ra ngoài mép ảnh?
- [ ] `occluded_level` và `truncated` đúng định nghĩa số ở 2.3?
- [ ] Vùng đông đúc đã dùng `ignore_region` thay vì gán lẻ tẻ không đáng tin?

### 5.2 Kiểm tra chéo
- Lấy mẫu ≥ 10 ảnh (25%) để hai người gán độc lập, đối chiếu: lệch lớp, lệch box (IoU < 0.7), sót/thừa box.
- Mọi bất đồng đều ghi lại: ảnh nào, vật nào, mỗi người gán gì, quyết định cuối cùng.
- Ảnh có `can_review` luôn được người thứ hai xem lại.

### 5.3 Phiên bản
- Mọi thay đổi **làm đổi nhãn** (thêm/bớt lớp, đổi ngưỡng, đổi quy tắc) → tăng phiên bản guideline và ghi vào bảng dưới.
- Chỉnh câu chữ không làm đổi nhãn → không tăng phiên bản.
- Ca mơ hồ chưa có quy tắc: tick `can_review`, ghi vào "Ca chờ quyết định", họp thống nhất rồi mới thêm vào mục 4.

| Phiên bản | Thay đổi | Ảnh dẫn chứng |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên. | — |
| v1 | Quy tắc box amodal cho vật bị che (3.3.1). Thêm `bus`, `three_wheeler`, `bicycle`; định nghĩa `car/van/truck` bằng "thùng hàng riêng"; định nghĩa số cho `occluded_level`; ngưỡng kích thước 20/40 px; quy tắc người trên xe hở/kín; quy tắc vật giả; `ignore_region` cho đám đông; 30 ca khó. | 001–040 |

### 5.4 Ca chờ quyết định (cần nhóm/giảng viên chốt)
1. Xích lô đạp (023) xếp `bicycle` hay `three_wheeler`?
2. Pickup dân dụng (Hilux/Ranger) xếp `truck` (đang chọn) hay `car`?
3. Xe đang được chở trên xe chở xe (002): dùng `ignore_region` (đang chọn) hay gán từng `car`?
4. Xe điêu khắc/nghệ thuật bằng kim loại (011): gán hay không?

---

## 6. Quy tắc chốt khi calibration (v1.1)


| Mã | Quy tắc | Lý do (ca/ảnh dẫn chứng) | Ảnh hưởng |
|---|---|--|---|
| R-01 |  |  |  |

| Phiên bản | Thay đổi | Ảnh dẫn chứng |
|---|--|--|
| v1.1 |  |  |
