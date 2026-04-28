# AWS VPC Architecture & Network Troubleshooting

Hands-on VPC build and network troubleshooting scenarios practised on AWS Free Tier
— part of my transition into Cloud Support Engineering.

---

## Environment

- Amazon VPC — Custom VPC with Public and Private Subnets
- Internet Gateway and NAT Gateway
- Route Tables
- Network ACLs (NACLs) and Security Groups
- Amazon EC2 — Ubuntu Server (Free Tier)
- AWS Management Console

---

## Architecture Built

```
Custom VPC (10.0.0.0/16)
│
├── Public Subnet (10.0.1.0/24)
│   ├── EC2 Instance (Bastion / Web Server)
│   └── Internet Gateway attached via Route Table
│
└── Private Subnet (10.0.2.0/24)
    ├── EC2 Instance (App / DB layer)
    └── NAT Gateway for outbound internet access
```

---

## Scenario 1 — Custom VPC Build with Public and Private Subnets

**Steps Performed:**

1. Created custom VPC — CIDR: 10.0.0.0/16
2. Created Public Subnet — CIDR: 10.0.1.0/24, AZ: ap-south-1a
3. Created Private Subnet — CIDR: 10.0.2.0/24, AZ: ap-south-1a
4. Created and attached Internet Gateway to VPC
5. Created Public Route Table — added route 0.0.0.0/0 → Internet Gateway
6. Associated Public Route Table with Public Subnet
7. Launched EC2 instance in Public Subnet — verified internet access
8. Created NAT Gateway in Public Subnet — allocated Elastic IP
9. Created Private Route Table — added route 0.0.0.0/0 → NAT Gateway
10. Associated Private Route Table with Private Subnet
11. Launched EC2 instance in Private Subnet
12. SSH into Private EC2 via Public EC2 (Bastion)
13. Verified outbound internet from Private instance

```bash
# From Private EC2 — outbound connectivity test
curl -I https://google.com
ping 8.8.8.8
```

**Result:**
End-to-end connectivity validated.
Private subnet instance could reach internet via NAT Gateway.
No inbound internet access to private instance — as expected.

**Key Learning:**
Internet Gateway enables two-way internet for public subnets.
NAT Gateway enables outbound-only internet for private subnets.
Route Tables control traffic flow — subnet association is critical.

---

## Scenario 2 — Subnet Routing Failure (Missing Route Table Entry)

**Problem:**
EC2 instance in public subnet could not reach the internet
after VPC reconfiguration.

**Investigation Steps:**
1. Confirmed EC2 instance was in Running state
2. Confirmed Internet Gateway was attached to VPC
3. Checked Public Route Table — route 0.0.0.0/0 was missing
4. Confirmed subnet was associated with correct Route Table
5. Added missing route: Destination 0.0.0.0/0 → Target Internet Gateway
6. Tested internet connectivity from EC2

```bash
curl -I https://google.com
```

**Resolution:**
Added missing route entry in Public Route Table.
Internet connectivity restored immediately.

**Root Cause:**
Route Table entry for 0.0.0.0/0 was accidentally deleted
during VPC reconfiguration — breaking outbound internet path.

**Key Learning:**
Always verify Route Table entries when subnet connectivity fails.
Internet Gateway alone is not enough — route must exist in Route Table.
Check subnet-to-Route Table association as well.

---

## Scenario 3 — NACL Misconfiguration Blocking Traffic

**Problem:**
EC2 instance was reachable via SSH but HTTP traffic was blocked
despite Security Group allowing port 80.

**Investigation Steps:**
1. Confirmed Security Group inbound rule for port 80 was present
2. Checked NACL inbound rules — port 80 was denied (DENY rule present)
3. Checked NACL outbound rules — ephemeral ports (1024–65535) were blocked
4. Updated NACL inbound: Allow port 80 from 0.0.0.0/0
5. Updated NACL outbound: Allow ephemeral ports 1024–65535 to 0.0.0.0/0
6. Tested HTTP access — website accessible

**Resolution:**
Corrected NACL inbound and outbound rules.
HTTP traffic restored successfully.

**Root Cause:**
NACL had explicit DENY rule for port 80 inbound.
Outbound ephemeral ports were also blocked — NACL is stateless
so both directions must be explicitly allowed.

**Key Learning:**
NACLs are stateless — both inbound and outbound rules required.
Security Groups are stateful — only inbound rule needed.
NACL rules are evaluated in order — lower rule number = higher priority.
Always check NACLs when Security Group is correctly configured
but traffic is still blocked.

---

## Stateful vs Stateless — Security Group vs NACL

| Feature | Security Group | NACL |
|---|---|---|
| Type | Stateful | Stateless |
| Applied at | Instance level | Subnet level |
| Inbound rule | Required | Required |
| Outbound rule | Auto-allowed | Must be explicit |
| Rule evaluation | All rules evaluated | Rules in order (lowest first) |
| Default behaviour | Deny all inbound | Allow all (default NACL) |

---

## Skills Demonstrated

| Skill | Scenario |
|---|---|
| Custom VPC build | 1 |
| Subnet design — public and private | 1 |
| Internet Gateway configuration | 1, 2 |
| NAT Gateway setup | 1 |
| Route Table configuration | 1, 2 |
| Routing failure diagnosis | 2 |
| NACL troubleshooting | 3 |
| Security Group vs NACL understanding | 3 |
| End-to-end connectivity validation | 1 |
| Root cause analysis (RCA) | 2, 3 |

---

## About

6+ years of Operations & Support experience at Accenture and Genpact
— transitioning into AWS Cloud Support Engineering.
All scenarios practised on AWS Free Tier.
