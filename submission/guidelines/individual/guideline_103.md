# Guideline gán nhãn cá nhân — v0.1 — Thành viên 103

- **Phiên bản:** v0.1 (Bản cá nhân)
- **Ngày:** 2026-11-16
- **Người viết:** Thành viên 103
- **Phạm vi:** 40 ảnh data/images/img_001.jpg–img_040.jpg; Bounding box trong CVAT và xuất COCO 1.0.

---

## 1. Mục tiêu

Tập dữ liệu dùng để huấn luyện mô hình phát hiện đối tượng 2D (2D Object Detection) cho hệ thống thị giác máy tính trên phương tiện tự hành và camera giao thông thông minh trong môi trường đô thị phức tạp.

- **Đối tượng nhận diện:** Người đi bộ, người điều khiển phương tiện, xe con, xe tải, xe van, xe máy.
- **Rủi ro an toàn:**
  - Bỏ sót người đi bộ hoặc người đi xe máy (`person`, `motorcycle`) là lỗi đặc biệt nghiêm trọng có thể dẫn tới va chạm thực tế.
  - Nhận diện nhầm ma-nơ-canh (`img_033.jpg`), tượng điêu khắc (`img_017.jpg`, `img_027.jpg`) hoặc hình in trên pano quảng cáo (`img_018.jpg`) sẽ dẫn tới hiện tượng phanh gấp không cần thiết (phantom braking).
  - Bounding box bị lỏng hoặc cắt xén thân xe gây ước lượng sai khoảng cách và thời gian chạm (TTC).

---

## 2. Lớp và thuộc tính

### 2.1. Lớp đối tượng

| Lớp | Định nghĩa | Ví dụ | Phản ví dụ / Dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con chở người từ 4–9 chỗ (sedan, SUV, hatchback, wagon, crossover). | Xe sedan trong mưa (`img_015.jpg`), xe wagon (`img_024.jpg`). | Xe bán tải (`truck`), xe van (`van`), xe đồ chơi (`img_038.jpg`). |
| `truck` | Phương tiện cơ giới chuyên chở hàng hóa có cabin riêng và thùng/sàn chở hàng; xe đầu kéo; xe bán tải (pickup truck). | Xe chở xe ô tô (`img_002.jpg`), xe tải giao hàng (`img_007.jpg`), xe bán tải chở hàng (`img_012.jpg`), xe tải nhỏ (`img_036.jpg`). | Xe con chở hàng trong cốp (`car`), xe van thùng liền (`van`). |
| `van` | Xe chở người hoặc hàng dạng thân liền khối (unibody), trần cao đuôi vuông; xe cơ giới 3 bánh chở khách (Tuk-tuk, Auto-rickshaw). | Xe Ford Transit, xe tuk-tuk (`img_004.jpg`, `img_035.jpg`). | Xe tải thùng rời (`truck`), xe SUV (`car`). |
| `motorcycle` | Xe 2 bánh gắn máy (xe máy, xe mô tô phân khối lớn); xe máy có thùng phụ (sidecar). | Xe gắn máy đường phố (`img_005.jpg`, `img_026.jpg`), xe sidecar (`img_022.jpg`). | Xe đạp thông thường, xe xích lô (`img_006.jpg`). |
| `person` | Con người còn sống thuộc mọi lứa tuổi, tư thế (đi bộ, đứng, ngồi, người điều khiển/ngồi sau xe máy, xe thô sơ); người mặc trang phục mascot. | Người bán hàng (`img_003.jpg`, `img_010.jpg`), người đi xe máy (`img_005.jpg`), người hóa trang (`img_011.jpg`). | Tượng người (`img_017.jpg`), ma-nơ-canh (`img_033.jpg`), hình người trên quảng cáo (`img_018.jpg`). |
| `ignore_region` | Vùng bỏ qua: khu vực không gán từng đối tượng đơn lẻ, không tính lỗi bỏ sót. | Đám đông/xe ở xa có chiều cao < 20px (`img_008.jpg`, `img_025.jpg`), xe buýt (`img_032.jpg`), tủ kính đồ chơi (`img_038.jpg`). | Không dùng để che lấp các ca bất đồng có thể giải quyết được. |

