# Guideline gán nhãn — v0-cá-nhân — thành viên 104

- Phiên bản: v0.1 (bản cá nhân, Pha 1, viết độc lập trước họp nhóm)
- Ngày: 2026-11-16
- Người viết: 101 (mã giả định — đổi lại cho đúng mã được cấp trong TEAM.md)
- Cơ sở: đã xem cả 40 ảnh `img_001`–`img_040`, chưa vẽ nhãn.

## 1. Mục tiêu

Nhãn huấn luyện mô hình phát hiện vật 2D (bounding box) cho camera giao thông / xe tự lái trên ảnh đường phố.
Mô hình chỉ nên học **người và phương tiện thật đang tham gia giao thông**.

Sai nguy hiểm nhất:
- Dạy mô hình coi **tượng, mô hình, hình in, phản chiếu** là xe/người thật (false positive: xe tự lái phanh gấp vì một tấm biển quảng cáo).
- **Bỏ sót** người đi xe máy ở gần (false negative).
- Hai người vẽ **khác số box** cho cùng một cặp người + xe máy, làm IoU/IAA thấp mà thực ra là lỗi guideline.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con chở người, gồm sedan, hatchback, SUV, taxi 4 chỗ. | img_031 (sedan, SUV), img_015, img_023 (Honda đen) | Pickup chở hàng (xem `truck`); xe mô hình/đồ chơi (img_038) |
| `truck` | Xe có thùng hàng riêng hoặc đầu kéo: xe tải, **pickup**, xe chở xe. | img_036 (xe tải nhỏ xanh), img_002 (xe chở xe), img_007 (Isuzu), img_012 | Van thùng kín liền khối |
| `van` | Xe thùng kín liền khối (cabin và khoang chở thông nhau) cỡ nhỏ, kể cả minibus ≤ ~16 chỗ. | img_005 (van xám nền sau), img_016, img_009 (minibus) | Xe tải có thùng rời → `truck` |
| `motorcycle` | Xe hai bánh có động cơ (xe máy, tay ga, mô tô), **kể cả xe có thùng bên (sidecar)**. | img_006, img_021, img_022 | Xe đạp, xích lô, xe ba bánh (xem dưới) |
| `person` | Người thật, đi bộ hoặc ngồi/đứng trên xe. | img_014, img_034, img_032 | Tượng, hình người in, manocanh, phản chiếu |
| `ignore_region` | Vùng không gán từng vật được (quá đông/mờ/nhỏ) và vùng đã loại. | Bãi xe máy dày img_030, đám đông img_032 | Không dùng để "giấu" vật rõ mà lười vẽ |

**Vật không thuộc lớp nào** (xe buýt lớn, xe đạp, xích lô, tuk-tuk/xe lam, xe ba gác): **không gán box và không tạo lớp mới**. Nếu chúng chiếm diện tích lớn và dễ bị mô hình hiểu nhầm thì ghi vấn đề cấp ảnh để reviewer xem (xem mục 4, mã `Q-xx`). Taxonomy v0 không có lớp `bus`, `bicycle`, `three_wheeler` — **đây là lỗ hổng cần nhóm quyết**.

### Thuộc tính

**`occluded_level`** (tính trên phần vật bị che so với phần ước lượng của cả vật):

| Mức | Nghĩa | Ngưỡng |
|---|---|---|
| 0 | Thấy rõ, đủ hình | che < 10% |
| 1 | Che ít | 10–30% |
| 2 | Che nhiều | 30–70% (vẫn nhận ra lớp) |
| 3 | Gần như không rõ | > 70%; chỉ vẽ nếu còn nhận ra lớp, nếu không thì bỏ |

**`truncated`**: `yes` khi vật **bị cắt bởi mép ảnh** (không phải bị vật khác che). Che bởi vật khác dùng `occluded_level`.

**`can_review`**: tick khi mình không chắc **lớp** hoặc **có nên vẽ**; không tick cho lỗi chỉ là box hơi lệch.

## 3. Quy tắc vẽ box

