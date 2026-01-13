
# 1️⃣ VPC CƠ BẢN

> **Quy ước CIDR dùng trong diagram (ví dụ)**
>
> - VPC chính: `10.0.0.0/16`
> - Public subnets: `10.0.1.0/24` (AZ-a), `10.0.2.0/24` (AZ-b)
> - Private app subnets: `10.0.11.0/24` (AZ-a), `10.0.12.0/24` (AZ-b)
> - Isolated DB subnet (nếu có): `10.0.21.0/24`
> - On-prem (ví dụ): `192.168.0.0/16`
> - Client VPN client CIDR (ví dụ): `10.100.0.0/22`

### 1. Single VPC – Public only

* 1 VPC
* 1 Public Subnet
* IGW
* EC2 có Public IP
  👉 Demo / test nhanh

**ASCII diagram**

```text
Internet
  |
 [IGW]
  |
 +----------------------------------------------+
 | VPC (10.0.0.0/16)                            |
 |  +----------------------------------------+  |
 |  | Public Subnet (10.0.1.0/24)           |  |
 |  |  EC2 (Public IP / EIP)                |  |
 |  +----------------------------------------+  |
 +----------------------------------------------+
```

**Routing table records (nếu có)**

- Public Subnet RT
  - `0.0.0.0/0 -> IGW`
  - (tuỳ chọn dual-stack) `::/0 -> IGW`

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-01)

---

### 2. Single VPC – Public + Private

* Public Subnet: ALB / Bastion
* Private Subnet: EC2 / App
* NAT Gateway
  👉 Web app cơ bản

**ASCII diagram**

```text
Internet
  |
 [IGW]
  |
 +--------------------------------------------------------------+
 | VPC (10.0.0.0/16)                                            |
 |  +--------------------------+      +------------------------+ |
|  | Public Subnet (10.0.1.0/24)|      | Private Subnet (10.0.11.0/24)|
 |  |  ALB / Bastion           |      |  EC2 / App             | |
 |  |  NAT Gateway             |      |                        | |
 |  +--------------------------+      +------------------------+ |
 |                 |                            |
 |                 +---- egress: NAT -> IGW ----+
 +--------------------------------------------------------------+
```

**Routing table records (nếu có)**

- Public Subnet RT
  - `0.0.0.0/0 -> IGW`
- Private Subnet RT
  - `0.0.0.0/0 -> NAT Gateway`

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-02)

---

### 3. Multi-AZ Public + Private

* 2–3 AZ
* Public subnet mỗi AZ
* Private subnet mỗi AZ
* ALB + NAT Gateway mỗi AZ
  👉 HA production chuẩn AWS

**ASCII diagram**

```text
Internet
  |
 [IGW]
  |
 +-------------------------------------------------------------------+
 | VPC (10.0.0.0/16)                                                 |
 |                 +-------------------+                             |
 |                 | ALB (public)      |                             |
 |                 +---------+---------+                             |
 |                           |                                       |
 |      +--------------------+--------------------+                  |
 |      |                                         |                  |
 |  +---------------------+                +---------------------+   |
 |  | AZ-a                |                | AZ-b                |   |
 |  | +-----------------+ |                | +-----------------+ |   |
|  | | Public 10.0.1.0/24| |              | | Public 10.0.2.0/24| |  |
 |  | |  NAT-a          | |                | |  NAT-b          | |   |
 |  | +--------+--------+ |                | +--------+--------+ |   |
 |  |          |          |                |          |          |   |
 |  | +-----------------+ |                | +-----------------+ |   |
|  | | Private 10.0.11.0/24| |             | | Private 10.0.12.0/24| | |
 |  | |  EC2/App-a      | |                | |  EC2/App-b      | |   |
 |  | +-----------------+ |                | +-----------------+ |   |
 |  +---------------------+                +---------------------+   |
 +-------------------------------------------------------------------+
```

**Routing table records (nếu có)**

- Public Subnet RT (mỗi AZ)
  - `0.0.0.0/0 -> IGW`
- Private Subnet RT (mỗi AZ)
  - `0.0.0.0/0 -> NAT Gateway (cùng AZ)`

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-03)

---

# 2️⃣ INTERNET ACCESS PATTERNS

### 4. Public-facing ALB → Private EC2

* ALB public
* EC2 private
* Outbound qua NAT
  👉 Web chuẩn

**ASCII diagram**

```text
Internet
  |
 [ALB (public)]
  |
 +--------------------------------------------------+
 | VPC (10.0.0.0/16)                                |
 |  +--------------------------+                    |
|  | Private Subnet (10.0.11.0/24)                 |
 |  |  EC2/App                                    | |
 |  +--------------------------+                    |
 |                     |                            |
 |                 [NAT GW] -> [IGW] -> Internet     |
 +--------------------------------------------------+
```

**Routing table records (nếu có)**

- Subnet chứa ALB (public)
  - `0.0.0.0/0 -> IGW`
