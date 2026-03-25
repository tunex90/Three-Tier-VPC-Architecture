# Three-Tier-VPC-Architecture
This project implements a highly available, production‑grade VPC in AWS using a classic three‑tier architecture across two Availability Zones. It focuses entirely on networking, routing, security isolation, and connectivity validation — no application servers or databases were deployed.


🏗️ Architecture Overview
VPC
- CIDR: 10.0.0.0/16
- Region: us-east-1
- Availability Zones: us-east-1a and us-east-1b

🌐 Subnet Design
| Tier | Subnet Name     | CIDR          | AZ         | Public/Private |
|------|------------------|---------------|------------|----------------|
| Web  | Web-Public-A     | 10.0.1.0/24   | us-east-1a | Public         |
| Web  | Web-Public-B     | 10.0.2.0/24   | us-east-1b | Public         |
| App  | App-Private-A    | 10.0.11.0/24  | us-east-1a | Private        |
| App  | App-Private-B    | 10.0.12.0/24  | us-east-1b | Private        |
| DB   | DB-Private-A     | 10.0.21.0/24  | us-east-1a | Private        |
| DB   | DB-Private-B     | 10.0.22.0/24  | us-east-1b | Private        |

🌍 Internet & NAT Gateways
- 1 Internet Gateway (IGW)
- 2 NAT Gateways (one per AZ for high availability)

🛣️ Route Tables
- Public-RT → Routes 0.0.0.0/0 to IGW
- Private-RT → Routes 0.0.0.0/0 to NAT Gateways

🔐 Security Groups
Web-SG
- SSH (22) from anywhere
- HTTP (80) from anywhere
- ICMP from anywhere
App-SG
- SSH (22) from Web-SG
- ICMP from Web-SG
- TCP 8080 from Web-SG

DB-SG
- SSH (22) from App-SG
- ICMP from App-SG
- TCP 3306 from App-SG

🖥️ EC2 Instances
| Name      | Subnet         | Security Group | Public IP |
|-----------|----------------|----------------|-----------|
| Bastion   | Web-Public-A   | Web-SG         | Yes       |
| Web-VM    | Web-Public-B   | Web-SG         | Yes       |
| App-VM    | App-Private-A  | App-SG         | No        |
| DB-VM     | DB-Private-B   | DB-SG          | No        |


🔌 Connectivity Tests
1. Public Instances → Internet
Both Bastion and Web-VM successfully:
- ping 8.8.8.8
- curl ifconfig.me

2. Private Instances → Internet (via NAT)
App-VM and DB-VM successfully:
- Reached the internet through NAT Gateways
- Returned valid public IPs via curl ifconfig.me
3. Tier-to-Tier Connectivity
- Web → App: Success
- App → DB: Success
- Web → DB: Blocked (correct — DB-SG only allows App-SG)
4. Bastion Jump Host
- SSH from laptop → Bastion → App/DB
- Direct SSH to private IPs: Blocked (correct)

📸 Screenshots
All screenshots (VPC, subnets, route tables, EC2 list, ping tests, SSH tests) are included in the /screenshots folder.

📝 What I Learned
1. How real production VPCs are structured
This project helped me understand:
- Multi‑AZ design
- Public vs private subnets
- Tiered isolation
- Routing and NAT behavior
2. How NAT Gateways work
Private instances need:
- A private subnet
- A route table pointing to NAT
- No public IP

3. How to test and troubleshoot networking
I validated:
- Internet access
- Tier communication
- Security group restrictions
- Bastion-only SSH access
4. How to interpret expected failures
Some failures (like Web → DB ping or laptop → private SSH) are correct behavior, not errors.

📄 Short Report
What worked?
- All subnets, route tables, and gateways were configured correctly
- Public and private instances reached the internet
- Tier-to-tier communication matched the security group design
- Bastion host access worked exactly as intended
What was the hardest part?
Understanding why certain traffic was blocked:
- Web → DB ping failed (correct — SG rules)
- Laptop → private IP SSH timed out (correct — no public exposure)
This reinforced how routing and SGs interact.

Why separate App and DB subnets?
To enforce strict security isolation:
- App tier is private and only reachable from Web tier
- DB tier is even more restricted — only App tier can reach it
- Enables compliance, least privilege, and reduced blast radius
- Matches real enterprise architecture patterns





