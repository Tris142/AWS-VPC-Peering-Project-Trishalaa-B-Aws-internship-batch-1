# AWS-VPC-Peering-Project
# AWS VPC Peering – Secure Private Network Connectivity

## 📌 Project Overview

This project demonstrates how to establish secure private network communication between two Amazon Virtual Private Clouds (VPCs) using **AWS VPC Peering**.

Two VPCs are configured with separate CIDR ranges, private and public subnets, EC2 instances, route tables, security groups and an Internet Gateway.

A Bastion Host is deployed in the public subnet to securely access private EC2 instances through SSH. VPC Peering enables communication between application servers in different VPCs using their private IP addresses without exposing them to the public internet.

## 🎯 Project Objectives

- Create two independent AWS VPCs.
- Configure public and private subnets.
- Deploy EC2 instances in different subnets.
- Configure an Internet Gateway for Bastion Host access.
- Establish a VPC Peering connection.
- Configure route tables for bidirectional communication.
- Implement secure access using Security Groups.
- Access private EC2 instances through a Bastion Host.
- Test connectivity using SSH and ICMP (Ping).

## 🏗️ Architecture

### Network Architecture

**VPC A**
- Public Subnet – Bastion Host EC2
- Private Subnet – Application EC2
- Internet Gateway
- Public and Private Route Tables

**VPC B**
- Private Subnet – Application EC2
- Private Route Table
- No Internet Gateway

**VPC Peering**
- Connects VPC A and VPC B.
- Enables private IP communication.
- Supports bidirectional communication through appropriate routes.

### Architecture Diagram

Add your project architecture diagram here:

`architecture/vpc-peering-architecture.png`

## 🛠️ AWS Services Used

| AWS Service | Purpose |
|---|---|
| Amazon VPC | Create isolated virtual networks |
| EC2 | Host Bastion and application servers |
| VPC Peering | Connect two VPCs privately |
| Internet Gateway | Provide internet access to Bastion |
| Route Tables | Control network traffic routing |
| Subnets | Separate public and private resources |
| Security Groups | Control inbound and outbound traffic |
| Key Pair | Secure SSH authentication |

## 🌐 IP Address Planning

### VPC Configuration

| Resource | CIDR Block |
|---|---|
| VPC A | 10.10.0.0/16 |
| VPC B | 10.20.0.0/16 |

### Subnet Configuration

| Subnet | VPC | CIDR |
|---|---|---|
| Public Subnet A | VPC A | 10.10.1.0/24 |
| Private Subnet A | VPC A | 10.10.2.0/24 |
| Private Subnet B | VPC B | 10.20.1.0/24 |

### EC2 Instance Configuration

| Instance | Subnet | Private IP |
|---|---|---|
| Bastion EC2 | Public Subnet A | 10.10.1.10 |
| App-A EC2 | Private Subnet A | 10.10.2.10 |
| App-B EC2 | Private Subnet B | 10.20.1.10 |

**Note:** Only the Bastion Host receives a public IPv4 address. Both application servers use private IP addresses.

## ⚙️ Implementation Steps

### Step 1: Create VPC A

1. Open AWS Management Console.
2. Navigate to VPC Dashboard.
3. Select Create VPC.
4. Choose VPC only.
5. Configure:
   - Name: `ewb-vpc-a`
   - IPv4 CIDR: `10.10.0.0/16`
6. Create the VPC.

### Step 2: Create Subnets in VPC A

Create two subnets:

**Public Subnet**
- Name: `ewb-a-public`
- CIDR: `10.10.1.0/24`

**Private Subnet**
- Name: `ewb-a-private`
- CIDR: `10.10.2.0/24`

### Step 3: Configure Internet Gateway

1. Navigate to Internet Gateways.
2. Create an Internet Gateway named `ewb-igw-a`.
3. Attach it to `ewb-vpc-a`.
4. Enable auto-assign public IPv4 for the public subnet, or ensure the Bastion launch configuration assigns a public IP.

### Step 4: Configure Route Tables for VPC A

**Public Route Table: `ewb-rt-a-public`**