- Subnet chứa EC2 (private)
  - `0.0.0.0/0 -> NAT Gateway`

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-04)

---

### 5. Private-only (No Internet)

* Không IGW
* Không NAT
* Access qua VPN / Direct Connect
  👉 Internal system

**ASCII diagram**

```text
On-prem (192.168.0.0/16)
   |
  VPN/DX
   |
 [VGW/TGW]
   |
 +----------------------------------------------+
 | VPC (10.0.0.0/16)                            |
 |  +----------------------------------------+  |
 |  | Private Subnet (10.0.11.0/24)          |  |
 |  |  EC2/App                               |  |
 |  +----------------------------------------+  |
 +----------------------------------------------+
```

**Routing table records (nếu có)**

- Private Subnet RT
  - `192.168.0.0/16 -> VGW/TGW`
  - (không có) `0.0.0.0/0 -> IGW/NAT`

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-05)

---

### 6. Egress-only VPC

* No inbound internet
* Chỉ outbound qua NAT / VPC Endpoint
  👉 Batch job / outbound-only service

**ASCII diagram**

```text
(No inbound)
Internet  X

 +--------------------------------------------------------------+
 | VPC (10.0.0.0/16)                                            |
 |  +------------------------+                                 |
|  | Private Subnet (10.0.11.0/24)                            |
 |  |  EC2/Job                                                  | |
 |  +-----------+------------+                                 |
 |              |                                              |
 |      +-------+--------+            +----------------------+  |
 |      | VPC Endpoint(s)|            | NAT GW (public subnet)| |
 |      +-------+--------+            +----------+-----------+  |
 |              |                                |              |
 |     AWS services (private)               [IGW] -> Internet    |
 +--------------------------------------------------------------+
```

**Routing table records (nếu có)**

- Private Subnet RT
  - `0.0.0.0/0 -> NAT Gateway` (nếu cần internet egress)
  - `pl-<S3/DDB> -> Gateway Endpoint` (nếu dùng)

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-06)

---

### 7. IPv6-only / Dual-stack

* IPv4 + IPv6
* Egress-only IGW (IPv6)
  👉 Modern networking

**ASCII diagram**

```text
IPv6 Internet
  |
[Egress-only IGW]
  |
+------------------------------------------------------------------+
| VPC dual-stack (IPv4 10.0.0.0/16, IPv6 2001:db8:1234::/56)       |
|  +------------------------------------------------------------+  |
|  | Private Subnet (IPv4 10.0.11.0/24, IPv6 2001:db8:1234:11::/64)|
|  |  Workload                                                   | |
|  +------------------------------------------------------------+  |
+------------------------------------------------------------------+

IPv4 egress (nếu cần): Workload -> NAT GW -> IGW -> IPv4 Internet
```

**Routing table records (nếu có)**

- Private Subnet RT
  - `::/0 -> Egress-only IGW`
  - (tuỳ chọn) `0.0.0.0/0 -> NAT Gateway`

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-07)

---

# 3️⃣ ON-PREMISE CONNECTIVITY

### 8. Site-to-Site VPN

* On-prem ↔ VPC
* IPsec VPN
  👉 Kết nối nhanh, chi phí thấp

**ASCII diagram**

```text
On-prem (192.168.0.0/16)
  |
[Customer Gateway]
  ||  (IPsec)
[VGW/TGW]
  |
 +----------------------------------------------+
 | VPC (10.0.0.0/16)                            |
 |  +----------------------------------------+  |
 |  | Private Subnet (10.0.11.0/24)          |  |
 |  |  Workloads                             |  |
 |  +----------------------------------------+  |
 +----------------------------------------------+
```

**Routing table records (nếu có)**

- VPC Subnet RT
  - `192.168.0.0/16 -> VGW/TGW`
- (tuỳ chọn) Route propagation bật trên route table (nếu dùng BGP)

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-08)

---

### 9. Client VPN

* User → AWS
* OpenVPN-based
  👉 Remote dev / admin access

**ASCII diagram**

```text
Remote User
  |
[Client VPN Endpoint] (Client CIDR: 10.100.0.0/22)
  |
(associate)
  v
+----------------------------------------------+
| VPC (10.0.0.0/16)                            |
|  +----------------------------------------+  |
|  | Subnet association (vd 10.0.11.0/24)   |  |
|  |  Private resources                      |  |
|  +----------------------------------------+  |
+----------------------------------------------+
```

**Routing table records (nếu có)**

- Client VPN route table (trên dịch vụ Client VPN)
  - `10.0.0.0/16 -> associated subnet (10.0.11.0/24)`
- VPC Subnet RT
  - Thường không cần route đặc biệt (đi theo `local` trong VPC)

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-09)

---

### 10. Direct Connect

* Dedicated line
* Stable latency
  👉 Enterprise, large data

**ASCII diagram**

