# **AWS SageMaker Cheat Sheet**

# ⚡ STAGE 1 — ULTRA-FAST READ

🧠 MEMORY ANCHORS

* Dịch vụ **ML quản lý toàn phần** cho việc xây dựng, huấn luyện và triển khai mô hình.
* Hỗ trợ **toàn bộ workflow**: chuẩn bị dữ liệu → huấn luyện → triển khai.
* Tự động mở rộng cho **huấn luyện & dự đoán**.

Analogy: 🏗️ *SageMaker giống như một nhà máy cho mô hình ML: dữ liệu thô vào, mô hình đã huấn luyện ra sẵn sàng triển khai.*

Keywords: **ML, Training, Deployment, Scalability, End-to-end, Managed, AI**

# 📝 STAGE 2 — PRE-EXAM READ

1️⃣ 🔍 SERVICE OVERVIEW

* **Là gì**: AWS SageMaker là dịch vụ quản lý hoàn toàn cho Machine Learning.
* **Mục đích**: Giảm thiểu phức tạp trong vòng đời ML và overhead hạ tầng.
* **Giá trị cốt lõi**: Xây dựng, huấn luyện, tinh chỉnh, và triển khai mô hình ML nhanh chóng mà không cần quản lý server.
* 🧠 Keywords: **ML lifecycle, Managed service, Scalability**

2️⃣ 🛡️ VẤN ĐỀ / RỦI RO GIẢI QUYẾT

* Giảm lỗi từ việc setup hạ tầng thủ công.
* Giải quyết **tắc nghẽn về khả năng mở rộng** khi huấn luyện và dự đoán.
* Loại bỏ overhead triển khai phức tạp.
* Nếu không dùng: Dev team tốn nhiều tuần để quản lý server & pipeline.
* 🧠 Keywords: **Risk, Automation, Scalability**

3️⃣ 📦 USE CASES (THỰC TẾ)

* Huấn luyện mô hình dự đoán cho **phát hiện gian lận, gợi ý, NLP, computer vision**.
* Phù hợp cho startup & enterprise cần **triển khai ML nhanh**.
* Tốt nhất khi cần **hạ tầng quản lý và huấn luyện mở rộng**.
* 🧠 Keywords: **Use case, Best fit, ML model**

4️⃣ 🧠 EXAM COVERAGE & TRAPS

* Khái niệm quan trọng: **Training jobs, Endpoints, Notebook Instances, AutoML (SageMaker Autopilot)**
* Bẫy thường gặp: Nhầm **SageMaker với Lambda**; Lambda không thể xử lý huấn luyện ML lớn.
* Không dùng cho tính toán hoặc lưu trữ chung.
* 🧠 Keywords: **Exam tip, Anti-pattern, End-to-end ML**

# 📚 STAGE 3 — FULL UNDERSTANDING

5️⃣ 🧩 CORE COMPONENTS & ARCHITECTURE

* **Notebook Instances** 📝: Jupyter notebooks để chuẩn bị dữ liệu và thử nghiệm.
* **Training Jobs** 🔥: Huấn luyện mô hình trên hạ tầng phân tán.
* **Hyperparameter Tuning Jobs** ⚙️: Tìm các tham số tối ưu tự động.
* **Models & Endpoints** 🚀: Triển khai mô hình huấn luyện để dự đoán thời gian thực hoặc batch.
* **SageMaker Studio** 🖥️: IDE thống nhất cho toàn bộ workflow ML.

Flow tổng quan: Dữ liệu trong S3 📦 → Notebook 📝 → Training Job 🔥 → Model 🚀 → Endpoint/API 🌐

6️⃣ 🔄 INTEGRATIONS & RELATED SERVICES

* **Amazon S3** 📦: Lưu trữ dataset & artifacts mô hình.
* **AWS Lambda** ⚡: Kích hoạt sự kiện inference.
* **Amazon CloudWatch** 📊: Giám sát metrics/log.
* **AWS Step Functions** 🔄: Orchestrate workflow ML.
* **Amazon ECR** 🐳: Lưu Docker image cho container ML tùy chỉnh.
* 🧠 Keywords: **Integration, Event-driven, Automation**

7️⃣ ⚖️ ƯU & NHƯỢC ĐIỂM

* ✅ Ưu điểm: Quản lý hoàn toàn, mở rộng, giảm overhead DevOps, hỗ trợ AutoML.
* ⚠️ Hạn chế: Có thể **tốn kém cho huấn luyện lớn**, học curve cho pipeline tùy chỉnh, không phù hợp cho ML nhỏ.
* 🧠 Keywords: **Benefit, Limitation, Cost**

8️⃣ 🧪 SCENARIOS & DECISION GUIDE

* Chọn SageMaker khi cần **ML end-to-end quản lý, huấn luyện & triển khai mở rộng**.
* So sánh:

  * **Lambda**: Không dùng cho huấn luyện ML lớn.
  * **EC2 + stack ML thủ công**: Kiểm soát nhiều nhưng overhead cao.
  * **Amazon Rekognition / Comprehend**: AI sẵn, không cần huấn luyện.
* 🧠 Keywords: **Choose when, Compare, Managed vs Custom**

❓ Q&A (TẬP TRUNG EXAM)

1. Q: SageMaker có hỗ trợ dự đoán real-time không?
   A: ✅ Có, qua **Endpoints**.
2. Q: Lambda thay SageMaker được không cho dataset >1GB?
   A: ❌ Không, Lambda giới hạn memory/time.
3. Q: SageMaker tự lưu dữ liệu không?
   A: ❌ Dữ liệu phải trong **S3**.
4. Q: SageMaker Studio có bắt buộc để huấn luyện không?
   A: ❌ Không, Notebook Instances vẫn dùng được.
5. Q: Dịch vụ AutoML tốt nhất?
   A: ✅ **SageMaker Autopilot**.

🏭 MÔ HÌNH / KIẾN TRÚC PHỔ BIẾN

1. Flow cơ bản: S3 📦 → Notebook 📝 → Training 🔥 → Model 🚀 → Endpoint 🌐
2. Batch inference: S3 Input 📦 → Batch Transform 🔄 → S3 Output 📤
3. AutoML: S3 Dataset 📦 → Autopilot ⚙️ → Model 🚀 → Endpoint 🌐
4. Orchestration pipeline ML: Step Functions 🔄 → Training 🔥 → Model 🚀 → Endpoint 🌐