#### Xử lý đối tượng ngoài taxonomy:
- Xe đạp, xích lô: KHÔNG gán box cho xe; người đạp xe/ngồi trên xe vẫn gán `person`.
- Xe buýt: Gán box `ignore_region` vì taxonomy không có lớp `bus`.
- Động vật, tượng, đồ chơi, hình in: Không gán box vào các lớp thật.

### 2.2. Thuộc tính

1. **`occluded_level` (Mức độ che khuất):**
   - `0`: Không che khuất (< 10% diện tích bị che).
   - `1`: Che nhẹ (10% - 35% diện tích bị che).
   - `2`: Che trung bình (35% - 70% diện tích bị che).
   - `3`: Che nặng (> 70% diện tích bị che nhưng còn nhận biết chắc chắn).
2. **`truncated` (Cắt mép ảnh):**
   - `no`: Vật thể trọn vẹn trong khung hình.
   - `yes`: Bị cắt ngang bởi ít nhất một cạnh mép ảnh.
3. **`can_review` (Cần xem xét lại):**
   - `false`: Đã rõ ràng.
   - `true`: Trường hợp mơ hồ, chưa chắc chắn, cần đưa ra phiên họp nhóm.

---

## 3. Quy tắc vẽ box

1. **Ôm sát phần nhìn thấy:** Bounding box phải bám khít các điểm ảnh biên nhìn thấy, không vẽ bao trùm khoảng không vô hình bị che khuất.
2. **Người và xe máy (R-03):** Tách 2 box riêng biệt. 1 box `motorcycle` ôm trọn xe (kể cả bánh xe, ghi đông, gương), 1 box `person` ôm trọn người lái từ đỉnh đầu/mũ bảo hiểm xuống chân. Xe chở 2 người thì có 1 box `motorcycle` và 2 box `person`.
3. **Xe chở xe (R-01):** Xe chuyên chở là 1 box `truck`. Mỗi xe con trên giá được gán 1 box `car` riêng với `occluded_level` phù hợp.
4. **Hàng hóa trên xe (R-15, R-16):** Hàng gắn chặt trên thùng xe tải hoặc sau xe máy thì tính vào bounding box của phương tiện đó.
5. **Vật quá nhỏ (R-12):** Đối tượng cao < 20 pixels không gán box đơn lẻ; gán `ignore_region` nếu là đám đông/vùng xa.
6. **Phản chiếu, tượng, ma-nơ-canh, hình in quảng cáo:** Tuyệt đối không gán nhãn người hoặc phương tiện thật.

---

## 4. Ca khó và đề xuất xử lý từ 40 ảnh