```text
On-prem (192.168.0.0/16)
  |
[Direct Connect]
  |
[DXGW/VGW/TGW]
  |
 +----------------------------------------------+
 | VPC (10.0.0.0/16)                            |
 |  +----------------------------------------+  |
 |  | Private Subnet (10.0.11.0/24)          |  |
 |  |  Workloads                             |  |
 |  +----------------------------------------+  |
 +----------------------------------------------+
```

**Routing table records (nếu có)**

- VPC Subnet RT
  - `192.168.0.0/16 -> VGW/TGW`
- (nếu dùng BGP) routes có thể được propagated tự động

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-10)

---

### 11. VPN backup cho Direct Connect

* DX + VPN failover
  👉 High availability

**ASCII diagram**

```text
                 +-------------------+
On-prem ---------|  DX (primary)     |----+
                 +-------------------+    |
                                            v
                 +-------------------+   [VGW/TGW]
On-prem ---------|  VPN (backup)     |----+
                 +-------------------+
                                            |
                                   +-------------------+
                                   | VPC (10.0.0.0/16)  |
                                   | Private 10.0.11.0/24 |
                                   +-------------------+
```

**Routing table records (nếu có)**

- VPC Subnet RT
  - `192.168.0.0/16 -> VGW/TGW` (1 route; path selection do BGP/propagation)

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-11)

---

# 4️⃣ MULTI-VPC PATTERNS

### 12. VPC Peering (1–1)

* Không transitive
* Simple
  👉 Small scale

**ASCII diagram**

```text
+---------------------------+     peering      +---------------------------+
| VPC A (10.0.0.0/16)       |<--------------->| VPC B (10.1.0.0/16)       |
|  +---------------------+  |                |  +---------------------+  |
|  | Subnet 10.0.11.0/24 |  |                |  | Subnet 10.1.11.0/24 |  |
|  |  resources          |  |                |  |  resources          |  |
|  +---------------------+  |                |  +---------------------+  |
+---------------------------+                +---------------------------+
```

**Routing table records (nếu có)**

- VPC A route table(s)
  - `10.1.0.0/16 -> pcx-...`
- VPC B route table(s)
  - `10.0.0.0/16 -> pcx-...`

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-12)

---

### 13. VPC Peering mesh

* Many VPC peered
  👉 ❌ Không scale

**ASCII diagram**

```text
  [VPC A 10.0.0.0/16]---pcx---[VPC B 10.1.0.0/16]
    |  \         /  |
    |   pcx   pcx   |
    |     \   /     |
    pcx   [VPC C 10.2.0.0/16]  pcx
    |         |      |
  [VPC D 10.3.0.0/16]---pcx---[VPC E 10.4.0.0/16]

(Số kết nối ~ N*(N-1)/2)
```

**Routing table records (nếu có)**

- Mỗi VPC phải có route tới CIDR của mọi VPC còn lại qua đúng `pcx-...`

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-13)

---

### 14. Transit Gateway – Hub & Spoke

* Central TGW
* Spoke VPC
  👉 Enterprise standard

**ASCII diagram**

```text
+-------------------------+     +-------------------------+
| Spoke-1 (10.10.0.0/16)  |     | Spoke-2 (10.20.0.0/16)  |
+-----------+-------------+     +-----------+-------------+
      \                     /
       \                   /
        v                 v
     +-------------------+
     | Transit Gateway   |
     +---------+---------+
         |
       +-------+-------+
       | Hub (10.0.0.0/16) |
       +---------------+
 (optional: VPN/DX attachment to on-prem)
```

**Routing table records (nếu có)**

- Spoke VPC subnet RTs
  - `Other VPC CIDR(s) -> TGW`
- TGW route table
  - `Spoke-1 CIDR -> attachment(spoke-1)`
  - `Spoke-2 CIDR -> attachment(spoke-2)`
  - `Hub CIDR -> attachment(hub)`

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-14)

---

### 15. Transit Gateway + On-prem

* VPC ↔ TGW ↔ On-prem
  👉 Hybrid network

**ASCII diagram**

```text
 +---------------------+   +---------------------+
 | Spoke (10.10.0.0/16)    |   | Spoke (10.20.0.0/16)    |
 +----------+----------+   +----------+----------+
            \                 /
             \               /
              v             v
                +-------------------+
On-prem 192.168.0.0/16 <--VPN/DX--> | TGW |
                +--------+----------+
                         |
                 +-------+-------+
                 | Shared (10.0.0.0/16)|
                 +---------------+
```

**Routing table records (nếu có)**

- VPC subnet RTs
  - `On-prem CIDR(s) -> TGW`
- TGW route table
  - `On-prem CIDR(s) -> VPN/DX attachment`
  - `VPC CIDR(s) -> VPC attachments`

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-15)

---

### 16. Shared Services VPC

* Central: AD, Bastion, Firewall
* App VPC connect TGW
  👉 Large org

**ASCII diagram**

