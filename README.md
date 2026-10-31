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

```text
                  Windows Laptop
                       |
                    PuTTY
                       |
                 Public IPv4
                       |
                Internet Gateway
                       |
        +-----------------------------+
        |           VPC A             |
        |       10.10.0.0/16          |
        |                             |
        |  Public Subnet               |
        |  10.10.1.0/24               |
        |                             |
        |  Bastion Host                |
        |  10.10.1.10                 |
        |          |                  |
        |          |                  |
        |  Private Subnet              |
        |  10.10.2.0/24               |
        |                             |
        |  App-A                      |
        |  10.10.2.10                 |
        +-------------+---------------+
                      |
                 VPC Peering
                      |
        +-------------+---------------+
        |           VPC B             |
        |       10.20.0.0/16          |
        |                             |
        |  Private Subnet              |
        |  10.20.1.0/24               |
        |                             |
        |  App-B                      |
        |  10.20.1.10                 |
        +-----------------------------+
```

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

The project architecture diagram was added here:

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

## Network Configuration

| Resource | Name | CIDR / Private IP |
|---|---|---|
| VPC A | ewb-vpc-a | 10.10.0.0/16 |
| Public Subnet A | ewb-a-public | 10.10.1.0/24 |
| Private Subnet A | ewb-a-private | 10.10.2.0/24 |
| VPC B | ewb-vpc-b | 10.20.0.0/16 |
| Private Subnet B | ewb-b-private | 10.20.1.0/24 |
| Bastion | ewb-bastion | 10.10.1.10 |
| App-A | ewb-app-a | 10.10.2.10 |
| App-B | ewb-app-b | 10.20.1.10 |

AWS Region: Mumbai (`ap-south-1`)

Operating System: Amazon Linux 2023

SSH Username: `ec2-user`

SSH Port: `22`

Key Pair: `ewb-lab`

## Implementation Steps

### 1. Create VPC A

Create a VPC with the following configuration:

- Name: `ewb-vpc-a`
- IPv4 CIDR: `10.10.0.0/16`

### 2. Create VPC A Subnets

Create two subnets:

Public Subnet:
- Name: `ewb-a-public`
- CIDR: `10.10.1.0/24`

Private Subnet:
- Name: `ewb-a-private`
- CIDR: `10.10.2.0/24`

### 3. Create Internet Gateway

- Name: `ewb-igw-a`
- Attach it to `ewb-vpc-a`.

### 4. Configure VPC A Route Tables

Public Route Table:
- Name: `ewb-rt-a-public`
- Associate with `ewb-a-public`.
- Add default route `0.0.0.0/0` pointing to the Internet Gateway.

Private Route Table:
- Name: `ewb-rt-a-private`
- Associate with `ewb-a-private`.
- Retain the local route initially.

### 5. Create VPC B

Create another VPC:

- Name: `ewb-vpc-b`
- IPv4 CIDR: `10.20.0.0/16`

Create its private subnet:

- Name: `ewb-b-private`
- CIDR: `10.20.1.0/24`

### 6. Configure VPC B Route Table

- Name: `ewb-rt-b-private`
- Associate with `ewb-b-private`.
- Retain the local route initially.

### 7. Configure Security Groups

#### Bastion Security Group

Name: `ewb-sg-bastion`

Inbound:
- SSH TCP 22 from the administrator's public IP address `/32`.

#### App-A Security Group

Name: `ewb-sg-app-a`

Inbound:
- SSH TCP 22 from `10.10.1.10/32`.
- ICMP Echo Request from `10.20.1.10/32`.

#### App-B Security Group

Name: `ewb-sg-app-b`

Inbound:
- SSH TCP 22 from `10.10.1.10/32`.
- ICMP Echo Request from `10.10.2.10/32`.

Keep default outbound allow-all rules for this lab. Use the appropriate network ACL rules if custom NACLs are configured.

### 8. Create EC2 Instances

Launch three Amazon Linux 2023 EC2 instances.

Bastion:
- Name: `ewb-bastion`
- VPC: `ewb-vpc-a`
- Subnet: `ewb-a-public`
- Private IP: `10.10.1.10`
- Public IPv4: Enabled
- Security Group: `ewb-sg-bastion`

App-A:
- Name: `ewb-app-a`
- VPC: `ewb-vpc-a`
- Subnet: `ewb-a-private`
- Private IP: `10.10.2.10`
- Public IPv4: Disabled
- Security Group: `ewb-sg-app-a`