Associate with `ewb-a-public`.

| Destination | Target |
|---|---|
| 10.10.0.0/16 | local |
| 0.0.0.0/0 | Internet Gateway |

**Private Route Table: `ewb-rt-a-private`**

Associate with `ewb-a-private`.

| Destination | Target |
|---|---|
| 10.10.0.0/16 | local |

### Step 5: Create VPC B

Create another VPC:

- Name: `ewb-vpc-b`
- IPv4 CIDR: `10.20.0.0/16`

Create a private subnet:

- Name: `ewb-b-private`
- CIDR: `10.20.1.0/24`

Create a route table named `ewb-rt-b-private` and associate it with the private subnet.

Initial route:

| Destination | Target |
|---|---|
| 10.20.0.0/16 | local |

### Step 6: Configure Security Groups

Create three Security Groups.

**Bastion Security Group – `ewb-sg-bastion`**

Inbound:
- SSH – TCP 22 – My IP (/32)

**App-A Security Group – `ewb-sg-app-a`**

Inbound:
- SSH – TCP 22 – `10.10.1.10/32`
- ICMP Echo Request – `10.20.1.10/32`

**App-B Security Group – `ewb-sg-app-b`**

Inbound:
- SSH – TCP 22 – `10.10.1.10/32`
- ICMP Echo Request – `10.10.2.10/32`

Keep the default outbound allow-all rules for this lab.

### Step 7: Create EC2 Instances

Launch three EC2 instances using Amazon Linux 2023.

**Bastion Host**
- Instance: `ewb-bastion`
- VPC: VPC A
- Subnet: Public Subnet A
- Private IP: `10.10.1.10`
- Public IP: Enabled
- Security Group: `ewb-sg-bastion`

**Application Server A**
- Instance: `ewb-app-a`
- VPC: VPC A
- Subnet: Private Subnet A
- Private IP: `10.10.2.10`
- Public IP: Disabled
- Security Group: `ewb-sg-app-a`

**Application Server B**
- Instance: `ewb-app-b`
- VPC: VPC B
- Subnet: Private Subnet B
- Private IP: `10.20.1.10`
- Public IP: Disabled
- Security Group: `ewb-sg-app-b`

Use the same EC2 key pair for all three instances.

### Step 8: Connect to Bastion Host

Set the private key permissions:

```bash
chmod 400 ewb-lab.pem
```

Connect to the Bastion Host:

```bash
ssh -i ewb-lab.pem ec2-user@<BASTION_PUBLIC_IP>
```

### Step 9: Configure SSH ProxyJump

Configure SSH on your local machine using `~/.ssh/config`.

```sshconfig
Host ewb-bastion
    HostName <BASTION_PUBLIC_IP>
    User ec2-user
    IdentityFile ~/.ssh/ewb-lab.pem
    IdentitiesOnly yes

Host ewb-app-a
    HostName 10.10.2.10
    User ec2-user
    IdentityFile ~/.ssh/ewb-lab.pem
    IdentitiesOnly yes
    ProxyJump ewb-bastion

Host ewb-app-b
    HostName 10.20.1.10
    User ec2-user
    IdentityFile ~/.ssh/ewb-lab.pem
    IdentitiesOnly yes
    ProxyJump ewb-bastion
```

Set permissions:

```bash
chmod 600 ~/.ssh/config
```

Connect to Application Server A:

```bash
ssh ewb-app-a
```

Connect to Application Server B:

```bash
ssh ewb-app-b
```

Before configuring VPC Peering, cross-VPC connectivity should not work.

### Step 10: Create VPC Peering Connection

1. Navigate to VPC Dashboard.
2. Select Peering Connections.
3. Click Create Peering Connection.
4. Configure:

Requester VPC: `ewb-vpc-a`

Accepter VPC: `ewb-vpc-b`

Name: `ewb-a-b-peer`

5. Create the peering connection.
6. Select the connection.
7. Choose Actions → Accept Request.
8. Verify that the status becomes Active.

### Step 11: Configure Route Tables for VPC Peering

Update all three route tables.

**VPC A Private Route Table**

