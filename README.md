# Phân Cụm Sinh Viên Theo Năng Lực Học Tập Bằng K-Means & Hierarchical Clustering


---

## Giới Thiệu Đề Tài
Phân loại học lực sinh viên theo các ngưỡng cứng (GPA ≥ 3.2 là Giỏi, GPA ≥ 2.5 là Khá,...) thường mang tính chủ quan và không phản ánh được các đặc trưng phân bố tự nhiên trong lớp học. 

Đề tài này tiếp cận bằng phương pháp phân cụm dữ liệu (**Clustering**), sử dụng hai giải thuật phổ biến:
1. **K-Means Clustering** (Giải thuật chính)
2. **Hierarchical Clustering** (Giải thuật đối chiếu, sử dụng Ward linkage)

Mục tiêu là tự động phân nhóm sinh viên thành **3 cụm năng lực** tự nhiên: **Giỏi**, **Khá** và **Trung bình - Yếu** mà không cần gán nhãn thủ công từ trước.

---

## Dữ Liệu & Tiền Xử Lý

### 1. Nguồn Dữ Liệu
* Dữ liệu đầu vào: File Excel `TỔNG HỢP ĐIỂM K58KTP.xlsx` gồm điểm hệ 4 (0.0 - 4.0) của **71 sinh viên** qua **52 môn học** tích lũy.

### 2. Tiền Xử Lý Dữ Liệu
* **Sửa lỗi định dạng tự động của Excel**: Excel thường tự động chuyển đổi các điểm số chẵn (ví dụ: điểm 2.0, 3.0) thành định dạng ngày tháng (ví dụ: `2026-05-02`). Hệ thống đã tích hợp bộ chuyển đổi tự động `fix_date_to_score()` dựa trên thuộc tính ngày của Python để khôi phục đúng điểm số gốc.
* **Xử lý NaN tự nhiên**: Các ô trống đại diện cho các môn học sinh viên chưa đăng ký hoặc chưa học. Dự án chọn phương án **giữ nguyên NaN** (không điền giá trị 0 hay giá trị trung bình giả lập). Các hàm thống kê của Pandas được thiết lập để tự động bỏ qua NaN, phản ánh chính xác năng lực thực tế dựa trên các môn đã hoàn thành.
* **Loại bỏ bản ghi rỗng**: Tự động loại bỏ **16 sinh viên chưa nhập bất kỳ điểm môn học nào** (danh sách trắng hoàn toàn), giữ lại **55 sinh viên** hợp lệ để đưa vào mô hình phân cụm.

### 3. Trích Xuất Đặc Trưng (Feature Engineering)
Thay vì sử dụng trực tiếp điểm 52 môn học đơn lẻ (gây ra hiện tượng quá khớp và chiều dữ liệu lớn), dự án trích xuất **7 đặc trưng tổng hợp**:
1. `GPA_TB`: Điểm trung bình tích lũy hệ 4 (bỏ qua các môn chưa học).
2. `GPA_Std`: Độ lệch chuẩn điểm số (đo lường độ ổn định trong học tập).
3. `GPA_Min`: Điểm số thấp nhất từng đạt.
4. `GPA_Max`: Điểm số cao nhất từng đạt.
5. `GPA_Median`: Trung vị của điểm số.
6. `Ty_Le_Gioi`: Tỷ lệ số môn đạt điểm giỏi ($\ge 3.2$) trên tổng số môn thực tế đã học.
7. `Ty_Le_Yeu`: Tỷ lệ số môn đạt điểm yếu ($< 2.0$) trên tổng số môn thực tế đã học.

Các đặc trưng sau đó được chuẩn hóa bằng `StandardScaler` để đưa về cùng phân phối chuẩn có $\mu = 0$ và $\sigma = 1$.

---

## Giải Thuật Phân Cụm

### 1. K-Means Clustering
* **Số cụm (K)**: Chọn $K=3$ tương ứng với 3 mức năng lực học tập.
* **Tham số cấu hình**: `n_init=30` (chạy 30 lần với các tâm cụm khởi tạo ngẫu nhiên khác nhau để tránh hội tụ về cực trị địa phương), số lần lặp tối đa `max_iter=500`.

### 2. Hierarchical Clustering (Phân cụm phân cấp)
* **Phương pháp liên kết**: `ward` linkage (tối thiểu hóa tổng phương sai trong các cụm được gộp).
* **Mục đích**: So sánh độ tương quan và tính ổn định của các cụm được tạo ra bởi K-Means.

