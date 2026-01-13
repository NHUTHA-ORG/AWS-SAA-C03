# AWS CLI setup – Networking patterns (SAA-C03)

> Mục tiêu: cung cấp “recipe” AWS CLI dạng skeleton cho từng pattern trong [topic/networking.md](../topic/networking.md).
>
> Ghi chú:
> - Các lệnh dưới đây ưu tiên dạng tham khảo/khung triển khai; tuỳ môi trường bạn sẽ cần bổ sung `--tag-specifications`, `--security-group-ids`, `--user-data`, IAM roles, policies…
> - CIDR ví dụ khớp với file networking:
>   - VPC chính: `10.0.0.0/16`
>   - Public: `10.0.1.0/24`, `10.0.2.0/24`
>   - Private: `10.0.11.0/24`, `10.0.12.0/24`
>   - Isolated DB: `10.0.21.0/24`
> - Thứ tự chung khi dựng VPC: `create-vpc` → `create-subnet` → (IGW/NAT/EP/TGW/...) → route tables → SG/NACL → instances/services.

## Conventions

Các snippet dùng biến môi trường. Bạn có thể chạy theo PowerShell hoặc Bash.

PowerShell (gợi ý):

```powershell
$REGION="ap-southeast-1"
$AZA="$REGION`a"
$AZB="$REGION`b"
$VPC_CIDR="10.0.0.0/16"
$PUB_A_CIDR="10.0.1.0/24"
$PUB_B_CIDR="10.0.2.0/24"
$PRIV_A_CIDR="10.0.11.0/24"
$PRIV_B_CIDR="10.0.12.0/24"
$DB_CIDR="10.0.21.0/24"
$ONPREM_CIDR="192.168.0.0/16"
$CLIENTVPN_CIDR="10.100.0.0/22"

aws configure set region $REGION
```

Bash (gợi ý):

```bash
export REGION=ap-southeast-1
export AZA=${REGION}a
export AZB=${REGION}b
export VPC_CIDR=10.0.0.0/16
export PUB_A_CIDR=10.0.1.0/24
export PUB_B_CIDR=10.0.2.0/24
export PRIV_A_CIDR=10.0.11.0/24
export PRIV_B_CIDR=10.0.12.0/24
export DB_CIDR=10.0.21.0/24
export ONPREM_CIDR=192.168.0.0/16
export CLIENTVPN_CIDR=10.100.0.0/22

aws configure set region "$REGION"
```

---

## muc-01 – Single VPC – Public only

```bash
VPC_ID=$(aws ec2 create-vpc --cidr-block $VPC_CIDR --query 'Vpc.VpcId' --output text)
IGW_ID=$(aws ec2 create-internet-gateway --query 'InternetGateway.InternetGatewayId' --output text)
aws ec2 attach-internet-gateway --vpc-id $VPC_ID --internet-gateway-id $IGW_ID

PUB_SUBNET_ID=$(aws ec2 create-subnet --vpc-id $VPC_ID --availability-zone $AZA --cidr-block $PUB_A_CIDR --query 'Subnet.SubnetId' --output text)
RTB_ID=$(aws ec2 create-route-table --vpc-id $VPC_ID --query 'RouteTable.RouteTableId' --output text)
aws ec2 create-route --route-table-id $RTB_ID --destination-cidr-block 0.0.0.0/0 --gateway-id $IGW_ID
aws ec2 associate-route-table --route-table-id $RTB_ID --subnet-id $PUB_SUBNET_ID

# EC2 (public IP): đảm bảo subnet bật map-public-ip-on-launch hoặc gán EIP
aws ec2 modify-subnet-attribute --subnet-id $PUB_SUBNET_ID --map-public-ip-on-launch
```

## muc-02 – Single VPC – Public + Private

```bash
# VPC + IGW + Public subnet (muc-01)
# Private subnet + NAT GW
PRIV_SUBNET_ID=$(aws ec2 create-subnet --vpc-id $VPC_ID --availability-zone $AZA --cidr-block $PRIV_A_CIDR --query 'Subnet.SubnetId' --output text)

