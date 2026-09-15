# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Nguyễn Hải Nam (Cá nhân)`
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

Bổ sung của nhóm (nếu có): `Không có.`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Trong vòng 25 frame có thể dự đoán được quỹ đạo di chuyển của xe một cách chính xác. |
| Xe bị che lâu hơn ngưỡng trên | Cắt track hiện tại, tạo track ID mới khi xe lộ diện hoàn toàn trở lại. | Tránh nối nhầm ID giữa các xe giống nhau sau một thời gian dài mất dấu tích. |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Khi xe đi ra ngoài, không thể xác minh xe đi vào có đúng là xe trước đó không. |
| Hai xe cắt nhau / chồng lên nhau | Bbox của xe nào gần camera hơn (không bị che) thì vẽ đè lên xe kia, duy trì ID cho cả 2. | Phản ánh đúng độ sâu không gian và sự che khuất trong thực tế. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `Kích thước bbox >= 15x15 pixel và nhận diện rõ đầu/đuôi xe.` |
| Xe đang đỗ, không di chuyển | `Vẫn duy trì gán nhãn và ID bình thường nếu xe nằm trong khu vực quan sát.` |
| Keyframe đặt dày ở đâu | `Đặt dày ở các đoạn xe thay đổi vận tốc, chuyển hướng, hoặc bị che khuất một phần.` |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / frame 120-160 / ID 4`
- Tình huống: `Xe ô tô con màu đen đi ngang qua gốc cây to bên đường, bị che khuất hoàn toàn khoảng 30 frame.`
- Quyết định: `Cắt ID 4 tại frame 120. Tạo ID mới (ID 8) tại frame 160 khi xe bắt đầu đi ra khỏi cây.`
- Lý do: `Thời gian che khuất là 30 frame, lớn hơn ngưỡng 25 frame quy định, do đó áp dụng luật tạo track mới.`

### Ca 2
- Clip / frame / ID: `clip_01 / frame 215 / ID 12`
- Tình huống: `Xe tải lớn (ID 10) chạy ngang che mất 80% xe con (ID 12) bên kia đường.`
- Quyết định: `Vẫn giữ ID 12, chỉ vẽ bbox ôm phần nóc và mũi xe lộ ra ngoài.`
- Lý do: `Che khuất một phần và thời gian ngắn (chỉ 10 frame), áp dụng luật giữ ID và ôm phần nhìn thấy được.`

### Ca 3
- Clip / frame / ID: `clip_02 / frame 45 / ID 6`
- Tình huống: `Xe bắt đầu di chuyển khỏi mép trái màn hình, chỉ còn lại rất ít thân xe.`
- Quyết định: `Kết thúc track sớm (không vẽ bbox) thay vì cố bám tới lúc chỉ còn 1 pixel.`
- Lý do: `Phần thân xe còn lại quá nhỏ (chưa tới 10% xe), khó vẽ bbox ổn định và không còn ý nghĩa tracking.`

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- `Bổ sung luật xử lý đối với các xe đậu thành hàng sát nhau ở bãi đỗ nhưng không tham gia vào luồng giao thông chính: Chỉ gán những xe bắt đầu di chuyển hoặc có khả năng vào đường chính.`
- `Làm rõ hơn quy định vẽ bbox đối với xe tải có thùng kéo dài hoặc kéo theo container: bbox phải bao gồm toàn bộ phần rơ-moóc kéo theo thay vì chỉ vẽ đầu kéo.`
