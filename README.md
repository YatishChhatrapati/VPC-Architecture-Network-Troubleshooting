**VPC Architecture & Network Troubleshooting**

**Scenario 1 — Custom VPC Setup**

Task:
Built VPC with public and private subnets.

Components Created:

VPC
Public Subnet
Private Subnet
Internet Gateway
NAT Gateway
Route Tables

**Scenario 2 — Private Subnet Internet Access Issue**

Problem:
Instances in private subnet unable to access internet.

Investigation:

Checked Route Table → Missing NAT route
Verified NAT Gateway

Fix:
Added route:

0.0.0.0/0 → NAT Gateway

Result:
Outbound internet access restored.


**Scenario 3 — NACL Misconfiguration**

Problem:
Traffic blocked despite correct Security Group.

Investigation:

Checked NACL rules → Deny entry present

Fix:

Updated inbound/outbound rules
Allowed required ports

Root Cause:
NACL is stateless — both inbound & outbound must be allowed.

Key Learning:
Security Group = Stateful
NACL = Stateless
