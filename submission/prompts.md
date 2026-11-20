# Dùng AI

## Bổ sung ngày 19/11/2026 — Data Validation

- Người yêu cầu: Nguyễn Trường Giang, phụ trách Data Validation.
- Công cụ: Codex. Prompt: “tôi là người thực hiện phần data_validation. Bạn hãy đọc folder và roadmap-nhóm rồi thực hiện làm phần bài làm của tôi vào trong folder đi”.
- AI triển khai 9 kiểm tra luật, CLI, 13 unit tests, hướng dẫn tích hợp và rules.json.
- Điểm cần lưu ý: chưa tích hợp candidates của Người 2; không coi rules.json là kết quả review cuối cùng. Lệnh chạy script ban đầu thiếu -X utf8 gây lỗi in tiếng Việt trên Windows; đã đổi cách gọi, không sửa script lab.
- AI kiểm tra: 13/13 unit tests đạt; seed 12007 có 12 lỗi luật, seed 12008 có 13, F1 lỗi luật của cả hai ≈ 1.0 theo bộ chấm làm tròn; lô sạch calib seed 5 có 0 lỗi luật. Chi tiết ở DATA_VALIDATION.md.
- Nhóm cần tự đọc code, chạy lại và xác nhận kết quả; các kiểm thử trên do AI chạy, không phải bằng chứng nhóm đã review bằng mắt. AI không tạo review confirmed/rejected.

Bảng dưới ghi lại việc sử dụng AI trong cuộc trò chuyện chuẩn bị lab. Tên thành viên và kết quả kiểm tra của nhóm cần được bổ sung trung thực; việc AI đọc tài liệu không có nghĩa là nhóm đã tự kiểm tra hoặc chạy validator.

| # | Ai dùng | Công cụ | Prompt (tóm tắt) | AI sai ở đâu / điểm cần lưu ý | Nhóm kiểm lại thế nào |
|---|---|---|---|---|---|
| 1 | Lê Tuấn Khôi (2A202603024) | ChatGPT / Codex | Đọc và tóm tắt repo làm gì. | Chưa ghi nhận lỗi cụ thể. | AI đã đối chiếu README, GUIDE và cấu trúc repo; **nhóm chưa ghi nhận kết quả tự kiểm**. |
| 2 | Lê Tuấn Khôi (2A202603024) | ChatGPT / Codex | Hỏi kiến thức cần có, nhiệm vụ validator và quyết định Hold/Release/rework. | Giải thích ban đầu về tỷ lệ ảnh lỗi chưa nêu ngay quy tắc riêng của lab: mỗi `scene_leak` tính là một ảnh sai split. | AI đã đối chiếu lại GUIDE khi lập roadmap; **nhóm cần tự kiểm lại quy tắc và ghi kết quả**. |
| 3 | Lê Tuấn Khôi (2A202603024) | ChatGPT / Codex | Hỏi cách xác định dấu hiệu lỗi và chọn ngưỡng. | Các ngưỡng tỷ lệ phủ mask 50%, 80%, 95% được đưa ra để minh họa, không phải ngưỡng do repo quy định. | AI đã đối chiếu danh sách lỗi với DELIVERY; **nhóm cần kiểm guideline và tự đo ngưỡng thực tế**. |
| 4 | Lê Tuấn Khôi (2A202603024) | ChatGPT / Codex | Lập roadmap từ chuẩn bị đến báo cáo và nộp, phân tích checkpoint. | Cần đối chiếu các mốc, lệnh và điều kiện với tài liệu gốc; chưa ghi nhận lỗi cụ thể khác. | AI đã đọc GUIDE, CHECKPOINTS, RULES, RUBRIC và SUBMISSION; **nhóm chưa ghi nhận kết quả tự kiểm**. |
| 5 | Lê Tuấn Khôi (2A202603024) | ChatGPT / Codex | Tạo file HTML đẹp, dễ nhìn để nhóm đọc roadmap. | HTML là bản tóm tắt; các ô checklist không lưu trạng thái khi tải lại. | AI đã kiểm tra file có thể được phân tích cú pháp HTML; **nhóm cần mở trang và kiểm nội dung, lệnh**. |
| 6 | Lê Tuấn Khôi (2A202603024) | ChatGPT / Codex | Chia việc cho 4 người và lập đặc tả theo PRD. | Phân công là đề xuất của AI; đề gốc quy định nhóm 5–6 người, cần xác nhận nhóm 4 người với giảng viên. | AI đã đối chiếu phạm vi với tài liệu lab; **nhóm cần xác nhận số thành viên và tự duyệt phân công**. |
| 7 | Lê Tuấn Khôi (2A202603024) | ChatGPT / Codex | Tạo bốn file công việc trong `work/`, kiểm tra và ghi chú phụ thuộc artifact giữa các phần. | Các file là kế hoạch làm việc, không phải bằng chứng validator, review hay bàn giao đã hoàn thành. | AI đã đọc lại bốn file và phân tích phụ thuộc; **nhóm cần kiểm tra, phân người và xác nhận giao diện tích hợp**. |
| 8 | Hà Khánh | Antigravity (Gemini) | Đọc toàn bộ dự án, đặc tả Phần 2 trong `work/02-annotation-review.md` và kết hợp roadmap 4 người mới để hướng dẫn triển khai Phần 2 đúng quy tắc. | Bản `roadmap-nhom.html` đang có trên nhánh là bản cũ (ghi 5–6 người); số liệu thực tế chưa có vì nhóm chưa tải dữ liệu và chưa chốt seed. | AI đã đối chiếu toàn bộ docs, scripts, rules, rubric và roadmap mới; **Hà Khánh và nhóm cần xác nhận số thành viên (4 người) với giảng viên, chốt seed và kiểm lại kế hoạch**. |
| 9 | Hà Khánh | Antigravity (Gemini) | Thiết kế và viết module đối chiếu `validator/cross_check.py` (REV-01..REV-06), công cụ xem ảnh `tools/review_viewer.py` (REV-07..REV-08), unit test `tests/test_cross_check.py` và cập nhật tài liệu tích hợp. | (1) Lần đầu viết `tools/review_viewer.py` bị lỗi cú pháp f-string do dấu `{}` trong mã JavaScript của `_render_html` và cần cấu hình UTF-8 cho stdout trên Windows; đã phát hiện qua unit test và sửa sang template chuỗi thuần. (2) Ngưỡng mặc định `DEFAULT_THRESHOLDS` trong `cross_check.py` là ước lượng hình học ban đầu, **chưa đo hiệu chỉnh trên lô sạch `runs/clean` và ảnh COCO thật**. (3) Chưa tạo `validator/validate.py` để giữ đúng điểm dừng phối hợp với Người 1. | Đã chạy unit test `tests/test_cross_check.py` (3/3 test PASS trên fixture tổng hợp, cùng 3/3 test không cần dữ liệu của `tests/test_pipeline.py`) và kiểm tra AST xác nhận `validator/cross_check.py` chỉ dùng thư viện chuẩn + `pngutil.py`. **Hà Khánh cần tự đọc hiểu code (chuẩn bị vấn đáp), chạy đo và chỉnh ngưỡng trên `runs/clean` sau khi có dữ liệu, chốt tích hợp `validator/validate.py` với Người 1 và cùng nhóm tự xem ảnh thật**. |

