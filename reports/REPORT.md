# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Lê Đức Tú - 2A202602045`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | 10 phút |
| Thời gian gán `clip_01` | 20 phút |
| Số track đã vẽ trong `clip_01` | 8 |
| Số keyframe trung bình mỗi track | 45 |

Ba tình huống khó:

1. Bbox trôi ở frame 39, track 3; cần thêm keyframe quanh đoạn này.
2. Bbox trôi ở frame 82, track 5; cần thêm keyframe khi xe thay đổi chuyển động.
3. Bbox trôi liên tiếp ở frame 100–103, track 5; cần đặt keyframe dày hơn thay vì chỉ nội suy dài.

## 2. Tự kiểm và kiểm chéo

- Lượt 1 (ID): Chưa có log kiểm chéo.
- Lượt 2 (frame đầu/cuối): Chưa có log kiểm chéo.
- Lượt 3 (frame giữa): Chưa có log kiểm chéo.

Chưa có `reports/review_partner.md`, nên chưa thể báo cáo người kiểm chéo và số lỗi hai bên tìm được.

## 3. Pre-gold lock và chấm trước/sau rework

Đã có `evidence/pre-gold/clip_01/manifest.json` và snapshot `gt.txt`. Manifest ghi SHA-256 `21401b0b3834f5dafdaae9a406a96559258aaac3a8416ca55511bae5946a9c2f`, khóa lúc `2026-09-15T03:37:19.760310+00:00`, với 562 row, 190 frame và 8 track. SHA-256 của annotation hiện tại trùng snapshot pre-gold, nên chưa có thay đổi sau khi khóa.

| Bản | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Pre-gold | 0.789 | 0.779 | 0.800 | 0.861 | 0.960 | 0.921 | 0.846 | 17 | 28 | 0 |
| Bản hiện tại | 0.789 | 0.779 | 0.800 | 0.861 | 0.960 | 0.921 | 0.846 | 17 | 28 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**.

| Loại lỗi | Frame | ID | Đã sửa / cần sửa |
| --- | --- | --- | --- |
| Bbox trôi | 39 | 3 | Thêm keyframe quanh frame 39 |
| Bbox trôi | 82 | 5 | Thêm keyframe quanh frame 82 |
| Bbox trôi | 100–103 | 5 | Thêm keyframe dày hơn |

## 4. Kết quả model: ByteTrack control vs ReID treatment

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13 |
| Weights / tracker | `yolo26n.pt`; `bytetrack.yaml`; `botsort-reid.yaml` |
| conf / IoU / imgsz / classes | 0.25 / 0.70 / 960 / `[2, 5, 7]` |
| Device | GPU `0` |
| Persist | `True` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Cá nhân tôi vs gold | 0.789 | 0.779 | 0.800 | 0.861 | 0.960 | 0.921 | 0.846 | 17 | 28 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.731 | 0.677 | 0.790 | 0.851 | 0.888 | 0.765 | 0.834 | 103 | 27 | 2 |

## 5. Phân tích

### 1. MOTA và IDF1

MOTA của tôi thấp hơn IDF1 (`0.921 < 0.960`). IDSW bằng 0, nên lỗi còn lại chủ yếu là FP, FN và bbox. MOTA chỉ phạt một lần cho mỗi ID switch, còn IDF1 đo tính nhất quán ID trên toàn bộ track nên nhạy hơn với lỗi định danh.

### 2. ByteTrack và ReID

ReID cải thiện IDF1 từ `0.875` lên `0.900` và AssA từ `0.776` lên `0.820`; IDSW vẫn là 2. Với track gold 5, ByteTrack đổi ID ở frame 94 (`23 → 32`), còn ReID vẫn đổi ID ở frame 87 (`17 → 18`). Vì vậy ReID cải thiện tổng thể nhưng chưa loại bỏ được ID switch. Đây là so sánh hệ thống, không cô lập tác động riêng của ReID vì hai tracker có implementation khác nhau.

### 3. DetA, FP và FN

ReID tăng DetA từ `0.649` lên `0.711`, giảm FN từ 54 xuống 26, nhưng FP tăng nhẹ từ 88 lên 91. Lỗi còn lại gồm cả detection/bbox và association: vẫn có 2 IDSW, các bbox lệch, track thừa và track mất đoạn.

### 4. Một chỗ tôi đúng và ReID sai

ReID tạo track ID 7 từ frame 16–116 trong 43 frame nhưng không khớp track nào trong gold. Đây là false positive; việc tôi không gán track này là đúng.

### 5. Một chỗ ReID khiến tôi xem lại annotation

Ở frame 101, track gold 5, bbox của tôi chỉ đạt IoU 0.54 với gold; sai khác giữa ReID và nhãn tay cũng xuống khoảng 0.50. Tôi cần thêm keyframe quanh frame 100–103 để tránh bbox trôi.

## 6. Nếu phải gán thêm 10 clip nữa

Tôi sẽ bổ sung vào `GUIDELINE_MINI.md` quy tắc đặt keyframe dày khi xe đổi tốc độ, đổi hướng hoặc bị che; kiểm tra cửa sổ ±5 frame quanh các đoạn đó; và giữ ID nhất quán khi xe bị che ngắn hạn hoặc cắt nhau.

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
