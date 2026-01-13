Dưới đây là **SAA-C03 Pre-Exam Checklist** được tối ưu cho **1–3 ngày trước kỳ thi**, tập trung đúng trọng tâm hay ra đề, dễ nhớ – đúng kiểu “lướt là kích hoạt trí nhớ” cho Solutions Architect 👇

---

# ✅ AWS SAA-C03 — PRE-EXAM CHECKLIST

## 🧠 1. MINDSET TRƯỚC KHI VÀO ĐỀ

✔ Luôn hỏi **“AWS muốn mình dùng managed service nào?”**
✔ Ưu tiên: **Managed > Serverless > PaaS > IaaS**
✔ Ưu tiên: **HA → Fault tolerant → Cost-effective → Minimal ops**
✔ Tránh: EC2 tự build khi **RDS / DynamoDB / SQS / Lambda** đủ dùng

---

## 🌐 2. NETWORKING (VERY HIGH WEIGHT)

### VPC Core

✔ CIDR, public vs private subnet
✔ Route Table → IGW vs NAT Gateway
✔ Security Group (stateful) vs NACL (stateless)

### Connectivity

✔ **VPC Peering** – same region/account, no transitive
✔ **Transit Gateway** – hub-and-spoke, multi VPC/on-prem
✔ **Site-to-Site VPN** – quick, IPsec
✔ **Direct Connect** – stable, high bandwidth

### DNS & Traffic

✔ **Route 53 routing policies**

* Simple / Weighted
* Latency
* Failover (health check)
* Geolocation / Geoproximity

---

## 💾 3. STORAGE (ALWAYS TESTED)

### S3

✔ Storage Classes:

* Standard
* IA / One-Zone IA
* Glacier Instant / Flexible / Deep Archive

✔ Features:

* Versioning
* Lifecycle
* CRR / SRR
* Object Lock (WORM)

✔ Security:

* Bucket Policy vs IAM Policy
* **S3 Block Public Access**

### Hybrid Storage

✔ **Storage Gateway**

* File → NFS/SMB → S3
* Volume → iSCSI → block
* Tape → virtual tape library

---

## 🗄 4. DATABASES (EXAM FAVORITE)

### Relational

✔ **RDS** (MySQL, Postgres, Oracle, SQL Server)
✔ **Aurora** (MySQL/Postgres compatible)

* Multi-AZ
* Read Replica
* Aurora Global DB

### NoSQL

✔ **DynamoDB**

* Partition key vs Sort key
* On-demand vs Provisioned
* DAX (cache)
* Global Table

### Other

✔ **ElastiCache** – Redis vs Memcached
✔ **Redshift** – OLAP
✔ **Athena** – SQL on S3

---

## ⚙️ 5. COMPUTE

### EC2

✔ Instance types: General / Compute / Memory / Storage
✔ Auto Scaling Group
✔ Placement Group:

* Cluster
* Spread
* Partition

### Serverless

✔ **Lambda**

* Event sources
* Timeout / Memory
* Stateless

✔ **ECS vs EKS vs Fargate**

* ECS = AWS native
* EKS = Kubernetes
* Fargate = no server mgmt

---

## 🔐 6. SECURITY & IAM (HIGH CONFUSION AREA)

✔ IAM User vs Role vs Policy
✔ AssumeRole (cross-account)
✔ Resource-based vs Identity-based policy

✔ Encryption:

* KMS (CMK)
* SSE-S3 vs SSE-KMS vs SSE-C

✔ Network security:

* SG vs NACL
* PrivateLink

---

## 🧩 7. APPLICATION INTEGRATION

✔ **SQS**

* Standard vs FIFO
* DLQ

✔ **SNS**

* Fan-out
* Push model

✔ **EventBridge**

* Event-driven architecture
* SaaS integration

✔ **Step Functions**

* Workflow orchestration

---

## 📈 8. MONITORING & GOVERNANCE

✔ CloudWatch:

* Metrics
* Logs
* Alarms

✔ CloudTrail – API auditing
✔ AWS Config – compliance

---

## 💰 9. COST OPTIMIZATION (SNEAKY QUESTIONS)

✔ On-Demand vs Reserved vs Savings Plan
✔ Spot instances
✔ S3 lifecycle
✔ Right-sizing EC2

---

## 🎯 10. EXAM QUESTION PATTERNS

🔑 Keywords → Instant Answer:

* “**minimum operational overhead**” → Serverless
* “**high availability**” → Multi-AZ
* “**low latency globally**” → CloudFront / Global Accelerator
* “**decouple**” → SQS / SNS
* “**on-prem + AWS**” → VPN / Direct Connect / Storage Gateway

---

## 🧪 11. FINAL NIGHT CHECK

✔ Không học dịch vụ mới
✔ Ôn sai lầm thường gặp
✔ Ngủ đủ
✔ Đọc kỹ **EXACT requirement** trong câu hỏi

---

Nếu bạn muốn, mình có thể:

* 🔥 Tạo **1-page SAA-C03 brain dump**
* 🧠 Làm **rapid-fire 20 câu dễ sai nhất**
* 🧪 Review **mock exam strategy**
* ⏱ Lập **2-day / 24-hour cram plan**

👉 Bạn đang thi **bao lâu nữa**?
