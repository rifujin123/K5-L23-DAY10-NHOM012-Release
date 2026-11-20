# Người 1: Data Validation - Lê Tuấn Khôi

## Mục tiêu

Xây dựng phần kiểm tra tự động các lỗi theo luật của lô dữ liệu. Validator phải chạy được trên lô luyện tập và đề ẩn, xuất lỗi theo đúng định dạng để nhóm review và bàn giao.

## Đầu vào

Thư mục `delivery/` được truyền vào, gồm manifest, metadata, jobs, classes, ảnh, nhãn detection và dữ liệu panoptic.

## Phụ thuộc và điểm dừng cho agent

- **Cần trước khi chạy trên dữ liệu thật:** `runs/r<seed>/delivery/` đã được tạo và nhóm đã chốt seed. Nếu chưa có, có thể đọc đặc tả và chuẩn bị logic; không tự chọn seed hoặc tự tạo dữ liệu thay nhóm khi chưa có thông tin.
- **Có thể làm song song với Người 2:** viết các kiểm tra luật. Trước khi cùng sửa `validator/validate.py`, hai người cần chốt giao diện module và ai tích hợp entrypoint; không ghi đè phần của người khác.
- **Cần từ Người 2 để nghiệm thu tích hợp:** chức năng tạo candidates và giao diện ghép vào entrypoint. Nếu chưa có, tiếp tục kiểm tra phần luật độc lập, ghi rõ tích hợp còn chờ.
- **Bàn giao:** `rules.json` sinh từ lô thật cho Người 2. Không tự tạo lỗi giả hoặc báo hoàn thành tích hợp khi artifact chưa sẵn sàng.

## Yêu cầu chức năng

| Mã | Loại lỗi | Cần kiểm tra |
|---|---|---|
| VAL-01 | `missing_image` | Manifest khai báo ảnh nhưng thiếu file. |
| VAL-02 | `wrong_hash` | Tính lại SHA-256 của file và so với manifest. |
| VAL-03 | `invalid_class` | Class ID detection không hợp lệ. |
| VAL-04 | `bbox_out_of_bounds` | Box vượt khỏi ảnh. |
| VAL-05 | `duplicate_box` | Box cùng lớp, cùng kích thước và gần như trùng vị trí. |
| VAL-06 | `scene_leak` | Một scene xuất hiện ở cả train và validation. |
| VAL-07 | `guideline_mismatch` | Guideline của ảnh khác guideline ghi trong job. |
| VAL-08 | `invalid_category_id` | Category ID panoptic không tồn tại trong `categories`. |
| VAL-09 | `thing_overlap` | Đếm pixel của segment và đối chiếu với `area` khai báo theo quy tắc lab. |

## Giao diện và đầu ra

Phối hợp với Người 2 để cung cấp một giao diện thống nhất:

```text
python validator/validate.py <delivery> --out <errors.json> [--candidates <candidates.json>]
```

Kết quả lỗi luật có thể ghi riêng vào `rules.json` để gộp với review. Định dạng:

```json
{"errors": [{"type": "missing_image", "target": "ten_anh"}]}
```

`scene_leak` dùng `scene_id` làm target; các loại khác dùng tên ảnh không có đuôi. Bàn giao `rules.json` cho Người 2 và thống nhất giao diện tích hợp trước khi viết sâu.

## Ràng buộc

- Validator chỉ đọc `delivery/` được truyền vào; không đọc `_private/`, `data_source/`, annotation COCO và không dùng mạng.
- Chỉ dùng thư viện chuẩn Python; được import `scripts/pngutil.py`.
- Không gắn cứng seed, tên ảnh hoặc đáp án.
- Tính hash trên bytes; kiểm tra box trùng bằng lớp, vị trí và kích thước, không chỉ dùng IoU thấp.
- Không sửa `scripts/` hoặc dữ liệu v1.

## Kiểm tra và nghiệm thu

- [ ] Hỗ trợ đủ 9 loại lỗi, đúng `type` và `target`.
- [ ] Chạy trên ít nhất 2 seed; mục tiêu F1 lỗi luật khoảng 1.0 theo checkpoint của lab.
- [ ] Chạy trên lô sạch không báo lỗi luật.
- [ ] Chạy được với tên ảnh dạng `img_0001` và tích hợp được với phần candidates.
- [ ] Giải thích được logic hash, box, scene leak và guideline theo job.
- [ ] Bàn giao kết quả đúng hạn để nhóm tổng hợp `errors.json`.

## Việc chung

Tham gia xem ảnh, làm đề ẩn, ghi lại việc dùng AI và có ít nhất 2 commit có nội dung. Xem [định dạng lỗi](../docs/DELIVERY.md), [hướng dẫn](../GUIDE.md) và [quy tắc](../RULES.md).
