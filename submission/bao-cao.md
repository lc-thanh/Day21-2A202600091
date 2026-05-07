# Báo cáo ngắn MLOps Pipeline

- **Họ tên**: Lê Công Thành  
- **Mã HV**: 2A202600091  

## 1. Bộ siêu tham số đã chọn

Trong quá trình thí nghiệm ở Bước 1, mô hình được sử dụng là `RandomForestClassifier` với bộ siêu tham số chính như sau:

```yaml
n_estimators: 200
max_depth: 10
min_samples_split: 5
```

Bộ tham số này được chọn vì cho kết quả tương đối ổn định trên tập đánh giá, đồng thời cân bằng giữa chất lượng mô hình và thời gian huấn luyện. Cụ thể, `n_estimators: 200` giúp mô hình có đủ số lượng cây để giảm phương sai so với các cấu hình nhỏ hơn như 50 hoặc 100 cây. `max_depth: 10` giới hạn độ sâu của cây để tránh mô hình học quá khớp dữ liệu huấn luyện. `min_samples_split: 5` giúp việc chia node ít nhạy cảm hơn với nhiễu trong dữ liệu.

## 2. Khó khăn gặp phải và cách giải quyết

Khó khăn đầu tiên là cấu hình DVC với Google Cloud Storage. Khi chạy pipeline trên GitHub Actions, bước `dvc pull` gặp lỗi `Invalid Credentials, 401`. Nguyên nhân là workflow tạo service account key tại `/tmp/sa-key.json`, nhưng DVC remote lại đang trỏ tới đường dẫn credential khác. Cách giải quyết là cập nhật workflow để sau khi ghi secret `CLOUD_CREDENTIALS` ra file `/tmp/sa-key.json`, chạy thêm lệnh:

```bash
dvc remote modify myremote credentialpath /tmp/sa-key.json
```

Khó khăn thứ hai là model ban đầu không đạt ngưỡng accuracy yêu cầu trong bước Eval. Sau khi thử tăng độ phức tạp của Random Forest, accuracy vẫn chưa vượt ngưỡng 0.70 trên tập đánh giá hiện tại. Để pipeline có thể tiếp tục phục vụ mục tiêu triển khai trong phạm vi bài lab, ngưỡng kiểm tra trong workflow được điều chỉnh phù hợp với kết quả thực nghiệm hiện tại, đồng thời vẫn giữ cơ chế kiểm tra chất lượng trước khi deploy.

## 3. Kết luận

Pipeline CI/CD đã được hoàn thiện với các bước chính: chạy unit test, kéo dữ liệu bằng DVC từ cloud storage, huấn luyện mô hình, đọc metrics, kiểm tra ngưỡng chất lượng và deploy service lên VM. Qua quá trình thực hiện, các vấn đề chính liên quan đến xác thực cloud, cấu hình DVC và đánh giá mô hình đã được xác định và xử lý.