EIP_ALLOC_ID=$(aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text)
NAT_ID=$(aws ec2 create-nat-gateway --subnet-id $PUB_SUBNET_ID --allocation-id $EIP_ALLOC_ID --query 'NatGateway.NatGatewayId' --output text)

PRIV_RTB_ID=$(aws ec2 create-route-table --vpc-id $VPC_ID --query 'RouteTable.RouteTableId' --output text)
aws ec2 create-route --route-table-id $PRIV_RTB_ID --destination-cidr-block 0.0.0.0/0 --nat-gateway-id $NAT_ID
aws ec2 associate-route-table --route-table-id $PRIV_RTB_ID --subnet-id $PRIV_SUBNET_ID
```

## muc-03 – Multi-AZ Public + Private

```bash
# 2 AZ public+private + NAT per AZ
PUB_A_ID=$(aws ec2 create-subnet --vpc-id $VPC_ID --availability-zone $AZA --cidr-block $PUB_A_CIDR --query 'Subnet.SubnetId' --output text)
PUB_B_ID=$(aws ec2 create-subnet --vpc-id $VPC_ID --availability-zone $AZB --cidr-block $PUB_B_CIDR --query 'Subnet.SubnetId' --output text)
aws ec2 modify-subnet-attribute --subnet-id $PUB_A_ID --map-public-ip-on-launch
aws ec2 modify-subnet-attribute --subnet-id $PUB_B_ID --map-public-ip-on-launch

# route table public (có thể dùng 1 RTB cho nhiều public subnets)
PUB_RTB_ID=$(aws ec2 create-route-table --vpc-id $VPC_ID --query 'RouteTable.RouteTableId' --output text)
aws ec2 create-route --route-table-id $PUB_RTB_ID --destination-cidr-block 0.0.0.0/0 --gateway-id $IGW_ID
aws ec2 associate-route-table --route-table-id $PUB_RTB_ID --subnet-id $PUB_A_ID
aws ec2 associate-route-table --route-table-id $PUB_RTB_ID --subnet-id $PUB_B_ID

# NAT per AZ + private RT per AZ
EIP_A=$(aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text)
NAT_A=$(aws ec2 create-nat-gateway --subnet-id $PUB_A_ID --allocation-id $EIP_A --query 'NatGateway.NatGatewayId' --output text)
PRIV_A_ID=$(aws ec2 create-subnet --vpc-id $VPC_ID --availability-zone $AZA --cidr-block $PRIV_A_CIDR --query 'Subnet.SubnetId' --output text)
PRIV_A_RTB=$(aws ec2 create-route-table --vpc-id $VPC_ID --query 'RouteTable.RouteTableId' --output text)
aws ec2 create-route --route-table-id $PRIV_A_RTB --destination-cidr-block 0.0.0.0/0 --nat-gateway-id $NAT_A
aws ec2 associate-route-table --route-table-id $PRIV_A_RTB --subnet-id $PRIV_A_ID

EIP_B=$(aws ec2 allocate-address --domain vpc --query 'AllocationId' --output text)
NAT_B=$(aws ec2 create-nat-gateway --subnet-id $PUB_B_ID --allocation-id $EIP_B --query 'NatGateway.NatGatewayId' --output text)
PRIV_B_ID=$(aws ec2 create-subnet --vpc-id $VPC_ID --availability-zone $AZB --cidr-block $PRIV_B_CIDR --query 'Subnet.SubnetId' --output text)
PRIV_B_RTB=$(aws ec2 create-route-table --vpc-id $VPC_ID --query 'RouteTable.RouteTableId' --output text)
aws ec2 create-route --route-table-id $PRIV_B_RTB --destination-cidr-block 0.0.0.0/0 --nat-gateway-id $NAT_B
aws ec2 associate-route-table --route-table-id $PRIV_B_RTB --subnet-id $PRIV_B_ID
```

## muc-04 – Public-facing ALB → Private EC2

```bash
# Nền: muc-02/muc-03 (ALB public subnet; EC2 private subnet + NAT)
# ALB skeleton (requires target group + listeners)
aws elbv2 create-load-balancer --name demo-alb --scheme internet-facing --type application \
  --subnets $PUB_A_ID $PUB_B_ID
