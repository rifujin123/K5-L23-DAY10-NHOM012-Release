# Đối chiếu nhãn và review ảnh

Ngày cập nhật: 20/11/2026. Hà Khánh phụ trách phần đối chiếu nhãn và chuẩn bị review ảnh (`REV-01` đến `REV-08`). Validator chỉ đọc dữ liệu được giao trong `delivery/`; nhóm giữ nguyên v1 và không sửa các script của đề.

## 1. Bằng chứng đo nhiễu và hiệu chỉnh ngưỡng trên lô sạch (`runs/clean`)

Nhóm chạy thử trên lô sạch seed `5` với tùy chọn `--no-faults`. Lô này có 50 ảnh thuộc pool `calib`, tách biệt với 250 ảnh luyện tập. Vì `runs/` không được commit, kết quả ứng viên được lưu tại:
- [submission/annotation_review/clean-candidates.json](annotation_review/clean-candidates.json)

### Lệnh chạy lại

```powershell
py -X utf8 scripts/inject_release.py --seed 5 --no-faults --out runs/clean
py -X utf8 -B validator/validate.py runs/clean/delivery --out runs/clean/rules.json --candidates runs/clean/candidates.json
```

### Số liệu đo thực tế trên `runs/clean` (50 ảnh sạch)

| Chỉ số đo trên `runs/clean/delivery` | Giá trị thực tế |
|---|---:|
| Số ảnh quét | `50 / 50` |
| Số lỗi luật (`runs/clean/rules.json`) | `0` |
| Số ứng viên gán nhãn (`runs/clean/candidates.json`) | `0` (`0` ảnh) |
| Tổng cặp vật cùng lớp đo được (mọi kích thước) | `435` |
| Tỷ lệ vật sạch có `mask_in_box = 1.0000` | `435 / 435` (`100%`) |
| `IoU` thấp nhất trong `299` vật nhìn rõ (`area ≥ 0,3%` ảnh) | `0.9082` |
| Độ lệch tâm chuẩn hóa tối đa trong `299` vật nhìn rõ | `0.040` |
| Segment thing (non-crowd) không khớp box | `1` segment rất nhỏ (`299 px`, `0,083%` ảnh) |
| Cặp khác lớp chồng nhau (`IoU ≥ 0,3`) chưa khớp cùng lớp | `0` |

### Lý do chọn bộ ngưỡng cuối (`DEFAULT_THRESHOLDS` trong `validator/cross_check.py`)

| Nhóm dấu hiệu | Ngưỡng chốt | Căn cứ từ số liệu `runs/clean` & kiểm tra thực tế |
|---|---|---|
| Lọc kích thước tối thiểu | `min_box_area_ratio = 0.0035`, `min_seg_px = 600` (sàn `150 px` cho drift) | Bỏ các vật quá bé khó quan sát bằng mắt trên ảnh nhỏ (~240×320). |
| Khóa ghép 1-1 trước `instance_merge` | `exact_match_iou = 0.80`, `exact_match_mask = 0.90` | Trên `runs/clean` mọi vật nhìn rõ có `IoU ≥ 0.908`, `mask_in_box = 1.0`. Khóa ghép 1-1 trước giúp loại lỗi báo nhầm chồng lấn góc nhìn trước/sau (đã gặp ở ảnh `000000002153` trên `r12007`). |
| `class_swap` / `category_swap` | `swap_min_iou = 0.60`, `swap_min_mask_in_box = 0.75` | Giữ `0` báo thừa trên `runs/clean` (không có cặp khác lớp chồng `IoU ≥ 0,3`), bắt rõ vật khớp vị trí nhưng sai tên lớp. |
| `bbox_drift` | `drift_max_mask_in_box = 0.95` (`drift_spill_max_iou = 0.92`) hoặc `drift_low_iou = 0.78` (`drift_low_iou_max_mask = 0.99`, `drift_min_center_shift = 0.08`) | Nới nhạy hơn ngưỡng ban đầu (`0.85` / `0.62`) để không bỏ sót drift nhẹ ở đề ẩn, nhưng vẫn cách an toàn vùng sạch (`mask_in_box = 1.0`, `IoU ≥ 0.908`, `shift ≤ 0.040`) → giữ `0` nhiễu trên `runs/clean`. |
| `missing_box` | `missing_min_seg_px = 800`, `missing_min_area_ratio = 0.004` (sàn `250 px`) | Cao hơn mức nhiễu lớn nhất trên `runs/clean` (`299 px` / `0,083%`) để không báo thừa vật quá nhỏ. |
| `instance_merge` | `merge_min_seg_px = 600`, `merge_max_box_iou = 0.65`, `merge_min_part_px = 150`, `merge_min_part_ratio = 0.12`, `merge_min_union_gain = 0.12` | `merge_max_box_iou < 0.65` và `union_gain ≥ 0.12` chặn báo nhầm cặp `duplicate_box` (chỉ lệch 1–2 px). |

