# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Nguyễn Lê Trân Châu`
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

Bổ sung của nhóm (nếu có): `...`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Giúp duy trì tính liên tục của cùng một đối tượng |
| Xe bị che lâu hơn ngưỡng trên | tạo **track mới** | Tránh gán nhầm ID khi không đủ bằng chứng để xác nhận là cùng xe |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Xe đã rời khỏi vùng quan sát nên lần xuất hiện lại được xem là một track mới |
| Hai xe cắt nhau / chồng lên nhau | giữ ID dựa trên **vị trí, hướng di chuyển và đặc điểm nhận dạng**; không đổi ID chỉ vì bị che/chồng bbox | Giảm nguy cơ ID switch khi hai xe đi gần hoặc cắt nhau |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: **IoU ≥ 0.5 với vùng xe quan sát được** |
| Xe đang đỗ, không di chuyển | vẫn giữ track nếu còn nhìn thấy; không xóa chỉ vì xe không di chuyển |
| Keyframe đặt dày ở đâu | đặt dày tại **frame bắt đầu/kết thúc track, lúc bị che, cắt nhau và khi bbox thay đổi mạnh** |
## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01/ 79–100/ ID 6`
- Tình huống: `Xe đã xuất hiện nhưng còn nhỏ và khó quan sát rõ`
- Quyết định: `Đặt Outside/xóa bbox ở các frame trước khi xe xuất hiện`
- Lý do: `Không nên duy trì track khi xe còn nhỏ và khó quan sát rõ`

### Ca 2
- Clip / frame / ID: `clip_01/ 149–151/ ID 4`
- Tình huống: `Xe vẫn còn một phần trong khung hình nhưng đang rời khỏi vùng quan sát`
- Quyết định: `Đặt Outside/xóa bbox sau khi xe rời gần  rời khung`
- Lý do: `Đảm bảo bbox phản ánh đúng phần đối tượng quan sát được và thời gian tồn tại của track`

### Ca 3
- Clip / frame / ID: `clip_01/ 169–171/ ID 8`
- Tình huống: `Xe đã rời khỏi khung hình nhưng bbox vẫn còn được gán`
- Quyết định: `Đặt Outside/xóa bbox từ frame xe thực sự rời khỏi khung`
- Lý do: `Không duy trì track khi đối tượng không còn xuất hiện trong ảnh`

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- `Khi xe vừa xuất hiện hoặc đang rất nhỏ/mờ, vẫn giữ bbox nếu có đủ bằng chứng xác định đó là xe; không xóa chỉ vì kích thước nhỏ.`
- `Khi xe bị che hoặc rời khung, phải xác định rõ thời điểm bắt đầu/kết thúc quan sát được: giữ cùng ID nếu vẫn có đủ bằng chứng liên tục; đặt Outside khi xe thực sự không còn quan sát được, không kéo dài bbox theo suy đoán`
