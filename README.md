# Three-Tier-Highly-Available-AWS-Architecture-using-Terraform

## Project Overview

This project provisions a production-style three-tier web application infrastructure on AWS using Terraform.

<p align="center">
  <img src="https://github.com/Jayanidu-Abeysinghe/Three-Tier-Highly-Available-AWS-Architecture-using-Terraform/blob/main/images/architecture%20diagram.png" width="800"><br><br>
  <em>Architecture Diagram</em>
</p>


The infrastructure includes:

- Public Application Load Balancer (ALB)
- Web tier running Next.js + Nginx
- Internal ALB for backend communication
- Application tier using Auto Scaling Group (ASG)
- Private RDS MySQL database
- Modular Terraform architecture for maintainability and reuse

---

## Architecture Summary

The architecture contains three layers:

- Web tier
- Application tier
- Database tier

The web tier is deployed in public subnets behind a public Application Load Balancer.

The application tier is deployed in private subnets behind an internal ALB with Auto Scaling Groups for scalability and fault tolerance.

The database tier uses Amazon RDS MySQL in Multi-AZ private subnets for high availability.

Terraform was used to automate provisioning of:

- VPC
- Subnets
- Route tables
- NAT Gateway
- Security Groups
- Application Load Balancers
- Auto Scaling Group
- EC2 Instances
- RDS MySQL

<p align="center">
  <img src="https://github.com/Jayanidu-Abeysinghe/Three-Tier-Highly-Available-AWS-Architecture-using-Terraform/blob/main/images/Screenshot%202026-05-14%20192732.png" width="800"><br><br>
  <em>PM2 status</em>
</p>

<p align="center">
  <img src="https://github.com/Jayanidu-Abeysinghe/Three-Tier-Highly-Available-AWS-Architecture-using-Terraform/blob/main/images/Screenshot%202026-05-14%20190015.png" width="800"><br><br>
  <em>Access the application using ALB DNS</em>
</p>

---

## Architecture Workflow

```text
[Internet User]
       ↓ HTTP (80)

[Public ALB]
← Public Subnets (AZ1 + AZ2)

       ↓ HTTP (80)

[Web EC2 - Next.js + Nginx]
← Public Subnet

       ↓ HTTP (3001)

[Internal ALB]
← Private App Subnets (AZ1 + AZ2)

       ↓ HTTP (3001)

[Target Group for App Tier]
← Managed by Auto Scaling Group

       ↓ HTTP (3001)

[App EC2 - Node.js Backend]
← Private Subnets (AZ1 + AZ2)

       ↓ MySQL (3306)

[RDS MySQL - Multi-AZ]
← Private DB Subnets (AZ1 + AZ2)
```

---

## Request Flow

1. User request reaches the **Public ALB** through the Internet Gateway
2. Public ALB forwards traffic to **Web EC2 (Next.js + Nginx)** in public subnets
3. Web EC2 serves frontend content and reverse proxies API requests to the **Internal ALB**
4. Internal ALB distributes traffic to the **Application EC2 instances** in the Auto Scaling Group
5. Backend EC2 instances execute application logic and communicate with **RDS MySQL**
6. Database responses return through the application layer back to the user
7. Private application instances access the internet through the **NAT Gateway** for updates and package installations

---

## Technologies Used

- Terraform
- AWS EC2
- AWS VPC
- AWS ALB
- AWS Auto Scaling Group (ASG)
- AWS RDS MySQL
- AWS Security Groups
- AWS NAT Gateway
- AWS Internet Gateway
- Nginx
- Next.js
- Node.js
- Linux

---

## Repository Structure

<p align="center">
  <img src="https://github.com/Jayanidu-Abeysinghe/Three-Tier-Highly-Available-AWS-Architecture-using-Terraform/blob/main/images/Screenshot%202026-05-04%20213649.png" width="800"><br><br>
  <em>Terraform apply</em>
</p>

