# AWS Infrastructure with Terraform

![Terraform](https://img.shields.io/badge/Terraform-1.5+-7B42BC?logo=terraform&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-EC2%20%7C%20VPC%20%7C%20ALB%20%7C%20S3-FF9900?logo=amazonaws&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-blue.svg)

A production-ready **three-tier cloud infrastructure** deployed on AWS using Terraform. This project provisions a highly available web application with a multi-AZ setup, Application Load Balancer, and object storage. It demonstrates core cloud engineering principles: network isolation, high availability, fault tolerance, and Infrastructure as Code.

---

## 📑 Table of Contents

- [Architecture](#-architecture)
- [Key Features](#-key-features)
- [Resources Created](#-resources-created)
- [Prerequisites](#-prerequisites)
- [Getting Started](#-getting-started)
- [Variables](#-variables)
- [Outputs](#-outputs)
- [Security Configuration](#-security-configuration)
- [Project Structure](#-project-structure)
- [Design Decisions](#-design-decisions)
- [Future Enhancements](#-future-enhancements)
- [Author](#-author)

---

🏗 Architecture

graph TB
    Internet((Internet))
    ALB[Application Load Balancer<br/>Internet-facing]
    
    subgraph VPC[VPC - 10.0.0.0/16]
        subgraph AZ1[Availability Zone us-east-1a]
            Sub1[Public Subnet<br/>10.0.0.0/24]
            EC2_1[EC2 Web Server 1<br/>t2.micro]
        end
        subgraph AZ2[Availability Zone us-east-1b]
            Sub2[Public Subnet<br/>10.0.1.0/24]
            EC2_2[EC2 Web Server 2<br/>t2.micro]
        end
        IGW[Internet Gateway]
        RT[Route Table<br/>0.0.0.0/0 → IGW]
        SG[Security Group<br/>Allow HTTP:80, SSH:22]
    end
    
    S3[S3 Bucket<br/>Static Assets]
    
    Internet --> ALB
    ALB -->|HTTP :80| EC2_1
    ALB -->|HTTP :80| EC2_2
    EC2_1 & EC2_2 -->|HTTPS/HTTP| S3
    Sub1 -.-> RT
    Sub2 -.-> RT
    RT --> IGW
    IGW --> Internet
    EC2_1 & EC2_2 -.-> SG



---

### 📄 `docs/KEY_FEATURES.md`

```markdown
# Key Features

- **High Availability**: Instances deployed in two separate Availability Zones (us-east-1a and us-east-1b).
- **Load Balancing**: Application Load Balancer distributes traffic evenly between the two instances, with health checks on `/`.
- **Infrastructure as Code**: Entire environment defined using Terraform, enabling repeatable deployments and version control.
- **Network Isolation**: Dedicated VPC, route tables, and security groups restrict access.
- **Scalable Design**: Easy to replace static EC2 instances with an Auto Scaling Group.
- **Object Storage**: S3 bucket provisioned for static content, decoupling storage from compute.
- **Auto-Healing**: Health checks ensure unhealthy instances are automatically taken out of rotation.
- **Cost-Effective**: Using t2.micro instances and S3 keeps the environment within free tier limits.
```

---

### 📄 `docs/RESOURCES.md`

```markdown
# Resources Created

| Resource Type | Name | Description |
|---------------|------|-------------|
| `aws_vpc` | myvpc | Custom VPC with configurable CIDR (default `10.0.0.0/16`) |
| `aws_subnet` | sub1 | Public subnet in `us-east-1a` (`10.0.0.0/24`) |
| `aws_subnet` | sub2 | Public subnet in `us-east-1b` (`10.0.1.0/24`) |
| `aws_internet_gateway` | igw | Internet Gateway attached to VPC |
| `aws_route_table` | RT | Route table with default route to Internet Gateway |
| `aws_route_table_association` | rta1, rta2 | Associates subnets with the route table |
| `aws_security_group` | webSg | Security group allowing HTTP (80) and SSH (22) inbound |
| `aws_instance` | webserver1 | EC2 instance in subnet 1 (us-east-1a) |
| `aws_instance` | webserver2 | EC2 instance in subnet 2 (us-east-1b) |
| `aws_s3_bucket` | example | S3 bucket for static assets |
| `aws_lb` | myalb | Application Load Balancer (internet-facing) |
| `aws_lb_target_group` | tg | Target group with HTTP health checks |
| `aws_lb_target_group_attachment` | attach1, attach2 | Registers EC2 instances with target group |
| `aws_lb_listener` | listener | Forwards HTTP traffic from ALB to target group |
```

---

### 📄 `docs/PREREQUISITES.md`

```markdown
# Prerequisites

- **Terraform** ≥ 1.0 installed ([Install Guide](https://learn.hashicorp.com/tutorials/terraform/install-cli))
- **AWS Account** with programmatic access (Access Key ID & Secret Access Key)
- **AWS CLI** configured locally (`aws configure`)
- Basic understanding of AWS VPC, EC2, and ELB services
```

---

### 📄 `docs/GETTING_STARTED.md`

````markdown
# Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/bharathyk2004/aws-terraform-three-tier.git
cd aws-terraform-three-tier
```

### 2. Configure Variables

Create a `terraform.tfvars` file (or edit `variables.tf` defaults) with your desired VPC CIDR:

```hcl
cidr = "10.0.0.0/16"
```

### 3. Initialize Terraform

```bash
terraform init
```

### 4. Review the Plan

```bash
terraform plan
```

### 5. Apply the Configuration

```bash
terraform apply
```

Type `yes` when prompted. After successful apply, Terraform will output the ALB DNS name.

### 6. Access the Application

```bash
terraform output loadbalancerdns
```

Open the DNS name in your browser. The load balancer will route traffic to one of the EC2 instances.

### 7. Clean Up Resources

```bash
terraform destroy
```

# Variables

| Variable | Description | Type | Default |
|----------|-------------|------|---------|
| `cidr` | CIDR block for the VPC | `string` | `"10.0.0.0/16"` |

Override via `terraform.tfvars` or `-var` flag:

```hcl
# terraform.tfvars
cidr = "10.0.0.0/16"


---

### 📄 `docs/OUTPUTS.md`

```markdown
# Outputs

| Output | Description |
|--------|-------------|
| `loadbalancerdns` | Public DNS name of the Application Load Balancer |
🔒 Security Configuration
Security Group Rules
Inbound:

Port	Protocol	Source	Description
80	TCP	0.0.0.0/0	HTTP access for web traffic
22	TCP	0.0.0.0/0	SSH access for administration
Outbound: All traffic allowed (0.0.0.0/0) to enable updates and S3 access.


⚠️ Production Hardening: Restrict SSH (port 22) to only your IP address. Replace the 0.0.0.0/0 with your public IP or use a bastion host.


🔐 HTTPS: Add an ACM certificate and listener on port 443 for encrypted traffic.


# Project Structure

```
.
├── main.tf              # Core infrastructure resources
├── variables.tf         # Input variable definitions
├── outputs.tf           # Output values
├── terraform.tfvars     # Variable values (add to .gitignore)
├── userdata.sh          # Bootstrap script for web server 1
├── userdata1.sh         # Bootstrap script for web server 2
├── docs/                # Documentation files
│   ├── ARCHITECTURE.md
│   ├── KEY_FEATURES.md
│   ├── RESOURCES.md
│   ├── PREREQUISITES.md
│   ├── GETTING_STARTED.md
│   ├── VARIABLES.md
│   ├── OUTPUTS.md
│   ├── SECURITY.md
│   ├── PROJECT_STRUCTURE.md
│   ├── DESIGN_DECISIONS.md
│   ├── FUTURE_ENHANCEMENTS.md
│   ├── AUTHOR.md
│   └── LICENSE.md
└── README.md            # Main documentation
```


# Design Decisions

- **Public Subnets Only**: Because the web servers need to serve traffic directly from the ALB, they reside in public subnets. A production three-tier architecture would typically place application servers in private subnets behind the ALB, with a NAT Gateway for outbound access. This project serves as a foundation for that evolution.

- **Two Availability Zones**: Using two AZs ensures high availability; if one AZ fails, the other instance continues serving traffic.

- **Application Load Balancer**: Chosen over Classic Load Balancer for advanced routing, health checks, and integration with target groups.

- **S3 for Static Assets**: Decouples static content from the web servers, improving performance and reducing load.

- **Terraform Modules Not Used**: Kept simple for learning, but the code can be refactored into reusable modules.


# Future Enhancements

- [ ] Add private subnets and NAT Gateway for a true three-tier architecture
- [ ] Introduce Auto Scaling Group to replace static instances
- [ ] Add RDS database for dynamic data (replacing S3 static usage)
- [ ] Use remote state (S3 backend + DynamoDB lock) for team collaboration
- [ ] Add CloudWatch alarms and SNS notifications
- [ ] Implement HTTPS with ACM certificate and Route 53 DNS
- [ ] Containerize the application with ECS or EKS
- [ ] Create a CI/CD pipeline to automatically deploy changes


👨‍💻 Author
Bharathy K