```text
 +-----------------------------+
 | Shared Services (10.0.0.0/16)   |
 | (AD, DNS, Bastion, FW, ...) |
 +--------------+--------------+
        |
      [TGW]
    ________|________
   /                 \
 +------------+     +------------+
 | App-1 10.10.0.0/16|   | App-2 10.20.0.0/16|
 +------------+     +------------+
```

**Routing table records (nếu có)**

- App VPC subnet RTs
  - `Shared-Services CIDR -> TGW`
- Shared Services VPC subnet RTs
  - `App VPC CIDR(s) -> TGW`

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-16)

---

### 17. Isolated VPC per workload

* Dev / Test / Prod
* Account separation
  👉 Best practice

**ASCII diagram**

```text
 +-------------------+  +-------------------+  +-------------------+
 | Dev (10.10.0.0/16)    |  | Test (10.20.0.0/16)   |  | Prod (10.30.0.0/16)   |
 | isolated/account  |  | isolated/account  |  | isolated/account  |
 +-------------------+  +-------------------+  +-------------------+

(Nếu cần liên thông: TGW/Peering/PrivateLink tuỳ tổ chức)
```

**Routing table records (nếu có)**

- Không bắt buộc có route đặc biệt (nếu thật sự isolated)
- Nếu kết nối qua TGW: `Other CIDR(s) -> TGW`

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-17)

---

# 5️⃣ MULTI-ACCOUNT NETWORKING

### 18. Networking account (Landing Zone)

* Central network account
* Other accounts attach TGW
  👉 AWS Control Tower

**ASCII diagram**

```text
      [Networking Account]
           [TGW]
      ________|________
     /        |        \
[App Acct-1] [App Acct-2] [Shared Svcs Acct]
 [VPC 10.10.0.0/16] [VPC 10.20.0.0/16] [VPC 10.0.0.0/16]
    |             |              |
 (attach)      (attach)       (attach)
```

**Routing table records (nếu có)**

- Trong các VPC (ở các account)
  - `Corporate CIDR / Other VPC CIDR(s) -> TGW`
- Trên TGW route table (central)
  - `VPC CIDR(s) -> đúng attachment`

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-18)

---

### 19. Centralized egress VPC

* NAT / Firewall ở 1 VPC
* Other VPC route traffic
  👉 Cost + security control

**ASCII diagram**

```text
 +---------------------+    +-------------------+    +---------------------+
| Spoke 10.10.0.0/16  |    | TGW               |    | Egress VPC 10.97.0.0/16 |
 | private subnets     |--->| (default route)   |--->| Firewall / NAT      |
 +---------------------+    +-------------------+    +----------+----------+
                                                           |
                                                         [IGW]
                                                           |
                                                        Internet
```

**Routing table records (nếu có)**

- Spoke VPC subnet RTs
  - `0.0.0.0/0 -> TGW`
  - (tuỳ chọn) `AWS service prefix -> VPC Endpoint (local)`
- TGW route table
  - `0.0.0.0/0 -> attachment(egress VPC)`
  - `Spoke CIDR(s) -> corresponding attachments`
- Egress VPC route tables (tuỳ thiết kế)
  - Public/NAT subnets: `0.0.0.0/0 -> IGW`
  - Return traffic tới spoke: `Spoke CIDR(s) -> TGW`

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-19)

---

### 20. Centralized ingress VPC

* ALB / NLB central
* Route to spoke VPC
  👉 Large SaaS

**ASCII diagram**

```text
Internet
  |
[ALB/NLB (Ingress VPC 10.98.0.0/16)]
  |
  [TGW]
  |
 +---------------------+  +---------------------+
 | Spoke 10.10.0.0/16  |  | Spoke 10.20.0.0/16  |
 | services            |  | services            |
 +---------------------+  +---------------------+
```

**Routing table records (nếu có)**

- Ingress VPC subnet RTs (private/attachment subnets)
  - `Spoke CIDR(s) -> TGW`
- Spoke VPC subnet RTs
  - `Ingress VPC CIDR -> TGW` (nếu cần response path)
- TGW route table
  - `Spoke CIDR(s) -> spoke attachments`
  - `Ingress CIDR -> ingress attachment`

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-20)

---

# 6️⃣ PRIVATE ACCESS TO AWS SERVICES

### 21. Gateway VPC Endpoint

* S3
* DynamoDB
  👉 Private S3/DDB

**ASCII diagram**

```text
+----------------------------------------------+
| VPC (10.0.0.0/16)                            |
|  +----------------------------------------+  |
|  | Private Subnet (10.0.11.0/24)          |  |
|  |  EC2/App                               |  |
|  +-------------------+--------------------+  |
|                      | (route via prefix-list)|
|               [Gateway Endpoint]              |
|                      |                        |
|                 [S3 / DynamoDB]               |
|              (no NAT/IGW path)                |
+----------------------------------------------+
```

