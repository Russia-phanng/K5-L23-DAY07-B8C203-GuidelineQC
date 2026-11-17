# Guideline gán nhãn — v1.0 (đã thống nhất) — nhóm 101, 102, 103, 104

- Phiên bản: **v1.0** — khóa cho lượt gán độc lập (Pha 3)
- Ngày: 2026-11-16
- Người viết: 101, 102, 103, 104 (họp Pha 2) · Người duyệt: người review bản cuối ghi trong `TEAM.md`
- Taxonomy đi kèm: `submission/taxonomy.json` (`version: v1.0`, mục `rules` giữ nguyên)
- Phạm vi: 40 ảnh `data/images/img_001.jpg`–`img_040.jpg`, bounding box trong CVAT, xuất COCO 1.0.

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện vật 2D cho camera giao thông / hỗ trợ lái xe trên ảnh đường phố.
Mô hình phải phát hiện **người và phương tiện cơ giới thật** thuộc 5 lớp dưới đây.

Sai nguy hiểm, theo thứ tự ưu tiên:
1. **Bỏ sót** người hoặc xe máy thật ở gần → nguy cơ va chạm.
2. **Gán vật giả thành thật** (tượng, ma-nơ-canh, ảnh in, phản chiếu, đồ chơi) → phanh gấp vô cớ.
3. **Hai người gán khác số box** cho cùng cảnh (đặc biệt người + xe máy) → IAA thấp do guideline, không phải do người gán.

Phép thử cho mọi quy tắc: *"Từ pixel trong ảnh, mô hình có đoán ra được điều này không?"* Nếu không, nhãn không được dựa vào nó.

## 2. Lớp và thuộc tính

### 2.1 Lớp

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm |
|---|---|---|---|
| `car` | Ô tô chở người thân liền ≤ 9 chỗ: sedan, hatchback, wagon, SUV, crossover, taxi; kể cả xe trang trí có kích thước thật chạy trên đường. | img_015 (xe đỏ, xe bạc), img_031, img_024 (Pacer đen), img_019 (xe diễu hành vàng), từng xe trên giá img_002 | Pickup → `truck`; minivan/van thân hộp → `van`; xe đồ chơi img_038 → không gán |
| `truck` | Xe cơ giới có cabin và thùng/sàn chở hàng tách khỏi cabin: xe tải, đầu kéo, xe chở xe, **pickup (bán tải)** kể cả có nắp thùng/camper. | img_002 (xe chở xe), img_007 (Isuzu + pickup), img_036, img_001, img_012, img_028, img_037 | Van thân liền → `van`; bia mộ hình xe tải img_027 → không gán |
| `van` | Xe thân hộp liền khối (cabin thông khoang chở), ≤ 16 chỗ hoặc van chở hàng cỡ nhỏ. | img_005 (van xám nền sau), img_016 (van trắng) | Minibus/xe buýt ≥ 17 chỗ (img_009) → `ignore_region`; pickup → `truck` |
| `motorcycle` | Xe **hai bánh** có động cơ (xe số, tay ga, mô tô, xe máy điện), kể cả **thùng sidecar gắn liền** (một box). | img_006, img_010, img_021, img_026, img_022 (sidecar) | Xe đạp, xích lô → không gán; tuk-tuk/xe ba bánh → `ignore_region`; điêu khắc kim loại img_011 → không gán |
| `person` | Người thật: đi bộ, đứng, ngồi, người lái hoặc ngồi sau xe, kể cả người mặc đồ hóa trang. | img_014, img_034, người lái img_026, người đạp xích lô img_023 | Tượng img_017, ma-nơ-canh img_033, ảnh in img_018, thú bông/mascot không có người bên trong img_019 |
| `ignore_region` | Vùng không gán từng vật: cụm vật không tách được, hoặc phương tiện cơ giới ngoài taxonomy. | Bãi xe dày img_030, đám đông xa img_032, xe buýt img_031/img_037, tuk-tuk img_004 | Không dùng cho vật rõ, tách được; **không** dùng cho vật giả |

### 2.2 Vật không thuộc lớp nào (R-10)