```

## muc-05 – Private-only (No Internet)

```bash
# Không tạo IGW/NAT.
# Kết nối on-prem qua VPN/DX -> VGW/TGW.
# Route mẫu (từ subnet private đi on-prem)
aws ec2 create-route --route-table-id $PRIV_A_RTB --destination-cidr-block $ONPREM_CIDR --gateway-id vgw-xxxxxxxx
```

## muc-06 – Egress-only VPC

```bash
# VPC có private subnets; outbound bằng NAT hoặc VPC endpoints
# NAT: dùng muc-02
# Gateway Endpoint cho S3/DDB:
aws ec2 create-vpc-endpoint --vpc-id $VPC_ID --vpc-endpoint-type Gateway \
  --service-name com.amazonaws.$REGION.s3 --route-table-ids $PRIV_A_RTB
```

## muc-07 – IPv6-only / Dual-stack

```bash
# Bật IPv6 cho VPC + subnet + egress-only IGW
aws ec2 associate-vpc-cidr-block --vpc-id $VPC_ID --amazon-provided-ipv6-cidr-block
EIGW_ID=$(aws ec2 create-egress-only-internet-gateway --vpc-id $VPC_ID --query 'EgressOnlyInternetGateway.EgressOnlyInternetGatewayId' --output text)
# Route IPv6 default
aws ec2 create-route --route-table-id $PRIV_A_RTB --destination-ipv6-cidr-block ::/0 --egress-only-internet-gateway-id $EIGW_ID
```

## muc-08 – Site-to-Site VPN

```bash
# Tạo VGW + attach VPC, sau đó tạo customer gateway + vpn connection
VGW_ID=$(aws ec2 create-vpn-gateway --type ipsec.1 --query 'VpnGateway.VpnGatewayId' --output text)
aws ec2 attach-vpn-gateway --vpc-id $VPC_ID --vpn-gateway-id $VGW_ID
CGW_ID=$(aws ec2 create-customer-gateway --type ipsec.1 --public-ip <ONPREM_PUBLIC_IP> --bgp-asn 65000 --query 'CustomerGateway.CustomerGatewayId' --output text)
VPN_ID=$(aws ec2 create-vpn-connection --type ipsec.1 --customer-gateway-id $CGW_ID --vpn-gateway-id $VGW_ID --query 'VpnConnection.VpnConnectionId' --output text)
# Route tới on-prem
aws ec2 create-route --route-table-id $PRIV_A_RTB --destination-cidr-block $ONPREM_CIDR --gateway-id $VGW_ID
```

## muc-09 – Client VPN

```bash
# Tạo Client VPN endpoint (cần cert ARN) + associate subnet + authorization
CVPN_ID=$(aws ec2 create-client-vpn-endpoint --client-cidr-block $CLIENTVPN_CIDR \
  --server-certificate-arn <ACM_CERT_ARN> --authentication-options Type=certificate-authentication,MutualAuthentication={ClientRootCertificateChainArn=<ROOT_CERT_ARN>} \
  --connection-log-options Enabled=false --query 'ClientVpnEndpointId' --output text)