## 2. Kết quả trên lô luyện tập `runs/r12007`

- Lô v1 `lab250_7fa680a7413b` có 250 ảnh trong manifest. Validator quét được 249 ảnh; một ảnh thiếu file được ghi nhận là `missing_image`.
- **Hàng đợi & review bằng mắt (cả 4 thành viên xem qua `tools/review_viewer.py`):**
  - Vòng review đầu sinh **`11` ứng viên trên `11` ảnh** (`missing_box: 3`, `instance_merge: 3`, `class_swap/category_swap: 3`, `bbox_drift: 2`).
  - Kết quả xem ảnh thật trong `runs/r12007/review_log.csv`: **`10 confirmed` + `1 rejected`** (ảnh `000000002153` là 3 người chồng lấn góc nhìn trước/sau, mỗi người đều có đủ box và mask riêng — đã ghi `rejected` kèm lý do và sửa thứ tự ghép 1-1 trước `instance_merge` trong `validator/cross_check.py`).
  - Sau khi vá lỗi thứ tự `instance_merge` và cập nhật ngưỡng hiệu chỉnh theo `runs/clean`, `validator/validate.py` sinh **`10` ứng viên trên `10` ảnh** cho `r12007` và vẫn **`0` ứng viên** trên `runs/clean`.
- **Gộp kết quả (`scripts/merge_review.py`) và tự đo (`scripts/score_findings.py --run runs/r12007`):**
  - `runs/r12007/errors.json`: **`22` lỗi** (`12` lỗi luật từ `rules.json` + `10` lỗi gán nhãn đã `confirmed`).
  - Tự đo lại trên run `lab250_7fa680a7413b` bằng `py -X utf8 -B scripts/score_findings.py --run runs/r12007`: **F1 tổng ≈ `1.0` · lỗi luật ≈ `1.0` · lỗi gán nhãn ≈ `0.9`**. Đây là kết quả làm tròn một chữ số thập phân.
- **Kiểm tra định dạng (`scripts/check_handover.py --run runs/r12007`):** trên run `lab250_7fa680a7413b`, kết quả thực chạy sau khi đồng bộ và đóng gói lại là **17 PASS, 0 FAIL**. PASS không chứng minh nhãn đúng.
- **Kiểm thử đơn vị:** `18/18` PASS (`tests/test_validation.py` 13/13 + `tests/test_cross_check.py` 5/5); AST xác nhận `validator/validate.py`, `validator/rules.py`, `validator/cross_check.py` chỉ dùng thư viện chuẩn Python + `scripts/pngutil.py`.

## 3. Các mục chung còn chờ cả nhóm

- Xác nhận quy mô nhóm 4 người với giảng viên/coach (đề gốc ghi 5–6 người).
- Đề ẩn đã có bốn file kết quả trong `submission/de_an/`; không commit `delivery/` của đề ẩn.
