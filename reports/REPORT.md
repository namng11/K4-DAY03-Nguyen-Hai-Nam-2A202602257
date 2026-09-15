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
| Bản pre-gold | - | - | - | - | - | - | - | - | - | - |
| Sau rework | 0.8337 | 0.8213 | 0.8473 | 0.8885 | 0.9716 | 0.9424 | 0.8768 | 25 | 8 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

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
| bạn vs gold | 0.8337 | 0.8213 | 0.8473 | 0.8885 | 0.9716 | 0.9424 | 0.8768 | 25 | 8 | 0 |
| ByteTrack control vs gold | 0.7085 | 0.6487 | 0.7761 | 0.8463 | 0.8746 | 0.7487 | 0.8226 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.7635 | 0.7110 | 0.8204 | 0.8721 | 0.9001 | 0.7923 | 0.8595 | 91 | 26 | 2 |
| ReID vs bạn | 0.7699 | 0.7085 | 0.8398 | 0.8830 | 0.9007 | 0.7966 | 0.8684 | 83 | 35 | 2 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`MOTA của tôi (0.9424) thấp hơn IDF1 (0.9716). Thường thì nếu MOTA cao nhưng IDF1 thấp cho thấy hệ thống detect bounding box rất tốt (ít FP/FN) nhưng khả năng duy trì ID kém. MOTA không phạt nặng lỗi ID vì công thức tính MOTA chủ yếu bị chi phối bởi FP và FN (xảy ra rất nhiều ở mọi frame), trong khi IDSW xảy ra ít thường xuyên hơn nên ít ảnh hưởng đến tổng điểm MOTA.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`BoT-SORT + ReID có IDF1 (0.9001) và AssA (0.8204) cao hơn ByteTrack (IDF1: 0.8746, AssA: 0.7761), số lượng IDSW thì như nhau (2 lỗi). Ví dụ: Tại khoảng frame 87-113, ReID nối lại track tốt hơn cho ID bị che khuất một phần nhờ matching đặc trưng ngoại hình. Tuy nhiên, cần lưu ý đây không hoàn toàn là hiệu ứng nhân quả (causal effect) của riêng module ReID, vì nền tảng của hai tracker (ByteTrack và BoT-SORT) có cơ chế association và lọc Kalman Filter khác nhau.`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`Giữa ByteTrack và BoT-SORT + ReID, DetA tăng từ 0.6487 lên 0.7110. FN giảm mạnh (từ 54 xuống 26), trong khi FP tăng nhẹ (88 lên 91). Cả hai đều dùng chung model detection đầu vào (yolo26n.pt), nhưng cơ chế match của BoT-SORT giữ lại nhiều bounding box thật hơn (tăng True Positives). Lỗi còn lại vẫn chia đều cho cả detector (do model yolo nhỏ nên sót xe xa) và association (IDSW = 2).`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`ReID so với nhãn của tôi có 35 FN và 83 FP. Có những chiếc xe bị khuất nhẹ mà tôi gán nhưng ReID không thể tracking liên tục và đánh dấu là FP. Điều này do con người hiểu context không gian liền mạch tốt hơn model chỉ phụ thuộc vào ReID embeddings.`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`ReID tìm được các "ghost_pred_tracks" hoặc "fragmented_tracks" mà tôi có thể đã gán thiếu ở các frame đầu khi xe vừa vào khung hình. Tuy nhiên, một số ghost_pred_tracks có reason "không khớp track tham chiếu nào" cũng minh chứng rằng ReID đôi khi tạo track ảo cho bóng cây, điều này khẳng định annotation của tôi (lọc kỹ bằng mắt) là chính xác.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`Tôi sẽ quy định rõ hơn về việc đánh giá mức độ che khuất cụ thể (ví dụ: chia che khuất 25%, 50% để có quyết định nhất quán hơn về việc duy trì hay bỏ bbox). Đồng thời, tôi sẽ tận dụng phím tắt nội suy (interpolation) hiệu quả hơn từ đầu để tiết kiệm thời gian gán.`

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
- [ ] `reports/review_partner.md` (Làm cá nhân độc lập)
- [x] `reports/REPORT.md` (file này)