aws ec2 associate-client-vpn-target-network --client-vpn-endpoint-id $CVPN_ID --subnet-id $PRIV_A_ID
aws ec2 authorize-client-vpn-ingress --client-vpn-endpoint-id $CVPN_ID --target-network-cidr $VPC_CIDR --authorize-all-groups
```

## muc-10 – Direct Connect

```bash
# DX chủ yếu cấu hình ở console + router; CLI thường dùng để tạo VGW/DXGW + attach.
DXGW_ID=$(aws directconnect create-direct-connect-gateway --direct-connect-gateway-name demo-dxgw --query 'directConnectGateway.directConnectGatewayId' --output text)
# Sau đó associate VGW/TGW với DXGW theo kiến trúc.
```

## muc-11 – VPN backup cho Direct Connect

```bash
# Tạo DX (primary) + site-to-site VPN (backup) và dùng BGP/AS-path/metrics để ưu tiên.
# CLI phần VPN tương tự muc-08.
```

## muc-12 – VPC Peering (1–1)

```bash
PCX_ID=$(aws ec2 create-vpc-peering-connection --vpc-id $VPC_A --peer-vpc-id $VPC_B --query 'VpcPeeringConnection.VpcPeeringConnectionId' --output text)
aws ec2 accept-vpc-peering-connection --vpc-peering-connection-id $PCX_ID
aws ec2 create-route --route-table-id $RTB_A --destination-cidr-block 10.1.0.0/16 --vpc-peering-connection-id $PCX_ID
aws ec2 create-route --route-table-id $RTB_B --destination-cidr-block 10.0.0.0/16 --vpc-peering-connection-id $PCX_ID
```

## muc-13 – VPC Peering mesh

```bash
# Lặp muc-12 cho từng cặp VPC (N*(N-1)/2). Không khuyến nghị.
```

## muc-14 – Transit Gateway – Hub & Spoke

```bash
TGW_ID=$(aws ec2 create-transit-gateway --query 'TransitGateway.TransitGatewayId' --output text)
ATT_HUB=$(aws ec2 create-transit-gateway-vpc-attachment --transit-gateway-id $TGW_ID --vpc-id $HUB_VPC --subnet-ids $HUB_TGW_SUBNETS --query 'TransitGatewayVpcAttachment.TransitGatewayAttachmentId' --output text)
ATT_SPOKE1=$(aws ec2 create-transit-gateway-vpc-attachment --transit-gateway-id $TGW_ID --vpc-id $SPOKE1_VPC --subnet-ids $SPOKE1_TGW_SUBNETS --query 'TransitGatewayVpcAttachment.TransitGatewayAttachmentId' --output text)
# Routes trong VPC: add route to TGW
aws ec2 create-route --route-table-id $SPOKE1_RTB --destination-cidr-block 10.0.0.0/16 --transit-gateway-id $TGW_ID
```

## muc-15 – Transit Gateway + On-prem

```bash
# Thêm VPN/DX attachment vào TGW
# CLI phần VPN tương tự muc-08 nhưng target là TGW.
```

## muc-16 – Shared Services VPC

```bash
# Dựng TGW (muc-14) + attach Shared Services VPC + attach App VPCs
# Route từ app -> shared và ngược lại qua TGW.
```

## muc-17 – Isolated VPC per workload

```bash
# Tạo VPC riêng cho Dev/Test/Prod (muc-01/muc-02), có thể tách account.
# Không tạo peering/tgw nếu muốn isolated hoàn toàn.
```

## muc-18 – Networking account (Landing Zone)

```bash
# Dựng TGW ở account network, share bằng RAM, các account khác tạo attachment.
aws ram create-resource-share --name tgw-share --resource-arns <TGW_ARN> --principals <ACCOUNT_ID_1> <ACCOUNT_ID_2>
```

## muc-19 – Centralized egress VPC

```bash
# TGW default route -> egress attachment
# Spoke VPC: 0.0.0.0/0 -> TGW
aws ec2 create-route --route-table-id $SPOKE_RTB --destination-cidr-block 0.0.0.0/0 --transit-gateway-id $TGW_ID
```

## muc-20 – Centralized ingress VPC

```bash
# Ingress VPC chạy ALB/NLB; spoke services nhận traffic qua TGW.
# ALB/NLB setup: aws elbv2 create-load-balancer ...
```

## muc-21 – Gateway VPC Endpoint

```bash
aws ec2 create-vpc-endpoint --vpc-id $VPC_ID --vpc-endpoint-type Gateway \
  --service-name com.amazonaws.$REGION.s3 --route-table-ids $PRIV_A_RTB $PRIV_B_RTB
```

## muc-22 – Interface VPC Endpoint (PrivateLink)

```bash
aws ec2 create-vpc-endpoint --vpc-id $VPC_ID --vpc-endpoint-type Interface \
  --service-name com.amazonaws.$REGION.ssm \
  --subnet-ids $PRIV_A_ID $PRIV_B_ID --private-dns-enabled
