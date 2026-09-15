# Khai báo bài thực hành cá nhân — Day 3 Video Tracking

Học viên thực hiện độc lập bài thực hành theo đúng quy chuẩn liêm chính học thuật của chương trình VinAI K4.

## Thông tin học viên

- **Họ và tên**: Vũ Minh Kiệt
- **MSSV VinAI / VinUni**: 2A202602300
- **MSSV HUST**: 20227128
- **Lớp / Khóa**: Toán Tin K67 — Đại học Bách Khoa Hà Nội
- **Kênh liên lạc**: kiet.vm227128@sis.hust.edu.vn / Discord VinAI K4
- **Hình thức thực hiện**: Cá nhân (kết hợp Peer Review với nhóm K4)

| Họ và tên | MSSV | Vai trò / phần việc | Artifact tự sở hữu |
| --- | --- | --- | --- |
| Vũ Minh Kiệt | 2A202602300 | Tự gán nhãn, QC, khóa Pre-Gold, chạy Tracker & viết báo cáo | Toàn bộ repo và 12 artifacts |

## Phần đóng góp và học được của người nộp repo này

- **Họ và tên / MSSV**: Vũ Minh Kiệt / 2A202602300
- **Tôi trực tiếp tạo hoặc chỉnh sửa những artifact nào**:
  1. `annotations/clip_02/gt.txt` (Warm-up MOT 1.1)
  2. `annotations/clip_01/gt.txt` (Core annotation 609 bboxes)
  3. `evidence/pre-gold/clip_01/gt.txt` & `manifest.json` (SHA-256 lock)
  4. `GUIDELINE_MINI.md` (Quy chuẩn ID & 3 ca mơ hồ)
  5. `reports/review_partner.md` (Self-QC 3 lượt & checklist)
  6. `outputs/eval_vs_gold.json` (Đánh giá với reference)
  7. `outputs/model_bytetrack_clip_01.txt` & `outputs/model_reid_clip_01.txt` (Control & Treatment tracking)
  8. `outputs/model_run_config.json` & các file JSON đánh giá model
  9. `reports/REPORT.md` (Báo cáo tổng kết toàn diện)
- **Finding hoặc quyết định annotation tôi chịu trách nhiệm**: Quyết định duy trì ID 3 cho xe đỗ cố định lề đường, và giữ nguyên một bounding box duy nhất cho xe buýt ID 4 khi bị cột biển báo che khuất thân xe.
- **Tôi học được gì về identity, occlusion, MOT hoặc ReID**: Hiểu sâu sắc sự khác biệt giữa Object Detection (tĩnh) và Video Tracking (động). Nắm vững cơ chế đánh cờ Outside để ngắt tracklet, vai trò của Linear Interpolation trong CVAT, và tại sao BoT-SORT kết hợp ReID appearance cues lại giúp cải thiện AssA và HOTA vượt bậc so với việc chỉ dùng chuyển động hình học IoU thuần túy.
- **Điều tôi đã kiểm lại độc lập trước khi nộp**: Chạy công cụ kiểm tra `check_mot_labels.py` đạt chuẩn 0 lỗi định dạng trên cả 2 clip; đối chiếu điểm số với Gold đạt IDF1 0.915, MOTA 0.826, MOTP 0.812 vượt mọi cổng yêu cầu.
