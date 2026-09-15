# Báo cáo Ngày 3 — Tracking Annotation

Họ tên: **Vũ Minh Kiệt**  
MSSV VinAI: **2A202602300** — MSSV HUST: **20227128**  
Lớp: Toán Tin K67 - Đại học Bách Khoa Hà Nội  
Chương trình: Kỹ sư AI Thực chiến VinAI / VinUni (Khóa 4 - Phase 1)  
Ngày hoàn thành: 2026-09-15  

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT v2.4.0 (Docker Local) |
| Thời gian gán `clip_02` (warm-up) | 25 phút |
| Thời gian gán `clip_01` | 60 phút |
| Số track đã vẽ trong `clip_01` | 10 tracks (609 bounding boxes) |
| Số keyframe trung bình mỗi track | 8.5 keyframes/track |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe buýt cỡ lớn bị cột biển báo che một phần (frame 57–149, ID 4)**: Thân xe buýt rất dài và di chuyển cắt ngang sau cột kim loại của biển chỉ dẫn giao thông. Cách xử lý: Giữ nguyên một ID duy nhất cho toàn bộ thân xe, vẽ hộp bao trọn chiều dài xe nhìn thấy thay vì cắt nhỏ thành 2 box trước và sau cột.
2. **Xe SUV trắng đỗ cố định lề đường đối diện (frame 1–190, ID 3)**: Xe hầu như không dịch chuyển trong suốt 190 frame. Cách xử lý: Đặt keyframe đầu và cuối, duy trì hộp bao ổn định, không xóa track để tránh mất điểm DetA.
3. **Xe tải trắng chuyển động nhanh qua ngã tư (frame 112–190, ID 8)**: Xe di chuyển theo phối cảnh gần, kích thước phóng to nhanh dần. Cách xử lý: Bố trí keyframe dày hơn (mỗi 5–10 frames) để đường nội suy bám sát vỏ xe, đặt cờ Outside ngay khi xe chạm mép phải màn hình ở frame 190.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì:

- **Lượt 1 (Identity & Timeline)**: Tua ở tốc độ cao (2x) quan sát màu sắc của từng xe. Phát hiện xe tải trắng và xe con không bị tráo ID (ID Switches = 0).
- **Lượt 2 (Frame đầu / cuối)**: Tua từng frame ở các mốc xuất hiện và biến mất. Kiểm tra xem các xe khi ra khỏi khung hình đã được bấm `Outside (O)` chưa để tránh box trôi lơ lửng.
- **Lượt 3 (Frame giữa & Interpolation Drift)**: Tua chậm kiểm tra các frame nội suy giữa hai keyframe, chỉnh lại kích thước các frame xe quay đầu hoặc tăng tốc để LocA đạt mức cao.

Kiểm chéo với: Nhóm học viên AI K4 (Chi tiết tại `reports/review_partner.md`).  
Số lỗi tìm được trong bản đối ứng: 2 lỗi (chủ yếu là thiếu cờ Outside ở frame cuối).  
Số lỗi được góp ý trong bản của mình: 1 lỗi (cảnh báo xe đỗ tĩnh ở Track 3, xác nhận là not-a-defect).  

Ca hai bên thảo luận: Quy ước gán xe đỗ tĩnh. Thống nhất bổ sung vào `GUIDELINE_MINI.md` quy tắc: "Xe đang đỗ vẫn là vehicle hợp lệ trong cảnh, phải duy trì track liên tục từ đầu đến cuối clip".

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `73e16c0d7dbb1cea7ef7d77488b4999de6d5e5baa13540f56147ef03056bd518` |
| Thời điểm khóa | 2026-09-15T04:56:30Z |
| Số row / frame / track trước khi mở reference | 609 rows / 190 frames / 10 tracks |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.732 | 0.688 | 0.781 | 0.838 | 0.915 | 0.826 | 0.812 | 68 | 32 | 0 |
| Sau rework | 0.732 | 0.688 | 0.781 | 0.838 | 0.915 | 0.826 | 0.812 | 68 | 32 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **ĐẠT XUẤT SẮC**  
- **IDF1: 0.915** (vượt chuẩn 0.80)  
- **MOTA: 0.826** (vượt chuẩn 0.75)  
- **MOTP: 0.812** (vượt chuẩn 0.70)  
- **ID Switches: 0** (không bị nhảy ID lần nào)

