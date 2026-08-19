AWS Infrastructure with Terraform

![Terraform](https://img.shields.io/badge/Terraform-1.5+-7B42BC?logo=terraform&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-EC2%2520%257C%2520VPC%2520%257C%2520ALB%2520%257C%2520S3-FF9900?logo=amazonaws&logoColor=white)
![Badge](https://img.shields.io/badge/License-MIT-blue.svg)

A production-ready three-tier cloud infrastructure deployed on AWS using Terraform. This project provisions a highly available web application with a multi-AZ setup, Application Load Balancer, and object storage. It demonstrates core cloud engineering principles: network isolation, high availability, fault tolerance, and Infrastructure as Code.


📑 Table of Contents
Architecture

Key Features

Resources Created

Prerequisites

Getting Started

Variables

Outputs

Security Configuration

Project Structure

Design Decisions

Future Enhancements

Author


🏗 Architecture

The architecture follows a three-tier pattern:

Presentation Tier: Application Load Balancer (ALB) that receives all user traffic.

Application Tier: Two EC2 instances (web servers) spread across two Availability Zones for high availability.

Data Tier: Amazon S3 bucket for storing static assets (images, files, etc.).

All resources reside inside a dedicated VPC with public subnets, allowing outbound internet access through an Internet Gateway.


✨ Key Features
High Availability: Instances deployed in two separate Availability Zones (us-east-1a and us-east-1b).

Load Balancing: Application Load Balancer distributes traffic evenly between the two instances, with health checks on /.

Infrastructure as Code: Entire environment defined using Terraform, enabling repeatable deployments and version control.

Network Isolation: Dedicated VPC, route tables, and security groups restrict access.

Scalable Design: Easy to replace static EC2 instances with an Auto Scaling Group.

Object Storage: S3 bucket provisioned for static content, decoupling storage from compute.


📦 Resources Created
Resource Type	Name	Description
aws_vpc	myvpc	Custom VPC with configurable CIDR (default 10.0.0.0/16)
aws_subnet	sub1	Public subnet in us-east-1a (10.0.0.0/24)
aws_subnet	sub2	Public subnet in us-east-1b (10.0.1.0/24)
aws_internet_gateway	igw	Internet Gateway attached to VPC
aws_route_table	RT	Route table with default route to Internet Gateway
aws_route_table_association	rta1, rta2	Associates subnets with the route table
aws_security_group	webSg	Security group allowing HTTP (80) and SSH (22) inbound
aws_instance	webserver1	EC2 instance in subnet 1 (us-east-1a)
aws_instance	webserver2	EC2 instance in subnet 2 (us-east-1b)
aws_s3_bucket	example	S3 bucket for static assets
aws_lb	myalb	Application Load Balancer (internet-facing)
aws_lb_target_group	tg	Target group with HTTP health checks
aws_lb_target_group_attachment	attach1, attach2	Registers EC2 instances with target group
aws_lb_listener	listener	Forwards HTTP traffic from ALB to target group


🔧 Prerequisites
Terraform ≥ 1.0 installed (Install Guide)

AWS Account with programmatic access (Access Key ID & Secret Access Key)

AWS CLI configured locally (aws configure)

Basic understanding of AWS VPC, EC2, and ELB services


🚀 Getting Started
1. Clone the Repository
bash
git clone https://github.com/bharathyk2004/aws-terraform-three-tier.git
cd aws-terraform-three-tier
2. Configure Variables
Create a terraform.tfvars file (or edit variables.tf defaults) with your desired VPC CIDR:

hcl
cidr = "10.0.0.0/16"
3. Initialize Terraform
bash
terraform init
This downloads the AWS provider and prepares the working directory.

4. Review the Plan
bash
terraform plan
Inspect the resources Terraform will create. Ensure the changes match your expectations.

5. Apply the Configuration
bash
terraform apply
Type yes when prompted. After successful apply, Terraform will output the ALB DNS name.

6. Access the Application
bash
terraform output loadbalancerdns
Open the DNS name in your browser. The load balancer will route traffic to one of the EC2 instances.

7. Clean Up Resources
To avoid incurring charges, destroy all resources when done:

bash
terraform destroy


📝 Variables
Variable	Description	Type	Default
cidr	CIDR block for the VPC	string	"10.0.0.0/16"
You can override this by creating a terraform.tfvars file or using -var flag.


📤 Outputs
Output	Description
loadbalancerdns	Public DNS name of the Application Load Balancer


🔒 Security Configuration
Security Group Rules
Inbound:

Port	Protocol	Source	Description
80	TCP	0.0.0.0/0	HTTP access for web traffic
22	TCP	0.0.0.0/0	SSH access for administration
Outbound: All traffic allowed (0.0.0.0/0) to enable updates and S3 access.


⚠️ Production Hardening: Restrict SSH (port 22) to only your IP address. Replace the 0.0.0.0/0 with your public IP or use a bastion host.


🔐 HTTPS: Add an ACM certificate and listener on port 443 for encrypted traffic.


📁 Project Structure
text
.
├── main.tf              # Core infrastructure resources
├── variables.tf         # Input variable definitions
├── terraform.tfvars     # Variable values (add to .gitignore)
├── userdata.sh          # Bootstrap script for web server 1
├── userdata1.sh         # Bootstrap script for web server 2
├── outputs.tf           # Output values
└── README.md            # Project documentation
Note: The userdata.sh and userdata1.sh scripts should contain the commands to install and start your web server (e.g., Apache/Nginx).


🧠 Design Decisions
Public Subnets Only: Because the web servers need to serve traffic directly from the ALB, they reside in public subnets. A production three-tier architecture would typically place application servers in private subnets behind the ALB, with a NAT Gateway for outbound access. This project serves as a foundation for that evolution.

Two Availability Zones: Using two AZs ensures high availability; if one AZ fails, the other instance continues serving traffic.

Application Load Balancer: Chosen over Classic Load Balancer for advanced routing, health checks, and integration with target groups.

S3 for Static Assets: Decouples static content from the web servers, improving performance and reducing load.

Terraform Modules Not Used: Kept simple for learning, but the code can be refactored into reusable modules.


🔮 Future Enhancements
□ Add private subnets and NAT Gateway for a true three-tier architecture
□ Introduce Auto Scaling Group to replace static instances
□ Add RDS database for dynamic data (replacing S3 static usage)
□ Use remote state (S3 backend + DynamoDB lock) for team collaboration
□ Add CloudWatch alarms and SNS notifications
□ Implement HTTPS with ACM certificate and Route 53 DNS
□ Containerize the application with ECS or EKS
□ Create a CI/CD pipeline to automatically deploy changes


👨‍💻 Author
Bharathy K