```text
.
├── main.tf
├── variables.tf
├── outputs.tf
├── terraform.tfvars
├── modules/
│   ├── network/
│   ├── security/
│   ├── alb/
│   ├── compute/
│   └── database/
└── scripts/
```

---

## Module Responsibilities

## network/

Creates:
- VPC
- Public/private subnets
- Internet Gateway
- NAT Gateway
- Route tables

---

## security/

Creates:
- Security groups for:
  - Public ALB
  - Internal ALB
  - Web tier
  - App tier
  - Database

---

## alb/

Creates:
- Public ALB
- Internal ALB
- Listeners
- Target groups

---

## compute/

Creates:
- Web EC2 instance
- Launch template
- Auto Scaling Group
- SSH key pair
- AMI lookup

---

## database/

Creates:
- DB subnet group
- RDS MySQL instance

---

## How Modules Connect

1. Root `main.tf` reads values from `terraform.tfvars`
2. `module.network` creates networking resources and exports subnet IDs
3. `module.security` creates security groups using the VPC ID
4. `module.alb` creates ALBs using subnet IDs and security groups
5. `module.compute` launches EC2 instances and Auto Scaling Groups
6. `module.database` creates the RDS instance using private DB subnets

---

## Terraform Concepts Used

## Variables

### `variables.tf`

Declares expected input variables.

Example:

```hcl
variable "vpc_cidr" {
  type = string
}
```

---

## terraform.tfvars

Provides actual values for variables.

Example:

```hcl
vpc_cidr = "10.0.0.0/16"
```

---

## Outputs

Modules expose values using `outputs.tf`.

Example:

```hcl
output "public_subnet_id" {
  value = aws_subnet.public.id
}
```

Root module accesses outputs using:

```hcl
module.network.public_subnet_id
```

---

## Quick Start

## Prerequisites

Install:
- Terraform
- AWS CLI

Configure AWS credentials:

```bash
aws configure
```

---

## Initialize Terraform

```bash
terraform init
```

---

## Preview Infrastructure Changes

```bash
terraform plan
```

---

## Apply Infrastructure

```bash
terraform apply
```

---

## Destroy Infrastructure

```bash
terraform destroy
```

---

## Terraform Best Practices Used

- Modular Terraform architecture
- Reusable modules
- Environment-specific variables
- Infrastructure as Code (IaC)
- Private subnet database deployment
- Separate security groups
- Gitignored secrets and private keys

---

## Terraform Drift

Terraform detects infrastructure drift when manual changes are made in AWS.

Example:
- Terraform expects `t2.micro`
- AWS manually changed to `t2.medium`

Run:

```bash
terraform plan
```

Terraform detects the difference and shows required corrections.

---

## Terraform `-target`

Used to apply only specific resources/modules.

Example:

```bash
terraform apply -target=module.compute
```

Useful for:
- debugging
- testing
- emergency fixes

Not recommended for regular production workflows.

---

## Outputs

After deployment Terraform provides:

- External ALB DNS
- Internal ALB DNS
- RDS endpoint

---

## Security Notes

- `terraform.tfvars` is gitignored
- `.pem` private keys are gitignored
- Avoid hardcoding AWS credentials
- Use environment variables or AWS profiles

Recommended future improvements:
- AWS Secrets Manager
- Remote Terraform backend

---

## Future Improvements

- Remote state using S3 + DynamoDB
- GitHub Actions CI/CD pipeline
- Terraform Workspaces
- CloudWatch monitoring
- AWS Secrets Manager integration
- Containerization with Docker
- Kubernetes/EKS deployment

---

## Learning Outcomes

Through this project I learned:

- Terraform module architecture
- Infrastructure as Code (IaC)
- AWS networking concepts
- Multi-tier architecture
- Auto Scaling and Load Balancing
- Terraform state management
- Drift detection
- Reusable infrastructure design patterns

---

## Notes

This project was created for learning and portfolio purposes to demonstrate Terraform-based AWS infrastructure deployment using a modular architecture.