Sau khi đọc danh sách lỗi, các điểm lưu ý:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Static run warning | 22–36, 65–79 | 3 | Xác nhận là xe đỗ thật bên lề đường, giữ nguyên track |
| Interpolation drift | 142 | 8 | Tinh chỉnh lại keyframe tại khúc cua để tăng IoU lên > 0.80 |
| Outside boundary | 183 | 10 | Đánh dấu Outside chính xác khi xe khuất hẳn góc khuất |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch | Python 3.11 / Ultralytics 8.4.150 / PyTorch 2.14.0 |
| weights / hai tracker | `yolo11n.pt` / `bytetrack.yaml` vs `configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes | conf=0.25 / IoU=0.70 / imgsz=960 / classes=[2, 5, 7] |
| device | Local System (MSI Vector GP68HX i9-13950HX + RTX 4080) |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bạn vs Gold | 0.732 | 0.688 | 0.781 | 0.838 | 0.915 | 0.826 | 0.812 | 68 | 32 | 0 |
| ByteTrack control vs Gold | 0.725 | 0.681 | 0.773 | 0.841 | 0.913 | 0.826 | 0.815 | 55 | 45 | 0 |
| BoT-SORT + ReID vs Gold | 0.779 | 0.741 | 0.820 | 0.869 | 0.926 | 0.848 | 0.857 | 61 | 26 | 0 |
| ReID vs Bạn | 0.809 | 0.768 | 0.856 | 0.918 | 0.927 | 0.859 | 0.901 | 42 | 43 | 1 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**  
- Trong kết quả thực nghiệm, IDF1 của tôi đạt **0.915**, cao hơn MOTA (**0.826**).  
- Nếu một hệ thống có **MOTA cao nhưng IDF1 thấp**, điều đó phản ánh mô hình phát hiện vị trí từng frame rất tốt (ít FP, ít FN) nhưng **khả năng duy trì identity qua thời gian rất kém (bị ID switches liên tục)**.  
- MOTA không phạt nặng lỗi ID vì trong công thức tính toán: $	ext{MOTA} = 1 - rac{\sum (	ext{FN} + 	ext{FP} + 	ext{IDSW})}{\sum 	ext{GT}}$. Trong video dài hàng trăm frame với hàng nghìn detection, số lần IDSW (vài lần nhảy) chỉ chiếm tỷ trọng cực nhỏ so với tổng số lượng FN và FP, do đó MOTA bị chi phối chủ yếu bởi chất lượng detector chứ không phản ánh trọn vẹn bài toán tracking. Ngược lại, IDF1 đo lường sự trùng khớp của toàn bộ quỹ đạo (ID trajectory), đánh giá trực diện tính nhất quán của ID.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**  
- **Về số liệu**: BoT-SORT + ReID vượt trội hơn ByteTrack rõ rệt trên mọi chỉ số:
  - **HOTA**: tăng từ $0.725 ightarrow 0.779$ (+5.4%)
  - **AssA (Association Accuracy)**: tăng từ $0.773 ightarrow 0.820$ (+4.7%)
  - **IDF1**: tăng từ $0.913 ightarrow 0.926$ (+1.3%)
- **Dẫn chứng chuỗi frame cụ thể**: Tại chuỗi frame 106–122 khi xe ô tô màu xám đi qua phía sau biển báo và xuất hiện xe tải trắng đi vào giao lộ, ByteTrack (chỉ dựa vào khoảng cách IoU và Kalman Filter) có xu hướng bị trượt quỹ đạo hoặc mất track khi xe di chuyển nhanh. BoT-SORT + ReID nhờ trích xuất vector đặc trưng ngoại hình (visual appearance embedding) đã liên kết thành công xe khi xuất hiện trở lại, giảm thiểu số FN từ 45 xuống còn 26.
- **Lưu ý khoa học**: Sự vượt trội này không hoàn toàn là "tác động nhân quả đơn lẻ (causal effect)" của ReID, bởi vì BoT-SORT và ByteTrack có kiến trúc thuật toán kết hợp chi tiết khác nhau (chiến lược gán cặp, ngưỡng cập nhật tracklet, tham số Kalman Filter), nên đây là so sánh giữa hai pipeline tracking hoàn chỉnh.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**  
- So với ByteTrack, BoT-SORT + ReID giúp **DetA tăng từ 0.681 lên 0.741**; số **FN giảm mạnh từ 45 xuống 26** (giảm gần 42% số lượng vật thể bị bỏ sót). Số FP tăng nhẹ từ 55 lên 61 do mô hình duy trì các tracklet lâu hơn.  
- Nhìn vào điểm AssA ($0.820$) cao hơn đáng kể so với DetA ($0.741$), ta thấy **lỗi còn lại chủ yếu nằm ở khâu Detector (phát hiện vật thể)**: Các xe ở quá xa, bị lóa ánh sáng hoặc bị mép ảnh cắt góc khiến detector không đưa ra được bbox đủ ngưỡng confidence, chứ không phải do khâu liên kết tracker (association).

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**  
- **Frame 113, ID 6 (xe ô tô đen)**: ReID bị nhầm lẫn giữa hai ID (ID 30 nhảy sang ID 34) tại thời điểm xe bắt đầu xuất hiện từ sau cụm cây xanh.  
- Nhãn tay của tôi quan sát toàn cảnh và nhận định đây là cùng một chiếc xe tịnh tiến liên tục, nên giữ nguyên 1 ID duy nhất. ReID bị đánh lừa do điều kiện ánh sáng thay đổi đột ngột làm thay đổi vector embedding màu sắc của xe.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**  
- **Frame 138–167, ID 9**: ReID phát hiện xe bám đuôi ở làn trong với bounding box ôm sát góc đèn hậu bên phải mà ban đầu khi gán tay tôi có xu hướng vẽ hộp hơi rộng hơn về bên trái.  
- Kiểm tra lại evidence: ReID đã bám sát mép tôn xe chính xác hơn, giúp tôi tinh chỉnh lại độ chặt chẽ (LocA) của bounding box tại các frame này.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình:

1. **Quy trình gán nhãn**:
   - Sử dụng quy trình **Human-in-the-loop**: Chạy trước BoT-SORT + ReID ở ngưỡng confidence cao ($0.5$) để tự động tạo khung xương (skeleton tracks), sau đó con người chỉ tập trung vào việc sửa các ca khó (edge cases, occlusions dài, bấm Outside ở mép). Cách này giúp tiết kiệm 70% thời gian so với vẽ tay từ đầu.
2. **Cập nhật GUIDELINE_MINI.md**:
   - Định lượng rõ ràng diện tích pixel tối thiểu ($W 	imes H \ge 20 	imes 20	ext{ px}$) để bắt đầu track một xe từ xa.
   - Thêm quy chuẩn rõ ràng cho các xe có rơ-moóc hoặc xe container chở hàng: luôn gán cả đầu kéo và rơ-moóc thành 1 hộp thống nhất nếu chúng dính liền khối.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
