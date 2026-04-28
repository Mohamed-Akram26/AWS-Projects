## 🔧 What Was Configured in Each Component & Why

---

### VPC — `akram-vpc`
- **CIDR `10.0.0.0/16`**
- **DNS hostnames + DNS support enabled** — so EC2 instances can talk to each other using DNS names

![VPC Created](Artifacts/01.-VPC-Created-with-DNS-Hostnames-Enabled.jpg.png)

---

### Subnets
- **4 subnets across 2 AZs** — for high availability
- **Public subnets (`10.0.1.0/24`, `10.0.2.0/24`)** — for resources that need internet access (Jumphost, NAT GW, ALB)
- **Private subnets (`10.0.3.0/24`, `10.0.4.0/24`)** — for app servers, no direct internet exposure

![Subnets Created](Artifacts/02.-Subnets-Created-4-Subnets-with-Auto-Assign-Public-IP.jpg.png)

---

### Internet Gateway — `akram-igw`
- **Attached to `akram-vpc`** — enables the VPC to communicate with the internet

![Internet Gateway](Artifacts/03.-Internet-Gateway-Created-and-Attached-to-VPC.jpg.png)

---

### Public Route Table — `akram-public-rt`
- **`0.0.0.0/0` → `akram-igw`** — any traffic going outside VPC routes through the Internet Gateway
- **`10.0.0.0/16` → local** — internal VPC traffic stays within VPC
- **Associated to both public subnets** — so Jumphost and NAT GW get internet access

![Public Route Table](Artifacts/05.-Public-Route-Table-Created-with-IGW-Route-and-Associated-to-Public-Subnets.jpg)

---

### Private Route Table — `akram-private-rt`
- **`0.0.0.0/0` → `akram-nat-gw-1`** — outbound internet traffic from private servers routes through NAT Gateway
- **`10.0.0.0/16` → local** — internal traffic stays within VPC
- **Associated to both private subnets** — private EC2s can reach internet (for updates etc.) but internet can't reach them directly

![Private Route Table](Artifacts/04.-Private-Route-Table-Created-and-Associated-to-Private-Subnets.jpg.png)

![Private Route Table Updated](Artifacts/08.-Private-Route-Table-Updated-with-NAT-Gateway-Route.jpg.png)

---

### NAT Gateways — `akram-nat-gw-1` & `akram-nat-gw-2`
- **Placed in public subnets** — needs internet access itself to forward traffic
- **`nat-gw-1` in public-subnet-1, `nat-gw-2` in public-subnet-2** — one per AZ for HA

![NAT Gateway 1](Artifacts/06.-NAT-Gateway-1-Available-in-Public-Subnet-1.jpg)

![NAT Gateway 2](Artifacts/07.-NAT-Gateway-2-Available-in-Public-Subnet-2.jpg.png)

---

### Security Group — `akram-alb-sg`
- **Inbound: HTTP port 80 from `0.0.0.0/0`** — allows anyone on internet to hit the ALB
- ALB is the only entry point to the application from outside

![ALB SG](Artifacts/11.-ALB-SG-Inbound-Rules-HTTP-Port-80.jpg.png)

---

### Security Group — `akram-ec2-sg`
- **Inbound: port 8000 from `akram-alb-sg`** — only ALB can send traffic to EC2 servers, nobody else
- **Inbound: port 22 from `akram-jumphost-sg`** — only Jumphost can SSH into private servers
- Follows **least privilege principle** — no unnecessary open ports

![EC2 SG](Artifacts/10.-EC2-SG-Inbound-Rules-Port-8000-from-ALB-SG-and-SSH-from-Jumphost-SG.jpg.png)

---

### Security Group — `akram-jumphost-sg`
- **Inbound: port 22 from My IP** — only your machine can SSH into the Jumphost
- Acts as the single controlled entry point for admin access

![Jumphost SG](Artifacts/09.-Jumphost-SG-Inbound-Rules-SSH-Port-22.jpg.png)

---

### Key Pair — `akram-key`
- **RSA, `.pem` format** — used to SSH into all EC2 instances
- Same key used for Jumphost and private servers

![Key Pair](Artifacts/12.-Key-Pair-Created-akram-key.jpg.png)

---