**Routing table records (nếu có)**

- Subnet RT (nơi workload chạy)
  - `pl-<S3>  -> vpce-gw` (Gateway Endpoint)
  - `pl-<DDB> -> vpce-gw` (Gateway Endpoint)

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-21)

---

### 22. Interface VPC Endpoint (PrivateLink)

* SSM
* Secrets Manager
* API Gateway
  👉 No NAT required

**ASCII diagram**

```text
+----------------------------------------------+
| VPC (10.0.0.0/16)                            |
|  +----------------------------------------+  |
|  | Private Subnet (10.0.11.0/24)          |  |
|  |  EC2/App                               |  |
|  +-------------------+--------------------+  |
|                      v (Private DNS)         |
|           [Interface Endpoint ENIs]          |
|                      |                      |
|     AWS services (SSM/Secrets/APIGW...)      |
+----------------------------------------------+
```

**Routing table records (nếu có)**

- Thường không cần thêm route trong route table (đi qua ENI trong VPC)
- (gợi ý) bật Private DNS để app gọi bằng hostname chuẩn

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-22)

---

### 23. Private API Gateway

* API Gateway + VPC Endpoint
  👉 Internal API

**ASCII diagram**

```text
+----------------------------------------------+
| VPC (10.0.0.0/16)                            |
|  +----------------------------------------+  |
|  | Private Subnet (10.0.11.0/24)          |  |
|  |  App client                            |  |
|  +-------------------+--------------------+  |
|                      |                       |
|     [Interface Endpoint: execute-api]        |
|                      |                       |
|           [Private API Gateway]              |
|                      |                       |
|        [Lambda / VPC Link / Backend]         |
+----------------------------------------------+
```

**Routing table records (nếu có)**

- Không cần thêm route table record (dùng Interface Endpoint)

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-23)

---

### 24. Private ALB + PrivateLink

* Expose service cross-VPC
  👉 Microservices

**ASCII diagram**

```text
 +------------------------------+         +------------------------------+
 | Consumer VPC (10.10.0.0/16)  |         | Provider VPC (10.20.0.0/16)  |
 |  +------------------------+  |         |  +------------------------+  |
 |  | Client (10.10.11/24)   |--+--PL---->|  | Endpoint Service (NLB) |  |
 |  +------------------------+  |         |  +-----------+------------+  |
 +------------------------------+         |              |               |
            |     [ALB (internal)]         |
            |              |               |
            |          [Targets]            |
            +------------------------------+
```

**Routing table records (nếu có)**

- Không cần route table record đặc biệt (PrivateLink dùng ENI/local routing)

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-24)

---

# 7️⃣ SECURITY & INSPECTION PATTERNS

### 25. Bastion Host

* SSH/RDP jump box
  👉 Classic (dần bỏ)

**ASCII diagram**

```text
Admin
  |
Internet
  |
[IGW]
  |
+----------------------------------------------+
| VPC (10.0.0.0/16)                            |
|  +------------------------+                  |
|  | Public Subnet (10.0.1.0/24)               |
|  |  Bastion host (SSH/RDP)  |                |
|  +-----------+------------+                  |
|              |                                 |
|              v                                 |
|  +------------------------+                  |
|  | Private Subnet (10.0.11.0/24)             |
|  |  EC2/App (no public IP)                  | |
|  +------------------------+                  |
+----------------------------------------------+
```

**Routing table records (nếu có)**

- Bastion subnet RT (public)
  - `0.0.0.0/0 -> IGW`
- Private subnet RT
  - `0.0.0.0/0 -> NAT Gateway` (nếu instance cần outbound)

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-25)

---

### 26. SSM Session Manager only

* No SSH
* No inbound
  👉 Best practice

**ASCII diagram**

```text
Admin -> AWS Console/CLI
       |
      SSM
       |
+--------------------------------------------------------------+
| VPC (10.0.0.0/16)                                            |
|  +------------------------+    +---------------------------+  |
|  | Private Subnet 10.0.11.0/24|  | Private Subnet 10.0.12.0/24 |  |
|  |  EC2 (SSM Agent + IAM)  |  |  Interface Endpoint ENIs   |  |
|  +-----------+------------+  +------------+--------------+  |
|              |                           |                 |
|              +-------- SSM traffic ------+                 |
|       (Alt) nếu không endpoint: EC2 -> NAT -> IGW -> SSM    |
+--------------------------------------------------------------+
```

**Routing table records (nếu có)**

- Nếu dùng Interface Endpoint: không cần route table record đặc biệt
- Nếu không dùng endpoint: private subnet thường cần `0.0.0.0/0 -> NAT Gateway`

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-26)

---

### 27. Network Firewall centralized

* AWS Network Firewall
* Ingress / egress inspection
  👉 Compliance

**ASCII diagram**

