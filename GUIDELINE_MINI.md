# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Học viên: **Vũ Minh Kiệt** — **MSSV Vin**: `2A202602300` — **MSSV HUST**: `20227128`  
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của học viên:
- Chỉ gán các xe 4 bánh có kích thước nhìn thấy từ $\ge 15\text{ px}$ theo chiều rộng hoặc chiều cao.
- Không gán xe mô hình đồ chơi hoặc xe máy có thùng chở hàng 3 bánh.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của học viên | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID nếu bị che **dưới 25 frame** ($\approx 2.0\text{ giây}$ tại $12.5\text{ fps}$) | Quỹ đạo chuyển động (motion trajectory) và vận tốc xe vẫn suy đoán được một cách tin cậy |
| Xe bị che lâu hơn ngưỡng trên | Đánh dấu `Outside (O)` khi bị che khuất hoàn toàn; nếu sau đó xuất hiện lại thì coi như **track mới** | Tránh rủi ro gán nhầm ID khi có xe khác đi ngang qua trong thời gian che khuất quá dài |
| Xe rời khung hình rồi quay lại | Mặc định: **track mới** | Chuẩn MOT quy định vật thể đã hoàn toàn ra khỏi frame xem như kết thúc vòng đời track |
| Hai xe cắt nhau / chồng lên nhau | Duy trì ID riêng của từng xe, xe bị che một phần vẽ bbox ôm phần nhìn thấy | Đảm bảo tính nhất quán (AssA cao), không hoán đổi ID (tránh ID Switches) |

## 3. Luật bbox

| Tình huống | Luật của học viên |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | Bbox chạm đúng mép ảnh ($x=0$ hoặc $y=0$ hoặc $x=960, y=540$), tuyệt đối không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | Bbox ôm sát phần **nhìn thấy được (visible region)** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu track từ frame đầu tiên xác định rõ ràng là xe bốn bánh (kích thước tối thiểu $\ge 15\text{ px}$) |
| Xe đang đỗ, không di chuyển | Gán 1 keyframe đầu và 1 keyframe cuối, duy trì ID liên tục suốt clip mà không xóa |
| Keyframe đặt dày ở đâu | Đặt keyframe dày tại các khúc cua, lúc xe giảm tốc/tăng tốc hoặc khi bị che khuất một phần |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1: Xe ô tô trắng đỗ cố định bên lề đường
- **Clip / frame / ID**: `clip_01` / frame 1–190 / `ID 3`
- **Tình huống**: Xe SUV trắng đỗ cố định bên lề đường đối diện, hầu như không di chuyển trong suốt 190 frame.
- **Quyết định**: Duy trì 1 track duy nhất từ frame 1 đến frame 190, không ngắt track.
- **Lý do**: Đây là xe thật đang hiện diện trong cảnh, giữ nguyên track giúp duy trì độ chính xác phát hiện (DetA) mà không làm tăng ID Switch.

### Ca 2: Xe buýt lớn màu vàng-xanh đi qua cột biển báo
- **Clip / frame / ID**: `clip_01` / frame 57–149 / `ID 4`
- **Tình huống**: Xe buýt cỡ lớn di chuyển từ phải sang trái và đi qua phía sau cột biển báo giao thông trung tâm (bị cột che một phần thân xe).
- **Quyết định**: Giữ nguyên ID 4 xuyên suốt, tại frame bị cột biển che thì bbox vẫn bao trọn toàn bộ thân xe buýt nhìn thấy, không tách làm 2 box riêng lẻ.
- **Lý do**: Xe buýt là một vật thể liền khối, việc bị thanh kim loại mảnh của cột biển che không làm đứt đoạn nhận diện vật thể.

### Ca 3: Xe tải trắng di chuyển nhanh qua ngã tư
- **Clip / frame / ID**: `clip_01` / frame 112–190 / `ID 8`
- **Tình huống**: Xe tải trắng thùng kín đi từ góc trái sang phải cắt ngang qua làn đường, kích thước thay đổi nhanh do góc quay phối cảnh.
- **Quyết định**: Đặt keyframe cách nhau mỗi 5–10 frames để hộp ôm sát cabin và thùng xe, tại frame 190 chạm mép phải thì giữ bbox chạm đúng mép $x=960$.
- **Lý do**: Xe chuyển động tịnh tiến nhanh theo phối cảnh gần, nếu đặt keyframe quá thưa sẽ bị hiện tượng trôi hộp (interpolation drift).
