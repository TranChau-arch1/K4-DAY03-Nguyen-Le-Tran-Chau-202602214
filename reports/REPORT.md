# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Nguyễn Lê Trân Châu`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `...` |
| Thời gian gán `clip_02` (warm-up) | `50` phút |
| Thời gian gán `clip_01` | `45` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `6` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `Xe che khuất lẫn nhau dẫn đến nguy cơ tráo ID`
2. `Lệch nội suy do chuyển động phi tuyến`
3. `...`

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `Bắt được 1 trường hợp xe bị che khuất một phần (ID 5) suýt bị tạo nhầm thành track mới, đã dùng phím Merge`
- Lượt 2: `Phát hiện xe ID 3 (ở frame 45) và ID 1 (ở frame 11) sau khi rời khỏi viền ảnh vẫn còn 1 frame bbox treo vô hình do chưa bấm Outside đã bổ sung bấm phím `O``
- Lượt 3: `Phát hiện hiện tượng trôi dạt nội suy (interpolation drift) khiến bbox bị hở mép đuôi xe ở frame 85 và 120`

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `a244e20cd119ebb1cae02abb9b26c34c8f05b780b23ef106c86e175538d6ab7a` |
| Thời điểm khóa | `19h30 - 19h50` |
| Số row / frame / track trước khi mở reference | `613 / 190 / 8` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.814 | 0.799 | 0.830 | 0.886 | 0.951 | 0.899 | 0.877 | 49 | 9 | 0 |
| Sau rework | 0.816 | 0.801 | 0.832 | 0.891 | 0.951 | 0.902 | 0.882 | 31 | 25 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox thừa trước khi đối tượng xuất hiện | 79–100 | 6 | Bấm Outside tại đúng frame xe bắt đầu xuất hiện/rời khung |
| Bbox thừa sau khi đối tượng rời khung | 149–151 | 4 | Bấm Outside tại đúng frame xe rời khung |
| Bbox thừa trước khi đối tượng xuất hiện | 76–78 | 5 | Bấm Outside tại đúng frame xe bắt đầu xuất hiện |
| Bbox thừa sau khi đối tượng rời khung | 169–171 | 8 | Bấm Outside tại đúng frame xe rời khung |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13` |
| weights / hai tracker | `yolo26n.pt / ByteTrack (bytetrack.yaml), BoT-SORT + ReID (botsort-reid.yaml)` |
| conf / IoU / imgsz / classes | `0.25 / 0.70 / 960 / [2, 5, 7]` |
| device | `0 (CUDA)` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.816 | 0.801 | 0.832 | 0.891 | 0.951 | 0.902 | 0.882 | 31 | 25 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.783 | 0.725 | 0.848 | 0.907 | 0.887 | 0.767 | 0.900 | 96 | 37 | 2 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`MOTA của mình thấp hơn IDF1 (0.902 < 0.951). Điều này cho thấy annotation có ít lỗi phát hiện nhưng vẫn cần đánh giá khả năng duy trì đúng ID. MOTA không phạt nặng lỗi ID vì IDSW chỉ là một thành phần nhỏ trong công thức MOTA.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`BoT-SORT + ReID có IDF1 0.900 > 0.875, AssA 0.820 > 0.776, nhưng IDSW đều = 2. Ví dụ ở frame 87, track 5 bị đổi ID 17 → 18 trong ReID; ByteTrack có lỗi tương tự ở frame 94, track 5 (23 → 32). ReID tốt hơn về association tổng thể, nhưng không thể kết luận causal effect riêng của ReID vì hai tracker implementation khác nhau.`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`ReID có DetA 0.711 > 0.649, nhưng FP 91 > 88 và FN 26 < 54 so với ByteTrack. Vì vậy lỗi còn lại nghiêng nhiều hơn về association/ID tracking, dù detector vẫn còn một số lỗi FP/FN`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`Frame 87, track 5: ReID đổi ID 17 → 18, trong khi annotation của mình giữ đúng track 5. Đây là trường hợp mình đúng, ReID sai`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`Frame 113, track 6, ReID đổi ID 24 → 31. Evidence cho thấy annotation của mình giữ track ổn định và IDSW của mình = 0, nên không có đủ bằng chứng để sửa annotation theo lỗi của model.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`Trong quy trình, mình sẽ gán theo từng clip → kiểm tra track continuity → chạy metric/gate → xem các frame có lỗi → sửa annotation → khóa pre-gold snapshot trước khi chuyển sang clip tiếp theo`

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
