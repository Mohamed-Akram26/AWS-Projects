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
