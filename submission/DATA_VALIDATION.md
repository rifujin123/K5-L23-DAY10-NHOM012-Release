# Bàn giao phần Data Validation

Ngày thực hiện: 19/11/2026. Phạm vi: triển khai ban đầu phần Data Validation (9 lỗi luật trong roadmap). Theo phân công hiện tại trong `TEAM.md`, Khôi phụ trách Data Validation; ghi chép dưới đây vẫn giữ lịch sử triển khai ban đầu của Giang.
Lô chính có sẵn khi bắt đầu: runs/r12007/delivery. TEAM.md chưa điền seed;
không tự điền danh tính hoặc xác nhận thay nhóm.

## Bài làm

- validator/rules.py: logic kiểm tra luật, API validate_rules(delivery).
- validator/validate.py: CLI xuất JSON.
- tests/test_validation.py: 13 kiểm thử với dữ liệu nhỏ tự tạo, tách biệt dữ liệu thật.
- runs/r12007/rules.json: artifact bàn giao Người 2.
- submission/data_validation/r12007-rules.json: bản sao để lưu cùng bài nộp.
- validator/README.md: giải thích thuật toán, ngưỡng và giao diện tích hợp.

## Kết quả thực chạy

| Lô | Số ảnh manifest | Số lỗi luật | F1 lỗi luật (bộ chấm làm tròn) |
|---|---:|---:|---:|
| r12007 | 250 | 12 | ≈ 1.0 |
| validation-r12008 | 250 | 13 | ≈ 1.0 |
| validation-clean, seed 5, pool calib | 50 | 0 | Không chấm F1; kiểm tra không báo lỗi |

13/13 unit tests đạt, gồm đủ 9 loại lỗi, tên img_0001, hash cả ba loại file,
box tự nhiên khác kích thước/class, dung sai làm tròn, NaN và chặn đường dẫn ngoài delivery.
F1 tổng ở hai lô lần lượt ≈ 0.7 và ≈ 0.8 vì phần này chỉ nộp lỗi luật;
không phải số đánh giá phần annotation review. Không suy ra F1 chính xác từ số làm tròn.

## Tái chạy

```powershell
python -X utf8 -B validator/validate.py runs/r12007/delivery --out runs/r12007/rules.json
python -X utf8 -B -m unittest discover -s tests -p test_validation.py -v
python -X utf8 -B validator/validate.py runs/validation-r12008/delivery --out runs/validation-r12008/rules.json
python -X utf8 -B validator/validate.py runs/validation-clean/delivery --out runs/validation-clean/rules.json
```

Hai lô kiểm thử phụ được tạo bằng scripts/inject_release.py với seed 12008
và seed 5 --no-faults, không thay lô chính. Lần chạy tạo lô gặp UnicodeEncodeError
ở câu print cuối trên console cp1252 sau khi đã tạo xong delivery.
Cách chạy lệnh tiếp theo là python -X utf8; không sửa scripts của đề.

Để đo, rules được sao chép sang errors.json trong các lô thử và chạy
scripts/score_findings.py --run <run>, mỗi seed một lần với toàn bộ kết quả.
errors.json hiện là bản lỗi luật để đo, chưa phải kết quả cuối sau review.
Không đọc _private, annotation nguồn hoặc dùng chế độ --truth-dir/--dev-truth.
Không chạy test_pipeline.py vì các bài test nội bộ đó có đọc ground truth.

## Còn cần người trong nhóm thực hiện

- Người 2 tích hợp candidates vào CLI, nhận rules và gộp với review thật.
- Nhóm tự xem ảnh; chưa có review confirmed/rejected nào do phần này tạo.
- Người 1 đọc lại logic, tự chạy các lệnh, giải thích khi vấn đáp và thực hiện
  commit bằng danh tính của mình. Chưa commit/push trong phiên này.
- Phần đề ẩn chưa thực hiện vì chưa có delivery đề ẩn được giao.

Không sửa scripts hay delivery v1. Không thay đổi phân công đang có trong work/01-data-validation.md.


## Cập nhật sau khi tích hợp PR #3 và review seed 12007

- `validator/validate.py` hiện tích hợp cờ `--candidates`; lần chạy hiện tại tạo `runs/r12007/rules.json` (12 lỗi luật) và `runs/r12007/candidates.json` (10 ứng viên).
- Review log được cung cấp đã được đưa vào `runs/r12007/review_log.csv`; `scripts/merge_review.py` tạo `errors.json` với 22 finding (12 luật + 10 review confirmed).
- Handover đã hoàn tất và `scripts/check_handover.py --run runs/r12007` đạt 17 PASS, 0 FAIL. Các kết quả luật trước đó vẫn cần thành viên tự đọc hiểu và xác nhận; AI không thay nhóm review ảnh.
