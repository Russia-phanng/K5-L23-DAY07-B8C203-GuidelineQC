# Guideline gán nhãn — <0.1> — <101>

> Copy file này (hoặc `guideline_v0_starter.md`) rồi viết. Một guideline đủ gồm **5 phần**. Câu nào của v0
> làm hai người đọc ra hai cách khác nhau thì phải viết lại cho một cách hiểu duy nhất.

- Phiên bản: v0.1
- Ngày: 16/11/2026
- Người viết / duyệt: Nguyễn Phương Nga

## 1. Mục tiêu
Mô hình sử dụng: Mô hình phát hiện vật thể 2D (Object Detection) phục vụ hệ thống nhận thức môi trường xung quanh cho xe tự hành / ADAS (Advanced Driver-Assistance Systems) và phân tích lưu lượng giao thông đô thị.
Mục đích sử dụng nhãn: Giúp mô hình nhận diện, phân loại và định vị chính xác các phương tiện cơ giới và người tham gia giao thông trên đường để dự đoán quỹ đạo, giữ khoảng cách an toàn và cảnh báo va chạm.
Lỗi nghiêm trọng cần tuyệt đối tránh:
Bỏ sót (False Negative) person và motorcycle: Gây nguy cơ tai nạn trực tiếp do hệ thống không phát hiện người đi bộ sang đường hoặc xe máy tạt đầu.
Nhầm lẫn kích thước phương tiện (car vs truck/van): Sai lệch trong việc ước lượng khoảng cách phanh an toàn và điểm mù của xe lớn.
Vẽ box quá lỏng hoặc lệch tâm: Làm sai lệch tọa độ tâm vật thể, khiến thuật toán tính sai vận tốc tương đối.

## 2. Lớp và thuộc tính
### Lớp
| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Phương tiện cơ giới 4 bánh chở khách cỡ nhỏ đến trung bình có khoang hành khách liền mạch. | Sedan, hatchback, SUV, crossover, taxi, xe bán tải (pickup truck) không có thùng hàng chuyên dụng; ví dụ img_015 | Không phải xe van 16 chỗ (van), xe tải nhỏ (truck), hoặc xe ba gác. |
| `truck` | Phương tiện cơ giới chuyên dụng để chở hàng hóa có khoang chở hàng tách biệt hoặc kích thước lớn. | Xe tải thùng kín, xe tải ben, xe container (đầu kéo + rơ-moóc), xe bồn, xe chở rác, xe cứu hỏa; ví dụ img_036 | Không phải xe bán tải (car) hay xe van chở hàng cỡ nhỏ (van). |
| `van` | Phương tiện cơ giới hình hộp có khoang chở khách hoặc chở hàng liền khối với cabin, lớn hơn xe con thông thường. | Xe khách 12–16 chỗ (Ford Transit, Hyundai Solati), xe minivan (Kia Carnival), xe tải van cỡ nhỏ (Suzuki Blind Van); ví dụ img_005 | Không phải SUV 7 chỗ (car) hay xe bus cỡ lớn 29–45 chỗ. |
| `motorcycle` | Phương tiện cơ giới 2 hoặc 3 bánh có động cơ dùng để di chuyển trên đường bộ. | Xe số, xe tay ga, xe côn tay, xe mô tô phân khối lớn, xe máy điện, xe ba gác máy; ví dụ img_006 | Không phải xe đạp thường, xe đạp điện có bàn đạp nhỏ (nếu không thuộc scope), hoặc người lái xe (person). |
| `person` | Con người đang tham gia giao thông hoặc xuất hiện trong khu vực đường đi, vỉa hè. | Người đi bộ, người điều khiển xe máy (motorcycle), người ngồi sau xe máy, công nhân làm đường, CSGT; ví dụ img_014 | Không phải ma-nơ-canh, hình người in trên biển quảng cáo/thân xe bus, tượng đài. |
| `ignore_region` | Vùng đa giác/chữ nhật bao quanh khu vực có mật độ vật thể quá dày đặc hoặc quá nhỏ không thể tách lẻ chính xác. | Bãi đỗ xe máy san sát ở xa, đám đông người đi bộ ở tít cuối đường, khu vực bị lóa/mờ nặng; ví dụ đám đông img_032 | Không dùng cho 2–3 xe đi gần nhau mà mắt thường vẫn phân tách được từng chiếc. |