```text
 +---------------------+     +-------------------+     +------------------------+
 | Spoke VPC (10.10.0.0/16)| --> | TGW               | --> | Inspection VPC (10.99.0.0/16)|
 +---------------------+     +-------------------+     |  AWS Network Firewall  |
                                     +-----------+------------+
                                             |
                                           [IGW/NAT]
                                             |
                                           Internet
```

**Routing table records (nếu có)**

- Spoke VPC subnet RTs
  - `0.0.0.0/0 -> TGW` (đẩy traffic về inspection/egress)
- Inspection VPC (tuỳ thiết kế)
  - `0.0.0.0/0 -> IGW/NAT` (ra internet)
  - `Spoke CIDR(s) -> TGW` (đường về)
- Các subnet trong inspection VPC có thể route `0.0.0.0/0` qua firewall endpoint

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-27)

---

### 28. Firewall appliance (Palo Alto, Fortinet)

* Third-party AMI
  👉 Advanced security

**ASCII diagram**

```text
 +---------------------+     +-------------------+     +------------------------+
 | Spoke VPC (10.10.0.0/16)| --> | TGW               | --> | Inspection VPC (10.99.0.0/16)|
 +---------------------+     +-------------------+     |  Subnet 10.99.11.0/24  |
                                     |   Firewall EC2 (ENI)   |
                                     +-----------+------------+
                                             |
                                           [IGW/NAT]
                                             |
                                           Internet
```

**Routing table records (nếu có)**

- Spoke VPC subnet RTs
  - `0.0.0.0/0 -> TGW`
- Inspection VPC subnet RTs (mô hình phổ biến)
  - `0.0.0.0/0 -> firewall-eni (next hop)`
  - `Spoke CIDR(s) -> TGW`

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-28)

---

### 29. NACL-based isolation

* Subnet-level control
  👉 Coarse-grained

**ASCII diagram**

```text
+----------------------------------------------+
| VPC (10.0.0.0/16)                            |
|  +------------------------+  NACL  +------------------------+ |
|  | Subnet A (10.0.11.0/24)  |<------>| Subnet B (10.0.12.0/24)  | |
|  |  instances             | rules  |  instances             | |
|  +------------------------+        +------------------------+ |
+----------------------------------------------+
```

**Routing table records (nếu có)**

- Không có route đặc biệt (NACL là layer filtering; route table không đổi)

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-29)

---

### 30. Security Group reference-based

* SG → SG
  👉 Fine-grained

**ASCII diagram**

```text
+-----------------------------------------------------------+
| VPC (10.0.0.0/16)                                         |
|  +---------------------+  +---------------------+         |
|  | Web/Public 10.0.1.0/24|  | App/Private 10.0.11.0/24     |
|  |  ALB (SG-web)       |->|  EC2/ECS (SG-app)             |
|  +---------------------+  +-----------+-----------------+ |
|                                   allow (SG->SG)          |
|                                            v              |
|                             +--------------------------+  |
|                             | DB/Isolated 10.0.21.0/24 |  |
|                             |  RDS (SG-db)             |  |
|                             +--------------------------+  |
+-----------------------------------------------------------+
```

**Routing table records (nếu có)**

- Không có route đặc biệt (SG là stateful filtering; route table không đổi)

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-30)

---

# 8️⃣ LOAD BALANCING & TRAFFIC FLOW

### 31. ALB → ECS/Fargate (private)

* Container-based app
  👉 Modern web

**ASCII diagram**

```text
Internet
  |
[ALB (public)]
  |
+--------------------------------------------------------------+
| VPC (10.0.0.0/16)                                            |
|  +------------------------+    +---------------------------+  |
|  | Public Subnet 10.0.1.0/24|  | Private Subnet 10.0.11.0/24 |  |
|  |  ALB ENIs              |    |  ECS/Fargate tasks        |  |
|  +------------------------+    +------------+--------------+  |
|                                           (egress)           |
|                                   NAT -> IGW -> Internet     |
+--------------------------------------------------------------+
```

**Routing table records (nếu có)**

- Public subnet RT (ALB subnets)
  - `0.0.0.0/0 -> IGW`
- Private subnet RT (tasks)
  - `0.0.0.0/0 -> NAT Gateway` (nếu cần outbound)
  - (tuỳ chọn) `pl-<S3/DDB> -> Gateway Endpoint`

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-31)

---

### 32. NLB → EC2 (TCP/UDP)

* High performance
  👉 Game / IoT

**ASCII diagram**

```text
Internet (TCP/UDP)
  |
[NLB]
  |
+----------------------------------------------+
| VPC (10.0.0.0/16)                            |
|  +------------------------+                  |
|  | Subnet (10.0.11.0/24)  |                  |
|  |  Targets: EC2 instances|                  |
|  +------------------------+                  |
|  (optional) egress: NAT -> IGW -> Internet   |
+----------------------------------------------+
```

**Routing table records (nếu có)**

- Nếu NLB internet-facing: subnets của NLB thường là public
  - `0.0.0.0/0 -> IGW`
