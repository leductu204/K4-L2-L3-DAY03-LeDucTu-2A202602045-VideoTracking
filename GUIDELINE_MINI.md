# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Lê Đức Tú`
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
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới ... frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | `...` |
| Xe bị che lâu hơn ngưỡng trên | `...` | `...` |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | `...` |
| Hai xe cắt nhau / chồng lên nhau | `...` | `...` |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `...` |
| Xe đang đỗ, không di chuyển | `...` |
| Keyframe đặt dày ở đâu | `...` |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / frame 39 / track 3`
- Tình huống: `BBox bị trôi giữa các keyframe, IoU với gold còn 0.51.`
- Quyết định: `Thêm keyframe quanh frame 39 và chỉnh bbox ôm sát xe.`
- Lý do: `Nội suy dài làm bbox lệch khỏi vật thể.`

### Ca 2
- Clip / frame / ID: `clip_01 / frame 82 / track 5`
- Tình huống: `BBox của xe bị lệch, IoU với gold chỉ 0.50.`
- Quyết định: `Thêm keyframe trước và sau frame 82; giữ nguyên ID 5.`
- Lý do: `Xe vẫn là cùng một đối tượng, lỗi nằm ở vị trí bbox chứ không phải định danh.`

### Ca 3
- Clip / frame / ID: `clip_01 / frame 100–103 / track 5`
- Tình huống: `BBox trôi liên tiếp trong nhiều frame khi xe thay đổi chuyển động.`
- Quyết định: `Đặt keyframe dày hơn trong đoạn 100–103 và kiểm tra lại nội suy.`
- Lý do: `Một keyframe đơn lẻ không đủ giữ bbox chính xác qua đoạn chuyển động nhanh.`
## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Không có gì mơ hồ nữa cả.
