# AWS VPC 2-Tier Architecture with ALB & Auto Scaling

## 🚀 Overview

This project demonstrates a **production-style web application infrastructure on AWS** where the application is secure, highly available, and auto-scalable.

---

## 🏗️ Architecture Summary

- Application servers are deployed in **private subnets** with **no public IPs**
- Direct internet access to servers is **blocked**
- All user traffic flows through an **Application Load Balancer (ALB)**

---

## 🌐 Traffic Flow

- Users send HTTP requests to the **ALB (Port 80)**
- ALB distributes traffic across servers using **round-robin**
- Servers are deployed across **multiple Availability Zones (AZs)**

---

## 🔒 Security Design

- Private EC2 instances are not publicly accessible
- **Jumphost (Bastion Host)** is the only entry point for SSH

---

## 🌍 Internet Access for Private Servers

- Private servers do not have direct internet access
- **NAT Gateways** are used for outbound traffic

---

## ⚙️ High Availability & Auto Scaling

- **Auto Scaling Group (ASG)** ensures availability
- Automatically replaces unhealthy instances
- Uses **Launch Template** with pre-configured setup

---

## 🔁 End-to-End Flow
- User → ALB → Private EC2 (via Target Group)
# ↓
- ASG maintains instances
  ↓
- NAT GW provides outbound access
  ↓
- Jumphost provides admin access

---

## ✅ Key Features

- Multi-AZ deployment
- Private subnet isolation
- Load balancing (ALB)
- Auto healing (ASG)
- Secure SSH access (Jumphost)
- Controlled internet access (NAT Gateway)
