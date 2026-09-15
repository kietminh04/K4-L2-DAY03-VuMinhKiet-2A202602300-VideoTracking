# Peer review — Day 3

Chép file này thành `reports/review_partner.md`. Reviewer chỉ ghi finding; tác giả
tự sửa bài của mình và điền closure.

| Trường | Giá trị |
| --- | --- |
| Author | Vũ Minh Kiệt (MSSV Vin: 2A202602300 / HUST: 20227128) |
| Reviewer | Nhóm học viên AI K4 / Peer Reviewer |
| Pair ID | K4-PAIR-DAY03-01 |
| CVAT version | v2.4.0 (Local Docker Engine) |
| Thời điểm review | 2026-09-15 11:30 (UTC+7) |

## Danh sách finding

Mỗi finding bắt buộc có `frame + ID + lỗi + sửa thế nào`. Nếu chưa thống nhất,
dùng `needs-review`; không ép tác giả sửa theo cảm tính.

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure: fixed / not-a-defect / needs-review |
| ---: | ---: | ---: | ---: | --- | --- | --- | --- |
| 1 | 0..189 | 1..190 | 3 | Static Run | Track 3 đứng im suốt 190 frame do xe SUV trắng đỗ lề đường | Không phải lỗi quên bấm Outside; đây là xe đỗ thật | fixed (not-a-defect) |
| 2 | 13 | 14 | 2 | Gap / Đứt quãng | Xe ô tô chuyển động bị mất 1 frame giữa chừng khi xuất thô từ detector | Bật interpolation trong CVAT để nối liền frame 13 sang 15 | fixed |
| 3 | 148 | 149 | 4 | Outside timing | Thân xe buýt lớn đi hết mép trái màn hình | Bấm phím Outside (O) tại frame 149 để tránh box trôi lơ lửng | fixed |

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | Đạt 10 tracks hợp lệ trên clip_01 (chỉ xe car, bus, truck) |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | IDSW = 0, toàn bộ 10 tracks giữ ID nhất quán |
| Occlusion ngắn giữ ID; crossing không đổi ID | PASS | Xe buýt ID 4 qua cột biển báo giữ nguyên ID |
| Entry/exit đúng; không box treo sau khi xe rời khung | PASS | Đã có cờ outside: True cho các track kết thúc trước frame 189 |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | PASS | Bbox ôm sát thân xe, không vượt biên 960x540 |
| Frame giữa hai keyframe không bị interpolation drift | PASS | LocA đạt 0.838 và MOTP đạt 0.812 |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | Validator check_mot_labels.py đạt 0 LỖI |
| Mọi finding có cách sửa và closure do tác giả điền | PASS | Đã cập nhật closure cho cả 3 findings |

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | PASS | Tua nhanh từ đầu đến cuối: không bị đổi màu/nhảy ID |
| 2 — endpoint/scope | PASS | Kiểm tra frame đầu và frame cuối của từng xe: xuất hiện và biến mất đúng nhịp |
| 3 — geometry/interpolation | PASS | Tua chậm: hộp ôm khít vỏ xe, không bị trôi khi xe rẽ/đổi hướng |

## Exit ticket

1. Finding quan trọng nhất và rule dùng để kết luận: Cần duy trì cờ Outside (O) đúng frame xe khuất hẳn, tránh để box tồn tại dạng tĩnh ở mép ảnh.
2. Một finding tác giả đóng là `not-a-defect`, kèm lý do: Track 3 bị validator cảnh báo đứng yên, nhưng đây là xe thật đang đỗ bên đường.
3. Một rule cần Lab Coach làm rõ: Quy ước ngưỡng số frame tối đa để phục hồi ID xe khi bị che khuất hoàn toàn trong cảnh giao thông phức tạp.
