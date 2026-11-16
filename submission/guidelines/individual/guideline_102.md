# Guideline gán nhãn — v1.0 cá nhân — 102

- Phiên bản: v1.0 (bản độc lập Pha 1, phát triển từ v0)
- Ngày: 2026-11-16
- Người viết: học viên 102
- Phạm vi: 40 ảnh `data/images/img_001.jpg`–`img_040.jpg`; bounding box trong CVAT, xuất COCO 1.0.

## 1. Mục tiêu

Tạo nhãn nhất quán cho mô hình phát hiện người và phương tiện thật trong ảnh đường phố, phục vụ phân tích giao thông/camera hỗ trợ lái xe. Gán nhầm tượng, ảnh in, hình phản chiếu, đồ chơi hoặc xe trưng bày thành người/xe thật tạo false positive, có thể khiến hệ thống báo người đi bộ hay phương tiện ở nơi không có thật; bỏ sót người/xe thật gây rủi ro an toàn. Chỉ gán đối tượng thật thuộc taxonomy bên dưới.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa | Ví dụ trong bộ ảnh | Phản ví dụ / dễ nhầm |
|---|---|---|---|
| `car` | Ô tô chở người, gồm sedan, hatchback, SUV, wagon và xe con điện. | Ô tô trên đường; từng ô tô chở trên xe vận chuyển ở `img_002`. | Xe tải chở ô tô không phải `car`; xe mô hình/đồ chơi và xe tượng không gán. |
| `truck` | Phương tiện cơ giới có cabin và thùng/sàn chở hàng hoặc chuyên dụng, kích thước xe tải. | Đầu kéo/xe chở xe ở `img_002`; pickup thật dùng chở hàng ở `img_001`, `img_012`. | Hàng hóa trên thùng không phải đối tượng; pickup tượng/trang trí ở `img_027` không gán. |
| `van` | Xe cơ giới chở người/hàng thân kín, nhỏ hơn xe tải, không thuộc dạng ô tô con. | Van thân kín nếu xuất hiện. | Không dùng `van` cho pickup hở thùng; nếu khó phân biệt van và truck, dùng `can_review` và ghi vấn đề. |
| `motorcycle` | Xe máy hai bánh hoặc xe cơ giới nhỏ dạng xe máy; trong guideline cá nhân này, xe tuk-tuk/auto-rickshaw có động cơ được xếp vào lớp gần nhất là `motorcycle`. | Xe máy, xe máy có sidecar (`img_022`); tuk-tuk (`img_004`, `img_035`). | Cyclo/xe đạp không động cơ không thuộc lớp; không coi người lái là một phần box xe. |
| `person` | Người thật, kể cả người đi bộ, người lái/ngồi trên phương tiện nếu nhìn thấy đủ để xác định là người thật. | Người đi bộ, người bán hàng, người lái xe. | Tượng (`img_017`), ma-nơ-canh (`img_033`), hình người trên billboard (`img_018`), nhân vật mascot/đồ chơi (`img_019`) không gán. |
| `ignore_region` | Vùng chứa nhiều đối tượng quá nhỏ/dày đặc hoặc khó tách để gán từng instance đáng tin cậy. | Một cụm xe/người xa ở hậu cảnh nếu từng vật thấp dưới 20 px hoặc chen lấp không thể tách. | Không dùng để che đi đối tượng rõ ràng, riêng lẻ và đủ lớn. |

**Vật không thuộc lớp:** không tạo lớp mới; bỏ qua vật giả, vật không phải người/xe trong taxonomy và xe đạp/cyclo không động cơ. Tuk-tuk và auto-rickshaw có động cơ tạm gán `motorcycle` để bảo toàn đối tượng giao thông cơ giới; ghi `can_review` nếu nhóm muốn một taxonomy riêng.

### Thuộc tính

| Thuộc tính | Giá trị và quy tắc |
|---|---|
| `occluded_level` | `0`: che 0% (thấy trọn vật); `1`: che >0% đến <30%; `2`: che từ 30% đến 70%, tính cả hai đầu; `3`: che >70% nhưng vẫn nhận diện chắc chắn được lớp. Tính tỷ lệ phần diện tích/hình dáng vật bị vật khác che, không tính phần nằm ngoài khung ảnh. Nếu không thể ước lượng tin cậy, bật `can_review`. |
| `truncated` | `yes` khi vật thật bị khung ảnh cắt mất một phần; `no` nếu toàn bộ vật nằm trong ảnh. Bị vật khác che không đồng nghĩa bị cắt mép. |
| `can_review` | Bật khi lớp/ranh giới box/độ che chưa chắc, hoặc có cách hiểu taxonomy cần nhóm quyết định. Ghi mã ảnh và lý do trong ghi chú/QC. |