App-B:
- Name: `ewb-app-b`
- VPC: `ewb-vpc-b`
- Subnet: `ewb-b-private`
- Private IP: `10.20.1.10`
- Public IPv4: Disabled
- Security Group: `ewb-sg-app-b`

Use the same EC2 key pair, `ewb-lab`, for all three instances.

## PuTTY and Pageant Configuration

### 1. Convert PEM to PPK

PuTTY uses the PPK private-key format.

1. Open PuTTYgen.
2. Click Load.
3. Select `ewb-lab.pem`.
4. Click Save private key.
5. Save as `ewb-lab.ppk`.

Keep the private key securely on your Windows laptop.

### 2. Load Key into Pageant

1. Open Pageant.
2. Right-click the Pageant system tray icon.
3. Select Add Key.
4. Choose `ewb-lab.ppk`.
5. Enter the passphrase if requested.

Keep Pageant running while connecting.

### 3. Configure PuTTY for Bastion

Open PuTTY.

Session:
- Host Name: `ec2-user@YOUR_BASTION_PUBLIC_IP`
- Port: `22`
- Connection Type: SSH

Authentication:
- Open Connection → SSH → Auth → Credentials.
- Select `ewb-lab.ppk` as the private key if using direct key authentication.
- Enable Allow agent forwarding.

Save the session as `ewb-bastion`.

Replace `YOUR_BASTION_PUBLIC_IP` with the actual public IPv4 address displayed in AWS.

### 4. Connect to Bastion

Open the saved PuTTY session.

After successful authentication, the terminal prompt should resemble:

```bash
[ec2-user@ip-10-10-1-10 ~]$
```

Verify the instance:

```bash
hostname
hostname -I
cat /etc/os-release
ip addr show
ip route
```

Expected private IP:

```text
10.10.1.10
```

### 5. Verify SSH Agent Forwarding

Run this command on Bastion:

```bash
ssh-add -L
```

It should display the public key loaded in Pageant.

If it displays `The agent has no identities`, check that Pageant has the key loaded and PuTTY agent forwarding is enabled. Reconnect to Bastion after changing the settings.

### 6. Connect to App-A from Bastion

Run:

```bash
ssh ec2-user@10.10.2.10
```

Verify:

```bash
hostname
hostname -I
ip route
```

Expected private IP:

```text
10.10.2.10
```

### 7. Return to Bastion

Run:

```bash
exit
```

### 8. Connect to App-B from Bastion

After VPC Peering is active and routing is configured, run:

```bash
ssh ec2-user@10.20.1.10
```

Verify:

```bash
hostname
hostname -I
ip route
```

Expected private IP:

```text
10.20.1.10
```

Important: The private key is forwarded from Pageant. It must not be copied to Bastion.

## VPC Peering Configuration

### 1. Create VPC Peering Connection

Navigate to:

AWS Console → VPC → Peering Connections → Create Peering Connection.

Configuration:

- Name: `ewb-a-b-peer`
- Requester VPC: `ewb-vpc-a`
- Accepter VPC: `ewb-vpc-b`
- Account: My account
- Region: Same Region

Create the connection and accept the request.

Wait until the peering status becomes `Active`.

### 2. Update Route Tables

VPC A Public Route Table (`ewb-rt-a-public`):

| Destination | Target |
|---|---|
| 10.10.0.0/16 | local |
| 0.0.0.0/0 | Internet Gateway |
| 10.20.0.0/16 | VPC Peering Connection |

VPC A Private Route Table (`ewb-rt-a-private`):

| Destination | Target |
|---|---|
| 10.10.0.0/16 | local |
| 10.20.0.0/16 | VPC Peering Connection |

VPC B Private Route Table (`ewb-rt-b-private`):

| Destination | Target |
|---|---|
| 10.20.0.0/16 | local |
| 10.10.0.0/16 | VPC Peering Connection |

Replace the peering target with your actual `pcx-...` connection ID.

VPC Peering does not automatically add routes. Both sides must have routes to each other's CIDR blocks.

## Connectivity Testing

### 1. Baseline Test Before Peering

From App-A, run:

```bash
ping -c 4 10.20.1.10
```

Expected: No replies before peering routes are configured.

Capture the baseline result.

### 2. Test App-A to App-B After Peering

Connect to App-A through Bastion:

