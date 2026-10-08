# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Nguyễn Huy Hoàng |
| MSSV | 2A202602738 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/Hoang-H-Nguyen/K4-L3-DAY21-NguyenHuyHoang-2A202602738-CI-CD-for-AI-Systems |
| Ngày nộp | 08/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---:|---:|---:|---:|---:|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 2 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.8740 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Lần 3 có F1 cao nhất (0.7149), hơn lần 1 (0.7109) và vượt ngưỡng 0.65. Accuracy cao nhất lại thuộc lần 1 (0.8780), cho thấy accuracy không đại diện đầy đủ cho khả năng nhận diện lớp thu nhập cao. Lần 2 với ít cây, cây nông và learning rate thấp có F1 thấp nhất. Ba lần chạy gợi ý sự đánh đổi giữa số cây và tốc độ học, nhưng chưa đủ để kết luận tổng quát.

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Lớp dương (thu nhập trên 50K) chiếm 24,8%, lớp âm chiếm 75,2%. Nếu luôn đoán “thu nhập thấp”, accuracy vẫn khoảng 75,2% dù bỏ sót toàn bộ người thu nhập cao. F1 lớp dương kết hợp precision và recall, cho biết mô hình nhận diện nhóm này tốt đến đâu và kiểm soát dự đoán dương sai thế nào. Vì cần đánh giá riêng lớp thiểu số, mã dùng `f1_score(y_eval, preds)` cho bài toán nhị phân, không dùng `average="macro"` hay `average="weighted"` để gộp điểm hai lớp. Quality Gate yêu cầu F1 ít nhất 0.65.

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| Cài scikit-learn trên Python 3.13 lỗi | Phiên bản ghim chưa hỗ trợ Python đó | Tạo lại môi trường bằng Python 3.12 và ghim dependency tương thích MLflow. |
| GitHub chặn push vì khóa GCP trong commit DVC | `sa-key.json` từng bị stage trước khi thêm ignore | Thu hồi khóa cũ, tạo khóa mới, xóa khỏi lịch sử chưa push và lưu bằng GitHub Secret. |
| Release SSH timeout | VM đã ở trạng thái dừng, IP công khai đổi sau khi khởi động | Bật lại VM, cập nhật `SERVER_HOST`, chạy lại pipeline và xác nhận health check. |

## 4. So Sánh Bước 2 và Bước 3

| | f1_score | accuracy |
|---|---:|---:|
| Bước 2 (22.361 mẫu) | 0.7149 | 0.8740 |
| Bước 3 (44.722 mẫu) | 0.7354 | 0.8820 |

**Nhận xét:** Sau khi thêm 22.361 mẫu cùng nguồn, F1 tăng 0.0205 và accuracy tăng 0.0080; cả hai run đều qua ngưỡng. Bước 3 chạy thủ công trên commit dữ liệu vì GitHub không tự tạo push run. Ảnh chứng minh bốn job thành công, nhưng chưa chứng minh kích hoạt tự động.