**R-01 Box ôm phần nhìn thấy.** Box bao phần pixel nhìn thấy của vật; không "đoán nối" phần bị che. Ngoại lệ: bánh/chắn bùn bị che nhẹ vẫn bao vào nếu thấy rõ hình dạng tổng thể.

**R-02 Người đi xe máy = 2 box riêng (xe + người), người ngồi trong box xe cho phép chồng lấn.**
- 1 box `motorcycle`: bao thân xe và hai bánh nhìn thấy, **không** bao toàn thân người.
- 1 box `person` cho **mỗi người** trên xe (cả người lái và người ngồi sau), bao phần người nhìn thấy (đầu đến chân/ phần không bị che), gồm mũ bảo hiểm.
- Box hai lớp chồng lấn nhiều là bình thường. Thử trên img_026 (2 người + 1 xe → 3 box) và img_009 (xe chở đồ + 3 người, người phụ nữ bán hàng đứng cạnh) — quy tắc dùng nhất quán.
- Với img_005: nhiều người ngồi sát nhau, xe máy chỉ lộ một phần → vẽ `person` cho từng khuôn mặt/thân nhìn thấy, xe máy chỉ vẽ khi ≥ 30% thân xe thấy (R-04), nếu không để vào can_review.

**R-03 Xe có nhiều phần hàng hóa / tải ngoài khổ.** Box xe ôm thân xe, **không** kéo dài theo vật tải nhô ra (ống nước img_029, hàng chất cao img_035). Người cầm vật dài vẫn là `person` riêng.

**R-04 Ngưỡng kích thước.** Box cao < 20 px (theo `rules.min_box_height_px`) → **không vẽ**, nếu nhiều vật nhỏ thì khoanh một `ignore_region`.

**R-05 Cắt mép ảnh.** Vẽ phần nhìn thấy, `truncated=yes`. Nếu thấy dưới ~15% vật và khó nhận lớp thì bỏ (không vẽ). Ví dụ: xe đỏ trái và xe bạc phải img_015 (vẽ, truncated); người ở mép phải img_016 (vẽ nếu đủ cao ≥ 20 px).

**R-06 Vật giả — KHÔNG gán nhãn.** Tượng, mô hình, đồ chơi, xe trưng bày làm quà tặng/mộ, hình người/xe in trên biển quảng cáo, manocanh, xe diễu hành giả dạng: **không vẽ**. Lý do: mô hình giao thông không cần nhận ra chúng như vật thật; nếu hai người vẽ khác nhau sẽ phá IAA. Ghi vào bảng mục 4 để đối chiếu.

**R-07 Phản chiếu / ảnh trong gương.** Không vẽ vật chỉ xuất hiện qua phản chiếu (gương chiếu hậu, kính, mặt đường ướt). Vật thật phía trước gương vẫn vẽ bình thường.

**R-08 Đám đông và bãi xe dày.** Khi không tách được từng vật, khoanh `ignore_region` quanh cụm; vẫn vẽ các vật tách được rõ ở rìa cụm.

**R-09 Xe chở xe / xe chở vật là xe.** Vẽ **1 box `truck`** cho xe chở; xe được chở ở trên là vật tải nên **không** vẽ riêng (mô hình sẽ thấy là xe thật khi đứng độc lập, nhưng ở đây nó là tải).