```bash
ssh ec2-user@10.10.2.10
```

Run:

```bash
ping -c 4 10.20.1.10
```

Expected successful result:

```text
4 packets transmitted, 4 received, 0% packet loss
```

### 3. Test App-B to App-A

Return to Bastion:

```bash
exit
```

Connect to App-B:

```bash
ssh ec2-user@10.20.1.10
```

Run:

```bash
ping -c 4 10.10.2.10
```

Expected successful result:

```text
4 packets transmitted, 4 received, 0% packet loss
```

### 4. Controlled ICMP Failure Test

Temporarily remove the ICMP Echo Request inbound rule from `ewb-sg-app-b` that allows source `10.10.2.10/32`.

From App-A:

```bash
ping -c 4 10.20.1.10
```

Expected: No replies.

Restore the ICMP rule immediately and run the ping command again.

Expected: Successful replies.

This demonstrates that security groups control traffic even when VPC Peering is active.

## Useful Linux Commands

| Purpose | Command |
|---|---|
| Check hostname | `hostname` |
| Check private IP | `hostname -I` |
| Check OS | `cat /etc/os-release` |
| Check network interfaces | `ip addr show` |
| Check routing table | `ip route` |
| Check SSH service | `sudo systemctl status sshd` |
| Start SSH service | `sudo systemctl enable --now sshd` |
| Check Pageant forwarded key | `ssh-add -L` |
| Connect to App-A | `ssh ec2-user@10.10.2.10` |
| Connect to App-B | `ssh ec2-user@10.20.1.10` |
| Test App-A to App-B | `ping -c 4 10.20.1.10` |
| Test App-B to App-A | `ping -c 4 10.10.2.10` |
| Return to Bastion | `exit` |

## Troubleshooting

### Permission Denied (Public Key)

Possible causes:
- Incorrect key pair.
- Pageant does not have the correct key loaded.
- PuTTY agent forwarding is disabled.
- Wrong SSH username.
- App-A or App-B was launched with a different key pair.

Useful command on Bastion:

```bash
ssh-add -L
```

### Connection Timed Out

Check:
- EC2 instance is running.
- Correct public IP is used for Bastion.
- Bastion security group permits SSH TCP 22 from your current public IP.
- Public route table has `0.0.0.0/0` pointing to the Internet Gateway.
- The Internet Gateway is attached to VPC A.

### Private Instance SSH Failure

Check:
- App-A/App-B security group permits TCP 22 from `10.10.1.10/32`.
- The instance uses the correct key pair.
- The private IP address is correct.
- VPC Peering is active for App-B access.
- The required routes are present.

### Ping Failure

Check:
- VPC Peering status is Active.
- Both VPC route tables contain the correct peering routes.
- Security groups allow ICMP Echo Request.
- Network ACLs allow request and return traffic.
- Destination instance is running.

## Security Best Practices

- Use a Bastion Host as the controlled SSH entry point.
- Keep private application instances without public IP addresses.
- Restrict SSH access to known source IP addresses.
- Use SSH agent forwarding instead of copying private keys to Bastion.
- Never upload PEM or PPK private keys to GitHub.
- Use IAM permissions according to least privilege.
- Delete unused AWS resources after the lab to avoid unnecessary charges.

## Project Outcomes

After completing this project, the following concepts are demonstrated:

- AWS VPC and subnet creation.
- Public and private subnet architecture.
- Bastion Host configuration.
- SSH connectivity using PuTTY and Pageant.
- VPC Peering setup.
- Route table configuration.
- Security group traffic control.
- Private EC2-to-EC2 communication.
- Linux network troubleshooting.
- ICMP connectivity testing.

## Screenshots

Added actual AWS and PuTTY screenshots to the `screenshots/` folder

**AWS & DevOps Project**

Technologies: AWS, Amazon EC2, Amazon VPC, VPC Peering, Linux, SSH, PuTTY, Pageant.

## 🚀 Conclusion

Successfully designing and implementing this AWS VPC Peering architecture demonstrates how separate AWS virtual networks can communicate securely through private IP addresses.

The project highlights practical knowledge of AWS networking, network isolation, secure remote access, routing and cloud infrastructure management.

## 👨‍💻 Author

**Trishalaa B**

AWS & DevOps Engineer | Cloud Computing Enthusiast

GitHub: https://github.com/Tris142

Portfolio: https://tris142.github.io/portfolio

---
