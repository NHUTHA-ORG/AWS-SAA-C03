# **Amazon CloudWatch**

# ⚡ STAGE 1 — ULTRA-FAST READ (30–60s)

🧠 **MEMORY ANCHORS**

* 👁️ **Giám sát** metrics, logs, events của AWS resources
* 🚨 **Phát hiện & cảnh báo** khi hệ thống bất thường
* 🔄 **Tự động phản ứng** qua alarms & integrations

**🌍 Analogy (đời thực)**
CloudWatch giống như **bảng đồng hồ + camera + chuông báo động** cho toàn bộ hệ thống AWS 🏭📊🔔

**🔑 Must-remember keywords**
Metrics · Logs · Alarms · Events · Monitoring

---

# 📝 STAGE 2 — PRE-EXAM READ

1️⃣ 🔍 SERVICE OVERVIEW

* **Amazon CloudWatch** là dịch vụ **Monitoring & Observability** cho AWS
* **WHY**: Biết hệ thống đang *sống hay chết*, *tốt hay xấu* theo thời gian thực
* **Value**:

  * Thu thập **Metrics**, **Logs**, **Events** tập trung
  * Tạo **Alarms** để cảnh báo & tự động hành động
* 🧠 Keywords: **Monitoring**, **Metrics**, **Logs**, **Alarms**

2️⃣ 🛡️ THREATS / PROBLEMS IT SOLVES

* ❌ Không biết server/app đang lỗi ở đâu
* ❌ Phát hiện sự cố quá muộn (downtime kéo dài)
* ❌ Không có dữ liệu để scale hay tối ưu chi phí

**Nếu KHÔNG dùng CloudWatch:**

* Không alert khi CPU spike, disk full, app crash
* Phải log thủ công, debug rất chậm

🧠 Keywords: **Threat**, **Detection**, **Risk**

3️⃣ 📦 USE CASES (REAL-WORLD)

* 📈 Monitor CPU, Memory, Latency của EC2 / RDS
* 📜 Centralized logging cho Lambda, ECS, API
* 🚨 Alert DevOps khi hệ thống vượt ngưỡng

**Who should use**:

* Startup → basic monitoring
* Enterprise → central observability
* DevOps / SRE / Security team

**Best fit when**:

* Cần **real-time visibility** & **alerting**

🧠 Keywords: **Use case**, **Best fit**

4️⃣ 🧠 EXAM COVERAGE & TRAPS
**Exam MUST know**:

* CloudWatch = metrics + logs + alarms
* Metrics mặc định cho nhiều AWS services
* Custom Metrics có thể push từ app

**Common traps**:

* ❌ Nhầm CloudWatch với CloudTrail
* ❌ Nghĩ CloudWatch tự audit security

**CloudWatch is NOT**:

* ❌ Audit log API calls (→ CloudTrail)
* ❌ Distributed tracing (→ X-Ray)

🧠 Keywords: **Exam tip**, **Anti-pattern**

---

# 📚 STAGE 3 — FULL UNDERSTANDING

5️⃣ 🧩 CORE COMPONENTS & ARCHITECTURE

* 📊 **Metrics** – số liệu thời gian thực (CPU, Latency)
* 📜 **Logs** – application & system logs
* 🚨 **Alarms** – trigger khi metric vượt threshold
* 📡 **Events (EventBridge)** – phản ứng theo sự kiện
* 📈 **Dashboards** – visualize tổng quan

**High-level flow**:
AWS Resource → Metrics / Logs → CloudWatch → Alarm → Action

6️⃣ 🔄 INTEGRATIONS & RELATED SERVICES

* 🔔 **SNS** – gửi alert Email / SMS
* ⚙️ **Auto Scaling** – scale EC2 tự động
* 🧠 **Lambda** – xử lý sự kiện / automation
* 🧾 **CloudTrail** – audit (bổ trợ, không thay thế)
* 🔍 **X-Ray** – tracing chi tiết

🧠 Keywords: **Integration**, **Event-driven**, **Automation**

7️⃣ ⚖️ PROS & LIMITATIONS
**✅ Benefits**

* Native AWS, bật là dùng
* Real-time monitoring
* Tích hợp rất mạnh

**⚠️ Limitations**

* UI logs không mạnh như ELK
* Metric custom tốn chi phí
* Không phải full APM

