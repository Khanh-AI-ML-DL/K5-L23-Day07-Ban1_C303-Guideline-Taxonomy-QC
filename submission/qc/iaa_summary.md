# Tóm tắt IAA (taxonomy v0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 4
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 229 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 229 | 0.818 | 93.9% | 59.0% | 94.8% | 164/36 | 0 | CHƯA ĐẠT |
| 101|102 | 6 | 22 | 0.876 | 90.9% | 68.2% | 100.0% | 10/0 | 0 | CHƯA ĐẠT |
| 101|103 | 9 | 71 | 0.816 | 100.0% | 52.1% | 94.4% | 82/5 | 0 | CHƯA ĐẠT |
| 101|104 | 5 | 35 | 0.792 | 74.3% | 40.0% | 91.4% | 50/4 | 0 | CHƯA ĐẠT |
| 102|103 | 5 | 28 | 0.868 | 96.4% | 71.4% | 100.0% | 2/7 | 0 | ĐẠT |
| 102|104 | 9 | 50 | 0.779 | 96.0% | 68.0% | 96.0% | 14/13 | 0 | CHƯA ĐẠT |
| 103|104 | 6 | 23 | 0.832 | 100.0% | 65.2% | 87.0% | 6/7 | 0 | CHƯA ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 437 ({'bo_sot_thua': 194, 'iou_thap': 103, 'sai_lop': 14, 'sai_thuoc_tinh': 97, 'bat_dong_ignore': 22, 'co_can_review': 7}), trên 35 ảnh cần calibration.