Quy tắc xử lý vật ngoài danh sách lớp:
Các phương tiện không nằm trong 5 lớp trên (ví dụ: xe bus lớn trên 29 chỗ, xe đạp, xe xích lô, xe lăn, chướng ngại vật/cọc tiêu): Không gán nhãn (bỏ qua hoàn toàn) nếu đứng riêng lẻ.
Nếu một cụm nhiều vật thể ngoài lớp hoặc vật thể thuộc lớp nhưng quá nhỏ/nhòe tạo thành một đám rối không thể phân định ranh giới từng đối tượng: Phủ một box ignore_region bao trùm toàn bộ cụm đó để mô hình không bị phạt oan (False Positive) khi dự đoán trúng.

### Thuộc tính

Áp dụng cho 5 lớp đối tượng chính (`car`, `truck`, `van`, `motorcycle`, `person`). Lớp `ignore_region` không có thuộc tính.

1. **`occluded_level`** (Mức độ bị vật thể khác trong ảnh che khuất):
   * **`0` (Mặc định):** Không bị che khuất, hoặc bị che **dưới 10%** diện tích toàn bộ vật thể. Nhìn rõ toàn vẹn hình dáng (ví dụ: xe tải đầu kéo ở tiền cảnh `img_002.jpg`, hai người đi bộ chính diện ở `img_014.jpg`).
   * **`1` (Che nhẹ):** Bị che từ **10% đến 30%** diện tích. Vẫn nhận diện dễ dàng cấu trúc chính và hướng di chuyển (ví dụ: xe con màu xanh dương và trắng xếp ở tầng trên xe lồng `img_002.jpg`, người ngồi phía trước trên xe máy `img_006.jpg`).
   * **`2` (Che vừa):** Bị che từ **trên 30% đến 60%** diện tích. Mất một phần lớn thân xe hoặc cơ thể nhưng vẫn đủ đặc trưng để xác định chính xác lớp (ví dụ: các xe con nằm ở tầng dưới của xe lồng `img_002.jpg`, các xe máy đỗ san sát ở phía sau `img_008.jpg`).
   * **`3` (Che nặng):** Bị che **trên 60% đến dưới 85%** diện tích, chỉ lộ một góc đầu xe, bánh xe hoặc một phần nhỏ cơ thể (ví dụ: các xe máy khuất sâu trong hàng xe đỗ vỉa hè `img_008.jpg`, người đi xe máy bị che khuất bởi dòng phương tiện ở tiền cảnh `img_006.jpg`). *Lưu ý: Nếu bị che từ 85% trở lên và không thể xác định chắc chắn thực thể, bỏ qua hoặc gộp vào `ignore_region`.*

2. **`truncated`** (Mức độ vật thể bị cắt bởi 4 cạnh viền khung hình):
   * **`no` (Mặc định):** Toàn bộ vật thể nằm trọn bên trong khung ảnh, hoặc chỉ chạm nhẹ mép dưới **5%** diện tích.
   * **`yes`:** Vật thể bị cắt mất một phần (từ **5% diện tích trở lên**) do nằm ở rìa trái, phải, trên hoặc dưới của bức ảnh (ví dụ: xe bán tải chở hàng ở góc dưới bên phải `img_012.jpg`, hai ô tô đỏ và bạc bị cắt đôi ở hai mép trái/phải `img_015.jpg`, xe taxi trắng ở mép trái `img_006.jpg`).

3. **`can_review`** (Cờ đánh dấu cần QC/Leader kiểm tra lại):
   * **`false` (Mặc định):** Nhãn đã rõ ràng, tuân thủ đúng các quy tắc trong guideline.
   * **`true`:** Tích chọn khi gặp ca mơ hồ về ranh giới lớp, mức độ che khuất hoặc phạm vi gán nhãn để Leader/QC chốt phương án (ví dụ: xe tuk-tuk 3 bánh ở `img_004.jpg`, người và xe bị nhòe chuyển động mạnh ở `img_014.jpg`, các ô tô bị che sau hàng rào sắt ở `img_018.jpg`).

## 3. Quy tắc vẽ box

