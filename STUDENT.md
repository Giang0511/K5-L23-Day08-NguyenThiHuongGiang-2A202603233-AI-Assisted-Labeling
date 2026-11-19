# Thông tin học viên

- Họ và tên: Nguyễn Thị Hương Giang
- MSHV: 2A202603233
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: Giang0511
- Repo bài nộp (URL): https://github.com/Giang0511/K5-L23-Day08-NguyenThiHuongGiang-2A202603233-AI-Assisted-Labeling
- Commit chốt bài (hash): 29a1b60
- Máy: Windows 11 Pro, Intel Core i7-11850H, RAM 64GB, NVIDIA T1200 Laptop GPU 4GB, CUDA.
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 598 / 7 / 76 (mẫu số: 683 box máy); added: 39 (mẫu số: 644 đối tượng)
- So với gold: precision / recall của **máy** 53.9% (48/89) / 81.4% (48/59)
- Bắt bẫy: 8/12 = 66.7%
- Thời gian: gán tay 3.75 phút/ảnh (15 phút / 4 ảnh); duyệt có pre-label 3.67 phút/ảnh (88 phút / 24 ảnh)
- Lỗi QC: 87.5% theo ảnh (7/8), 4.3% theo đối tượng (10/232)
- **Quyết định:** Hold — lý do một dòng: guideline v1.1 còn câu hỏi mở số 4 chưa chốt (ngưỡng "còn thấy ≥ 20%" của người trong xe ba bánh có mui, chưa có ca đo được trong 24 ảnh); ngoài ra ảnh lỗi 87.5% > 2% và bắt bẫy 66.7% < 80% nên sau khi chốt câu 4 vẫn phải Rework.