```

## muc-23 – Private API Gateway

```bash
# Tạo Interface endpoint execute-api, rồi tạo API Gateway kiểu PRIVATE (thường làm qua apigateway CLI)
aws ec2 create-vpc-endpoint --vpc-id $VPC_ID --vpc-endpoint-type Interface \
  --service-name com.amazonaws.$REGION.execute-api --subnet-ids $PRIV_A_ID $PRIV_B_ID --private-dns-enabled
```

## muc-24 – Private ALB + PrivateLink

```bash
# Provider: NLB + endpoint service
aws ec2 create-vpc-endpoint-service-configuration --network-load-balancer-arns <NLB_ARN> --acceptance-required
# Consumer: create interface endpoint tới service name
aws ec2 create-vpc-endpoint --vpc-id <CONSUMER_VPC_ID> --vpc-endpoint-type Interface \
  --service-name com.amazonaws.vpce.$REGION.vpce-svc-xxxxxxxx --subnet-ids <CONSUMER_SUBNETS>
```

## muc-25 – Bastion Host

```bash
# Nền: VPC public+private (muc-02). Bastion đặt ở public subnet.
aws ec2 run-instances --image-id <AMI> --instance-type t3.micro --subnet-id $PUB_A_ID --associate-public-ip-address
```

## muc-26 – SSM Session Manager only

```bash
# Dùng interface endpoints (muc-22) cho ssm, ec2messages, ssmmessages.
for svc in ssm ec2messages ssmmessages; do
  aws ec2 create-vpc-endpoint --vpc-id $VPC_ID --vpc-endpoint-type Interface \
    --service-name com.amazonaws.$REGION.$svc --subnet-ids $PRIV_A_ID $PRIV_B_ID --private-dns-enabled
done
```

## muc-27 – Network Firewall centralized

```bash
# Tạo Firewall policy + firewall trong Inspection VPC (khá dài); tham khảo AWS docs.
# Các bước lõi: aws network-firewall create-firewall-policy -> create-firewall -> update route tables to firewall endpoints.
```

## muc-28 – Firewall appliance (Palo Alto, Fortinet)

```bash
# Launch marketplace AMI trong inspection subnet + cấu hình route table next-hop tới ENI.
# aws ec2 run-instances ... (marketplace AMI)
```

## muc-29 – NACL-based isolation

```bash
NACL_ID=$(aws ec2 create-network-acl --vpc-id $VPC_ID --query 'NetworkAcl.NetworkAclId' --output text)
aws ec2 create-network-acl-entry --network-acl-id $NACL_ID --ingress --rule-number 100 --protocol tcp --port-range From=443,To=443 --cidr-block 10.0.11.0/24 --rule-action allow
```

## muc-30 – Security Group reference-based

```bash
SG_WEB=$(aws ec2 create-security-group --vpc-id $VPC_ID --group-name sg-web --description web --query 'GroupId' --output text)
SG_APP=$(aws ec2 create-security-group --vpc-id $VPC_ID --group-name sg-app --description app --query 'GroupId' --output text)
aws ec2 authorize-security-group-ingress --group-id $SG_APP --protocol tcp --port 443 --source-group $SG_WEB
```

## muc-31 – ALB → ECS/Fargate (private)

```bash
# ECS/Fargate thường dựng bằng CloudFormation/CDK/terraform; CLI có thể tạo cluster/service.
aws ecs create-cluster --cluster-name demo
```

## muc-32 – NLB → EC2 (TCP/UDP)

```bash
aws elbv2 create-load-balancer --name demo-nlb --scheme internet-facing --type network --subnets $PUB_A_ID $PUB_B_ID
```

## muc-33 – ALB + WAF

```bash
# WAFv2: create-web-acl và associate với ALB ARN
aws wafv2 create-web-acl --name demo-waf --scope REGIONAL --default-action Allow={} \
  --visibility-config SampledRequestsEnabled=true,CloudWatchMetricsEnabled=true,MetricName=demo \
  --rules '[]'
