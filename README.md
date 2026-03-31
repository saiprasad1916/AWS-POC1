Task 1: Highly Available 3-Tier Web Application on AWS

 1. Objective

Design and deploy a highly available, scalable, and fault-tolerant 3-tier web application on AWS using best practices.

The architecture ensures:

 High Availability across multiple Availability Zones (AZs)
 Auto Scaling based on demand
 Secure HTTPS communication
 Separation of concerns (Web, App, Storage)


 2. Architecture Overview

Flow:

Internet 
   ↓
Application Load Balancer (HTTPS:443)
   ↓
Target Group (EC2 Instances - Auto Scaling Group)
   ↓
EBS (Application Data)
   ↓
S3 (Static Assets)


 Components:

    VPC with Public & Private Subnets
    EC2 instances (App Layer)
    Auto Scaling Group (ASG)
    Application Load Balancer (ALB)
    S3 Bucket (Static Content)
    EBS Volumes (Encrypted)
    AWS Backup (Snapshots)

3. Step-by-Step Implementation

 Step 1: VPC and Networking Setup

 Objective

Create isolated network with high availability.

 Steps

1. Create VPC:
 ![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(434).png)
 ![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(435).png) 
2. Create Subnets:
    Public Subnet (AZ1): 10.0.1.0/24
    Public Subnet (AZ2): 10.0.2.0/24
    Private Subnet (AZ1): 10.0.3.0/24
    Private Subnet (AZ2): 10.0.4.0/24
3. Attach Internet Gateway (IGW)  
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(433).png)
5. Configure Route Tables:
    Public Route → IGW
    Private Route → NAT Gateway
6. Create NAT Gateway in Public Subnet 
Subnet Creation:

Create subnets with the below data.

Subnet Type	AZ	CIDR
Public-1	ap-south-1a	10.0.1.0/24
Private-1	ap-south-1a	10.0.2.0/24
Public-2	ap-south-1b	10.0.3.0/24
Private-2	ap-south-1b	10.0.4.0/24
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(427).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(428).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(429).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(430).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(431).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(432).png)

Step 3: Create Golden AMI

 Objective

Standardize application deployment.

 Steps
1. Launch EC2 instance
2. Install web server:
   sudo apt update
   sudo apt install nginx -y  
3. Deploy sample app:
   echo "Welcome to Production App" > /var/www/html/index.html
4. Create AMI from instance → Golden AMI
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(436).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(437).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(438).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(439).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(440).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(441).png)

 Step 4: Launch Template Creation

 Configuration:
   AMI: Golden AMI
   Instance Type: t3.micro
   Key Pair: Existing
Security Group:
  Allow HTTP (80) from ALB
 User Data:
bash:
 !/bin/bash
 systemctl start nginx
 systemctl enable nginx
 ![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(438).png)
 ![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(439).png)
 ![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(440).png)
 ![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(441).png)
 ![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(442).png)
 ![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(443).png)
 ![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(444).png)

 Step 5: Auto Scaling Group (ASG)

 Configuration:
 Subnets: Private Subnets (2 AZs)
   Desired: 2
   Min: 2
   Max: 6

 Scaling Policy:
   Scale Out: CPU > 60% Scale In: CPU < 30%
 Cooldown: 300 seconds
screenshots:
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(453).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(454).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(455).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(456).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(457).png)
 Step 6: EBS Volume Configuration

 Steps

1. Attach 20GB gp3 volume to each EC2
2. Enable encryption using KMS
Note: the options need to be check in before creating image , launch templete creation

 Step 7: Application Load Balancer (ALB)

 Steps:
1. Create ALB in Public Subnets
2. Listener:
    HTTP (80) → Redirect to HTTPS
    HTTPS (443)
3. Attach ACM Certificate (self-signed)
4. Create Target Group:

    Protocol: HTTP
    Port: 80
 Step 8: Enable Advanced ALB Features
 Configurations:
 Enable Cross-Zone Load Balancing
 Enable Sticky Sessions:
   Type: Duration-based
   Duration: 1 hour

screenshots:
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(450).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(451).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(452).png)

Alarm Setup :
 We have to set up alarm for scale up and scale down.
60>Scale out
30< Scale In
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(460).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(461).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(462).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(463).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(464).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(465).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(466).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(467).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(468).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(469).png)

Load Balancer:

Accessing application through DNS name from load balancer

Copy and paste dns name url in browser to access application.
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(458).png)
![Alt text](https://github.com/saiprasad1916/Task1_AWS/blob/master/scrrenshots/Screenshot%20(459).png)

S3 BUCKET CREATION:
create S3 bucket using versioning, enbling bucket key.

 9. Conclusion

This architecture demonstrates:

 High Availability
 Fault Tolerance
 Auto Scaling
 Secure Communication
 



