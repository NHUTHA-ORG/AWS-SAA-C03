```bash
# Liệt kê các VPC
aws ec2 describe-vpcs --query "Vpcs[*].{ID:VpcId,CIDR:CidrBlock,Name:Tags[?Key=='Name']|[0].Value}" --output table

# Liệt kê route tables
aws ec2 describe-route-tables --query "RouteTables[*].{ID:RouteTableId,VPC:VpcId,Name:Tags[?Key=='Name']|[0].Value}" --output table

# Tạo Gateway Endpoint
aws ec2 create-vpc-endpoint \
    --vpc-id <VPC_ID> \
    --service-name com.amazonaws.<region>.s3 \
    --route-table-ids <ROUTE_TABLE_ID_1> <ROUTE_TABLE_ID_2> \
    --region <region> \
    --profile <aws_profile>

# Bật Private DNS (nếu cần), Điều này giúp apps dùng bucket.s3.<region>.amazonaws.com bình thường mà không phải đổi URL.
aws ec2 modify-vpc-endpoint \
    --vpc-endpoint-id <VPCE_ID> \
    --private-dns-enabled

# Kiểm tra route table đã có entry mới không:   
aws ec2 describe-route-tables --route-table-ids <ROUTE_TABLE_ID> --query "RouteTables[*].Routes" --output table

```

``` bash
# Các EC2 Instances trong VPC 
aws ec2 describe-instances --filters "Name=vpc-id,Values=<VPC_ID>" --query "Reservations[*].Instances[*].{ID:InstanceId,Type:InstanceType,State:State.Name,Subnet:SubnetId,Name:Tags[?Key=='Name']|[0].Value}" --output table

# Các EC2 Instances trong SUBNET
aws ec2 describe-instances --filters "Name=subnet-id,Values=<SUBNET_ID>" --query "Reservations[*].Instances[*].{ID:InstanceId,Type:InstanceType,State:State.Name,Name:Tags[?Key=='Name']|[0].Value}" --output table

# Network Interfaces (ENI)
aws ec2 describe-network-interfaces --filters "Name=vpc-id,Values=<VPC_ID>" --query "NetworkInterfaces[*].{ID:NetworkInterfaceId,Subnet:SubnetId,Instance:Attachment.InstanceId,PrivateIP:PrivateIpAddress}" --output table

# NAT Gateways
aws ec2 describe-nat-gateways --filter "Name=vpc-id,Values=<VPC_ID>" --query "NatGateways[*].{ID:NatGatewayId,Subnet:SubnetId,State:State}" --output table

# Route Tables
aws ec2 describe-route-tables --filters "Name=vpc-id,Values=<VPC_ID>" --query "RouteTables[*].{ID:RouteTableId,Subnets:Associations[*].SubnetId}" --output table


# S3 Gateway Endpoints
aws ec2 describe-vpc-endpoints --filters "Name=vpc-id,Values=<VPC_ID>" --query "VpcEndpoints[*].{ID:VpcEndpointId,Service:ServiceName,State:State,RouteTables:RouteTableIds}" --output table


# RDS Instances
aws rds describe-db-instances --query "DBInstances[*].{ID:DBInstanceIdentifier,VPC:DBSubnetGroup.VpcId,SubnetGroup:DBSubnetGroup.DBSubnetGroupName,Status:DBInstanceStatus}" --output table


# Elastic Load Balancers (ELB / ALB / NLB)
aws elbv2 describe-load-balancers --query "LoadBalancers[*].{Name:LoadBalancerName,Type:Type,VPC:VpcId,SubnetIds:AvailabilityZones[*].SubnetId,State:State.Code}" --output table

```