| Ảnh | Tình huống quan sát | Rule ID | Đề xuất xử lý | Lý do |
|---|---|---|---|---|
| `img_001.jpg` | Xe bán tải đỗ trước quán ăn | `R-06` | Gán `truck` cho xe bán tải; gán `person` cho thực khách. | Xe bán tải có cabin và thùng chở hàng riêng biệt. |
| `img_002.jpg` | Xe lồng chở nhiều ô tô con | `R-01` | 1 box `truck` cho xe chở; từng box `car` cho mỗi xe con trên giá. | Xe con trên giá vẫn là phương tiện giao thông hoàn chỉnh. |
| `img_003.jpg` | Người bán mít bên xe đạp | `R-04` | Gán `person` cho người bán; không gán xe đạp. | Taxonomy không có lớp `bicycle`. |
| `img_004.jpg` | Xe Tuk-tuk đón khách | `R-02` | Gán `van` cho mỗi xe tuk-tuk. | Xe cơ giới 3 bánh chở khách có mái che phù hợp nhóm `van`. |
| `img_005.jpg` | Dòng người lái xe máy TP.HCM | `R-03` | Tách riêng 1 box `motorcycle` và 1 box `person` cho mỗi người lái. | Chuẩn hóa quy tắc tách người và phương tiện. |
| `img_006.jpg` | Đoàn xích lô tại Huế | `R-04` | Gán `person` cho người đạp và du khách; không gán xích lô. | Xích lô là phương tiện ngoài taxonomy. |
| `img_008.jpg` | Chợ đêm đông người ở xa | `R-12` | Gán box cho người gần; khoanh `ignore_region` cho đám đông xa < 20px. | Tuân thủ ngưỡng `min_box_height_px = 20`. |
| `img_011.jpg` | Người hóa trang Predator bên mô tô | `R-05` | Gán `person` cho nhân vật; gán `motorcycle` cho xe. | Người thật mặc đồ hóa trang có khả năng di chuyển. |
| `img_012.jpg` | Xe bán tải chất đầy nông sản | `R-15` | Gán `truck` bao toàn bộ xe và sọt hàng trên thùng. | Hàng hóa gắn liền với xe tạo thành kích thước vật cản thực tế. |
| `img_013.jpg` | Mặt đường mưa phản chiếu ánh đèn | `R-07` | Không gán box cho bóng phản chiếu dưới nước. | Tránh hiện tượng false positive phanh gấp. |
| `img_017.jpg` | Tượng Duke of Wellington cưỡi ngựa | `R-08` | Không gán box cho tượng người hay ngựa. | Tượng đá tĩnh, không phải thực thể sống. |
| `img_018.jpg` | Pano quảng cáo chính trị gia | `R-09` | Không gán `person` cho ảnh in trên biển quảng cáo. | Ảnh in 2D không tham gia giao thông. |
| `img_019.jpg` | Xe diễu hành mô hình Dangermouse | `R-10` | Gán `car` cho xe mô hình diễu hành có động cơ. | Xe kích thước thật lưu thông trên đường. |
| `img_020.jpg` | Ảnh phản chiếu qua gương chiếu hậu | `R-07` | Không gán box cho hình ảnh trong gương xe. | Ảnh ảo trong gương không phải chướng ngại vật phía trước. |
| `img_022.jpg` | Xe mô tô có thùng phụ (Sidecar) | `R-11` | 1 box `motorcycle` bao xe và thùng phụ; 2 box `person` cho 2 người. | Thùng phụ gắn liền cơ học với khung xe máy. |
| `img_026.jpg` | Nam nữ đi xe máy trong hầm | `R-03` | 1 box `motorcycle` cho xe; 2 box `person` cho 2 người. | Nhận diện độc lập từng người tham gia giao thông. |
| `img_027.jpg` | Bia mộ điêu khắc hình xe tải | `R-13` | Không gán `truck` cho bia mộ đá. | Khối đá điêu khắc trong nghĩa trang. |
| `img_028.jpg` | Xe tải chở tượng ngựa | `R-14` | Gán `truck` bao gồm cả tượng ngựa; không gán tượng ngựa. | Tượng là hàng hóa chuyên chở trên thùng xe. |
| `img_029.jpg` | Xe máy chở kiện hàng cồng kềnh | `R-16` | Box `motorcycle` bao trọn xe và kiện hàng; box `person` cho người lái. | Khối hàng cồng kềnh gắn chặt và cản trở không gian chuyển động. |
| `img_032.jpg` | Xe buýt đỗ ở trạm | `R-18` | Vẽ 1 box `ignore_region` bao quanh xe buýt. | Không có lớp `bus` trong taxonomy, tránh gán sai lớp. |
| `img_033.jpg` | Ma-nơ-canh trong tiệm thời trang | `R-17` | Không gán `person` cho ma-nơ-canh. | Ma-nơ-canh trong tủ kính không phải người thật. |
| `img_035.jpg` | Xe Auto-rickshaw chở học sinh | `R-18` | Gán `van` cho thân xe; gán `person` cho từng học sinh thò người ra ngoài. | Đảm bảo tính toán đúng vị trí người có nguy cơ va chạm. |
| `img_036.jpg` | Xe tải nhỏ CMC Veryca | `R-20` | Gán `truck` cho xe tải nhỏ. | Xe có cabin và thùng kim loại riêng biệt. |
| `img_037.jpg` | Xe bán tải có nắp thùng kín | `R-21` | Gán `truck` cho xe bán tải có mui cứng. | Cấu tạo khung gầm xe tải nhẹ. |
| `img_038.jpg` | Tủ kính đồ chơi ô tô, búp bê | `R-22` | Vẽ `ignore_region` bao quanh tủ kính đồ chơi. | Đồ chơi trong tủ trưng bày không phải phương tiện thật. |

---

## 5. QC và đề xuất cho nhóm

- Đề xuất áp dụng ngưỡng IoU $\ge 0.85$ với box lớn và $\ge 0.70$ với box nhỏ.
- Tất cả các ca annotator còn băn khoăn phải bật `can_review = true` để đưa ra phiên calibration.
- Đề xuất đưa các quy tắc ca khó trên vào bản `guideline_v1.0_agreed.md` chung của cả nhóm.