🧠 Keywords: **Benefit**, **Limitation**

8️⃣ 🧪 SCENARIOS & DECISION GUIDE

* **Choose CloudWatch when**:

  * Monitor health, performance
  * Cần alert & auto-scale

**Compare nhanh**:

* CloudWatch vs CloudTrail → *Monitoring* vs *Audit*
* CloudWatch vs X-Ray → *Metrics* vs *Tracing*
* CloudWatch vs OpenSearch → *Basic logs* vs *Advanced search*

🧠 Keywords: **Choose when**, **Compare**

---

❓ Q&A (EXAM TRAPS)
1️⃣ EC2 CPU spike → alert & scale?
👉 CloudWatch Alarm + Auto Scaling

2️⃣ Ai gọi API DeleteBucket?
👉 CloudTrail (không phải CloudWatch)

3️⃣ Muốn log Lambda execution?
👉 CloudWatch Logs (default)

4️⃣ Monitor custom business metric?
👉 CloudWatch Custom Metrics

---

🏭 MÔ HÌNH / KIẾN TRÚC PHỔ BIẾN

* 🖥️ **EC2 Monitoring**
  EC2 → CloudWatch Metrics → Alarm → SNS

* ⚙️ **Auto Scaling Pattern**
  CloudWatch Alarm → Auto Scaling Group

* 🔁 **Serverless Observability**
  Lambda → CloudWatch Logs → Alarm

* 📊 **Central Dashboard**
  Multiple services → CloudWatch Dashboard

---

---

# 🧠 CHỨC NĂNG ĐẶC BIỆT / ÍT NGƯỜI NHỚ (RẤT HAY RA THI)

🧪 **CloudWatch Synthetics – Canaries** 🐦

* **What**: Chạy **script giả lập người dùng** (synthetic user) theo lịch
* **Why**: Phát hiện lỗi **trước khi user thật gặp**
* **Test được**:

  * Website uptime (HTTP check)
  * API endpoints
  * Login flow / checkout flow
* **Output**:

  * Metrics (latency, success rate)
  * Screenshots, HAR files
  * Logs + Alarms

🧠 Keywords: **Canary**, **Synthetic monitoring**, **Proactive detection**

**Exam tip** ⚠️

* User hỏi: *"Làm sao kiểm tra website/app hoạt động đúng ngay cả khi chưa có traffic?"*
  👉 **CloudWatch Synthetics Canaries** (KHÔNG phải ALB Health Check)

---

🔍 **CloudWatch Contributor Insights**

* **What**: Phân tích **ai / cái gì đóng góp nhiều nhất** vào metric/log
* **Use**:

  * Top IP gây nhiều request
  * Top user gây nhiều error
* **Best for**: Phát hiện bottleneck, traffic bất thường

🧠 Keywords: **Top contributors**, **Analysis**

---

📜 **CloudWatch Logs Insights** 🔎

* **What**: Query logs bằng ngôn ngữ riêng (không cần index trước)
* **Why**: Debug nhanh, tìm error patterns
* **Exam trap**:

  * Không phải OpenSearch
  * Không dùng cho full-text search lâu dài

🧠 Keywords: **Query**, **Log analysis**

---

⏱️ **High-Resolution Metrics**

* Metrics với độ phân giải **1s** (thay vì 1 phút)
* Dùng cho system cần phản ứng rất nhanh
* ⚠️ Có **chi phí cao hơn**

🧠 Keywords: **1-second metrics**, **High resolution**

---

🤖 **Anomaly Detection** 📉📈

* CloudWatch tự học baseline
* Alert khi metric **lệch khỏi hành vi bình thường**
* Không cần set threshold cứng

🧠 Keywords: **ML-based**, **Baseline**, **Anomaly**

---

🎯 FINAL TAKEAWAY (UPDATED)

* **CloudWatch không chỉ là Metrics + Alarms**
* Có thể:

  * 🐦 Test app trước user (Canaries)
  * 🧠 Tự phát hiện bất thường (Anomaly Detection)
  * 🔍 Phân tích log & contributor rất nhanh

👉 Nếu đề thi nhấn mạnh: *"chủ động phát hiện lỗi, không chờ user báo"*
👉 **CloudWatch Synthetics (Canaries)**
