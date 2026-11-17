# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 4
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 193 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 193 | 0.844 | 98.4% | 62.2% | 89.1% | 178/53 | 9 | CHƯA ĐẠT |
| 101|102 | 6 | 24 | 0.869 | 95.8% | 79.2% | 100.0% | 3/5 | 0 | ĐẠT |
| 101|103 | 9 | 48 | 0.854 | 100.0% | 64.6% | 89.6% | 74/7 | 9 | ĐẠT |
| 101|104 | 5 | 29 | 0.807 | 100.0% | 51.7% | 89.7% | 14/14 | 0 | CHƯA ĐẠT |
| 102|103 | 5 | 25 | 0.892 | 100.0% | 40.0% | 68.0% | 26/3 | 0 | ĐẠT |
| 102|104 | 9 | 47 | 0.818 | 100.0% | 76.6% | 91.5% | 57/8 | 0 | CHƯA ĐẠT |
| 103|104 | 6 | 20 | 0.843 | 90.0% | 45.0% | 95.0% | 4/16 | 0 | CHƯA ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 478 ({'iou_thap': 73, 'sai_thuoc_tinh': 84, 'bo_sot_thua': 134, 'co_can_review': 56, 'bat_dong_ignore': 128, 'sai_lop': 3}), trên 34 ảnh cần calibration.
