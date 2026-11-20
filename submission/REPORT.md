# Báo cáo nhóm 12

## 1. Số liệu luyện tập

- Seed: `12007`, release v1: `lab250_7fa680a7413b`.
- Khi tự đo bằng `scripts/score_findings.py`, nhóm nhận được F1 tổng khoảng 1,0; F1 lỗi luật khoảng 1,0 và F1 lỗi gán nhãn khoảng 0,9. Script làm tròn đến một chữ số thập phân. Validator phát hiện 12 lỗi luật.
- Phần đối chiếu quét được 249/250 ảnh; ảnh `000000021839` bị thiếu nên không thể quét. Hàng đợi cuối có 10 ứng viên trên 10 ảnh. Trong quá trình xem ảnh, nhóm xác nhận 10 trường hợp và bác bỏ một ứng viên của lượt chạy trước.
- Với lô sạch seed 5, kết quả lưu trong `submission/annotation_review/clean-candidates.json` cho thấy đã quét 50 ảnh và không có ứng viên. Kết quả lỗi luật lưu trong `submission/data_validation/clean-rules.json` cũng là 0. Nhóm chọn ngưỡng dựa trên các kết quả này: chẳng hạn, một cặp được xem là khớp chắc khi IoU ≥ 0,80 và ít nhất 90% mask nằm trong box. Chi tiết cách chọn ngưỡng có trong `ANNOTATION_REVIEW.md`. Lô sạch không còn trong workspace hiện tại nên chưa được chạy lại ở đây.

## 2. Vì sao báo hết ứng viên làm F1 giảm?

Detection và panoptic có thể khác nhau vì vật bị che, nằm trong đám đông hoặc chồng lên vật khác. Những khác biệt này chưa chắc là lỗi. Ví dụ, nhóm đã bác bỏ một ứng viên `instance_merge` vì ba người trong ảnh chỉ đứng chồng lên nhau theo góc nhìn. Nếu báo cả trường hợp đó là lỗi, số báo nhầm sẽ tăng, làm precision và F1 giảm. Số ứng viên được xác nhận cũng không đủ để suy ra recall vì có thể còn lỗi nằm ngoài hàng đợi review.

## 3. Quyết định và lineage

`errors.json` có 22 kết quả, tương ứng 21 target duy nhất trên 250 ảnh; `scene_leak` được tính là một target theo bộ kiểm. Tỷ lệ là `21/250 = 8,4%`, vượt ngưỡng 2%, nên nhóm quyết định **HOLD** v1. Lineage ghi lại quyết định này và release `lab250_7fa680a7413b_v2` có parent là v1. Run `R-021` đã train trên v1 nên cần được xem lại hoặc train lại: việc sửa dataset không tự sửa model đã học.

V2 là lô detection gồm 239 ảnh và 1.638 box. Nhóm loại 11 ảnh bị thiếu hoặc có lỗi detection khỏi v2 và giao lại cho đúng người gán, người kiểm theo `jobs.csv`. Ảnh `000000005600` được mở kiểm tra và vẫn hiển thị bình thường, nên được giữ lại với hash mới trong manifest. Guideline trong handover được lấy theo job; các ảnh còn lại của `scene_801` được đưa về split `val`. Sáu trường hợp lỗi chỉ thuộc panoptic được giữ trong lô detection và ghi chú để báo team panoptic.

Sau khi đóng gói, nhóm chạy `check_handover.py` và đạt 17/17 mục. Kết quả này cho biết các file và số liệu nhất quán; việc đánh giá nhãn vẫn cần người xem ảnh.

## 4. Hash chứng minh gì, không chứng minh gì?

Hash giúp kiểm tra file có thay đổi so với lúc được tính hash hay không. Nếu hash khác, file hiện tại và bản hash không còn khớp. Tuy nhiên, hash khớp không có nghĩa nhãn đúng: một box sai vẫn có hash hợp lệ nếu file không bị thay đổi sau khi tính hash.

## 5. Truy vết

Team AI báo có thể bỏ sót box `cup` bị che hoặc nằm ở mép ảnh trong hai cảnh `scene_609` và `scene_868`. Theo changelog, v1.2 cho phép bỏ qua một số trường hợp này, còn v1.3 yêu cầu gán box. Nhóm tìm các ảnh có box `cup` (ID 41) và thuộc job dùng v1.2, rồi chia thành hai tầng:

- Tầng A, thuộc hai cảnh được báo: `000000002157`, `000000002431`, `000000002592`, `000000002685`.
- Tầng B, thuộc các cảnh khác: `000000003661`, `000000003845`, `000000003934`.
- Báo `A02` và `Q03`, annotator và reviewer của job `J02`.

Nhóm lấy phiên bản guideline từ `jobs.csv` vì thông tin ở từng ảnh trong metadata có thể bị sai. Đây là danh sách cần kiểm tra theo tiêu chí truy vết, không phải danh sách ảnh đã xác nhận lỗi. Kết quả nằm trong `runs/r12007/trace.json`.

## 6. Bug hoặc điểm cần chốt

- Bản ghi trước đây trong `ANNOTATION_REVIEW.md` nêu release `lab250_823c03178b31`; kết quả F1 ở mục 1 đã được tự đo lại trên run cuối `lab250_7fa680a7413b`.
- Nhóm hiện khai báo 4 thành viên trong `TEAM.md`, trong khi `RULES.md` nêu nhóm 5–6 người; xác nhận quy mô nhóm với giảng viên.
- Đề ẩn đã có bốn file trong `submission/de_an/`: validator báo 11 lỗi luật và 9 ứng viên; review log do thành viên cung cấp có 9 dòng `confirmed`, đã gộp thành 20 finding. Trace gồm 3 ảnh tầng A, 4 ảnh tầng B và 5 người cần thông báo. Không suy ra F1 từ các số này.