| Loại vật | Xử lý | Lý do |
|---|---|---|
| Phương tiện cơ giới **ngoài taxonomy**: xe buýt, minibus ≥ 17 chỗ, tuk-tuk, auto-rickshaw, xe ba bánh chở hàng, xe lam | 1 box `ignore_region` ôm sát thân xe (kể cả hàng chở) | Là vật cản thật; mô hình đoán trúng thì không bị tính sai, không đoán thì không bị tính bỏ sót |
| Xe **không động cơ**: xe đạp, xích lô, xe đẩy, xe gánh | Không gán xe; người trên/cạnh xe vẫn gán `person` | Không phải phương tiện cơ giới trong taxonomy |
| **Vật giả**: tượng, điêu khắc, bia mộ, ma-nơ-canh, ảnh in/biển quảng cáo/tranh tường, đồ chơi, mô hình, thú bông | **Không gán gì**, kể cả `ignore_region` | Mô hình phải học đây là mẫu âm; phủ ignore sẽ xóa tín hiệu đó |
| Phản chiếu (gương, kính, mặt đường ướt), xe ego | Không gán gì | Không phải vật trực tiếp trong cảnh |
| Động vật, cọc tiêu, biển báo, hàng hóa đứng riêng | Không gán | Ngoài phạm vi |

### 2.3 Thuộc tính

Áp dụng cho 5 lớp vật. `ignore_region` để giá trị mặc định.

**`occluded_level`** — phần **vật bị vật khác che** (không tính phần nằm ngoài khung ảnh), ước lượng theo diện tích hình dáng cả vật:

| Mức | Ngưỡng | Ví dụ |
|---|---|---|
| `0` | che < 10% | 2 người đi bộ img_014 |
| `1` | 10% đến < 30% | xe sau song rào img_018; người đứng sau thùng xe img_001 |
| `2` | 30% đến 70% | các xe tầng dưới giá chở img_002; van xám bị người che img_005 |
| `3` | > 70% nhưng **vẫn chắc chắn lớp** | xe máy gần như khuất sau người img_005 |

Che > 70% và không chắc lớp → không vẽ box; nếu là cụm thì đưa vào `ignore_region` (R-08).

**`truncated`** — `yes` khi vật bị **mép ảnh** cắt mất một phần (box chạm mép ảnh và vật còn tiếp ra ngoài); ngược lại `no`. Bị vật khác che **không** phải truncated.

**`can_review`** — tick khi không chắc **lớp** hoặc **có nên vẽ** box này. Không tick chỉ vì box hơi lệch vài pixel. Mỗi box tick phải ghi lý do ngắn trong ghi chú cá nhân.

## 3. Quy tắc vẽ box