AI không tự review và không tạo các verdict confirmed/rejected; review log được cung cấp là kết quả của thành viên nhóm. AI đã mở ảnh `000000005600` để xác nhận ảnh còn nguyên trước khi cập nhật hash v2, và `000000009590` để ghi ví dụ cho ca bảo mật. Nếu sau đó dùng AI để hỏi về ảnh COCO hoặc hỗ trợ các công việc khác, thêm dòng mới và ghi rõ việc kiểm tra của nhóm. Các kết luận `confirmed`/`rejected` phải do thành viên nhóm xem ảnh và ghi nhận.

## Bổ sung ngày 20/11/2026 — đề ẩn và handover

- Người yêu cầu: Lê Tuấn Khôi, phụ trách các yêu cầu về đề ẩn và handover trong phiên này. Công cụ: ChatGPT / Codex.
- Prompt (tóm tắt): kiểm tra kết quả validator đề ẩn; tạo trang review cho 9 ứng viên; kiểm tra review log, gộp lỗi luật với kết quả đã xác nhận và tạo trace theo ticket; kiểm tra run và handover Người 3.
- AI thực hiện: dùng công cụ review sẵn có để tạo trang; kiểm tra review log 9 dòng do thành viên cung cấp, giữ bản 11 lỗi luật và gộp 9 dòng `confirmed` thành 20 finding; đối chiếu ticket, changelog, metadata, jobs và labels để tạo trace lớp `bench`. AI không tự tạo verdict hay ghi chú review.
- Handover: AI phát hiện release trong run cũ khác parent của v2; sau khi nhận run `lab250_7fa680a7413b` từ Người 3, đối chiếu các file với `submission/`, đóng gói lại trên máy Windows vì H15 không khớp, và chạy `check_handover.py`: 17 PASS, 0 FAIL. Kết quả PASS kiểm tra tính nhất quán, không chứng minh nhãn đúng.
- Kiểm lại: AI đã kiểm tra đủ 9 ảnh ứng viên có dòng review, mọi lỗi gán nhãn trong kết quả đều có `confirmed`, và trace dùng guideline theo job. Nhóm cần tự đối chiếu trace và các ghi chú review; chưa ghi nhận kết quả nhóm tự kiểm cho các bước này.
