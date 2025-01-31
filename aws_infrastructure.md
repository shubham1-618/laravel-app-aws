# AWS Infrastructure Setup Guide for Laravel Application

## Prerequisites
- AWS CLI installed and configured with appropriate credentials
- Account with necessary IAM permissions to create resources
- AWS region: ap-south-1 (Mumbai)

## 1. VPC Setup
### Create VPC
```bash
aws ec2 create-vpc \
    --cidr-block 10.0.0.0/16 \
    --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=laravel-vpc},{Key=Environment,Value=production},{Key=Project,Value=laravel}]'
```
Save the VPC ID: `vpc-00ade31e06f8a37f2`

### Create Internet Gateway
```bash
aws ec2 create-internet-gateway \
    --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=laravel-igw},{Key=Environment,Value=production},{Key=Project,Value=laravel}]'
```
Save the IGW ID: `igw-000a481702233e30d`

### Attach Internet Gateway to VPC
```bash
aws ec2 attach-internet-gateway \
    --vpc-id vpc-00ade31e06f8a37f2 \
    --internet-gateway-id igw-000a481702233e30d
```

## 2. Subnet Configuration
### Create Public Subnet
```bash
aws ec2 create-subnet \
    --vpc-id vpc-00ade31e06f8a37f2 \
    --cidr-block 10.0.1.0/24 \
    --availability-zone ap-south-1a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=laravel-public-1a},{Key=Type,Value=Public},{Key=Environment,Value=production}]'
```
Save the Public Subnet ID: `subnet-0eee116d20ee34413`

### Create Private Subnets
```bash
# Private Subnet in ap-south-1a
aws ec2 create-subnet \
    --vpc-id vpc-00ade31e06f8a37f2 \
    --cidr-block 10.0.2.0/24 \
    --availability-zone ap-south-1a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=laravel-private-1a},{Key=Type,Value=Private},{Key=Environment,Value=production}]'

# Private Subnet in ap-south-1b
aws ec2 create-subnet \
    --vpc-id vpc-00ade31e06f8a37f2 \
    --cidr-block 10.0.3.0/24 \
    --availability-zone ap-south-1b \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=laravel-private-1b},{Key=Type,Value=Private},{Key=Environment,Value=production}]'
```
Save Private Subnet IDs:
- Private Subnet 1a: `subnet-0413af3888b57728a`
- Private Subnet 1b: `subnet-0fc4e2d22108ea7d2`

## 3. Route Table Configuration
### Create Route Table
```bash
aws ec2 create-route-table \
    --vpc-id vpc-00ade31e06f8a37f2 \
    --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=laravel-public-rt},{Key=Environment,Value=production},{Key=Project,Value=laravel}]'
```
Save Route Table ID: `rtb-055f25e3da3814d25`

### Configure Routes and Associations
```bash
# Create route to Internet Gateway
aws ec2 create-route \
    --route-table-id rtb-055f25e3da3814d25 \
    --destination-cidr-block 0.0.0.0/0 \
    --gateway-id igw-000a481702233e30d

# Associate with public subnet
aws ec2 associate-route-table \
    --subnet-id subnet-0eee116d20ee34413 \
    --route-table-id rtb-055f25e3da3814d25
```

## 4. Security Group Setup
### Create RDS Security Group
```bash
aws ec2 create-security-group \
    --group-name laravel-rds-sg \
    --description "Security group for Laravel RDS" \
    --vpc-id vpc-00ade31e06f8a37f2
```
Save RDS Security Group ID: `sg-0f61891a3cdd1b962`

### Create EC2 Security Group
```bash
aws ec2 create-security-group \
    --group-name laravel-ec2-sg \
    --description "Security group for Laravel EC2 instances" \
    --vpc-id vpc-00ade31e06f8a37f2
```
Save EC2 Security Group ID: `sg-0c969660d42950669`

### Configure Security Group Rules
```bash
# Allow MySQL access from EC2 to RDS
aws ec2 authorize-security-group-ingress \
    --group-id sg-0f61891a3cdd1b962 \
    --protocol tcp \
    --port 3306 \
    --source-group sg-0c969660d42950669

# Allow SSH access to EC2
aws ec2 authorize-security-group-ingress \
    --group-id sg-0c969660d42950669 \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0

# Allow HTTP access to EC2
aws ec2 authorize-security-group-ingress \
    --group-id sg-0c969660d42950669 \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0
```

## 5. Database Setup
### Create DB Subnet Group
```bash
aws rds create-db-subnet-group \
    --db-subnet-group-name laravel-db-subnet-group \
    --db-subnet-group-description "Subnet group for Laravel RDS" \
    --subnet-ids subnet-0413af3888b57728a subnet-0fc4e2d22108ea7d2
```

### Launch RDS Instance
```bash
aws rds create-db-instance \
    --db-instance-identifier laravel-prod-db \
    --db-instance-class db.t3.micro \
    --engine mysql \
    --master-username admin \
    --master-user-password admin123 \
    --allocated-storage 20 \
    --vpc-security-group-ids sg-0f61891a3cdd1b962 \
    --db-subnet-group-name laravel-db-subnet-group
```

## 6. Cache Setup
### Create ElastiCache Subnet Group
```bash
aws elasticache create-cache-subnet-group \
    --cache-subnet-group-name laravel-cache-subnet-group \
    --cache-subnet-group-description "Subnet group for Laravel Redis" \
    --subnet-ids subnet-0413af3888b57728a subnet-0fc4e2d22108ea7d2
```

### Launch Redis Cluster
```bash
aws elasticache create-cache-cluster \
    --cache-cluster-id laravel-prod-cache \
    --engine redis \
    --cache-node-type cache.t3.micro \
    --num-cache-nodes 1 \
    --security-group-ids sg-0f61891a3cdd1b962 \
    --cache-subnet-group-name laravel-cache-subnet-group
```

## 7. EC2 Instance Setup
### Create Key Pair
```bash
aws ec2 create-key-pair \
    --key-name laravel-prod-key \
    --query 'KeyMaterial' \
    --output text > laravel-prod-key.pem

chmod 400 laravel-prod-key.pem
```

### Launch EC2 Instance
```bash
aws ec2 run-instances \
    --image-id ami-0287a05f0ef0e9d9a \
    --instance-type t2.micro \
    --key-name laravel-prod-key \
    --security-group-ids sg-0c969660d42950669 \
    --subnet-id subnet-0eee116d20ee34413 \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=laravel-web-01},{Key=Environment,Value=production},{Key=Project,Value=laravel}]' \
    --associate-public-ip-address
```

## 8. Connect to EC2 Instance
Once the instance is running and you have its public IP:
```bash
ssh -i laravel-prod-key.pem ubuntu@<EC2-PUBLIC-IP>
```

## Important Security Notes
1. The current setup uses `admin123` as the database password. In production, use a strong password and store it securely.
2. The SSH and HTTP access is currently open to all IPs (0.0.0.0/0). Consider restricting to specific IP ranges.
3. Consider implementing AWS KMS for key management.
4. Enable automated backups for RDS in production.
5. Consider using AWS Secrets Manager for credential management.

## Cleanup Instructions
To avoid unnecessary charges, remember to delete resources when not needed:
1. Terminate EC2 instances
2. Delete RDS instances
3. Delete ElastiCache clusters
4. Delete security groups
5. Delete subnets
6. Detach and delete Internet Gateway
7. Delete VPC