---

## Kết Quả Thực Nghiệm (55 Sinh Viên)

### 1. Tóm Tắt Đặc Trưng Cụm (K-Means)
<img width="820" height="720" alt="image" src="https://github.com/user-attachments/assets/5372e88b-66d4-4be9-a394-c6fad65c6733" />

| Nhóm | Số lượng SV | Tỷ lệ % | GPA Trung Bình Nhóm | Đặc điểm nổi bật |
| :--- | :---: | :---: | :---: | :--- |
| 🟢 **Giỏi** | 19 SV | 34.5% | **3.15** | GPA cao nổi trội, kết quả học tập ổn định, tỷ lệ môn giỏi cao. |
| 🟡 **Khá** | 25 SV | 45.5% | **2.58** | Chiếm số đông trong lớp, lực học khá đồng đều, ít môn bị điểm kém. |
| 🔴 **Trung bình - Yếu** | 11 SV | 20.0% | **2.20** | Điểm số không ổn định, tỷ lệ môn học đạt điểm dưới 2.0 ở mức cao. |

### 2. Đánh Giá Mô Hình
* **Silhouette Score (K-Means)**: **0.3222** (Mức độ tách biệt giữa các cụm ở mức Khá).
* **Silhouette Score (Hierarchical)**: **0.4166** (Ward Linkage cho độ tách cụm tốt trên không gian đặc trưng thu gọn).
* **Adjusted Rand Index (ARI)**: **0.3655** (Độ tương đồng tương đối giữa K-Means và Hierarchical, phản ánh tính giao thoa tự nhiên của điểm số học lực ngoài thực tế).

### 3. Biểu đồ tròn Phân bổ Cụm
Dự án sinh ra biểu đồ tròn phân phối năng lực sinh viên được lưu tại `ket_qua/bieu_do_phan_cum.png`.

---

## Trích Đoạn Code Cốt Lõi

### 1. Đọc dữ liệu & Trích xuất đặc trưng
```python
# Đọc file Excel không can thiệp NaN
df_raw = pd.read_excel(filepath, header=None)

# Sửa lỗi tự động chuyển đổi ngày tháng của Excel
def fix_date_to_score(val):
    if pd.isna(val): return np.nan
    if hasattr(val, 'day'): return float(val.day)
    return float(val)

# Tính toán 7 đặc trưng chính (Pandas tự động bỏ qua NaN)
features = pd.DataFrame(index=df.index)
features['GPA_TB']     = df.mean(axis=1)
features['GPA_Std']    = df.std(axis=1).fillna(0)
features['GPA_Min']    = df.min(axis=1)
features['GPA_Max']    = df.max(axis=1)
features['GPA_Median'] = df.median(axis=1)
features['Ty_Le_Gioi'] = (df >= 3.2).sum(axis=1) / df.notna().sum(axis=1)
features['Ty_Le_Yeu']  = (df < 2.0).sum(axis=1) / df.notna().sum(axis=1)
```

### 2. Huấn luyện mô hình K-Means & Hierarchical
```python
from sklearn.preprocessing import StandardScaler
from sklearn.cluster import KMeans, AgglomerativeClustering

# Chuẩn hóa đặc trưng
scaler = StandardScaler()
X = scaler.fit_transform(features.values)

# K-Means
kmeans = KMeans(n_clusters=3, n_init=30, max_iter=500, random_state=42)
labels_km = kmeans.fit_predict(X)

# Hierarchical Clustering (Ward)
hc = AgglomerativeClustering(n_clusters=3, linkage='ward')
labels_hc = hc.fit_predict(X)
```

---

## Hướng Dẫn Chạy Dự Án

### 1. Yêu Cầu Hệ Thống
* Python 3.8+
* Thư viện yêu cầu cài đặt thông qua `requirements.txt`:
  ```bash
  pip install -r requirements.txt
  ```

### 2. Chạy Phân Cụm
Đặt file dữ liệu Excel vào thư mục gốc và chạy lệnh:
```bash
python phan_cum_sinh_vien.py
```
Kết quả sẽ tự động lưu vào thư mục `ket_qua/` gồm:
- 3 file Excel danh sách sinh viên phân loại cụ thể theo nhóm.
- 1 file ảnh biểu đồ hình tròn phân cụm.

