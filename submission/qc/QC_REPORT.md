# Báo cáo QC — Nhóm 101–104 (thành viên 101, 102, 103, 104)

> **Nộp 2026-11-17 — trạng thái Hold** (chạy lại sau khi 104 nộp lại `104.json`; bảng quyết định và IAA cũ lưu ở `submission/qc/_old_iaa_v1/`).
> Chưa có: sửa tay trên CVAT và người review vòng 1 (xem mục 5).

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Đặc tả và QC nhãn bounding box 2D (5 lớp `car`, `truck`, `van`, `motorcycle`, `person` + `ignore_region`) để huấn luyện/đánh giá mô hình phát hiện người và phương tiện trên ảnh đường phố. Lô chỉ 40 ảnh Wikimedia Commons, thiên về Đông Nam Á và có chủ đích nhiều ca biên (tượng, phản chiếu, đồ chơi, xe chở xe), nên **không đại diện** cho phân bố camera giao thông thật; chỉ dùng cho học tập, thử quy trình QC và làm tập kiểm ca biên. Taxonomy không có `bus`, `bicycle`, xe ba bánh: các xe này nằm trong `ignore_region`, mô hình huấn luyện từ lô này không học được chúng.
- Nguồn và phép sử dụng dữ liệu: 40 ảnh Wikimedia Commons, ghi công từng ảnh trong `data/ATTRIBUTION.csv`. Giấy phép: CC BY-SA 4.0 (15), CC BY 2.0 (7), CC BY-SA 2.0 (6), CC BY 4.0 (5), CC0 (3), CC BY-SA 3.0 (3), Public domain (1). 24 ảnh CC BY-SA: chia sẻ lại phải giữ ghi công và cùng giấy phép. Ảnh không sửa, không đổi tên.
- Manifest và checksum nhãn cuối: `submission/qc/batch_manifest.json`; `submission/qc/labels_calibrated.json` SHA-256 `3b3f7364b7e9b8a3c585c42ae2ca2c7e4ab8b3a85024f90d026693879872dcbe` (40 ảnh, 433 annotation, sinh 2026-11-17).
- Guideline và taxonomy khi gán / khi bàn giao: Khi gán: `guideline_v1.0_agreed.md` + `taxonomy.json` `v1.0` (khóa 2026-11-16). Khi bàn giao: `guideline_v1.1_calibrated.md` (thay đổi **semantic**, `submission/guideline_change.json`: viết lại R-08, thêm R-14, R-15, làm rõ R-05); taxonomy **giữ v1.0** vì không đổi lớp/thuộc tính, không có `taxonomy_v1.1.json`.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80 / 80. `split_assign.py --members 101,102,103,104` seed 42, mỗi ảnh 2 người, mỗi người 20 ảnh. Cả 4 file `submission/labels/<mã>.json` qua `validate_labels.py --all` (101 có 2 cảnh báo vật nhỏ: img_009 #40, img_039 #211).
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: Mẫu số **193 cặp box ghép** (IoU ≥ 0.5, ghép 1-1) trên 40 ảnh. IoU TB **0.844** (< 0.85, **CHƯA ĐẠT**); cùng lớp **98.4%** (đạt ≥ 95%); cùng `occluded_level` 62.2%; cùng `truncated` 89.1%. Theo cặp: 101|102 0.869 ĐẠT (24 cặp), 101|103 0.854 ĐẠT (48), 102|103 0.892 ĐẠT (25), 101|104 0.807 (29), 102|104 0.818 (47), 103|104 0.843 (20) CHƯA ĐẠT. IoU và % lớp **không** tính box chưa ghép. Lần đo đầu (trước khi 104 nộp lại): 192 cặp, IoU 0.843, 485 ca.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: Box chưa ghép 178 (phía A) / 53 (phía B) — không nằm trong mẫu số 193; 9 box loại vì cả hai cùng phủ ignore. 478 ca bất thường trên 34 ảnh: `bo_sot_thua` 134, `bat_dong_ignore` 128, `sai_thuoc_tinh` 84, `iou_thap` 73, `co_can_review` 56, `sai_lop` 3. Số box mỗi người rất lệch: 101 = 226 (25 ignore), 102 = 190 (6), 103 = 123 (16), 104 = 136 (2, nhưng 47 box tick `can_review`). Phân tích: bất đồng ignore/bỏ sót tập trung ở ảnh đông (img_021: 37 ca, img_003: 27, img_011: 19, img_032: 18, img_030: 15) và **cả hai người cùng lệch** theo hai cách đọc R-08 v1.0 ("tách được" không đo được) → lỗi guideline; các ca `sai_lop` và box trùng là **một người lệch** (104 ở img_016/img_040, 101 ở img_037/img_002). Bản nộp lại của 104 đã sửa img_022 (thêm `person` sidecar) và bỏ box trùng, nhưng gán tuk-tuk thành `motorcycle` thay vì `ignore_region` (img_016, img_040) — vẫn lệch R-10.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: Chưa đối chiếu ca tham chiếu của coach (hết giờ). Ứng viên tự kiểm: img_018 (101 và 103 đều gán 5 `car`; xe tối màu biển 9AE 2365 có thể là pickup → `truck` theo v1.0 — cần mở ảnh kiểm); img_024 (cả 101 và 103 không vẽ box riêng cho xe mô hình — đúng R-07, nhưng chỉ 101 bao nó vào box xe chở theo R-05). Điều IAA **không** xác nhận: hai người cùng bỏ một vật, hoặc cùng gán vật giả thành thật, vẫn cho IoU/đồng thuận cao.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: `submission/qc/calibration_decisions.csv` có đủ **478 / 478** dòng, mỗi ca IAA một dòng, `phien_ban_guideline=v1.1`. Theo quy tắc: R-08 v1.1 279, R-01 91, R-14 73, R-04 11, R-10/R-15 11, R-06 7, R-05 v1.1 2, R-09 1, định nghĩa lớp 3 (C209 img_016 → `van`; C210 img_016 → `truck`, 104 gán motorcycle; C455 img_037 → `truck`). Phạm vi: 248 ca `lo` (R-08 đổi nghĩa), 230 ca `anh`. Người chốt: soạn theo v1.1, **chờ nhóm duyệt** (cột `nguoi_chot`). Kiểm lại: chỉ bằng script (validator lab, `build_final.py` qua); **chưa** có người khác người sửa kiểm trên CVAT — lý do Hold.
- Tác động v1.1 lên nhãn và ảnh: Thay đổi `semantic` → `prepare_calibration.py` đưa **cả 40 ảnh** vào phạm vi sửa (`work/calib/images/`). Nhãn nháp đã áp quyết định tự động: `work/calib/auto_v1.1_draft.json` (433 annotation, 0 `can_review`; đổi lớp 2 box img_016 → `van`/`truck`, xóa 3 box tuk-tuk của 104, thêm 4 vùng ignore R-10 của 103, sửa tay 2 box img_016/img_040, cắt box về trong khung ảnh). Nhãn sau sửa `submission/qc/labels_calibrated_2d3b1a1143bd6932.json` **là bản nháp áp quy tắc tự động, không qua sửa tay trên CVAT** do hết giờ. Không cần mapping lớp (taxonomy không đổi).
- Tham chiếu bằng chứng khi Rework: Không áp dụng (vòng 1 Hold).

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: chưa có người review độc lập; phiếu vòng 1 tạo 2026-11-17, chưa điền
- Đã duyệt / cả lô: 0 / 40 (đã lấy mẫu 40/40 bằng qc_sample.py, chưa duyệt)
- Ảnh lỗi / ảnh đã duyệt: chưa đo (0 ảnh đã duyệt)
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: chưa đo
- Vật bỏ sót, ảnh và vùng: chưa đo; rủi ro biết trước: các ảnh đông (img_003, 011, 021, 032) — vật ≥ 20 px mà cả hai người cùng bỏ sót không hiện trong IAA
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Lỗi lặp trước calibration: tuk-tuk gán sai lớp (img_016, img_040), vùng ignore phủ vật tách được (img_011, img_021). Còn lại: box lấy trung bình ở 73 ca IoU thấp chưa chỉnh tay; img_018 nghi pickup gán `car` ở cả hai người.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Máy chặn được: `validate_labels.py` chặn box vượt khung ảnh (img_005 #57 trong bản đồng thuận do làm tròn — đã sửa `scripts/common.py`, IAA chạy lại không đổi). Cần người đọc ảnh: 104 vẽ hai box khác lớp (`car` + `truck`) trùng nhau trên cùng pickup ở img_016 — hợp lệ về cấu trúc, chỉ phát hiện khi mở ảnh chồng nhãn `work/viz/img_016.png`.

## 5. Quyết định bàn giao

- Quyết định: Hold
- Lý do, người nhận và phạm vi sử dụng: Thiếu bằng chứng — nhãn calibration là nháp áp quy tắc tự động, chưa sửa tay trên CVAT và chưa có người review độc lập; IAA trước calibration chưa đạt (IoU 0.844). Không bàn giao để huấn luyện; chỉ dùng nội bộ lớp làm minh chứng quy trình.
- Nếu còn việc: (1) 101/102/103 kiểm và chỉnh 40 ảnh trên CVAT từ `work/calib/auto_v1.1_draft.json`, export lại, chạy `build_final.py`; (2) một người khác người sửa điền `qc_review.csv` vòng 1, chạy `qc_report.py --round 1`; (3) điền cột `kiem_lai`, `nguoi_chot` trong calibration_decisions.csv.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Chưa có người ngoài nhóm kiểm; đường lần theo đề xuất — ca C210: img_016 → R-10, R-15 + định nghĩa `truck` trong guideline v1.1 → pickup [0,278,328,266] lớp `truck`, tuk-tuk trái là `ignore_region` trong labels_calibrated.json