## 3. Quy tắc vẽ box

1. Vẽ một box cho mỗi instance thật, hình chữ nhật nhỏ nhất ôm toàn bộ phần nhìn thấy của chính đối tượng; không kéo box theo bóng, khoảng trống hay vật che. Người và xe là hai instance riêng: box người lái không nhập vào box xe.
2. Đối tượng bị che: box chỉ ôm phần nhìn thấy, không suy diễn phần khuất. Gán `occluded_level` theo ngưỡng ở mục 2. Nếu phần còn thấy không đủ nhận diện chắc chắn, bỏ qua và dùng `ignore_region` cho cụm không thể phân giải hoặc ghi `can_review`.
3. Đối tượng bị cắt bởi biên ảnh: box kéo sát biên ảnh, chỉ bao phần nằm trong ảnh; đặt `truncated=yes`. Không đánh `truncated` chỉ vì bị xe/vật khác che.
4. Không gán người/xe giả hoặc không hoạt động trong giao thông: tượng, ma-nơ-canh, hình trên billboard/ảnh, hình phản chiếu trong gương/kính, xe đồ chơi/mô hình/xe trưng bày. Người thật đứng cạnh chúng vẫn được gán riêng. Xe thật chở xe: gán xe chở và từng ô tô thật đang được chở (`img_002`); mỗi xe có box riêng.
5. Pickup thật có khả năng di chuyển/chở hàng là `truck`; pickup làm tượng/monument là bỏ qua. Tuk-tuk/auto-rickshaw có động cơ tạm xếp `motorcycle`, box riêng toàn thân xe; người lái/ngồi thấy rõ có box `person` độc lập.
6. Nếu chiều cao box nhìn thấy dưới 20 px, không gán instance riêng. Nếu nhiều vật nhỏ/dày đặc khiến không thể tách ổn định, vẽ một `ignore_region` bao cụm; không dùng ignore cho một vật lớn, rõ. Ngưỡng 20 px theo kích thước ảnh gốc, trước khi hiển thị/resize.
7. Không gán xe đạp, cyclo, động vật, biển báo, cọc tiêu, hàng hóa hoặc nội thất/chi tiết xe vì chúng không có lớp trong taxonomy hiện tại. Với xe kéo/cyclo, không nhầm người đạp với xe cơ giới; chỉ gán người thật nếu đủ rõ.
8. Xe chính người chụp đang ngồi trong (ego vehicle) không được gán nếu chỉ thấy một phần nội thất/khung xe; gương chiếu hậu và cảnh giao thông phản chiếu trong gương không phải đối tượng trong cảnh (`img_020`).

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định độc lập | Lý do |
|---|---|---|---|---|
| `img_002` | Xe tải chở nhiều ô tô | R-VEH-01 | Gán `truck` cho xe chở và `car` riêng cho từng ô tô nhìn thấy; không gộp thành một box. | Mỗi phương tiện thật là instance riêng, dù đang được vận chuyển. |
| `img_004` | Hai tuk-tuk đỗ bên đường, có tài xế | R-VEH-02 | Mỗi tuk-tuk là `motorcycle` theo quy ước lớp gần nhất; tài xế là `person` nếu nhìn thấy rõ. | Có động cơ và tham gia giao thông; taxonomy thiếu lớp tuk-tuk riêng. |
| `img_006`, `img_009`, `img_023`, `img_034`, `img_039` | Cyclo/xe đạp ba bánh và người trong cảnh đường phố Huế | R-VEH-03 | Không gán cyclo như xe cơ giới; gán người thật nhìn rõ thành `person`. | Cyclo không nằm trong các lớp phương tiện hiện có. |
| `img_017` | Tượng người cưỡi ngựa, có vật trang trí trên đầu | R-FAKE-01 | Không gán người, ngựa hoặc vật trang trí. | Tượng không phải người/xe thật. |
| `img_018` | Billboard có ảnh chân dung | R-FAKE-02 | Không gán người trong ảnh in; xe thật phía dưới được gán nếu đạt ngưỡng kích thước và đủ rõ. | Hình ảnh in không phải người hiện diện trong cảnh. |
| `img_019` | Xe tạo hình nhân vật/xe diễu hành và mascot chuột | R-FAKE-03 | Bỏ qua xe trang trí/đạo cụ và mascot; gán người thật đang lái nếu nhận diện rõ. | Đạo cụ/xe trình diễn không đại diện phương tiện giao thông thông thường; người lái là người thật. |
| `img_020` | Xe và đường phố chỉ xuất hiện trong gương chiếu hậu | R-FAKE-04 | Không gán các xe trong ảnh phản chiếu; không gán gương/xe ego. | Phản chiếu không phải vật thể trực tiếp trong cảnh được chụp. |
| `img_027` | Pickup làm tượng tại nghĩa trang | R-FAKE-05 | Không gán pickup tượng; không gán bia/vật khác. | Xe bất động dạng monument, không phải phương tiện thật. |
| `img_028` | Ngựa đứng trên thùng pickup thật | R-VEH-04 | Gán pickup `truck`; không gán ngựa; không gộp ngựa vào box xe. | Pickup là phương tiện thật; động vật ngoài taxonomy. |
| `img_033` | Ma-nơ-canh trong cửa kính, có phản chiếu | R-FAKE-06 | Không gán ma-nơ-canh hay người/hình phản chiếu; người thật ngoài phố nếu có và đủ rõ thì gán riêng. | Ma-nơ-canh và phản chiếu không phải người thật. |
| `img_035` | Auto-rickshaw/tuk-tuk chở quá tải, có người và xe máy | R-VEH-02 | Mỗi auto-rickshaw có động cơ là `motorcycle`; người thật nhìn thấy gán `person`; xe máy khác là instance riêng. | Áp dụng cùng quy ước tuk-tuk, tách người khỏi xe. |
| `img_038` | Cửa hàng đồ chơi, nhiều xe mô hình và phản chiếu kính | R-FAKE-07 | Không gán mô hình xe, đồ chơi, hình phản chiếu; người thật nhìn qua kính chỉ gán nếu hiện diện trực tiếp và phân biệt rõ. | Vật trưng bày và ảnh phản chiếu không phải phương tiện/người thật. |

