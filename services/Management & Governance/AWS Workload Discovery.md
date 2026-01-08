# **AWS Workload Discovery (on AWS)**

# ⚡ STAGE 1 — ULTRA-FAST READ (30–60s)

🧠 MEMORY ANCHORS

* 🔍 **Automatically discovers** AWS resources across **multiple accounts & Regions**
* 🗺️ **Maps relationships** between workloads (compute, network, storage)
* ⚡ **Fast inventory & visualization** with minimal setup (agentless)

🏙️ Real-world analogy:

> Giống như **Google Maps cho AWS** — quét toàn bộ “thành phố cloud” và vẽ ra các tuyến đường kết nối giữa tài nguyên

🔑 Must-remember keywords:

* **Discovery**
* **Inventory**
* **Relationship mapping**
* **Multi-account**
* **Visualization**

---

# 📝 STAGE 2 — PRE-EXAM READ

1️⃣ 🔍 SERVICE OVERVIEW

* **AWS Workload Discovery** là gì?

  * Một **AWS Solution** giúp **tự động khám phá (discover)** và **lập bản đồ (map)** các workload đang chạy trên AWS
* WHY – Tại sao tồn tại?

  * Giải quyết bài toán **không biết đang có những resource nào** và **chúng liên quan với nhau ra sao**
* Key value:

  * Nhanh 🚀, trực quan 🗺️, **không cần cài agent**
* 🧠 Keywords:

  * **Discovery**, **Visualization**, **Inventory**, **Workload**

2️⃣ 🛡️ THREATS / PROBLEMS IT SOLVES

* Vấn đề phổ biến:

  * ❌ Không có **resource inventory**
  * ❌ Không hiểu **dependency giữa các service**
  * ❌ Khó audit, migrate, optimize
* Tình huống thường gặp:

  * Nhân sự cũ nghỉ việc, **không có tài liệu kiến trúc**
* Nếu KHÔNG dùng:

  * Dễ xóa nhầm resource, migrate sai, cấu hình thiếu HA
* 🧠 Keywords:

  * **Risk**, **Visibility**, **Dependency**

3️⃣ 📦 USE CASES (REAL-WORLD)

* 🏢 Enterprise nhiều account / Region
* 🔁 Chuẩn bị **migration, modernization, cost optimization**
* 🔍 Audit & hiểu hệ thống hiện tại
* BEST choice khi:

  * Cần **toàn cảnh nhanh**, không cần chi tiết mức OS
* Ai nên dùng?

  * **Solutions Architect**, **Cloud team**, **Platform team**
* 🧠 Keywords:

  * **Use case**, **Best fit**

4️⃣ 🧠 EXAM COVERAGE & TRAPS

* Exam rất hay hỏi:

  * “Discover & map workloads across accounts” ➝ **AWS Workload Discovery**
* Trap thường gặp:

  * ❌ Nhầm với **AWS Application Discovery Service** (on-prem discovery)
  * ❌ Nhầm với **AWS Config** (compliance, not visualization)
* NOT dùng cho:

  * ❌ Threat detection
  * ❌ Performance monitoring
* 🧠 Keywords:

  * **Exam tip**, **Anti-pattern**

---

# 📚 STAGE 3 — FULL UNDERSTANDING

5️⃣ 🧩 CORE COMPONENTS & ARCHITECTURE

* 🧠 **Discovery Engine** 🔍

  * Quét AWS APIs để thu thập metadata
* 🗂️ **Resource Inventory** 📦

  * EC2, VPC, Subnet, ALB, RDS, Security Group…
* 🗺️ **Relationship Mapping**

  * Hiển thị dependency giữa compute – network – data
* 📊 **Visualization Dashboard**

  * Xem kiến trúc dạng sơ đồ

Flow tổng quan:

> AWS Accounts ➝ API Discovery ➝ Inventory ➝ Relationship Map ➝ Visual Diagram

6️⃣ 🔄 INTEGRATIONS & RELATED SERVICES

* **AWS Organizations** – multi-account discovery
* **IAM** – cross-account access (read-only)
* **Amazon EC2 / VPC / RDS / ELB** – data sources
* Bổ trợ tốt cho:

  * **Migration Hub**, **Well-Architected Review**
* 🧠 Keywords:

  * **Integration**, **Automation**, **Multi-account**

7️⃣ ⚖️ PROS & LIMITATIONS

* ✅ Pros:

  * Nhanh, dễ triển khai
  * Không agent
  * Trực quan, dễ hiểu cho non-technical
* ⚠️ Limitations:

  * Không đi sâu OS-level
  * Không real-time monitoring
  * Là **solution**, không phải core managed service
* 🧠 Keywords:

  * **Benefit**, **Limitation**

8️⃣ 🧪 SCENARIOS & DECISION GUIDE

* Chọn **AWS Workload Discovery** khi:

  * Cần **inventory + architecture map nhanh**
* So sánh nhanh:

  * vs **AWS Config** ➝ Config = compliance & change tracking
  * vs **CloudWatch** ➝ Monitoring, không phải discovery
  * vs **Application Discovery Service** ➝ On-prem ➝ AWS
* 🧠 Keywords:

  * **Choose when**, **Compare**

---

❓ Q&A (EXAM TRAPS)

* Q: Multi-account, không có tài liệu kiến trúc, cần map nhanh?

  * A: **AWS Workload Discovery** ➝ discovery + visualization
* Q: Discover server dependency trước khi migrate từ on-prem?

  * A: ❌ Không phải ➝ **Application Discovery Service**
* Q: Muốn theo dõi config drift?

  * A: ❌ Không dùng ➝ **AWS Config**

---

🏭 MÔ HÌNH / KIẾN TRÚC PHỔ BIẾN

* 🏢 **Multi-account visibility**

  * AWS Organizations ➝ Workload Discovery ➝ Global map
* 🔄 **Pre-migration assessment**

  * Existing AWS workloads ➝ Discovery ➝ Migration planning
* 🛡️ **Audit & documentation**

  * Discovery ➝ Diagram ➝ Architecture docs

✅ KEY TAKEAWAY:

> Khi đề bài nói **"discover + map AWS workloads"**, hãy nghĩ ngay đến **AWS Workload Discovery**