* **[R01] Box ôm sát phần nhìn thấy (Tight Bounding Box):** Vẽ hình chữ nhật bao khít tối đa các điểm ảnh thực tế của vật thể (tính cả gương chiếu hậu, phụ kiện gắn liền), không chừa khoảng trống thừa thãi xung quanh. **Không** vẽ đoán phần bị che khuất hoặc phần nằm ngoài khung ảnh.
* **[R02] Vật bị che khuất một phần (Occlusion):** Nếu vật thể bị vật khác chắn ngang (ví dụ: cột đèn, hàng rào sắt, hoặc xe khác) nhưng vẫn nhìn thấy các phần rời rạc của cùng một vật thể, hãy vẽ **1 box duy nhất** bao trùm toàn bộ các phần đang hiển thị.
* **[R03] Vật bị cắt mép khung ảnh (Truncation):** Nếu vật thể bị cắt mất một phần bởi cạnh biên của bức ảnh, vẽ box chạm sát đường viền biên ảnh tại vị trí cắt và đánh dấu thuộc tính `truncated = yes`.
* **[R04] Cấu trúc phức tạp / Nhiều thành phần (Compositional Objects):**
  * **Người và xe máy:** Tách riêng biệt từng box cho người và phương tiện. Vẽ 1 box `motorcycle` ôm phần xe và 1 box `person` riêng cho người lái/người ngồi sau (bao gồm cả mũ bảo hiểm và balo).
  * **Xe tải chở container / Xe lồng chở ô tô:** Vẽ **1 box duy nhất** bao trùm toàn bộ cả phần đầu kéo và phần rơ-moóc/thùng chở hàng phía sau.
  * **Hàng hóa trên xe:** Nếu hàng hóa được xếp trên xe bán tải hoặc xe tải, bao gồm cả phần hàng hóa nhô lên vào chung box của phương tiện.
* **[R05] Kích thước tối thiểu (Minimum Size):** Chỉ gán nhãn đơn lẻ khi vật thể có kích thước chiều rộng hoặc chiều cao từ **15 pixel trở lên** (trên ảnh tiêu chuẩn) và có thể nhận diện rõ ràng lớp. Các vật thể quá nhỏ ở tít xa không đủ độ chi tiết sẽ không gán đơn lẻ.
* **[R06] Mật độ dày đặc (Crowded/Dense Regions):** Khi gặp các nhóm đối tượng đan xen dày đặc ở khoảng cách xa (như bãi đỗ xe máy hoặc đám đông lớn), nếu việc tách từng box gây mất thời gian và dễ sai lệch, hãy dùng 1 vùng **`ignore_region`** phủ trọn cụm đó.

