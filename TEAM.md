# Thông tin nhóm

- Tên nhóm: Ban1_C303
- Mã thành viên (ba chữ số): 101, 102, 103, 104
- Người điều phối: 102
- Người review bản cuối: 104
- Commit chốt bài (điền ở kênh nộp sau khi commit):

## Kết quả tóm tắt (điền cuối bài)

- Lượt gán hoàn tất / được giao: 80/80
- IAA trước calibration (kèm mẫu số): IoU TB 0.818 và cùng lớp 93.9% trên 229 cặp box ghép 
- Số ảnh phải calibration: 40 / 40 (phạm vi cả lô, 35 ảnh bị cờ với 437 ca)
- Vòng QC và số ảnh đã duyệt / cả lô: vòng 2, 40/40 (vòng 1 ra Hold do còn cờ can_review)
- Tỉ lệ lỗi: 0.0% theo ảnh (0/40), 0.0% theo đối tượng (0/428)
- **Quyết định** (Release / Hold / Rework) và lý do một dòng: Release, vòng 2 đạt 0/40 ảnh lỗi và không còn box can_review; còn 4 box ignore_region phủ vật giả cần xóa và rà lại ở vòng 3.
