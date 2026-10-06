# Question 2: Custom VPC with an Ubuntu EC2 Web Server

**AWS Project Assignment, Set 1 (AWS Only)**

## Problem Statement
Create a custom AWS VPC with one public subnet and deploy an Ubuntu EC2 instance. Configure an Internet Gateway, route table, and Security Group, install Apache or Nginx on the instance, and host a basic HTML webpage accessible through the public IP.

## Project Overview
| Item | Details |
|---|---|
| Region | Asia Pacific (Mumbai), `ap-south-1` |
| Web server | Nginx on Ubuntu 24.04 LTS |
| Instance | `Ubuntu-Web-Server`, t3.micro |
| Public IP | 13.206.86.123 |
| Result URL | http://13.206.86.123 |

## Components
| Component | Name / Value | Role |
|---|---|---|
| VPC | `my-web-vpc`, 10.0.0.0/16 | Isolated private network |
| Public subnet | `Public-Subnet`, 10.0.1.0/24 | Address range for the server, auto-assign public IPv4 enabled |
| Internet Gateway | `my-web-IGW` | Door between the VPC and the internet |
| Route table | `Public-Route-Table` | Sends 0.0.0.0/0 traffic to the gateway |
| Security group | `web-sg` | Firewall allowing SSH (22) and HTTP (80) |
| EC2 instance | `Ubuntu-Web-Server` | Ubuntu server that serves the HTML page |

## Architecture
```
Internet
   |
[ my-web-IGW ]
   |
[ Public-Route-Table ]  0.0.0.0/0 -> IGW
   |
+------------------ my-web-vpc (10.0.0.0/16) ------------------+
|   +------------- Public-Subnet (10.0.1.0/24) -------------+  |
|   |   [ web-sg: SSH 22 from My IP, HTTP 80 from anywhere ] |  |
|   |   Ubuntu-Web-Server (EC2, Nginx) -> public IP          |  |
|   +--------------------------------------------------------+  |
+--------------------------------------------------------------+
```

## Implementation Steps

### 1. Create the VPC
VPC service, **Create VPC**, choose **VPC only**. Name `my-web-vpc`, IPv4 CIDR `10.0.0.0/16`.

### 2. Create the public subnet
Subnets, **Create subnet** in `my-web-vpc`. Name `Public-Subnet`, CIDR `10.0.1.0/24`. Then **Actions, Edit subnet settings, Enable auto-assign public IPv4 address**.

### 3. Create and attach the Internet Gateway
Internet gateways, **Create** `my-web-IGW`, then **Actions, Attach to VPC**, and select `my-web-vpc`.

### 4. Create the route table
Route tables, **Create** `Public-Route-Table` in `my-web-vpc`. Add route **0.0.0.0/0** with target `my-web-IGW`. Under **Subnet associations**, associate `Public-Subnet`.

### 5. Create the security group
Security groups, **Create** `web-sg` in `my-web-vpc` with these inbound rules:

| Type | Port | Source |
|---|---|---|
| SSH | 22 | My IP |
| HTTP | 80 | 0.0.0.0/0 |

### 6. Launch the EC2 instance
Name `Ubuntu-Web-Server`, AMI **Ubuntu 24.04 LTS**, type **t3.micro**, new key pair (`web-key.pem`). In network settings choose `my-web-vpc`, `Public-Subnet`, auto-assign public IP **Enable**, and select the existing `web-sg`.

### 7. Connect and install Nginx
```bash
ssh -i "web-key.pem" ubuntu@13.206.86.123
sudo apt update
sudo apt install -y nginx
sudo systemctl enable --now nginx
systemctl status nginx
```
The status shows **active (running)**. The HTML page is then written to `/var/www/html/index.html`.

## Testing and Results
Open `http://13.206.86.123` in a browser. It shows **"Hello from my custom AWS VPC!"**

Request path:
**Browser, Internet Gateway, Route table, Public subnet, Security group (port 80), EC2, Nginx**

| Test | Expected result | Result |
|---|---|---|
| `systemctl status nginx` | active (running) | Passed |
| Open `http://<public-ip>` | Custom HTML page is displayed | Passed |

## Conclusion
A custom VPC with a public subnet, Internet Gateway, route table and Security Group was built, and an Ubuntu EC2 instance running Nginx serves the HTML page on its public IP.

## Cleanup
To avoid charges: terminate the instance, then delete the security group, Internet Gateway, subnet and VPC.