- Nếu targets ở private subnets và cần outbound
  - `0.0.0.0/0 -> NAT Gateway`

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-32)

---

### 33. ALB + WAF

* Layer 7 protection
  👉 Public web

**ASCII diagram**

```text
Internet
  |
[AWS WAF]
  |
[ALB]
  |
+----------------------------------------------+
| VPC (10.0.0.0/16)                            |
|  +------------------------+                  |
|  | Private Subnet 10.0.11.0/24               |
|  |  Targets (EC2/ECS/Lambda)                 |
|  +------------------------+                  |
+----------------------------------------------+
```

**Routing table records (nếu có)**

- Thường giống mô hình ALB public: `0.0.0.0/0 -> IGW` ở public subnets

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-33)

---

### 34. CloudFront → ALB

* Edge caching
  👉 Global users

**ASCII diagram**

```text
Users (global)
  |
[CloudFront (edge)]
  |
(origin)
  v
+----------------------------------------------+
| Region VPC (10.0.0.0/16)                     |
|  +------------------------+                  |
|  | Public Subnet 10.0.1.0/24                 |
|  |  ALB (origin)           |                 |
|  +------------------------+                  |
|              |                               |
|        +-----v-------------------+            |
|        | Private Subnet 10.0.11.0/24          |
|        |  App                       |         |
|        +---------------------------+          |
+----------------------------------------------+
```

**Routing table records (nếu có)**

- Không có route đặc biệt do CloudFront tạo ra
- Nếu ALB public: public subnet RT có `0.0.0.0/0 -> IGW`

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-34)

---

### 35. CloudFront → S3 (private)

* OAC / OAI
  👉 Static site

**ASCII diagram**

```text
Users
  |
+-------------------+
| CloudFront         |
+---------+---------+
          | (OAC/OAI)
          v
+-------------------------------+
| S3 Bucket (Block Public Access)|
+-------------------------------+
```

**Routing table records (nếu có)**

- Không áp dụng route table VPC (S3 + CloudFront không cần route table)

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-35)

---

# 9️⃣ MULTI-REGION NETWORKING

### 36. Active–Passive Multi-Region

* Route 53 failover
  👉 DR

**ASCII diagram**

```text
Users
  |
[Route 53 Failover]
  |\
  | \ (health check)
  |  \
  v   v
Primary Region                     Secondary Region
+-------------------------+        +-------------------------+
| VPC (10.0.0.0/16)       |        | VPC (10.1.0.0/16)       |
| ALB/App (active)        |        | ALB/App (standby)       |
+-------------------------+        +-------------------------+
```

**Routing table records (nếu có)**

- Mỗi region dùng route table nội bộ như các pattern VPC tương ứng (public/private/NAT)

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-36)

---

### 37. Active–Active Multi-Region

* Latency-based routing
  👉 Global app

**ASCII diagram**

```text
Users (global)
  |
[Route 53 Latency/Geo]
  |\
  | \
  v  v
Region A                           Region B
+-------------------------+        +-------------------------+
| VPC (10.0.0.0/16)       |        | VPC (10.1.0.0/16)       |
| ALB/App (active)        |        | ALB/App (active)        |
+-------------------------+        +-------------------------+
```

**Routing table records (nếu có)**

- Tương tự: route table phụ thuộc mỗi VPC design (thường public ALB + private app + NAT)

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-37)

---

### 38. Cross-region VPC Peering

* Region A ↔ Region B
  👉 Low latency private

**ASCII diagram**

```text
Region A                          Region B
+---------------------------+  peering  +---------------------------+
| VPC A (10.0.0.0/16)       |<--------->| VPC B (10.1.0.0/16)       |
|  subnets/resources        |           |  subnets/resources        |
+---------------------------+           +---------------------------+
```

**Routing table records (nếu có)**

- VPC A route table(s)
  - `10.1.0.0/16 -> pcx-... (inter-region)`
- VPC B route table(s)
  - `10.0.0.0/16 -> pcx-... (inter-region)`

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-38)

---

### 39. TGW Inter-Region Peering

* Multi-region hub
  👉 Enterprise global

**ASCII diagram**

```text
Region A                                     Region B
+-------------------+    TGW peering     +-------------------+
| TGW-A             |<------------------>| TGW-B             |
+--------+----------+                    +----------+--------+
      |                                        |
 +-------+----------------+              +--------+---------------+
 | VPCs (vd 10.0.0.0/16)  |              | VPCs (vd 10.1.0.0/16)  |
 +------------------------+              +------------------------+
```

**Routing table records (nếu có)**

- VPC subnet RTs (mỗi region)
  - `Remote-region CIDR(s) -> local TGW`
- TGW-A route table
  - `Region B CIDR(s) -> tgw-peering attachment`
- TGW-B route table
  - `Region A CIDR(s) -> tgw-peering attachment`

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-39)

---