**R-10 Ảnh không có vật.** Vẫn giữ trong COCO với 0 box.

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
| img_014 | Hai người đi bộ rõ nét, xe máy mờ phía sau | R-01 | 2 `person` `occluded=0`. Xe máy mờ: vẽ nếu nhận ra lớp; không thì `ignore_region` | Blur mạnh | **H** (xe máy mờ) |
| img_031 | Phố đêm mưa, xe, xe buýt, phản chiếu ánh đèn | R-07, R-04 | `car` các xe thấy rõ; xe buýt: không gán; phản chiếu đèn trên đường: không vẽ | Có phản chiếu mạnh | **L**, bus **H** |
| img_034 | Người đi bộ băng đường + xe máy bên phải cắt mép | R-05 | `person` giữa; `car` đen; `motorcycle`+`person` bên phải `truncated=yes` | Cắt mép | **L** |
| img_036 | Xe tải nhỏ xanh, biển số bị làm mờ; xe máy đỗ phía sau | R-08 | `truck`; xe máy đỗ sau: `motorcycle` nếu tách được, còn lại `ignore_region` | | **L** |
| img_040 | Người lái xe máy áo đen che gần hết khung; taxi, tuk-tuk phía sau | R-05, R-02 | `person` `truncated=yes`; `motorcycle` chỉ thấy gương, vẽ nếu ≥ 30% thân (không) → để `person` + can_review; `car` taxi xanh `truncated=yes`, `occluded=1` | | **H** |
| img_006 | Hàng xe máy chồng chéo, xe đạp | R-02 | Từng cặp xe + người rõ; xe đạp: không gán; taxi trắng: `car` | | **L** |
| img_010 | Đám người bán hàng, xe máy, xe đẩy chuối | R-02 | `motorcycle`+`person` cho 2 cặp; người đứng đối diện: `person`; xe đẩy hàng: không gán | | **L** |
| img_016 | Tuk-tuk, van, xe tay ga xanh, người phụ nữ truncated | R-05 | Tay ga xanh: `motorcycle`+`person`; van trắng: `van`; người cuối phải: `person` `truncated=yes`; tuk-tuk: không gán | | **H** (tuk-tuk) |
| img_025 | Bãi biển: người đi bộ, taxi, xe máy kéo thùng | R-04 | `person` các người ≥ 20 px, người ngồi ghế xa: `ignore_region`; `car` taxi; `motorcycle`+`person` bên phải | | **L** |
| img_026 | 2 người trên 1 xe máy, tranh in cảng nền sau | R-02, R-06 | 1 `motorcycle` + 2 `person`. Tranh nền (có tàu, đảo): không vẽ | Ví dụ chuẩn R-02 | **L** |
| img_039 | Phố nhỏ, người đi xe máy nhỏ, xe đạp, quán | R-04, R-08 | Vẽ cặp xe+người ≥ 20 px; xa hơn: bỏ hoặc `ignore_region` | | **L** |

### Ca "cần hỏi reviewer" quan trọng nhất (tóm lại)

1. **Pickup**: `car` hay `truck` (img_001, 007, 012, 028, 037)?
2. **Xe buýt, tuk-tuk, xe ba bánh, xích lô, xe đạp**: ngoài taxonomy, nhưng xuất hiện ở ≥ 10 ảnh (002, 003, 004, 009, 016, 023, 031, 035, 037, 040). Có thêm lớp hay chỉ dùng `ignore_region`?
3. **Xe diễu hành giả hình xe** (img_019) và **tác phẩm điêu khắc dạng xe máy** (img_011).
4. **Sidecar** (img_022): 1 hay 2 box?
5. **Xe chở xe** (img_002): 1 `truck` hay thêm các `car` trên giá?

## 5. QC và phiên bản

- Ngưỡng QC lấy từ `taxonomy.json` > `rules`: `min_box_height_px=20`, `match_iou=0.5`, `iaa_iou=0.85`, `iaa_class=0.95`, `error_rate_img_max=0.02`.
- Ca mơ hồ: tick `can_review` trên box; ca chưa vẽ được: ghi cấp ảnh (ảnh, vùng, lý do) vào `image_issues.csv` sau khi khóa nhãn.
- Người quyết ca mơ hồ: reviewer/coach; trong nhóm là người điều phối ghi vào `calibration_decisions.csv`.
- Chưa thay đổi gì so với v0 về taxonomy; các đề xuất mới (thêm lớp, ngưỡng occluded) chờ nhóm.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản giảng viên | — |
| v0.1 (cá nhân 104) | Thêm R-01…R-10, ngưỡng occluded/truncated, bảng 40 ca | Pha 1, các ca ở mục 4 |