### EC2 — `akram-jumphost`
- **In `akram-public-subnet-1` with public IP** — accessible from internet via SSH
- **Purpose:** acts as a gateway to SSH into private servers that have no public IP
- Assigned with `akram-jumphost-sg`

![Jumphost EC2](Artifacts/13.-EC2-Jumphost-Running-in-Public-Subnet-1.jpg.png)

![SCP Key to Jumphost](Artifacts/16.-SCP-Key-Transferred-to-Jumphost.jpg)

---

### EC2 — `akram-server-1` & `akram-server-2`
- **In private subnets, no public IP** — not directly accessible from internet
- **Python HTTP server running on port 8000** — serving custom HTML identifying each server
- Assigned `akram-ec2-sg`

![Server 1](Artifacts/14.-EC2-Server-1-Running-in-Private-Subnet-1.jpg.png)

![Server 2](Artifacts/15.EC2-Server-2-Running-in-Private-Subnet-2.jpg.png)

![SSH to Server 1](Artifacts/17.-SSH-From-Jumphost-to-Private-Server-1.jpg)

![Python Server 1](Artifacts/18.-Python-Web-Server-Running-on-Server-1-Port-8000.jpg)

![SSH to Server 2](Artifacts/19.-SSH-From-Jumphost-to-Private-Server-2.jpg)

![Python Server 2](Artifacts/20.-Python-Web-Server-Running-on-Server-2-Port-8000.jpg)

---

### Target Group — `akram-tg`
- **Protocol HTTP, port 8000** — matches the port Python server is running on
- **Health check on `/`** — ALB pings this path to confirm server is alive before sending traffic
- **Both servers registered** — ALB uses this list to know where to forward requests

![Target Group Healthy](Artifacts/21.-Target-Group-akram-tg-Instances-Healthy.jpg)

---

### Application Load Balancer — `akram-alb`
- **Internet-facing** — has a public DNS, accessible from browser/curl
- **Spans both public subnets** — sits in AZ1 and AZ2 for HA
- **Listener on port 80** — forwards incoming HTTP traffic to `akram-tg`
- **Assigned `akram-alb-sg`** — only allows HTTP port 80 inbound
- **Round-robin by default** — distributes each request alternately between server-1 and server-2

![ALB Active](Artifacts/22.-ALB-akram-alb-Active-with-DNS-Configured.jpg)

![Load Balancing Verified](Artifacts/23.-ALB-Load-Balancing-Verified-via-Curl-Requests.jpg)

![Server 1 Response](Artifacts/24.-ALB-DNS-Response-Server-1.jpg)

![Server 2 Response](Artifacts/25.-ALB-DNS-Response-Server-2.jpg)

---

### Launch Template — `akram-lt`
- **AMI: Ubuntu 24.04, type: t2.micro, key: `akram-key`, SG: `akram-ec2-sg`** — defines the blueprint for ASG to launch instances
- **User data script** — automatically installs Python, creates `index.html` with server's hostname/IP, starts Python HTTP server on port 8000 on every new instance launch
- Any new instance ASG creates is **production-ready automatically** without manual SSH

![Launch Template](Artifacts/26.-Launch-Template-akram-lt-Created-for-ASG.jpg)

---

### Auto Scaling Group — `akram-asg`
- **Uses `akram-lt`** — so every instance it creates follows the same config
- **Subnets: both private subnets** — launches instances across 2 AZs for HA
- **Desired: 2, Min: 1, Max: 3** — maintains 2 running instances, can scale up to 3 under load, never goes below 1
- **Attached to `akram-tg`** — new instances are automatically registered to the target group
- **Health check type: ELB** — if ALB marks an instance unhealthy, ASG automatically terminates and replaces it

![ASG Created](Artifacts/27.-Auto-Scaling-Group-akram-asg-Created-attached-with-target-group.jpg)

![ASG Instances Healthy](Artifacts/28.-ASG-Instances-InService-and-Healthy.jpg)

![EC2 by ASG](Artifacts/29.-EC2-Instances-Launched-by-ASG.jpg)

![ASG Load Balancing](Artifacts/30.-ALB-ASG-Load-Balancing-Verified-via-Curl.jpg)

![ASG Instance 1 Response](Artifacts/31.-ALB-DNS-Response-ASG-Instance-1.jpg)

![ASG Instance 2 Response](Artifacts/32.-ALB-DNS-Response-ASG-Instance-2.jpg)
