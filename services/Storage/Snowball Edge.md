# **Amazon Snowball Edge**

# ⚡ STAGE 1 — ULTRA-FAST READ (30–60s)

🧠 MEMORY ANCHORS

🔹 **Tóm tắt nhanh**

* 📦 **Physical device** để **transfer dữ liệu lớn** (TB–PB) offline
* 🚚 Thay Internet bằng **shipping** → nhanh, ổn định, bảo mật
* 🧠 Có thể **compute tại chỗ (edge computing)**

🔹 **Ví dụ đời thực**

> Giống như **chuyển nhà bằng xe tải** 🚛 thay vì gửi từng món qua bưu điện 📮

🔹 **Must-remember keywords**
**Offline**, **Petabyte-scale**, **Edge computing**, **Encryption**, **Shipping**

# 📝 STAGE 2 — PRE-EXAM READ

1️⃣ 🔍 SERVICE OVERVIEW

* **Amazon Snowball Edge** là **physical data transfer device** của AWS
* Mục đích: **di chuyển dữ liệu khổng lồ** khi mạng **chậm / đắt / không khả thi**
* Giá trị cốt lõi:

  * 🚀 Faster than network
  * 🔐 Built-in **Encryption**
  * 🧠 Hỗ trợ **local compute & storage**
* 🧠 Keywords: **Offline transfer**, **Edge computing**, **Petabyte-scale**

2️⃣ 🛡️ THREATS / PROBLEMS IT SOLVES

* Vấn đề thường gặp:

  * 🌐 Băng thông Internet thấp
  * 💸 Chi phí truyền dữ liệu quá cao
  * ⏳ Migration mất nhiều tháng
* Rủi ro nếu KHÔNG dùng:

  * Migration delay
  * Business downtime
  * Infeasible data transfer
* Snowball Edge giải quyết bằng:

  * 🔐 **Encryption at rest**
  * 📦 **Tamper-resistant device**
* 🧠 Keywords: **Risk**, **Offline**, **Secure transfer**

3️⃣ 📦 USE CASES (REAL-WORLD)

* 🏢 **Data center migration** lên AWS
* 📸 Media, genomics, IoT thu thập dữ liệu tại biên
* 🌍 Vị trí remote (oil rig, ship, factory)
* Ai nên dùng:

  * Enterprise
  * Research / Media
  * Hybrid / Edge workloads
* Khi là BEST choice:

  * > 10–50 TB
  * Network không ổn định
* 🧠 Keywords: **Use case**, **Best fit**

4️⃣ 🧠 EXAM COVERAGE & TRAPS

* Cần nhớ cho exam:

  * Snowball = **offline + shipping**
  * Có **compute capability** (EC2-like)
* Trap thường gặp:

  * ❌ Dùng Snowball cho realtime data
  * ❌ Nhầm với **AWS DataSync** hoặc **S3 Transfer Acceleration**
* Snowball Edge KHÔNG dùng cho:

  * Low-latency
  * Continuous sync
* 🧠 Keywords: **Exam tip**, **Anti-pattern**

# 📚 STAGE 3 — FULL UNDERSTANDING

5️⃣ 🧩 CORE COMPONENTS & ARCHITECTURE

* 📦 **Snowball Edge Device**

  * Storage + Compute
* 🔐 **Encryption (KMS)**

  * Data auto-encrypted
* 🚚 **Shipping workflow**

  * Order → Ship → Load → Return
* ☁️ **AWS Import to S3 / EBS**
* 🧠 Keywords: 📦 Device | 🔐 KMS | ☁️ S3

6️⃣ 🔄 INTEGRATIONS & RELATED SERVICES

* **Amazon S3**: data destination
* **Amazon EC2**: edge compute workloads
* **AWS IAM**: access control
* **AWS DataSync**: alternative online transfer
* **NFS (Network File System)**: file-level access
* 🧠 Keywords: **Integration**, **Hybrid**, **Automation**

7️⃣ ⚖️ PROS & LIMITATIONS

* ✅ Advantages:

  * Very fast for large data
  * Secure by default
  * Works without Internet
* ❌ Limitations:

  * Physical logistics
  * Not real-time
  * Requires planning
* 🧠 Keywords: **Benefit**, **Limitation**

8️⃣ 🧪 SCENARIOS & DECISION GUIDE

* Chọn **Snowball Edge** khi:

  * Massive data
  * Poor connectivity
* So sánh nhanh:

  * Snowball vs DataSync → Offline vs Online
  * Snowball vs Snowmobile → PB vs EB scale
* 🧠 Keywords: **Choose when**, **Compare**

❓ Q&A (EXAM-FOCUSED)

* Q: Dữ liệu 200 TB, Internet chậm → chọn gì?

  * A: **Snowball Edge** (offline, faster)
* Q: Realtime sync on-prem → AWS?

  * A: ❌ Not Snowball → **DataSync**
* Q: Cần compute tại site remote?

  * A: ✅ Snowball Edge

🏭 MÔ HÌNH / KIẾN TRÚC PHỔ BIẾN

* 📦➡️☁️ **Data Center Migration**

  * On-prem ➝ Snowball ➝ S3
* 🧠📦 **Edge Processing**

  * Sensor ➝ Snowball Edge ➝ S3
* 🚢🌍 **Remote Location**

  * Ship / Rig ➝ Snowball ➝ AWS

🧠 CHỨC NĂNG ĐẶC BIỆT

* **Edge Computing** 🧠

  * Run EC2 / Lambda locally
  * Use case: preprocess data

* **Built-in Encryption** 🔐

  * Always-on, KMS-managed
  * Exam note: no manual setup needed

* **Offline-first Design** 📦

  * Designed for zero / low connectivity

* **NFS Support (VERY IMPORTANT FOR EXAM)** 📂

  * Là gì:

    * Snowball Edge **supports NFS interface** để mount như file server
  * Use case:

    * Legacy applications chỉ hỗ trợ **file-based (NFS)**
    * Backup / migration từ on-prem **NAS / file server**
  * Ghi chú exam:

    * ❗ Snowball ≠ object-only
    * ❗ NFS dùng để **copy file**, không phải shared filesystem lâu dài