| Mã | Quy tắc |
|---|---|
| **R-01** Phần nhìn thấy | Box là hình chữ nhật nhỏ nhất ôm các pixel **nhìn thấy** của vật, gồm gương, mũ bảo hiểm, ba lô. Không đoán phần bị che hoặc ngoài khung ảnh. Không kéo theo bóng. |
| **R-02** Bị che rời rạc | Vật bị cột/rào/xe khác chắn thành nhiều mảnh: **1 box** ôm tất cả mảnh nhìn thấy của cùng vật; đặt `occluded_level` theo 2.3. |
| **R-03** Người + xe máy | Luôn tách: **1 box `motorcycle`** (thân xe, hai bánh, ghi đông, gương) + **1 box `person` cho mỗi người** trên xe (lái và ngồi sau, từ mũ tới chân nhìn thấy). Hai box chồng nhau là bình thường. Ví dụ: img_026 → 1 xe + 2 người = **3 box**; img_010 xe trái → 1 xe + 2 người. Xe máy chỉ lộ < 30% thân (vd gương img_040) → không vẽ xe, vẫn vẽ người. |
| **R-04** Cắt mép | Box chạm mép ảnh tại chỗ cắt, `truncated=yes`. Vật còn nhìn thấy quá ít để chắc lớp → không vẽ. Ví dụ img_015 (2 xe hai bên), img_012 (pickup tiền cảnh), img_040 (người lái). |
| **R-05** Hàng hóa | Hàng **đặt trên / buộc vào** xe (thùng, bao, tượng chở trên thùng) nằm **trong** box xe: img_012, img_028, img_035. Vật **người cầm tay** không thuộc box xe hay box người: ống dài img_029. |
| **R-06** Xe chở xe | Xe chở = 1 box `truck` ôm đầu kéo + giá/rơ-moóc + xe trên giá. **Mỗi ô tô trên giá** còn có box `car` riêng, `occluded_level` theo phần bị khung giá che (img_002). Lý do: nhìn từ pixel, đó là ô tô thật; không gán thì mô hình bị phạt khi phát hiện đúng. Ngoại lệ: mô hình/đồ chơi trên nóc xe (img_024 xe vàng) → không gán (R-07). |
| **R-07** Vật giả | Không gán tượng, điêu khắc, ma-nơ-canh, ảnh in, đồ chơi, mô hình, bia mộ, thú bông (xem 2.2). Người thật đứng cạnh vẫn gán. Người mặc đồ hóa trang là `person`. |
| **R-08** Đám đông / bãi xe | Cụm vật chồng lấn không tách được ranh giới, hoặc nhiều vật < 20 px: 1 `ignore_region` ôm cả cụm. Vật ở rìa cụm tách rõ và ≥ 20 px vẫn vẽ box riêng. |
| **R-09** Kích thước tối thiểu | Không vẽ box vật nếu **chiều cao** box < **20 px** trên ảnh gốc (`rules.min_box_height_px`). |
| **R-10** Ngoài taxonomy | Xem bảng 2.2: phương tiện cơ giới ngoài lớp → `ignore_region` ôm sát xe; người ngồi **trong** xe đó không vẽ riêng (đã nằm trong vùng ignore); người ở ngoài thân xe vẫn vẽ. |
| **R-11** Phản chiếu / ego | Không gán vật thấy qua gương, kính, mặt nước; không gán xe của người chụp (img_020). |
| **R-12** Mờ chuyển động | Vật mờ vẫn là vật thật: vẽ nếu chắc lớp, ôm vùng vật rõ nhất; không chắc lớp → `can_review` (img_014 xe máy mờ phía sau). |
| **R-13** Ảnh không có vật | Ảnh vẫn có mặt trong COCO, 0 annotation (img_017, img_027, img_033, img_038). |

## 4. Ca khó đã chốt

Cột "Bất đồng Pha 1" ghi lựa chọn ban đầu của từng thành viên để đối chiếu.

