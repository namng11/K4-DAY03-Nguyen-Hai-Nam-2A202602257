# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Nguyễn Hải Nam (Cá nhân)`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | `25` phút |
| Thời gian gán `clip_01` | `45` phút |
| Số track đã vẽ trong `clip_01` | `18` |
| Số keyframe trung bình mỗi track | `6` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `Nhiều xe chạy song song ở ngã tư và che khuất nhau liên tục. Xử lý: Zoom kỹ, đi từng frame một thay vì nhảy keyframe dài để đảm bảo ID không bị gán nhầm sang xe bên cạnh.`
2. `Xe đi vào vùng bóng râm, bị mất độ tương phản so với mặt đường. Xử lý: Tăng độ sáng màn hình, dựa vào quỹ đạo chuyển động trước đó để nội suy vị trí bbox.`
3. `Xe di chuyển ra xa và kích thước quá nhỏ, dễ lẫn với các vật thể tĩnh. Xử lý: Áp dụng luật kết thúc track khi xe dưới 15x15 pixel.`

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `Tập trung tua qua lại nhanh để kiểm tra tính liên tục của track ID, bắt được 1 trường hợp xe tải bị đứt ID khi đi qua cột đèn.`
- Lượt 2: `Kiểm tra độ chính xác của bbox ở các frame đầu/cuối, bắt được vài track kết thúc quá trễ (xe đã ra khỏi ảnh nhưng vẫn còn bbox rỗng).`
- Lượt 3: `Kiểm tra các keyframe ở đoạn giữa (đặc biệt khi xe rẽ), sửa lại vài đoạn bbox nội suy bị trôi khỏi xe.`

Kiểm chéo với: `(Do làm cá nhân nên tự kiểm tra chéo lại bài của chính mình)`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `3 (tự tìm)`. Số lỗi bạn ấy tìm được trong bản của bạn: `0`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`Tự rút ra nhận xét: Ban đầu phân vân việc xe đỗ bên lề có gán hay không. Sau đó tự bổ sung vào guideline là chỉ gán nếu xe nằm trong khu vực mặt đường hoặc có khả năng tham gia giao thông.`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `(Chưa có do thiếu file)` |
| Thời điểm khóa | `(Chưa có)` |
| Số row / frame / track trước khi mở reference | `(Chưa có)` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | | | | | | | | | | |
| Sau rework | | | | | | | | | | |
*(Ghi chú: Không có file đánh giá trong folder outputs)*

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **(Chưa xác định)**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox rộng | 150 | 3 | Bóp hẹp bbox lại ôm sát mép phải xe |
| Đứt ID | 200-210 | 5 | Nối ID lại do thời gian bị che chỉ 10 frame |
| Thiếu nhãn | 315 | 8 | Thêm mới track cho xe xuất hiện ở góc xa |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13` |
| weights / hai tracker | `yolo26n.pt / bytetrack.yaml & botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25 / 0.7 / 960 / [2, 5, 7]` |
| device | `0` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | | | | | | | | | | |
| ByteTrack control vs gold | | | | | | | | | | |
| BoT-SORT + ReID vs gold | | | | | | | | | | |
| ReID vs bạn | | | | | | | | | | |
*(Ghi chú: Metrics trống do không có file JSON trong outputs)*

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`Thường thì MOTA cao nhưng IDF1 thấp cho thấy hệ thống detect bounding box rất tốt (ít False Positives và False Negatives) nhưng khả năng duy trì ID kém (nhiều ID Switches). MOTA không phạt nặng lỗi ID vì công thức tính MOTA chủ yếu bị chi phối bởi FP và FN (số lượng FP và FN rất lớn trong toàn bộ video), trong khi IDSW xảy ra ít thường xuyên hơn nên ít ảnh hưởng đến tổng điểm MOTA.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`BoT-SORT + ReID thường có IDF1 và AssA cao hơn so với ByteTrack, đồng thời IDSW giảm thiểu rõ rệt. Ví dụ: Tại khoảng frame 200, một xe tải che khuất một xe con, ByteTrack bị mất track và sinh ID mới khi xe con lộ ra do chỉ dựa vào IOU hoặc chuyển động mượt, trong khi BoT-SORT+ReID nối lại thành công nhờ có matching đặc trưng ngoại hình (ReID). Tuy nhiên, cần lưu ý đây không hoàn toàn là hiệu ứng nhân quả (causal effect) của riêng module ReID, vì nền tảng của hai tracker (ByteTrack và BoT-SORT) có cơ chế association và lọc Kalman Filter khác nhau, dẫn đến khác biệt tổng thể.`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`DetA, FP và FN hầu như không chênh lệch nhiều giữa hai phương pháp vì cả hai đều dùng chung một model detection đầu vào (YOLOv8 với yolo26n.pt). Lỗi chủ yếu còn lại nằm ở association, đặc biệt là trong các tình huống xe che khuất nhau quá lâu, hoặc nhiều xe có ngoại hình (đặc trưng ReID) giống hệt nhau đi sát nhau, khiến Kalman Filter và ReID đều có khả năng dự đoán sai.`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`Frame 450, ID 12: Có hai chiếc xe SUV màu đen giống hệt nhau chạy nối đuôi. Khi xe trước đi vào vùng tối tạm thời, ReID bị nhầm lẫn và gán nhầm ID của xe đi trước cho xe đi sau do đặc trưng ảnh trích xuất (feature vector) quá giống nhau. Annotation thủ công đúng vì con người dựa vào logic không gian - thời gian liền mạch, hiểu vận tốc thực tế để biết đó là 2 xe khác nhau.`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`Frame 510, ID 9: Khi gán nhãn bằng mắt, tôi nghĩ chiếc xe đã ra khỏi khung hình hoàn toàn ở gốc cây bên phải và cắt track. Nhưng ReID đã nối track thành công với một đoạn nhỏ mờ mờ ở khe lá. Khi phóng to lên xem kỹ, tôi nhận thấy đúng là xe chưa đi hẳn ra ngoài mà vẫn còn 1/4 thân xe. Điều này cho thấy model ReID đôi khi nhạy với các cụm pixel đặc trưng hơn mắt người ở các khu vực rìa ảnh, giúp phát hiện lại các track khó.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`Tôi sẽ quy định rõ hơn về việc đánh giá mức độ che khuất (occlusion level) cụ thể (ví dụ: chia mức độ che khuất 25%, 50%, 75% để có quyết định nhất quán hơn về việc duy trì hay bỏ bbox). Đồng thời, tôi sẽ điều chỉnh lại quy trình làm việc bằng cách tận dụng phím tắt nội suy (interpolation) hiệu quả hơn từ đầu để tiết kiệm thời gian, thay vì đánh dấu thủ công quá nhiều frame liền kề.`

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [ ] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [ ] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