`ewb-rt-a-private`

| Destination | Target |
|---|---|
| 10.10.0.0/16 | local |
| 10.20.0.0/16 | VPC Peering Connection |

**VPC B Private Route Table**

`ewb-rt-b-private`

| Destination | Target |
|---|---|
| 10.20.0.0/16 | local |
| 10.10.0.0/16 | VPC Peering Connection |

**VPC A Public Route Table**

`ewb-rt-a-public`

| Destination | Target |
|---|---|
| 10.10.0.0/16 | local |
| 0.0.0.0/0 | Internet Gateway |
| 10.20.0.0/16 | VPC Peering Connection |

These routes enable communication between both VPCs and allow the Bastion Host to reach the private EC2 instance in VPC B.

### Step 12: Test VPC Peering Connectivity

Connect to App-A:

```bash
ssh ewb-app-a
```

Check the instance:

```bash
hostname
hostname -I
```

Test connectivity to App-B:

```bash
ping -c 4 10.20.1.10
```

Expected result:

```text
4 packets transmitted, 4 received, 0% packet loss
```

Test reverse communication from App-B:

```bash
ssh ewb-app-b
ping -c 4 10.10.2.10
```

Expected result:

```text
4 packets transmitted, 4 received, 0% packet loss
```

The successful ping tests demonstrate private network communication between two different VPCs.

## 🔍 Validation and Testing

| Test Case | Expected Result |
|---|---|
| SSH to Bastion | Successful |
| SSH from Bastion to App-A | Successful |
| SSH from Bastion to App-B | Successful |
| Ping App-A to App-B before peering | Failed |
| Ping App-A to App-B after peering | Successful |
| Ping App-B to App-A | Successful |
| Private EC2 public IP assignment | Disabled |
| Peering connection status | Active |

## 🛡️ Security Implementation

- Restricted SSH access to the Bastion Host using the administrator's public IP.
- Restricted private EC2 SSH access to the Bastion Host's private IP.
- Used private IP addresses for application servers.
- Configured security groups with minimum required inbound access.
- Used SSH key-based authentication.
- Avoided direct public internet exposure of private EC2 instances.

## ⚠️ Troubleshooting

| Issue | Solution |
|---|---|
| SSH connection timeout | Verify security groups and public subnet routing |
| VPC Peering not working | Verify peering status is Active |
| Ping request timeout | Check route tables and ICMP inbound rules |
| Private EC2 inaccessible | Verify Bastion connectivity and ProxyJump configuration |
| Return traffic failure | Add the reverse VPC Peering route |
| SSH permission denied | Check PEM key, permissions and username |

Useful Linux commands:

```bash
hostname
hostname -I
ip route
ping -c 4 <PRIVATE_IP>
```

## 📸 Project Screenshots

Add your actual AWS console and terminal screenshots in the `screenshots/` directory.

Suggested screenshots:

1. VPC A configuration
2. VPC B configuration
3. Public and private subnet configuration
4. Internet Gateway attachment
5. Route table configuration
6. Security Group configuration
7. EC2 instances
8. Active VPC Peering connection
9. VPC A and VPC B peering routes
10. SSH connection to private EC2
11. Successful bidirectional ping test

## 📚 Key Learnings

Through this project, I gained practical knowledge of:

- AWS VPC architecture and CIDR planning.
- Public and private subnet configuration.
- Internet Gateway implementation.
- Route table configuration and traffic forwarding.
- VPC Peering and private network connectivity.
- EC2 instance deployment.
- Bastion Host and SSH ProxyJump.
- Security Group configuration.
- Network troubleshooting and connectivity testing.

## 🚀 Conclusion

Successfully designing and implementing this AWS VPC Peering architecture demonstrates how separate AWS virtual networks can communicate securely through private IP addresses.

The project highlights practical knowledge of AWS networking, network isolation, secure remote access, routing and cloud infrastructure management.

## 👨‍💻 Author

**Your Name**

AWS & DevOps Engineer | Cloud Computing Enthusiast

GitHub: https://github.com/Tris142

Portfolio: https://tris142.github.io/portfolio

---