Các trường hợp ảnh cụ thể khác được xử lý cùng quy tắc chung. Nếu phân loại tuk-tuk hoặc xe diễu hành gây bất đồng, bật `can_review`; đề xuất nhóm chốt quy ước trước khi khóa v1.0.

## 5. QC và phiên bản

- Ngưỡng QC mục tiêu: IoU box `>= 0.85`; đồng thuận lớp `>= 95%`; tỷ lệ ảnh có lỗi `<= 2%` (trên tổng số ảnh được QC). Đây là mục tiêu kiểm tra sau khi có nhãn độc lập, không phải số đo đã đạt ở Pha 1.
- Soát đủ 40 ảnh: không bỏ sót người/xe thật đủ điều kiện; không gán vật giả; kiểm tra box ôm sát, xe chở xe, box người tách box xe, thuộc tính che khuất/cắt mép và ngưỡng 20 px. Ca không thống nhất ghi `can_review` và chuyển nhóm quyết định.
- Người review cuối/nhóm quyết định các ca mơ hồ tại Pha 2; ghi ảnh, Rule ID, quyết định và lý do trong nhật ký calibration/QC. Không tự coi một ca đang chờ xem là đã đồng thuận.

| Phiên bản | Thay đổi | Lý do / ca dẫn tới thay đổi |
|---|---|---|
| v0 | Bản khởi đầu: chỉ liệt kê lớp, thuộc tính nhưng chưa định nghĩa ngưỡng/biên box và vật giả. | Cơ sở so sánh. |
| v1.0 cá nhân | Bổ sung ví dụ/phản ví dụ, ngưỡng che khuất 0/30/70%, cách vẽ box, ngưỡng vật cao 20 px, ignore cho đám đông và quy tắc vật giả/xe chở xe; thêm quyết định các ca khó ở 40 ảnh. | `img_002`, `img_004`, `img_017`–`img_020`, `img_027`–`img_028`, `img_033`, `img_035`, `img_038` và nhóm cyclo. |