# 🔟 SPECIALIZED / EDGE CASES

### 40. VPC + Outposts

* On-prem extension
  👉 Hybrid cloud

**ASCII diagram**

```text
AWS Region
 +----------------------------------------------+
 | VPC (10.0.0.0/16)                            |
 +---------------------------+------------------+
                    |
                (service link)
                    |
On-prem site
 +------------------------------+
 | Outposts Rack                |
 |  +------------------------+  |
|  | Outposts Subnet 10.0.60.0/24|
 |  |  EC2 on Outposts         |
 |  +------------------------+  |
 +------------------------------+
```

**Routing table records (nếu có)**

- Outposts subnet RT (tuỳ use-case)
  - `On-prem local CIDR(s) -> Local Gateway (LGW)` (nếu cần local routing)
  - `0.0.0.0/0 -> NAT/IGW (trong region)` (nếu egress đi AWS)

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-40)

---

### 41. Local Zones

* Ultra-low latency
  👉 Gaming / media

**ASCII diagram**

```text
AWS Region                              Local Zone
+----------------------------------+    +------------------------------+
| VPC (10.0.0.0/16)                |--->| Subnet (10.0.70.0/24)         |
| (extended VPC to Local Zone)     |    | Workloads                     |
+----------------------------------+    +------------------------------+
```

**Routing table records (nếu có)**

- Tương tự subnet thường: public subnet có `0.0.0.0/0 -> IGW`; private subnet có `0.0.0.0/0 -> NAT`

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-41)

---

### 42. Wavelength Zones

* Telco edge
  👉 5G apps

**ASCII diagram**

```text
5G Devices
  |
[Telco Network]
  |
[Wavelength Zone Subnet (10.0.50.0/24)] -> [EC2 on Wavelength]
  |
(optional backhaul)
  v
+----------------------------------------------+
| Region VPC (10.0.0.0/16) resources           |
+----------------------------------------------+
```

**Routing table records (nếu có)**

- Wavelength subnet RT (thường dùng Carrier Gateway)
  - `0.0.0.0/0 -> Carrier Gateway (cagw)`

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-42)

---

### 43. Dedicated tenancy VPC

* Compliance
  👉 Financial / gov

**ASCII diagram**

```text
+----------------------------------------------+
| VPC (10.0.0.0/16, tenancy=dedicated)         |
|  Dedicated Instances/Hosts                   |
+----------------------------------------------+
```

**Routing table records (nếu có)**

- Không có route đặc biệt (tenancy không ảnh hưởng route table)

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-43)

---

### 44. Bring Your Own IP (BYOIP)

* Own public IP range
  👉 Brand / compliance

**ASCII diagram**

```text
Your Public IP Range (BYOIP)
        |
      AWS
        |
[EIP allocated from BYOIP pool]
        |
[IGW] -> +----------------------------------------------+
   | VPC (10.0.0.0/16)                            |
   |  +------------------------+                  |
  |  | Public Subnet (10.0.1.0/24)               |
   |  |  Resource (EIP from BYOIP pool)           |
   |  +------------------------+                  |
   +----------------------------------------------+
```

**Routing table records (nếu có)**

- Không có route table record mới (BYOIP là IP management; route vẫn `0.0.0.0/0 -> IGW`)

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-44)

---

### 45. Multi-tier isolated subnets

* Web / App / DB subnet
  👉 Classic 3-tier

**ASCII diagram**

```text
Internet
  |
+--------------------------------------------------------------+
| VPC (10.0.0.0/16)                                            |
|  +------------------------+                                  |
|  | Web/Public 10.0.1.0/24  |  ALB/Web                         |
|  +-----------+------------+                                  |
|              |                                               |
|  +-----------v------------+                                  |
|  | App/Private 10.0.11.0/24|  App tier                        |
|  +-----------+------------+                                  |
|              |                                               |
|  +-----------v------------+                                  |
|  | DB/Isolated 10.0.21.0/24|  DB tier (no direct internet)    |
|  +------------------------+                                  |
+--------------------------------------------------------------+

Egress:
App (private) -> NAT -> IGW -> Internet (optional)
DB (isolated) -> (no direct internet)
```

**Routing table records (nếu có)**

- Web/Public subnet RT
  - `0.0.0.0/0 -> IGW`
- App/Private subnet RT
  - `0.0.0.0/0 -> NAT Gateway`
- DB/Isolated subnet RT
  - (khuyến nghị) không có `0.0.0.0/0` (chỉ `local` + routes nội bộ nếu cần)

**AWS CLI setup (tham khảo)**

- Xem: [aws-cli/networking-setup.md](aws-cli/networking-setup.md#muc-45)

---

# 🧠 CÁCH NHỚ NHANH CHO THI

> **90% đề AWS Network xoay quanh:**

* Public vs Private
* NAT vs VPC Endpoint
* Peering vs TGW
* VPN vs Direct Connect
* Centralized vs Decentralized

