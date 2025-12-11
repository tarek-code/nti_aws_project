# AWS Console Step-by-Step Guide

## Complete Manual Deployment Instructions

**Project:** GlobalBook Inc. - 3-Tier Multi-Region Architecture
**Method:** AWS Management Console (GUI)
**Purpose:** Detailed step-by-step instructions for creating the entire architecture manually using AWS Console

---

## Table of Contents

1. [Primary Region Setup](#primary-region-setup)
2. [Spare Region Setup](#spare-region-setup)
3. [Route 53 &amp; CloudFront Setup](#route-53--cloudfront-setup)

---

## Primary Region Setup

### Step 1: Create VPC

**Navigation:**

1. Sign in to AWS Management Console
2. Ensure you're in **us-east-1 (N. Virginia)** region (top right)
3. Go to **Services** → Search for **VPC** → Click **VPC**
4. In the left sidebar, click **Your VPCs**

**Creation Steps:**

1. Click the orange **Create VPC** button (top right)
2. In the **Create VPC** form, configure:
   - **Name tag:**
     - Enter: `primary-vpc`
   - **IPv4 CIDR block:**
     - Enter: `10.0.0.0/20`
     - This provides 4,096 IP addresses
   - **IPv6 CIDR block:**
     - Leave as **No IPv6 CIDR block**
   - **Tenancy:**
     - Select: **Default** (shared tenancy)
3. Click **Create VPC** button (bottom right)
4. You should see your VPC in the list with status "Available"

📸 **Screenshot Placeholder:** VPC creation form and VPC list showing primary-vpc

---

### Step 2: Create Internet Gateway

**Navigation:**

1. In VPC Dashboard, left sidebar → Click **Internet Gateways**

**Creation Steps:**

1. Click **Create Internet Gateway** button (top right)
2. In the **Create Internet Gateway** form:
   - **Name tag:**
     - Enter: `primary-igw`
3. Click **Create Internet Gateway** button
4. You'll see success message and the IGW in the list

**Attach Internet Gateway to VPC:**

1. Select the Internet Gateway you just created (check the box)
2. Click **Actions** dropdown (top right) → Select **Attach to VPC**
3. In the **Attach to VPC** dialog:
   - **Available VPCs:** Select `primary-vpc` from dropdown
4. Click **Attach Internet Gateway** button
5. Wait for status to change to "Attached"
6. The **State** column should show "Attached"

📸 **Screenshot Placeholder:** IGW creation form, attachment dialog, and IGW list showing "Attached" status

---

### Step 3: Create Regional NAT Gateway

**Navigation:**

1. In VPC Dashboard, left sidebar → Click **NAT Gateways**

**Creation Steps:**

1. Click **Create NAT Gateway** button (top right)
2. In the **Create NAT Gateway** form, configure:
   - **Name:**
     - Enter: `primary-regional-nat`
   - **Subnet:**
     - **Important:** For Regional NAT Gateway, you can select ANY subnet
     - Select one of your public subnets (we'll create them next, but you can come back)
     - OR: Select any existing subnet in your VPC
     - **Note:** Regional NAT Gateway doesn't require a public subnet like traditional NAT
   - **Elastic IP:**
     - **Leave blank** - Regional NAT Gateway doesn't require Elastic IP
     - The dropdown will be disabled or not shown
   - **Availability Mode:**
     - Select: **Regional** (this is the key setting!)
     - This enables automatic Multi-AZ expansion
     - Provides high availability without manual configuration
3. Click **Create NAT Gateway** button
4. **Important:** Wait for status to change to "Available"
   - This takes 2-5 minutes
   - Status will show "Pending" initially
   - Don't proceed until status is "Available"

📸 **Screenshot Placeholder:** NAT Gateway creation form showing Regional mode, and NAT Gateway list showing "Available" status

---

### Step 4: Create Public Subnets (Web Tier)

**Navigation:**

1. VPC Dashboard → Left sidebar → Click **Subnets**
2. Click **Create Subnet** button (top right)

**Subnet A - Creation Steps:**

1. **Configure Subnet:**
   - **VPC:**
     - Select `primary-vpc` from dropdown
   - **Subnet name:**
     - Enter: `Web-Public-A`
   - **Availability Zone:**
     - Click dropdown → Select first AZ (e.g., us-east-1a)
     - **Important:** Note which AZ this is for later steps
   - **IPv4 CIDR block:**
     - Enter: `10.0.0.0/24`
     - This provides 256 IP addresses
2. Click **Create Subnet** button
3. You'll see the subnet in the list
4. **Configure Auto-Assign IP Settings:**
   - Select the subnet `Web-Public-A` (check the box)
   - Click **Actions** dropdown → Select **Edit subnet settings**
   - In the dialog:
     - **Enable auto-assign public IPv4 address:** Uncheck this box
     - ALB will handle public access, instances don't need direct public IPs
   - Click **Save** button

**Subnet B - Creation Steps:**

1. Click **Create Subnet** button again
2. Repeat the same steps with these values:
   - **Subnet name:** `Web-Public-B`
   - **Availability Zone:** Select second AZ (e.g., us-east-1b)
     - **Important:** Must be different from Subnet A
   - **IPv4 CIDR block:** `10.0.1.0/24`
3. Configure auto-assign IP settings the same way (uncheck public IP)

📸 **Screenshot Placeholder:** Subnet creation form, subnet list showing both public subnets, and subnet settings dialog

---

### Step 5: Create Private Subnets (App Tier)

**Navigation:**

1. VPC Dashboard → Subnets → **Create Subnet**

**Subnet A1 - Creation Steps:**

1. **Configure Subnet:**
   - **VPC:** Select `primary-vpc`
   - **Subnet name:** Enter: `App-Private-A1`
   - **Availability Zone:** Select first AZ (same as PublicSubnetA, e.g., us-east-1a)
   - **IPv4 CIDR block:** Enter: `10.0.10.0/24`
2. Click **Create Subnet** button

**Subnet B1 - Creation Steps:**

1. Click **Create Subnet** button again
2. Repeat with these values:
   - **Subnet name:** `App-Private-B1`
   - **Availability Zone:** Select second AZ (same as PublicSubnetB, e.g., us-east-1b)
   - **IPv4 CIDR block:** `10.0.12.0/24`
3. Click **Create Subnet** button

📸 **Screenshot Placeholder:** App tier subnets creation and subnet list

---

### Step 6: Create Private Subnets (DB Tier)

**Navigation:**

1. VPC Dashboard → Subnets → **Create Subnet**

**Subnet A2 - Creation Steps:**

1. **Configure Subnet:**
   - **VPC:** Select `primary-vpc`
   - **Subnet name:** Enter: `DB-Private-A2`
   - **Availability Zone:** Select first AZ (e.g., us-east-1a)
   - **IPv4 CIDR block:** Enter: `10.0.11.0/24`
2. Click **Create Subnet** button

**Subnet B2 - Creation Steps:**

1. Click **Create Subnet** button again
2. Repeat with these values:
   - **Subnet name:** `DB-Private-B2`
   - **Availability Zone:** Select second AZ (e.g., us-east-1b)
   - **IPv4 CIDR block:** `10.0.13.0/24`
3. Click **Create Subnet** button

📸 **Screenshot Placeholder:** DB tier subnets creation and subnet list

---

### Step 7: Create Route Tables

**Public Route Table - Creation Steps:**

1. **Navigation:**
   - VPC Dashboard → Left sidebar → Click **Route Tables**
   - Click **Create Route Table** button (top right)
2. **Configure Route Table:**
   - **Name:** Enter: `Public-RT`
   - **VPC:** Select `primary-vpc` from dropdown
3. Click **Create Route Table** button
4. **Add Route to Internet Gateway:**
   - Select the `Public-RT` route table (check the box)
   - Click on the **Routes** tab (below the route table list)
   - Click **Edit routes** button
   - Click **Add route** button
   - **Configure Route:**
     - **Destination:** Enter `0.0.0.0/0`
     - **Target:** Click dropdown → Select **Internet Gateway** → Select `primary-igw`
   - Click **Save changes** button
5. **Associate Subnets:**
   - Click **Subnet associations** tab → Click **Edit subnet associations**
   - Check: `Web-Public-A`
   - Check: `Web-Public-B`
   - Click **Save associations**

**Private Route Table - Creation Steps:**

1. Click **Create Route Table** button again
2. **Configure Route Table:**
   - **Name:** Enter: `Private-RT`
   - **VPC:** Select `primary-vpc`
3. Click **Create Route Table** button
4. **Add Route to NAT Gateway:**
   - Select the `Private-RT` route table
   - Click **Routes** tab → **Edit routes** → **Add route**
   - **Configure Route:**
     - **Destination:** Enter `0.0.0.0/0`
     - **Target:** Click dropdown → Select **NAT Gateway** → Select `primary-regional-nat`
   - Click **Save changes**
5. **Associate Subnets:**
   - Click **Subnet associations** tab → **Edit subnet associations**
   - Check: `App-Private-A1`
   - Check: `App-Private-B1`
   - Check: `DB-Private-A2`
   - Check: `DB-Private-B2`
   - Click **Save associations**

📸 **Screenshot Placeholder:** Route table creation form, routes tab showing IGW and NAT routes, subnet associations

---

### Step 8: Create Security Groups

**Web Security Group - Creation Steps:**

1. **Navigation:**
   - VPC Dashboard → Left sidebar → Click **Security Groups**
   - Click **Create Security Group** button (top right)
2. **Basic Details:**
   - **Security group name:** Enter: `Web-SG`
   - **Description:** Enter: `Web/ALB ingress 80/443`
   - **VPC:** Select `primary-vpc` from dropdown
3. **Inbound Rules:**
   - Click **Add rule** button
   - **Rule 1:**
     - **Type:** Select **HTTP** from dropdown
     - **Source:** Select **Anywhere-IPv4** OR enter `0.0.0.0/0`
     - **Description:** Allow HTTP from Internet
     - Click **Add rule** button again
   - **Rule 2:**
     - **Type:** Select **HTTPS** from dropdown
     - **Source:** Select **Anywhere-IPv4** OR enter `0.0.0.0/0`
     - **Description:** Allow HTTPS from Internet
     - Click **Add rule** button again
   - **Rule 3:**
     - **Type:** Select **Custom TCP** from dropdown
     - **Port range:** Enter `80`
     - **Source:** Select **Custom** → Click **Select a security group** → Select `Web-SG` (self-reference)
     - **Description:** Allow ALB to reach instances
4. **Outbound Rules:**
   - Leave default (allows all outbound traffic)
5. Click **Create Security Group** button

**App ALB Security Group - Creation Steps:**

1. Click **Create Security Group** button again
2. **Basic Details:**
   - **Security group name:** Enter: `App-ALB-SG`
   - **Description:** Enter: `Internal App ALB`
   - **VPC:** Select `primary-vpc`
3. **Inbound Rules:**
   - Click **Add rule** button
   - **Rule 1:**
     - **Type:** Select **Custom TCP** from dropdown
     - **Port range:** Enter `80`
     - **Source:** Select **Custom** → Click **Select a security group** → Select `Web-SG`
     - **Description:** Allow from Web tier
4. **Outbound Rules:** Leave default
5. Click **Create Security Group** button

**App Security Group - Creation Steps:**

1. Click **Create Security Group** button again
2. **Basic Details:**
   - **Security group name:** Enter: `App-SG`
   - **Description:** Enter: `App ingress from App ALB`
   - **VPC:** Select `primary-vpc`
3. **Inbound Rules:**
   - Click **Add rule** button
   - **Rule 1:**
     - **Type:** Select **Custom TCP** from dropdown
     - **Port range:** Enter `80`
     - **Source:** Select **Custom** → Click **Select a security group** → Select `App-ALB-SG`
     - **Description:** Allow from App ALB
4. **Outbound Rules:** Leave default
5. Click **Create Security Group** button

**DB Security Group - Creation Steps:**

1. Click **Create Security Group** button again
2. **Basic Details:**
   - **Security group name:** Enter: `DB-SG`
   - **Description:** Enter: `DB ingress from app tier`
   - **VPC:** Select `primary-vpc`
3. **Inbound Rules:**
   - Click **Add rule** button
   - **Rule 1:**
     - **Type:** Select **MySQL/Aurora** from dropdown (or Custom TCP)
     - **Port range:** `3306` (auto-filled if MySQL/Aurora selected)
     - **Source:** Select **Custom** → Click **Select a security group** → Select `App-SG`
     - **Description:** Allow from App tier
4. **Outbound Rules:** Leave default
5. Click **Create Security Group** button

📸 **Screenshot Placeholder:** Security group creation forms for all 4 security groups, showing inbound rules configuration

---

### Step 9: Create Application Load Balancer (Web Tier)

**Navigation:**

1. Go to **Services** → Search for **EC2** → Click **EC2**
2. In the left sidebar, click **Load Balancers**
3. Click **Create Load Balancer** button (top right)

**Creation Steps:**

1. **Select Load Balancer Type:**
   - Click **Create** button under **Application Load Balancer**
2. **Basic Configuration:**
   - **Name:** Enter: `primary-alb`
   - **Scheme:** Select **Internet-facing**
   - **IP address type:** Select **IPv4**
3. **Network Mapping:**
   - **VPC:** Select `primary-vpc`
   - **Availability Zones:**
     - Check: `us-east-1a` → Select subnet `Web-Public-A`
     - Check: `us-east-1b` → Select subnet `Web-Public-B`
4. **Security Groups:**
   - Select: `Web-SG` (check the box)
   - Remove default security group if selected
5. **Listeners and Routing:**
   - **Protocol:** HTTP
   - **Port:** 80
   - **Default action:** Forward to new target group
   - Click **Create target group** link (opens in new tab/window)
     - **Target type:** Instances
     - **Target group name:** `primary-alb-tg`
     - **Protocol:** HTTP
     - **Port:** 80
     - **VPC:** Select `primary-vpc`
     - **Health check protocol:** HTTP
     - **Health check path:** `/`
     - **Advanced health check settings:**
       - **Interval:** 30 seconds
       - **Timeout:** 5 seconds
       - **Healthy threshold:** 2
       - **Unhealthy threshold:** 3
     - Click **Next** → Skip adding targets → Click **Create target group**
   - Return to ALB creation tab
   - **Default action:** Select `primary-alb-tg` from dropdown
6. Click **Create Load Balancer** button
7. Wait for status to change to "Active" (takes 2-5 minutes)

📸 **Screenshot Placeholder:** ALB creation form, target group creation form, ALB list showing "Active" status

---

### Step 10: Create Internal Application Load Balancer (App Tier)

**Navigation:**

1. EC2 Dashboard → Load Balancers → **Create Load Balancer**

**Creation Steps:**

1. **Select Load Balancer Type:**
   - Click **Create** button under **Application Load Balancer**
2. **Basic Configuration:**
   - **Name:** Enter: `app-internal-alb`
   - **Scheme:** Select **Internal**
   - **IP address type:** Select **IPv4**
3. **Network Mapping:**
   - **VPC:** Select `primary-vpc`
   - **Availability Zones:**
     - Check: `us-east-1a` → Select subnet `App-Private-A1`
     - Check: `us-east-1b` → Select subnet `App-Private-B1`
4. **Security Groups:**
   - Select: `App-ALB-SG` (check the box)
5. **Listeners and Routing:**
   - **Protocol:** HTTP
   - **Port:** 80
   - **Default action:** Forward to new target group
   - Click **Create target group** link
     - **Target type:** Instances
     - **Target group name:** `app-alb-tg`
     - **Protocol:** HTTP
     - **Port:** 80
     - **VPC:** Select `primary-vpc`
     - **Health check protocol:** HTTP
     - **Health check path:** `/`
     - **Advanced health check settings:** Same as Web ALB
     - Click **Next** → Skip adding targets → Click **Create target group**
   - Return to ALB creation tab
   - **Default action:** Select `app-alb-tg` from dropdown
6. Click **Create Load Balancer** button
7. Wait for status to change to "Active"

📸 **Screenshot Placeholder:** Internal ALB creation form and ALB list

---

### Step 11: Create Launch Templates

**Web Launch Template - Creation Steps:**

1. **Navigation:**
   - EC2 Dashboard → Left sidebar → Click **Launch Templates**
   - Click **Create Launch Template** button (top right)
2. **Launch Template Name and Description:**
   - **Launch template name:** Enter: `web-launch-template`
   - **Template version description:** Enter: `Web tier launch template`
3. **AMI:**
   - Click **Quick Start** tab
   - Select **Amazon Linux 2 AMI** (or search for latest Amazon Linux 2)
   - **Note:** Use AMI ID `ami-0c02fb55956c7d316` for us-east-1 (or find latest in AMI catalog)
4. **Instance Type:**
   - Select: `t3.micro`
5. **Key Pair:**
   - Select: `vockey` (or your key pair)
6. **Network Settings:**
   - **Security groups:** Select `Web-SG`
   - **Note:** Don't specify subnet here (ASG will handle it)
7. **Advanced Details:**
   - Scroll down to **User Data** section
   - Paste the following script:
     ```bash
     #!/bin/bash
     set -e
     yum update -y
     yum install -y httpd
     echo "<html><body><h1>Welcome to my ASG instance (Web)</h1><p>Instance is healthy!</p></body></html>" > /var/www/html/index.html
     echo "OK" > /var/www/html/health
     chmod 644 /var/www/html/index.html
     chmod 644 /var/www/html/health
     systemctl enable httpd
     systemctl start httpd
     sleep 15
     systemctl status httpd || systemctl restart httpd
     ```
8. Click **Create Launch Template** button

**App Launch Template - Creation Steps:**

1. Click **Create Launch Template** button again
2. **Launch Template Name and Description:**
   - **Launch template name:** Enter: `app-launch-template`
   - **Template version description:** Enter: `App tier launch template`
3. **AMI:** Same as Web (Amazon Linux 2)
4. **Instance Type:** `t3.micro`
5. **Key Pair:** `vockey`
6. **Network Settings:**
   - **Security groups:** Select `App-SG`
7. **Advanced Details:**
   - **User Data:** Paste the following script:
     ```bash
     #!/bin/bash
     set -e
     yum update -y
     yum install -y httpd
     echo "<html><body><h1>Welcome to my ASG instance (App)</h1><p>Instance is healthy!</p></body></html>" > /var/www/html/index.html
     echo "OK" > /var/www/html/health
     chmod 644 /var/www/html/index.html
     chmod 644 /var/www/html/health
     systemctl enable httpd
     systemctl start httpd
     sleep 15
     systemctl status httpd || systemctl restart httpd
     ```
8. Click **Create Launch Template** button

📸 **Screenshot Placeholder:** Launch template creation forms showing AMI selection, instance type, security groups, and user data sections

---

### Step 12: Create Auto Scaling Groups

**Web Auto Scaling Group - Creation Steps:**

1. **Navigation:**
   - EC2 Dashboard → Left sidebar → Click **Auto Scaling Groups**
   - Click **Create Auto Scaling Group** button (top right)
2. **Choose Launch Template or Configuration:**
   - Select: **Launch template**
   - **Launch template:** Select `web-launch-template`
   - **Version:** Select **Latest**
   - Click **Next** button
3. **Configure Auto Scaling Group:**
   - **Auto Scaling group name:** Enter: `web-asg`
   - **VPC:** Select `primary-vpc`
   - **Availability Zones and subnets:**
     - Check: `us-east-1a` → Select `Web-Public-A`
     - Check: `us-east-1b` → Select `Web-Public-B`
   - Click **Next** button
4. **Configure Advanced Options:**
   - **Load balancing:** Select **Attach to an existing load balancer**
   - **Choose a target group for your load balancer:** Select `primary-alb-tg`
   - **Health check type:** Select **ELB**
   - **Health check grace period:** Enter `300` (5 minutes)
   - Click **Next** button
5. **Configure Group Size and Scaling Policies:**
   - **Desired capacity:** Enter `2`
   - **Minimum capacity:** Enter `2`
   - **Maximum capacity:** Enter `4`
   - **Scaling policies:** Select **Target tracking scaling policy**
     - **Metric type:** Average CPU utilization
     - **Target value:** Enter `50`
     - **Instances need:** Enter `300` seconds warm-up
   - Click **Next** button
6. **Add Notifications:** Skip (or configure SNS later)
7. **Add Tags:**
   - **Key:** `Name`
   - **Value:** `web-asg`
   - Click **Add tag** → Click **Next** button
8. **Review:** Review all settings → Click **Create Auto Scaling Group** button

**App Auto Scaling Group - Creation Steps:**

1. Click **Create Auto Scaling Group** button again
2. **Choose Launch Template:**
   - **Launch template:** Select `app-launch-template`
   - **Version:** Latest
   - Click **Next**
3. **Configure Auto Scaling Group:**
   - **Auto Scaling group name:** Enter: `app-asg`
   - **VPC:** Select `primary-vpc`
   - **Availability Zones and subnets:**
     - Check: `us-east-1a` → Select `App-Private-A1`
     - Check: `us-east-1b` → Select `App-Private-B1`
   - Click **Next**
4. **Configure Advanced Options:**
   - **Load balancing:** Select **Attach to an existing load balancer**
   - **Choose a target group:** Select `app-alb-tg`
   - **Health check type:** Select **ELB**
   - **Health check grace period:** Enter `300`
   - Click **Next**
5. **Configure Group Size and Scaling Policies:**
   - **Desired capacity:** Enter `2`
   - **Minimum capacity:** Enter `2`
   - **Maximum capacity:** Enter `4`
   - **Scaling policies:** Target tracking scaling policy (CPU 50%)
   - Click **Next**
6. **Add Tags:**
   - **Key:** `Name`
   - **Value:** `app-asg`
   - Click **Next**
7. **Review:** Click **Create Auto Scaling Group** button

📸 **Screenshot Placeholder:** ASG creation wizard showing launch template selection, subnet selection, target group attachment, scaling policies

---

### Step 13: Create RDS Database

**Navigation:**

1. Go to **Services** → Search for **RDS** → Click **RDS**
2. In the left sidebar, click **Databases**
3. Click **Create Database** button (top right)

**Creation Steps:**

1. **Choose a database creation method:**
   - Select **Standard create**
2. **Engine Options:**
   - **Engine type:** Select **MariaDB**
   - **Version:** Select latest version (e.g., MariaDB 10.11.x)
3. **Templates:**
   - Select **Dev/Test** (or Free tier if available)
   - **Note:** Multi-AZ is not supported in sandbox
4. **Settings:**
   - **DB instance identifier:** Enter: `primary-mariadb`
   - **Master username:** Enter your database username (e.g., `admin`)
   - **Master password:** Enter a strong password (min 8 characters)
   - **Confirm password:** Re-enter password
5. **Instance Configuration:**
   - **DB instance class:** Select **db.t3.micro**
6. **Storage:**
   - **Storage type:** Select **General Purpose SSD (gp2)**
   - **Allocated storage:** Enter `20` GB
   - **Enable storage autoscaling:** Uncheck (optional)
7. **Connectivity:**
   - **VPC:** Select `primary-vpc`
   - **Subnet group:** Click **Create new DB subnet group**
     - **Name:** `primary-db-subnetgrp`
     - **Description:** `Primary DB subnets`
     - **VPC:** Select `primary-vpc`
     - **Availability Zones:** Select `us-east-1a` and `us-east-1b`
     - **Subnets:** Select `DB-Private-A2` and `DB-Private-B2`
     - Click **Create** button
   - **Publicly accessible:** Select **No**
   - **VPC security group:** Select **Choose existing**
     - Select: `DB-SG`
   - **Availability Zone:** Leave default (No preference)
8. **Database Authentication:**
   - Select **Password authentication**
9. **Additional Configuration:**
   - **Initial database name:** Enter: `globalbookdb`
   - **DB parameter group:** Leave default
   - **Backup:** Configure as needed (7 days retention)
   - **Enable encryption:** Optional (uncheck for sandbox)
   - **Enable Enhanced monitoring:** **Uncheck** (not supported in sandbox)
   - **Log exports:** Leave default
10. **Maintenance:**
    - Leave defaults
11. Click **Create Database** button
12. Wait for status to change to "Available" (takes 5-10 minutes)

📸 **Screenshot Placeholder:** RDS creation form showing MariaDB selection, instance class, subnet group creation, security group selection

---

### Step 14: Create SNS Topic and CloudWatch Alarms

**SNS Topic - Creation Steps:**

1. **Navigation:**
   - Go to **Services** → Search for **SNS** → Click **Simple Notification Service**
   - In the left sidebar, click **Topics**
   - Click **Create Topic** button (top right)
2. **Create Topic:**
   - **Type:** Select **Standard**
   - **Name:** Enter: `primary-alarms`
   - Click **Create Topic** button
3. **Create Subscription:**
   - Select the topic you just created
   - Click **Create Subscription** button
   - **Protocol:** Select **Email**
   - **Endpoint:** Enter: `dobiya4358@crsay.com`
   - Click **Create Subscription** button
   - **Important:** Check your email and confirm the subscription!

**CloudWatch Alarm for Web ASG - Creation Steps:**

1. **Navigation:**
   - Go to **Services** → Search for **CloudWatch** → Click **CloudWatch**
   - In the left sidebar, click **Alarms** → **All Alarms**
   - Click **Create Alarm** button (top right)
2. **Specify Metric and Conditions:**
   - Click **Select Metric** button
   - **Namespace:** Select **EC2**
   - **Metric name:** Select **CPUUtilization**
   - **Dimensions:** Click **Auto Scaling Group Name** → Select `web-asg`
   - Click **Select Metric** button
3. **Metric Conditions:**
   - **Statistic:** Select **Average**
   - **Period:** Select **5 minutes**
   - **Threshold type:** Select **Static**
   - **Whenever CPUUtilization is:** Select **Greater**
   - **than:** Enter `70`
   - Click **Next** button
4. **Configure Actions:**
   - **Alarm state trigger:** Check **Alarm**
   - **Notification:** Select **Send a notification to an SNS topic**
   - **Select an SNS topic:** Select `primary-alarms`
   - Click **Next** button
5. **Add Name and Description:**
   - **Alarm name:** Enter: `web-asg-high-cpu`
   - **Description:** Enter: `Alert when Web ASG CPU exceeds 70%`
   - Click **Next** button
6. **Preview and Create:** Review → Click **Create Alarm** button

**CloudWatch Alarm for App ASG - Creation Steps:**

1. Repeat the same steps with:
   - **Dimensions:** Select `app-asg`
   - **Alarm name:** `app-asg-high-cpu`
   - **Description:** `Alert when App ASG CPU exceeds 70%`

📸 **Screenshot Placeholder:** SNS topic creation, subscription creation, CloudWatch alarm creation forms

---

## Spare Region Setup

**Important:** Switch to **us-west-2 (Oregon)** region before starting!

### Step 1: Create Spare Region VPC

**Navigation:**

1. Click **region selector** (top right) → Select **us-west-2 (Oregon)**
2. Go to **Services** → **VPC** → **Your VPCs**

**Creation Steps:**

1. Click **Create VPC** button
2. **Configure:**
   - **Name tag:** `spare-vpc`
   - **IPv4 CIDR block:** `10.1.0.0/20`
3. Click **Create VPC** button

📸 **Screenshot Placeholder:** Spare VPC creation

---

### Step 2: Create Spare Region Internet Gateway

**Navigation:**

1. VPC Dashboard → **Internet Gateways**

**Creation Steps:**

1. Click **Create Internet Gateway** button
2. **Name tag:** Enter: `spare-igw`
3. Click **Create Internet Gateway** button
4. **Attach to VPC:**
   - Select the IGW → **Actions** → **Attach to VPC**
   - Select: `spare-vpc`
   - Click **Attach**

📸 **Screenshot Placeholder:** Spare IGW creation and attachment

---

### Step 3: Create Spare Region Subnets (Single AZ)

**Navigation:**

1. VPC Dashboard → **Subnets** → **Create Subnet**

**Web Subnet - Creation Steps:**

1. **VPC:** Select `spare-vpc`
2. **Subnet name:** `Spare-Web-Public`
3. **Availability Zone:** Select any single AZ (e.g., us-west-2a)
4. **IPv4 CIDR block:** `10.1.0.0/24`
5. Click **Create Subnet**
6. **Edit subnet settings:** Uncheck auto-assign public IP

**App Subnet - Creation Steps:**

1. Click **Create Subnet** again
2. **Subnet name:** `Spare-App-Private`
3. **Availability Zone:** Same AZ as Web subnet
4. **IPv4 CIDR block:** `10.1.10.0/24`
5. Click **Create Subnet**

**DB Subnet - Creation Steps:**

1. Click **Create Subnet** again
2. **Subnet name:** `Spare-DB-Private`
3. **Availability Zone:** Same AZ as other subnets
4. **IPv4 CIDR block:** `10.1.11.0/24`
5. Click **Create Subnet**

📸 **Screenshot Placeholder:** Spare region subnets

---

### Step 4: Create Spare Region Regional NAT Gateway

**Navigation:**

1. VPC Dashboard → **NAT Gateways** → **Create NAT Gateway**

**Creation Steps:**

1. **Name:** `spare-regional-nat`
2. **Subnet:** Select any subnet (Regional NAT doesn't require specific subnet)
3. **Availability Mode:** Select **Regional**
4. Click **Create NAT Gateway**
5. Wait for "Available" status

📸 **Screenshot Placeholder:** Spare Regional NAT Gateway

---

### Step 5: Create Spare Region Route Tables

**Public Route Table:**

1. VPC Dashboard → **Route Tables** → **Create Route Table**
2. **Name:** `Spare-Public-RT`
3. **VPC:** `spare-vpc`
4. **Add route:** `0.0.0.0/0` → `spare-igw`
5. **Associate subnet:** `Spare-Web-Public`

**Private Route Table:**

1. **Create Route Table** again
2. **Name:** `Spare-Private-RT`
3. **VPC:** `spare-vpc`
4. **Add route:** `0.0.0.0/0` → `spare-regional-nat`
5. **Associate subnets:** `Spare-App-Private`, `Spare-DB-Private`

📸 **Screenshot Placeholder:** Spare route tables

---

### Step 6: Create Spare Region Security Groups

**Web Security Group:**

1. VPC Dashboard → **Security Groups** → **Create Security Group**
2. **Name:** `Spare-Web-SG`
3. **Description:** `Web/NLB ingress 80/443`
4. **VPC:** `spare-vpc`
5. **Inbound Rules:**
   - HTTP (80) from `0.0.0.0/0`
   - HTTPS (443) from `0.0.0.0/0`

**App Security Group:**

1. **Create Security Group**
2. **Name:** `Spare-App-SG`
3. **Description:** `App ingress from web tier`
4. **VPC:** `spare-vpc`
5. **Inbound Rules:**
   - Custom TCP (80) from `10.1.0.0/24` (Web subnet CIDR)

**DB Security Group:**

1. **Create Security Group**
2. **Name:** `Spare-DB-SG`
3. **Description:** `DB ingress from app tier`
4. **VPC:** `spare-vpc`
5. **Inbound Rules:**
   - MySQL/Aurora (3306) from `Spare-App-SG` security group

📸 **Screenshot Placeholder:** Spare security groups

---

### Step 7: Create Network Load Balancer (Spare Region)

**Navigation:**

1. EC2 Dashboard → **Load Balancers** → **Create Load Balancer**

**Creation Steps:**

1. **Select Load Balancer Type:**
   - Click **Create** under **Network Load Balancer**
2. **Basic Configuration:**
   - **Name:** `spare-nlb`
   - **Scheme:** **Internet-facing**
   - **IP address type:** **IPv4**
3. **Network Mapping:**
   - **VPC:** `spare-vpc`
   - **Availability Zones:**
     - Check: `us-west-2a` → Select `Spare-Web-Public`
4. **Security Groups:** Select `Spare-Web-SG`
5. **Listeners and Routing:**
   - **Protocol:** TCP
   - **Port:** 80
   - **Default action:** Forward to new target group
   - **Create target group:**
     - **Target type:** Instances
     - **Name:** `spare-nlb-tg`
     - **Protocol:** TCP
     - **Port:** 80
     - **VPC:** `spare-vpc`
     - **Health check protocol:** TCP
     - **Health check port:** 80
     - Click **Create target group**
   - Return to NLB creation → Select `spare-nlb-tg`
6. Click **Create Load Balancer**
7. Wait for "Active" status

📸 **Screenshot Placeholder:** NLB creation form

---

### Step 8: Create Spare Region Launch Templates and ASGs

**Web Launch Template:**

1. EC2 → **Launch Templates** → **Create Launch Template**
2. **Name:** `spare-web-launch-template`
3. **AMI:** Amazon Linux 2 (use appropriate AMI for us-west-2)
4. **Instance type:** `t3.micro`
5. **Security groups:** `Spare-Web-SG`
6. **User Data:** Same Apache script as primary region
7. Click **Create Launch Template**

**App Launch Template:**

1. **Name:** `spare-app-launch-template`
2. **Security groups:** `Spare-App-SG`
3. Same configuration as Web template
4. Click **Create Launch Template**

**Web ASG:**

1. EC2 → **Auto Scaling Groups** → **Create Auto Scaling Group**
2. **Launch template:** `spare-web-launch-template`
3. **Name:** `spare-web-asg`
4. **Subnets:** `Spare-Web-Public` (single subnet)
5. **Target group:** `spare-nlb-tg`
6. **Desired/Min:** `1`, **Max:** `3`
7. **Health check:** ELB, grace period 300
8. **Scaling policy:** Target tracking (CPU 50%)
9. Click **Create Auto Scaling Group**

**App ASG:**

1. **Name:** `spare-app-asg`
2. **Subnets:** `Spare-App-Private`
3. **Target group:** None (no ALB in spare region)
4. **Desired/Min:** `1`, **Max:** `3`
5. Click **Create Auto Scaling Group**

📸 **Screenshot Placeholder:** Spare launch templates and ASGs

---

### Step 9: Create Spare Region RDS

**Navigation:**

1. RDS Dashboard → **Databases** → **Create Database**

**Creation Steps:**

1. **Engine:** MariaDB
2. **Template:** Dev/Test
3. **DB instance identifier:** `spare-mariadb`
4. **Instance class:** `db.t3.micro`
5. **Storage:** 20 GB gp2
6. **VPC:** `spare-vpc`
7. **Subnet group:** Create new with `Spare-DB-Private` subnet
8. **Publicly accessible:** No
9. **Security group:** `Spare-DB-SG`
10. **Database name:** `globalbookdb`
11. Click **Create Database**

📸 **Screenshot Placeholder:** Spare RDS creation

---

### Step 10: Create Spare Region SNS and CloudWatch Alarms

**SNS Topic:**

1. SNS Dashboard → **Topics** → **Create Topic**
2. **Name:** `spare-alarms`
3. **Subscription:** Email to `dobiya4358@crsay.com`
4. Confirm email subscription

**CloudWatch Alarms:**

1. Create alarms for `spare-web-asg` and `spare-app-asg` (same as primary region)
2. **Alarm names:** `spare-web-asg-high-cpu`, `spare-app-asg-high-cpu`
3. **SNS topic:** `spare-alarms`

📸 **Screenshot Placeholder:** Spare SNS and alarms

---

## Route 53 & CloudFront Setup

**Important:** Route 53 is global - you can configure it from any region!

### Step 1: Create Route 53 Hosted Zone

**Navigation:**

1. Go to **Services** → Search for **Route 53** → Click **Route 53**
2. In the left sidebar, click **Hosted zones**
3. Click **Create hosted zone** button

**Creation Steps:**

1. **Domain name:** Enter your domain (e.g., `globalbook.com`)
   - **Note:** If you don't have a domain, you can use a test domain or skip this step
2. **Type:** Select **Public hosted zone**
3. Click **Create hosted zone** button
4. **Note:** You'll receive 4 name servers - save these if you own the domain

📸 **Screenshot Placeholder:** Hosted zone creation form

---

### Step 2: Create Route 53 Health Check

**Navigation:**

1. Route 53 Dashboard → **Health checks**
2. Click **Create health check** button

**Creation Steps:**

1. **Health check name:** Enter: `Primary-Region-HealthCheck`
2. **What to monitor:** Select **Endpoint**
3. **Specify endpoint by:** Select **Domain name**
4. **Domain name:** Enter your primary ALB DNS name
   - **How to find it:** EC2 → Load Balancers → Select `primary-alb` → Copy "DNS name"
   - Example: `primary-alb-572260929.us-east-1.elb.amazonaws.com`
5. **Protocol:** Select **HTTPS** (or HTTP if ALB uses HTTP)
6. **Port:** Enter `443` (or `80` for HTTP)
7. **Path:** Enter `/` (or `/health`)
8. **Advanced configuration:**
   - **Request interval:** 30 seconds
   - **Failure threshold:** 3
   - **Health check regions:** Select multiple regions (us-east-1, us-west-2, eu-west-1)
9. Click **Next** → **Create health check** button

📸 **Screenshot Placeholder:** Health check creation form

---

### Step 3: Create Route 53 Record Sets (Failover)

**Primary Record (Failover) - Creation Steps:**

1. **Navigation:**
   - Route 53 Dashboard → **Hosted zones**
   - Click on your hosted zone (e.g., `globalbook.com`)
   - Click **Create record** button
2. **Configure Record:**
   - **Record name:** Enter `www` (for www.globalbook.com) or leave blank (for root domain)
   - **Record type:** Select **A**
   - **Alias:** Toggle **Alias** to **Yes**
3. **Route traffic to:**
   - Select **Alias to Application and Classic Load Balancer**
   - **Region:** Select **us-east-1**
   - **Load balancer:** Select `primary-alb`
4. **Routing policy:**
   - Select **Failover**
5. **Failover record type:**
   - Select **Primary**
6. **Health check:**
   - Select `Primary-Region-HealthCheck` (the health check we created)
7. **Record ID:**
   - Enter: `Primary-Region-ALB`
8. Click **Create records** button

**Secondary Record (Failover) - Creation Steps:**

1. Click **Create record** button again
2. **Record name:** Enter `www` (same as primary)
3. **Record type:** Select **A**
4. **Alias:** Toggle **Alias** to **Yes**
5. **Route traffic to:**
   - Select **Alias to Network Load Balancer**
   - **Region:** Select **us-west-2**
   - **Load balancer:** Select `spare-nlb`
6. **Routing policy:**
   - Select **Failover**
7. **Failover record type:**
   - Select **Secondary**
8. **Health check:**
   - **Leave blank** (secondary doesn't need health check)
9. **Record ID:**
   - Enter: `Spare-Region-NLB`
10. Click **Create records** button

📸 **Screenshot Placeholder:** Route 53 record creation forms showing failover configuration

---

### Step 4: Create CloudFront Distribution

**Navigation:**

1. Go to **Services** → Search for **CloudFront** → Click **CloudFront**
2. Click **Create Distribution** button (top right)

**Creation Steps:**

1. **Origin Settings:**
   - Click **Create Origin** button
   - **Origin domain:** Enter your primary ALB DNS name
     - Example: `primary-alb-572260929.us-east-1.elb.amazonaws.com`
   - **Name:** `primary-alb-origin`
   - **Origin protocol policy:** Select **HTTPS Only** (or Match Viewer)
   - **HTTP port:** 80
   - **HTTPS port:** 443
   - Click **Add custom header** (optional)
   - Click **Add origin** button
2. **Add Second Origin:**
   - Click **Create Origin** again
   - **Origin domain:** Enter your spare NLB DNS name
     - Example: `spare-nlb-123456789.elb.us-west-2.amazonaws.com`
   - **Name:** `spare-nlb-origin`
   - **Origin protocol policy:** Select **HTTP Only**
   - Click **Add origin** button
3. **Create Origin Group:**
   - Click **Create Origin Group** button
   - **Origin group name:** `failover-group`
   - **Failover criteria:**
     - **Status codes:** Enter `500,502,503,504`
   - **Members:**
     - **Primary origin:** Select `primary-alb-origin`
     - **Secondary origin:** Select `spare-nlb-origin`
   - Click **Create origin group** button
4. **Default Cache Behavior:**
   - **Path pattern:** `*` (default)
   - **Origin and origin groups:** Select `failover-group`
   - **Viewer protocol policy:** Select **Redirect HTTP to HTTPS**
   - **Allowed HTTP methods:** Select **GET, HEAD, OPTIONS**
   - **Cache policy:** Select **CachingOptimized** (or create custom)
   - **Compress objects automatically:** Check **Yes**
5. **Distribution Settings:**
   - **Price class:** Select **Use only North America and Europe** (or your preference)
   - **Alternate domain names (CNAMEs):** Leave blank (or add your domain)
   - **SSL certificate:** Select **Default CloudFront certificate** (for testing)
   - **Default root object:** Leave blank
   - **Comment:** Enter: `GlobalBook CDN with failover`
6. Click **Create Distribution** button
7. Wait for status to change to "Deployed" (takes 10-15 minutes)

📸 **Screenshot Placeholder:** CloudFront distribution creation form showing origins, origin group, and cache behavior

---

## Verification Steps

### Verify Primary Region:

1. **Check ALB:** EC2 → Load Balancers → `primary-alb` should be "Active"
2. **Check Target Groups:** EC2 → Target Groups → Instances should be "Healthy"
3. **Check ASGs:** EC2 → Auto Scaling Groups → Should have desired number of instances
4. **Test ALB:** Copy ALB DNS name → Open in browser → Should see welcome page

### Verify Spare Region:

1. **Check NLB:** EC2 → Load Balancers → `spare-nlb` should be "Active"
2. **Check Target Groups:** Instances should be "Healthy"
3. **Test NLB:** Copy NLB DNS name → Open in browser → Should see welcome page

### Verify Route 53:

1. **Check Health Check:** Route 53 → Health Checks → Should show "Healthy"
2. **Test DNS:** Use `nslookup www.yourdomain.com` or `dig www.yourdomain.com`
3. **Verify Failover:** Stop primary region instances → Health check should fail → Route 53 should route to spare region

### Verify CloudFront:

1. **Check Distribution:** CloudFront → Distributions → Status should be "Deployed"
2. **Test CloudFront:** Copy CloudFront domain name → Open in browser → Should see welcome page
3. **Verify Failover:** Test origin failover by stopping primary region

---

## Troubleshooting

### Instances Show as Unhealthy:

- Check security groups allow traffic from ALB/NLB
- Verify Apache is running: `sudo systemctl status httpd`
- Test locally: `curl http://localhost/`
- Check health check path matches your application
- Increase health check grace period in ASG

### ALB/NLB Not Accessible:

- Verify security groups allow inbound traffic (80/443)
- Check route tables have correct routes (IGW for public, NAT for private)
- Verify instances are in correct subnets
- Check target group health status

### Route 53 Not Routing:

- Verify health check is passing
- Check record sets are configured correctly
- Ensure ALB/NLB DNS names are correct
- Wait for DNS propagation (can take a few minutes)

### CloudFront Not Working:

- Verify distribution is "Deployed" (not "In Progress")
- Check origin groups are configured correctly
- Verify origins are accessible
- Test CloudFront domain name directly