| Ảnh | Tình huống | Rule | Quyết định nhóm | Bất đồng Pha 1 → lý do chốt |
|---|---|---|---|---|
| img_002 | Xe chở 8 ô tô + người đứng phải | R-06, R-09 | 1 `truck` (cả giá và xe trên giá) + `car` cho mỗi ô tô cao ≥ 20 px, occluded theo khung giá; `person` người đứng | 101, 104: chỉ truck · 102, 103: truck + car → chọn tách vì ô tô trên giá là ô tô thật nhìn thấy được |
| img_001, 007, 012, 028, 037 | Pickup | R-05, `truck` | `truck`; hàng/tượng ngựa trên thùng nằm trong box (img_012, img_028) | 101 định nghĩa pickup là car, còn lại truck → chọn `truck` vì có thùng chở tách cabin |
| img_004, 016, 035, 040 | Tuk-tuk, auto-rickshaw | R-10 | `ignore_region` ôm sát xe (kể cả hàng chở img_035); tài xế ngồi trong không vẽ riêng | 102: motorcycle · 103: van · 104: không gán · 101: review → không lớp nào đúng nghĩa; ignore tránh dạy sai lớp mà không phạt mô hình |
| img_009, 031, 037 | Xe buýt / minibus | R-10 | `ignore_region` ôm sát xe | 103: ignore · 101, 104: không gán → chọn ignore cho đồng nhất với tuk-tuk |
| img_019 | Xe diễu hành kích thước thật, người lái, thú bông | R-07, `car` | `car` cho xe; `person` người lái; không gán thú bông. Đám đông sau rào: R-08 | 102: bỏ xe · 103: car · 101, 104: car + review → xe thật có bánh, biển số, chạy trên đường |
| img_011 | Điêu khắc kim loại hình xe máy, đầu tượng lớn, người xa | R-07, R-09 | Không gán điêu khắc/đầu tượng; `person` người thật ≥ 20 px; van trắng xa: `van` nếu ≥ 20 px | 103: person + motorcycle (đọc là người hóa trang) → nhìn kỹ là tượng không có thân người, xe làm từ phế liệu không chạy được |
| img_022 | Xe máy sidecar | R-03 | 1 `motorcycle` (xe + thùng) + `person` mỗi người thấy được | Cả 4 thống nhất 1 box xe |
| img_029 | Xe máy, người cầm ống dài; xe đạp | R-03, R-05 | `motorcycle` + `person` lái; ống không vào box nào; người đạp xe: `person`; xe đạp không gán | 103 gộp ống vào box xe, 104 không → ống do người cầm, không gắn vào xe |
| img_026 | 2 người trên 1 xe máy, tranh tường in cảng | R-03, R-07 | 1 `motorcycle` + 2 `person`; không gán tàu/xe trong tranh | Thống nhất |
| img_005 | Nhiều người đội mũ trên xe máy sát nhau, van nền | R-03, R-02 | `person` cho mỗi người thấy rõ; `motorcycle` chỉ khi thấy ≥ 30% thân; van: `van` `occluded=2` | Thống nhất; ngưỡng 30% thân xe theo R-03 |
| img_003, 006, 023, 039 | Xe đạp, xích lô | R-10 | Không gán xe; `person` cho người đạp/ngồi | Thống nhất |
| img_017, 027, 033, 038 | Tượng, bia mộ, ma-nơ-canh, tủ đồ chơi | R-07, R-13 | Không gán gì; ảnh giữ 0 annotation | 103 đề xuất ignore cho tủ đồ chơi → không dùng ignore cho vật giả (mẫu âm) |
| img_018 | Biển quảng cáo ảnh người; ô tô sau hàng rào | R-07, R-02 | Không gán ảnh in; `car`/`truck` sau rào, `occluded=1` | Thống nhất |
| img_020 | Xe trong gương chiếu hậu | R-11 | Không gán gì | Thống nhất |
| img_013, 015, 031 | Phản chiếu trên đường ướt | R-11 | Không gán phản chiếu; vật thật ≥ 20 px vẫn gán; tượng rồng img_013 không gán | Thống nhất |
| img_008, 021, 030, 032 | Bãi xe dày, đám đông | R-08 | Vẽ vật tách rõ ≥ 20 px; cụm không tách được → `ignore_region` | Thống nhất; người vẫn tách được thì phải vẽ |
| img_014 | Xe máy mờ chuyển động | R-12 | 2 `person` rõ; xe/người mờ: vẽ nếu chắc lớp, kèm `can_review` | Thống nhất |

### Ca còn mở (ghi `can_review` khi gán)

- Ranh giới `van` / `car` với minivan 7–9 chỗ: theo hình dáng thân hộp → `van`; nếu không rõ thì `can_review`.
- Vật có chiều cao sát 20 px (img_013, img_039): đo trong CVAT; dưới 20 px thì không vẽ.

## 5. QC và phiên bản

- Ngưỡng QC (từ `taxonomy.json > rules`, không đổi): ghép box `match_iou = 0.5`; IAA đạt khi IoU ≥ `0.85` (box lớn, cao ≥ 50 px) / ≥ `0.70` (box nhỏ) và đồng thuận lớp ≥ `0.95`; tỉ lệ ảnh lỗi ≤ `2%`.
- Box chưa chắc: `can_review`. Không vẽ được: ghi `image, annotator, region, note` để nhóm gom vào `submission/qc/image_issues.csv` **sau khi khóa nhãn**.
- Ca mơ hồ mới phát sinh: nhóm chốt ở Pha 5, ghi `calibration_decisions.csv` với mã rule; quy tắc đổi nghĩa → v1.1 `semantic`, kiểm lại cả lô.
- v1.0 **khóa** trong lượt gán độc lập; không sửa giữa chừng.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản giảng viên: 6 lớp, 3 thuộc tính, chưa có ngưỡng | — |
| v1.0 | Định nghĩa lớp có ví dụ/phản ví dụ; ngưỡng occluded 10/30/70%; truncated; R-01…R-13; ngoài taxonomy → ignore; vật giả không gán; xe chở xe tách `car` | Hợp nhất 4 bản cá nhân 101–104; ca img_002, 004, 009, 011, 019, 022, 029, 035, 038 |
