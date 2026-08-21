# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Nguyễn Huy Hưng |
| MSSV | 2A202601204 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/HoangTuBangGia/K4-Track2-Day21-2A202601204-NguyenHuyHung |
| Ngày nộp | 21/08/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---:|---:|---:|---:|---:|
| 1 | 100 | 0.1 | 3 | 0.7109 | 0.878 |
| 2 | 50 | 0.05 | 2 | 0.6051 | 0.846 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.874 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Lần chạy 3 được chọn vì có F1 cao nhất, 0.7149, và vượt ngưỡng 0.65. Lần chạy 1 có accuracy cao nhất (0.878) nhưng F1 thấp hơn, cho thấy accuracy tốt nhất không đồng nghĩa nhận diện lớp thu nhập cao tốt nhất. Với learning rate 0.05 và chỉ 50 cây, mô hình học chưa đủ nên F1 giảm còn 0.6051. Tăng lên 200 cây với learning rate 0.1 giúp boosting sửa lỗi qua nhiều vòng hơn, đổi lại thời gian huấn luyện tăng.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Tập Adult mất cân bằng: chỉ 24,8% mẫu thuộc lớp thu nhập trên 50K. Mô hình luôn đoán “thu nhập thấp” vẫn đạt accuracy 0.752 nhưng không phát hiện được trường hợp thu nhập cao nào. F1 của lớp dương kết hợp precision và recall, phản ánh cả độ chính xác của dự đoán dương và khả năng tìm được lớp dương thực tế. Vì vậy Quality Gate kiểm tra `f1_score >= 0.65`. Lab dùng `f1_score(y_eval, preds)` mặc định cho lớp dương, không dùng `average="weighted"` hay `average="macro"`, vì phép tổng hợp có thể pha loãng ảnh hưởng của lớp cần quan tâm và khiến kết quả gây hiểu nhầm.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| MLflow báo thiếu `pkg_resources` | MLflow 2.13.0 chưa tương thích với setuptools mới | Ghim `setuptools<81` trong dependencies |
| Không thể tạo service-account JSON key | GCP áp policy `iam.disableServiceAccountKeyCreation` | Dùng ADC cục bộ và Workload Identity Federation cho GitHub Actions |
| Commit `.dvc` ban đầu không kích hoạt pipeline | Mẫu đường dẫn workflow chưa khớp file `.dvc` trực tiếp trong `data/` | Đổi trigger thành `data/*.dvc` và `data/**/*.dvc` |

---

## 4. So Sánh Bước 2 và Bước 3

| | f1_score | accuracy |
|---|---:|---:|
| Bước 2 (chỉ `train_batch1`) | 0.7149 | 0.874 |
| Bước 3 (thêm `train_batch2`) | 0.7354 | 0.882 |

**Nhận xét:** Khi tăng dữ liệu từ 22.361 lên 44.722 mẫu, F1 tăng 0.0205 và accuracy tăng 0.008. Batch mới giúp mô hình khái quát tốt hơn trong lần chạy này, nhưng thêm dữ liệu không phải lúc nào cũng cải thiện chỉ số vì còn phụ thuộc chất lượng và phân phối. Commit dữ liệu đã tự động đi qua cả bốn job mà không cần triển khai thủ công.
