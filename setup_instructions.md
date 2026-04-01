# ⚙️ Setup Instructions – AWS Scalable Web Architecture

This guide explains how to set up the **highly available and fault-tolerant web application architecture** on AWS, step by step.  
The goal is to ensure **zero downtime** for end-users while accessing the website.

---

## 🛠️ Prerequisites
- AWS Account with admin permissions
- Basic understanding of VPC, EC2, RDS, and IAM
- Domain name (optional, for Route 53 integration)
- Architecture diagram for reference (see `architecture.png`)

---

## 🔹 Step 1: Create VPC and Networking
1. Create a **VPC** (e.g., `10.0.0.0/16`).
2. Add **2 Public Subnets** (e.g., `10.0.1.0/24`, `10.0.2.0/24`) – across two Availability Zones.
3. Add **2 Private Subnets** (e.g., `10.0.3.0/24`, `10.0.4.0/24`) – across the same AZs.
4. Create and attach an **Internet Gateway (IGW)** to the VPC.
5. Create **Route Tables**:
   - Public Subnet → Route to IGW
   - Private Subnet → No direct Internet access (only via NAT if required).

---

## 🔹 Step 2: Deploy Application Load Balancer (ALB)
1. Create an **Application Load Balancer** in the public subnets.
2. Configure **Target Groups** for EC2 web instances.
3. Attach **Security Groups**:
   - Allow inbound HTTP/HTTPS on ALB
   - Restrict web servers to receive traffic only from ALB

---

## 🔹 Step 3: Configure Auto Scaling Group (ASG)
1. Create a **Launch Template** (AMI + Instance Type + Security Group).
2. Create an **Auto Scaling Group**:
   - Attach to ALB Target Group.
   - Minimum: 2 instances, Maximum: (based on budget).
   - Spread across **2 AZs** for high availability.

---

## 🔹 Step 4: Set Up Database (RDS – Aurora)
1. Launch an **Aurora DB Cluster** in private subnets.
2. Enable **Multi-AZ deployment**.
3. Add a **Read Replica** for read-heavy traffic.
4. Configure **Security Groups**:
   - Allow DB traffic only from web servers’ SG.
   - No public access.

---

## 🔹 Step 5: Configure Shared Storage (EFS)
1. Create an **Amazon EFS File System**.
2. Add **Mount Targets** in each private subnet.
3. Attach EFS to EC2 web instances (via user-data script or manual mount).
4. Store shared content (e.g., images, media files).

---

## 🔹 Step 6: Host Static Assets on S3 + CloudFront
1. Create an **S3 bucket** for static assets (CSS, JS, Images).
2. Enable **Bucket Policy** for CloudFront access.
3. Create a **CloudFront Distribution**:
   - Origin → S3 bucket
   - Cache static assets globally for low latency.

---

## 🔹 Step 7: Route Traffic with Route 53
1. Buy/Register a domain in Route 53 (or use an existing one).
2. Create a **Hosted Zone**.
3. Add **A Record / CNAME**:
   - Point domain → CloudFront distribution
   - CloudFront → ALB → EC2 Auto Scaling Group

---

## 🔹 Step 8: Security Best Practices
- Use **IAM Roles** for EC2 to access S3/EFS.
- Restrict inbound/outbound rules with **Security Groups & NACLs**.
- Enable **CloudWatch Monitoring** for Auto Scaling, ALB, and RDS.
- Enable **Backups for RDS**.

---

## Docker Setup

This project includes a sample web app containerized using Docker.

### Build Image
docker build -t aws-app .

### Run Container
docker run -p 8080:80 aws-app

## Enhancements

- Added Docker containerization for application deployment
- Prepared project for CI/CD integration

## ✅ Final Output
- Users access website via **Route 53 domain name**.
- Requests flow through **CloudFront → ALB → Auto Scaling EC2 Web Instances**.
- Dynamic content served via **Aurora RDS**.
- Shared assets stored on **EFS + S3**.
- Entire architecture is **scalable, fault-tolerant, and highly available**.

---

## 📖 References
- [AWS VPC Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)  
- [AWS ALB Documentation](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)  
- [AWS Auto Scaling Documentation](https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html)  
- [Amazon RDS Documentation](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html)  
- [Amazon EFS Documentation](https://docs.aws.amazon.com/efs/latest/ug/whatisefs.html)  

---
