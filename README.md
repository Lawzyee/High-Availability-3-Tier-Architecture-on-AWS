# High Availability 3-Tier Architecture on AWS

This repository contains the design and configuration steps for setting up a highly available 3-tier architecture on AWS. The architecture includes a web server, application server, and database server, all configured to ensure high availability and security.

## Architecture Overview

The architecture is designed to be highly available by utilizing multiple availability zones. The key components include:

- **VPC**: A Virtual Private Cloud with 4 subnets (1 public and 3 private).
- **Public Subnet**: Hosts the Bastion Host and Web Server.
- **Private Subnets**: Host the Application Server and Database Server.
- **Security Groups**: Configured to control inbound and outbound traffic for each component.
- **Route Tables**: Configured to route traffic between subnets and to the internet.
- **NAT Gateway**: Allows instances in private subnets to connect to the internet.
- **Internet Gateway**: Enables communication between the VPC and the internet.

## Components

### VPC and Subnets
- **VPC**: Created with a CIDR block of 192.168.0.0/16.
- **Subnets**: 
  - Public Subnet: 192.168.1.0/24
  - Private Subnet 1: 192.168.2.0/24
  - Private Subnet 2: 192.168.3.0/24
  - Private Subnet 3: 192.168.4.0/24 (in a different availability zone)

### Security Groups
- **Bastion Host**: Allows SSH access from your IP and HTTP/HTTPS traffic.
- **Web Server**: Allows HTTP/HTTPS traffic and SSH access from the Bastion Host.
- **App Server**: Allows ICMP traffic from the Web Server and SSH access from the Bastion Host.
- **Database Server**: Allows MySQL/Aurora traffic from the App Server and Bastion Host.

### Instances
- **Bastion Host**: Amazon Linux 2 AMI, t2.micro, in the public subnet.
- **Web Server**: Amazon Linux 2 AMI, t2.micro, in the public subnet with Apache installed.
- **App Server**: Amazon Linux 2 AMI, t2.micro, in the private subnet with MariaDB installed.
- **Database Server**: MariaDB instance in the private subnet.

## Configuration Steps

### Step 1: Create VPC and Subnets
1. Go to "Your VPCs" from the VPC service on the AWS management console and click on the orange "Create VPC" button.
2. Create a VPC with a name (e.g. Lawtechio VPC) and a CIDR block of 192.168.0.0/16. (leave everything else as default. Click create.)
3. Create 4 subnets:
   - Public Subnet:    192.168.1.0/24 in any availability zone.
   - Private Subnet 1: 192.168.2.0/24 in the same availability zone as the public subnet.
   - Private Subnet 2: 192.168.3.0/24 in the same availability zone as the public subnet.
   - Private Subnet 3: 192.168.4.0/24 in a different availability zone.

### Step 2: Set Up Routing
1. Allocate an Elastic IP address.
2. Create an Internet Gateway and attach it to the VPC.
3. Create a NAT Gateway in the public subnet and associate it with the Elastic IP.
4. Create route tables for public and private subnets:
   - Public Route Table: Add a route to the Internet Gateway.
   - Private Route Table: Add a route to the NAT Gateway.
5. Associate the public subnet with the public route table and the private subnets with the private route table.

### Step 3: Configure Security Groups
1. Create security groups for the Bastion Host, Web Server, App Server, and Database Server.
2. Configure inbound rules:
   - Bastion Host: SSH from your IP, HTTP/HTTPS from anywhere.
   - Web Server: SSH from Bastion Host, HTTP/HTTPS from anywhere.
   - App Server: ICMP from Web Server, SSH from Bastion Host, MySQL/Aurora from Database Server.
   - Database Server: MySQL/Aurora from App Server and Bastion Host.

### Step 4: Launch Instances
1. **Bastion Host**:
   - AMI: Amazon Linux 2
   - Instance Type: t2.micro
   - Subnet: Public Subnet
   - Security Group: Bastion Host SG
   - Auto-assign Public IP: Enabled
2. **Web Server**:
   - AMI: Amazon Linux 2
   - Instance Type: t2.micro
   - Subnet: Public Subnet
   - Security Group: Web Server SG
   - User Data:
     ```bash
     #!/bin/bash
     sudo yum update -y
     sudo amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
     sudo yum install -y httpd
     sudo systemctl start httpd
     sudo systemctl enable httpd
     ```
3. **App Server**:
   - AMI: Amazon Linux 2
   - Instance Type: t2.micro
   - Subnet: Private Subnet 1
   - Security Group: App Server SG
   - User Data:
     ```bash
     #!/bin/bash
     sudo yum install -y mariadb-server
     sudo service mariadb start
     ```

### Step 5: Create Database
1. Create a DB subnet group with Private Subnet 3 and Private Subnet 4.
2. Create a MariaDB instance:
   - Engine: MariaDB
   - Instance Class: db.t2.micro (Free Tier)
   - Master Username: Lawtehio
   - Master Password: Lawproject
   - VPC: Your VPC
   - Subnet Group: Your DB Subnet Group
   - Security Group: Database Server SG
   - Initial Database Name: mydb

### Step 6: Test Connections
1. SSH into the Bastion Host using the provided key pair.
2. Upload the SSH key to the Bastion Host.
3. SSH into the App Server from the Bastion Host.
4. Test connectivity:
   - Ping the Web Server from the App Server.
   - Connect to the Database Server from the App Server using MySQL.

## Diagram

The architecture diagram is available in the file [ArchitecturalDesign.png](ArchitecturalDesign.png).

## Conclusion

This setup provides a robust and highly available 3-tier architecture on AWS, suitable for hosting web applications with a secure and scalable backend.