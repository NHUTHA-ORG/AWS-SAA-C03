
## 1️⃣ File Gateway

👉 **Dùng cho file-based storage**

* Giao thức: **NFS / SMB**
* Lưu file dưới dạng **object trong Amazon S3**
* Phù hợp khi:

  * Muốn **chia sẻ file** giữa on-premises và AWS
  * Lift & shift file server lên cloud
  * Ứng dụng truy cập file (PDF, ảnh, video, log…)

📌 Ví dụ:

* On-prem app → NFS → File Gateway → S3

---

## 2️⃣ Volume Gateway

👉 **Dùng cho block storage (iSCSI)**
Có **2 mode**

### 🔹 a. Stored Volumes

* **100% dữ liệu nằm on-premises**
* AWS chỉ giữ **snapshot (backup)**
* Độ trễ thấp, truy cập local
* Phù hợp:

  * App cần **low latency**
  * DR / backup on-prem storage

### 🔹 b. Cached Volumes

* **Dữ liệu chính nằm trên AWS (S3)**
* On-prem chỉ giữ **cache**
* Giảm dung lượng storage local
* Phù hợp:

  * Dataset lớn
  * Không cần toàn bộ data truy cập thường xuyên

---

## 3️⃣ Tape Gateway

👉 **Thay thế băng từ (physical tape)**

* Giao thức: **iSCSI Virtual Tape Library (VTL)**
* Tương thích với backup software hiện có (Veeam, Veritas, Commvault…)
* Virtual tape lưu trên:

  * S3
  * Glacier / Glacier Deep Archive (lưu lâu, rẻ)

📌 Dùng khi:

* Muốn **bỏ băng từ**
* Giữ nguyên **workflow backup cũ**

---

## 📊 Tóm tắt nhanh

| Loại Gateway            | Giao thức   | Dữ liệu chính | Dùng khi           |
| ----------------------- | ----------- | ------------- | ------------------ |
| File Gateway            | NFS / SMB   | S3            | File share         |
| Volume Gateway (Stored) | iSCSI       | On-prem       | Low latency, DR    |
| Volume Gateway (Cached) | iSCSI       | AWS           | Giảm local storage |
| Tape Gateway            | iSCSI (VTL) | S3 / Glacier  | Backup tape        |

👉 **Tổng cộng: 3 loại chính**, trong đó **Volume Gateway có 2 mode**.


