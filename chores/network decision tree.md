```
START
 |
 |-- Resource có cần NETWORK trong VPC không?
        |
        |-- NO
        |     → S3, DynamoDB (public) → KHÔNG có ENI
        |
        |-- YES
              |
              |-- Resource chạy TRONG VPC?
                    |
                    |-- YES
                    |     |
                    |     |-- Loại resource?
                    |           |
                    |           |-- EC2
                    |           |     → Primary ENI (+ secondary ENI nếu attach thêm)
                    |           |
                    |           |-- ALB / NLB
                    |           |     → Mỗi AZ = 1 ENI
                    |           |
                    |           |-- RDS / ElastiCache
                    |           |     → ENI managed bởi AWS
                    |           |
                    |           |-- Lambda (VPC enabled)
                    |           |     → ENI được AWS tạo & reuse
                    |           |
                    |           |-- VPC Endpoint (Interface)
                    |                 → ENI + Private IP
                    |
                    |-- NO
                          |
                          |-- Service PUBLIC (S3, SNS...)
                                → Không có ENI
```