```

## muc-34 – CloudFront → ALB

```bash
# CloudFront distribution config khá dài; thường dùng template.
# Tối thiểu: aws cloudfront create-distribution --distribution-config file://cf.json
```

## muc-35 – CloudFront → S3 (private)

```bash
# Dùng OAC (khuyến nghị) + bucket policy chỉ cho CloudFront.
# aws cloudfront create-origin-access-control ...
```

## muc-36 – Active–Passive Multi-Region

```bash
# Route53 failover record
aws route53 change-resource-record-sets --hosted-zone-id <HZ_ID> --change-batch file://r53-failover.json
```

## muc-37 – Active–Active Multi-Region

```bash
# Route53 latency/weighted records
aws route53 change-resource-record-sets --hosted-zone-id <HZ_ID> --change-batch file://r53-latency.json
```

## muc-38 – Cross-region VPC Peering

```bash
# create-vpc-peering-connection có --peer-region
aws ec2 create-vpc-peering-connection --vpc-id <VPC_A> --peer-vpc-id <VPC_B> --peer-region <REGION_B>
```

## muc-39 – TGW Inter-Region Peering

```bash
# create-transit-gateway-peering-attachment giữa 2 TGW
aws ec2 create-transit-gateway-peering-attachment --transit-gateway-id <TGW_A> --peer-transit-gateway-id <TGW_B> --peer-region <REGION_B>
```

## muc-40 – VPC + Outposts

```bash
# Outposts thường cần cấu hình thêm local gateway và Outpost ARN.
# Subnet on Outposts: aws ec2 create-subnet --outpost-arn <OUTPOST_ARN> ...
```

## muc-41 – Local Zones

```bash
# create-subnet trong Local Zone AZ (vd ap-southeast-1-lz-1a)
aws ec2 create-subnet --vpc-id $VPC_ID --availability-zone <LOCAL_ZONE_AZ> --cidr-block 10.0.70.0/24
```

## muc-42 – Wavelength Zones

```bash
# Wavelength subnet + Carrier Gateway
CAGW_ID=$(aws ec2 create-carrier-gateway --vpc-id $VPC_ID --query 'CarrierGateway.CarrierGatewayId' --output text)
WAVE_SUBNET=$(aws ec2 create-subnet --vpc-id $VPC_ID --availability-zone <WAVELENGTH_AZ> --cidr-block 10.0.50.0/24 --query 'Subnet.SubnetId' --output text)
RTB=$(aws ec2 create-route-table --vpc-id $VPC_ID --query 'RouteTable.RouteTableId' --output text)
aws ec2 create-route --route-table-id $RTB --destination-cidr-block 0.0.0.0/0 --carrier-gateway-id $CAGW_ID
```

## muc-43 – Dedicated tenancy VPC

```bash
aws ec2 create-vpc --cidr-block $VPC_CIDR --instance-tenancy dedicated
```

## muc-44 – Bring Your Own IP (BYOIP)

```bash
# BYOIP require steps: provision + advertise + allocate.
aws ec2 provision-byoip-cidr --cidr <YOUR_PUBLIC_CIDR>
aws ec2 advertise-byoip-cidr --cidr <YOUR_PUBLIC_CIDR>
```

## muc-45 – Multi-tier isolated subnets

```bash
# Web/App/DB subnets + route tables
WEB_SUBNET=$(aws ec2 create-subnet --vpc-id $VPC_ID --availability-zone $AZA --cidr-block $PUB_A_CIDR --query 'Subnet.SubnetId' --output text)
APP_SUBNET=$(aws ec2 create-subnet --vpc-id $VPC_ID --availability-zone $AZA --cidr-block $PRIV_A_CIDR --query 'Subnet.SubnetId' --output text)
DB_SUBNET=$(aws ec2 create-subnet --vpc-id $VPC_ID --availability-zone $AZA --cidr-block $DB_CIDR --query 'Subnet.SubnetId' --output text)
# DB subnet route table: chỉ local (không add 0.0.0.0/0)
```
