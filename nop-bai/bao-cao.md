# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| **Họ và tên** | Lê Duy Đông |
| **MSSV** | 2A202601282 |
| **Lớp / Khóa** | K4 |
| **Repo GitHub** | https://github.com/leeduydong2005/K4-Track2-DAY21-2A202601282-LeDuyDong |
| **Ngày nộp** | 21/08/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.8780 |
| 2 | 50 | 0.05 | 2 | 0.6051 | 0.8460 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.8740 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Tôi chọn bộ tham số ở Lần 3 vì đạt điểm `f1_score` cao nhất (0.7149), vượt qua ngưỡng 0.65. Ban đầu ở Lần 1, mô hình có accuracy cao nhất (0.8780) nhưng F1 lại thấp hơn Lần 3. Đến Lần 2 khi giảm số cây và độ sâu để mô hình nhẹ hơn thì F1 tụt mạnh xuống 0.6051 (dưới ngưỡng) dù accuracy vẫn giữ ở mức 0.8460. Qua đó, tôi thấy rõ sự đánh đổi giữa learning_rate và n_estimators: với bài toán phân loại nhị phân này, mô hình GradientBoosting cần đủ độ sâu và số lượng cây để học được các đặc trưng phức tạp của nhóm thiểu số.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Tập dữ liệu Adult mất cân bằng rõ rệt khi chỉ có 24.8% người có thu nhập cao (>50K). Nếu xây dựng một mô hình "lười biếng" luôn gán nhãn thu nhập thấp cho mọi trường hợp thì độ chính xác (Accuracy) vẫn đạt tới 75.2%, tạo cảm giác mô hình rất tốt nhưng thực chất hoàn toàn vô dụng vì bỏ sót 100% đối tượng cần tìm.

Chỉ số `f1_score` tính riêng cho lớp dương (target = 1) giải quyết triệt để vấn đề này nhờ cân bằng giữa Precision (độ chuẩn xác) và Recall (độ bao phủ). Tôi không truyền tham số `average="weighted"` hay `average="macro"` vì các trọng số này sẽ bị lớp đa số (75.2%) kéo lên cao, khiến ngưỡng chặn Quality Gate mất đi tác dụng bảo vệ hệ thống.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| Lỗi build scikit-learn khi cài đặt trên máy local | Máy dùng Python 3.13 nên scikit-learn 1.4.2 cũ chưa có sẵn file wheel nhị phân | Đổi ràng buộc trong requirements.txt sang `>=1.5.2` để tương thích cả Python 3.10 trên CI và Python 3.13 local |
| Không tạo được bucket do lỗi Billing trên GCP | Project GCP mới chưa liên kết tài khoản thanh toán | Chuyển sang hạ tầng AWS (S3 và EC2) theo đúng hướng dẫn tương đương của lab |
| Lỗi unpickle module `_loss` và bảo mật SSH key | Khác biệt cấu trúc nội bộ giữa các bản sklearn và lỗi định dạng SSH key trên GitHub Secrets | Thêm alias tương thích module trong `src/serve.py` và copy chuẩn toàn bộ private key bằng PowerShell |

---

## 4. So Sánh Bước 2 và Bước 3 & Vai Trò Của Human-in-the-Loop (HITL)

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7149 | 0.8740 |
| Bước 3 (thêm `train_batch2`) | 0.7138 | 0.8760 |

**Nhận xét:** Khi bổ sung thêm 22.361 mẫu ở Bước 3, F1 giảm nhẹ khoảng 0.0011. Điều này rất hợp lý vì hai tập dữ liệu được chia ngẫu nhiên từ cùng một nguồn nên việc tăng kích thước dữ liệu cùng phân phối không tạo thêm đột phá. 

Tuy nhiên, bài học lớn nhất ở đây là vai trò của **Human-in-the-Loop (HITL)** trong MLOps: tự động hóa CI/CD giúp tăng tốc triển khai, nhưng con người vẫn đóng vai trò trung tâm trong việc thiết kế Quality Gate, kiểm soát trôi dạt dữ liệu (data drift), và quyết định xem khi nào cần can thiệp tái cấu trúc đặc trưng (feature engineering) thay vì chỉ tin tưởng mù quáng vào việc nạp thêm dữ liệu.