## 4. Ca khó
Loại **L** = loại theo quy tắc (tự quyết); **H** = hỏi reviewer / `can_review`.

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do | Loại |
|---|---|---|---|---|---|
| img_002 | Xe chở xe, 8 ô tô trên xe + 1 người đứng xa phải | R-09, R-04 | 1 `truck` bao cả đầu kéo và giá chở. Không vẽ 8 `car` trên giá. Người đứng cạnh xe: `person` nếu cao ≥ 20 px | Tải là phần của phương tiện; nếu vẽ riêng sẽ nhầm xe đang đỗ với xe đang chạy | **H** (cần chốt cả nhóm: nhiều người sẽ vẽ `car`) |
| img_024 | Xe nhỏ màu vàng đặt trên nóc xe đen (xe trưng bày) | R-06, R-09 | Vẽ `car` cho xe đen (Pacer). **Không** vẽ xe vàng trên nóc. Xe cam/ngà trong sảnh bảo tàng: vẽ nếu thấy rõ | Xe vàng là mô hình/tải | **L** |
| img_019 | Xe diễu hành giả hình xe thể thao, có người lái thật và hình chuột bông | R-06 | `person` cho người lái (thấy mặt). Xe: **ca khó** — nếu coi là xe đi trên đường thì `car`, nếu coi là float thì không vẽ. Tạm: **vẽ `car` và tick can_review** | Có biển số, bánh, chạy đường thật. Nhóm phải chốt "float" thuộc lớp nào | **H** |
| img_001 | Pickup Suzuki đỗ (xe tải nhỏ) + người đứng truncated phía trước + người ngồi bàn quán | R-01, R-05 | Pickup = `truck` (có thùng chở). Người đứng chắn: `person`, `occluded=1`, `truncated=yes`. Người ngồi trong quán: `person` mỗi người thấy rõ, `occluded=1–2` (bị bàn che) | Pickup nằm giữa `car` và `truck` — chọn `truck` vì có thùng hàng riêng | **H** (pickup) |
| img_007 | Gara tối: pickup trắng có thùng bọc + Isuzu | R-01 | Isuzu = `truck`; pickup trắng = `truck` (cùng quy tắc img_001). Xe phản chiếu trên sàn nước: không vẽ | Nhất quán pickup=truck | **L** (nếu chốt pickup=truck) |
| img_012 | Mui xe pickup chiếm khung hình, truncated; Mazda đang chạy phía trái | R-05 | `truck` (pickup đầu ảnh) `truncated=yes`; `car` Mazda; xe tải xa: vẽ nếu ≥ 20 px | Một phần thân xe nhìn thấy đủ nhận lớp | **L** |
| img_037 | Pickup có thùng camper kín phía sau; xe buýt xanh | R-01 | Pickup camper = `truck`. Xe buýt: **không gán** (không có lớp `bus`) và ghi cấp ảnh | Cấu hình kín như `van` nhưng cabin tách thùng → truck | **H** (bus) |
| img_009 | Minibus kem ở giữa phố | — | `van` (minibus) nếu nhóm đồng ý; ngược lại bỏ. Tạm: `van`, can_review | Taxonomy không có `bus` | **H** |
| img_005 | Ba–bốn người ngồi trên nhiều xe, sát nhau; van xám sau | R-02, R-01 | `person` cho từng người thấy rõ (`occluded=1–2`); xe máy vẽ khi thấy ≥ 30% thân; van = `van`, `occluded=2` vì bị người che | Gần như toàn bộ cảnh bị che chéo | **H** (xe máy bị che hoàn toàn) |
| img_021 | Đám đông xe máy lúc tan tầm | R-08 | Vẽ các cặp xe + người ở 2–3 hàng đầu; phần sâu bên trong → 1–2 `ignore_region` | Không tách được vật ở xa | **L** |
| img_030 | Bãi xe máy đỗ dày, 1 người đứng | R-08 | Vẽ xe gần camera tách rõ; còn lại `ignore_region`; `person` (bảo vệ áo xanh) vẽ. Poster người trên tường: không vẽ | Xe đỗ gối nhau, không đếm được | **L** |
| img_008 | Xe máy đỗ dày + xe ôm áo cam đang ngồi trên xe + người đứng | R-02, R-08 | Xe ôm gần (img_008 phải): `motorcycle` + `person`. Người đứng gần cột: `person`. Dãy xe đỗ xa: `ignore_region` | Mô hình cần học người lái thật vs xe đỗ | **L** |
| img_032 | Đám đông ~25 người chờ xe buýt, nhỏ và dày | R-08, R-04 | `ignore_region` cho cụm đông; vẽ `person` tách rõ ở hai bên nếu ≥ 20 px. Xe xanh trái: `car`, `truncated=yes` | Không tách được từng người | **L** |
| img_013 | Ban đêm: 2 người đi bộ nhỏ, xe xa, xe máy đỗ, tượng rồng | R-04, R-06 | `person` hai người nếu ≥ 20 px; xe trắng xa: `car` nếu ≥ 20 px; tượng rồng vàng: không vẽ; phản chiếu trên đường ướt: không vẽ | Ảnh tối, vật nhỏ — dễ lệch giữa 2 người | **H** (vật quanh ngưỡng 20 px) |
| img_017 | Tượng người cưỡi ngựa, đội cone | R-06 | Không vẽ gì. Cone không có lớp | Tượng = vật giả | **L** |
| img_018 | Biển quảng cáo có ảnh người; xe sau hàng rào | R-06, R-01 | Ảnh người in: không vẽ. Xe sau hàng rào: `car` + `occluded=1` | Hình in; xe bị song sắt che nhẹ | **L** |
| img_020 | Gương chiếu hậu thấy xe phía sau | R-07 | Không vẽ xe trong gương | Phản chiếu | **L** |
| img_027 | Bia mộ hình xe tải bằng đá đen | R-06 | Không vẽ | Điêu khắc | **L** |
| img_028 | Pickup cổ chở tượng ngựa trắng; Mazda đỏ phía sau | R-06 | `truck` (pickup cổ). Tượng ngựa: không có lớp. `car` đỏ phía sau: có (SUV) | Ngựa là tượng, không thuộc lớp nào | **L** |
| img_033 | Manocanh, phản chiếu kính, gương xe tải | R-06, R-07 | Không vẽ `person`; gương xe không phải vật cần gán. Có thể ghi cấp ảnh "không có vật" | Manocanh + phản chiếu | **L**; ảnh rỗng (R-10) |
| img_038 | Tủ trưng bày xe đồ chơi, vịt cao su | R-06 | Không vẽ | Mô hình thu nhỏ | **L**; ảnh rỗng |
| img_011 | Điêu khắc kim loại hình xe máy + đầu tượng lớn; vài người xa | R-06, R-04 | Không vẽ xe máy kim loại / tượng. `person` nếu ≥ 20 px (2 người trái); xe cứu thương/van xa: `van` nếu ≥ 20 px | Sculpture | **H** (xe kim loại: "có hình dạng xe máy nhưng là tác phẩm") |
| img_022 | Xe máy có sidecar, ảnh đen trắng cũ | R-02 | `motorcycle` 1 box bao cả xe + thùng bên; `person` cho người trong thùng (thấy đầu) | Sidecar là một phần xe | **H** (có nên vẽ 1 hay 2 box) |
| img_029 | Xe máy chở ống dài; người đẩy xe đạp | R-03, R-02 | `motorcycle` + `person` lái. Ống không tính vào box. Người trên xe đạp: `person`; xe đạp không gán | Vật tải nhô ra | **L** |
| img_004 | 2 xe tuk-tuk, tài xế ngồi trong | — | Tuk-tuk: không gán (không có lớp). Tài xế: `person` nếu thấy ≥ 20 px, `occluded=2` | Không thuộc lớp nào | **H** (ba bánh) |
| img_035 | Xe ba bánh chở hàng ngất ngưởng; xe máy lái ở giữa | R-03, R-02 | Xe ba bánh: không gán (hoặc `truck`: **can_review**); xe máy + người lái: gán; ô tô trắng: `car` | Xe ba bánh có thùng — mơ hồ | **H** |
| img_003 | Người gánh hàng; xe đạp; xe máy bị người che | R-01 | `person` cho từng người thấy rõ (cả người gánh); xe máy sau lưng: `motorcycle` `occluded=2`; xe đạp: không gán | Xe đạp không có lớp | **L** |
| img_023 | Xe xích lô có người đạp + ô tô + xe máy xa | — | `car` (Honda); `person` người đạp; xích lô: không gán (không có lớp); `motorcycle`+`person` xa nếu ≥ 20 px | Xích lô không phải lớp | **H** |
| img_015 | Xe mờ do chuyển động, phản chiếu mặt đường ướt | R-05, R-07 | `car` đỏ (trái, truncated), `car` bạc (phải, truncated); xe trắng + đen phía sau vẽ; phản chiếu xe trên đường: không vẽ | Motion blur vẫn là xe thật | **L** |

## 5. QC và phiên bản
- Ngưỡng QC lấy từ `taxonomy.json` > `rules`: `min_box_height_px=20`, `match_iou=0.5`, `iaa_iou=0.85`, `iaa_class=0.95`, `error_rate_img_max=0.02`.
- Ca mơ hồ: tick `can_review` trên box; ca chưa vẽ được: ghi cấp ảnh (ảnh, vùng, lý do) vào `image_issues.csv` sau khi khóa nhãn.
- Người quyết ca mơ hồ: reviewer/coach; trong nhóm là người điều phối ghi vào `calibration_decisions.csv`.
- Chưa thay đổi gì so với v0 về taxonomy; các đề xuất mới (thêm lớp, ngưỡng occluded) chờ nhóm.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản giảng viên | — |
| v0.1 (cá nhân 101) | Thêm R-01…R-10, ngưỡng occluded/truncated, bảng 40 ca | Pha 1, các ca ở mục 4 |
