# PythonBuddy ☁️🐍

**Cloud-Deployed Python Linter & Code Execution Platform**

[![AWS](https://img.shields.io/badge/AWS-Cloud-orange?logo=amazon-aws)](https://aws.amazon.com)
[![Docker](https://img.shields.io/badge/Docker-Containerized-blue?logo=docker)](https://www.docker.com)
[![React](https://img.shields.io/badge/React-Frontend-61DAFB?logo=react)](https://reactjs.org)
[![Flask](https://img.shields.io/badge/Flask-Backend-000000?logo=flask)](https://flask.palletsprojects.com)

---

## Overview

**PythonBuddy** is an online Python 3 linter and code execution environment featuring:

- ✅ Live Pylint syntax checking
- ✅ Python code execution in the browser
- ✅ Real-time error feedback and explanations

**This semester project** extended the original PythonBuddy by Ethan Chiu with **production-grade AWS cloud architecture**, emphasizing high availability, fault tolerance, and automated recovery.

---

## Cloud Architecture

Deployed using a **highly available, self-healing AWS infrastructure**:

| Component         | Technology                   | Purpose                       |
| ----------------- | ---------------------------- | ----------------------------- |
| **Frontend**      | React + S3                   | Static website hosting        |
| **Backend**       | Flask + Docker + EC2         | API and code execution        |
| **Load Balancer** | Application Load Balancer    | Traffic distribution          |
| **Scaling**       | Auto Scaling Group           | Automatic instance management |
| **Network**       | VPC (Public/Private Subnets) | Secure isolation              |

### Architecture Diagram

![PythonBuddy Cloud Architecture](./images/architecture-cloud.png)

**How It Works:**

The architecture is designed for **high availability** and **automatic recovery**. Here's the traffic flow:

1. **User Request** → User opens PythonBuddy in their browser and makes an HTTP request on port 80

2. **Application Load Balancer (ALB)** → Routes incoming traffic to healthy backend instances
   - Listens on port 80 (HTTP)
   - Forwards requests to EC2 instances on port 5000
   - Performs health checks every 30 seconds via `/api/health`
   - Only sends traffic to instances that pass health checks

3. **Auto Scaling Group** → Manages backend EC2 instances in private subnets
   - Maintains 2 running instances (Min: 1, Max: 4)
   - Launches new instances if one fails
   - Uses Launch Template with User Data for automatic Docker deployment

4. **EC2 Instances (Private)** → Run Dockerized Flask backend
   - Located in private subnets (no direct internet access)
   - Pull Docker images from Docker Hub via NAT Gateway
   - Execute Python code and run Pylint checks
   - Health endpoint responds with HTTP 200 when healthy

5. **Supporting Infrastructure:**
   - **VPC (10.0.0.0/16)**: Isolated network environment
   - **Public Subnets**: Host ALB and NAT Gateway
   - **Private Subnets (AZ-a & AZ-b)**: Host EC2 instances across multiple availability zones
   - **NAT Gateway**: Provides internet access for private instances (outbound only)
   - **Internet Gateway**: Connects VPC to the internet
   - **Security Groups**:
     - ALB accepts HTTP from anywhere (0.0.0.0/0)
     - EC2 instances only accept traffic from ALB on port 5000
   - **IAM Role**: Grants EC2 instances permissions for CloudWatch and Systems Manager
   - **AWS Systems Manager**: Enables remote management without SSH

**Architecture Components:**

```
┌─────────────────┐
│  User Browser   │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────────┐
│  Amazon S3                      │
│  (Static React Frontend)        │
└────────┬────────────────────────┘
         │
         ▼
┌─────────────────────────────────┐
│  Application Load Balancer      │
│  (HTTP :80 → :5000)             │
└────────┬────────────────────────┘
         │
         ▼
┌─────────────────────────────────┐
│  Auto Scaling Group             │
│  (Private Subnets)              │
│  ┌─────────────────────────┐   │
│  │  EC2 Instance           │   │
│  │  Docker + Flask         │   │
│  │  Port: 5000             │   │
│  └─────────────────────────┘   │
│  ┌─────────────────────────┐   │
│  │  EC2 Instance           │   │
│  │  Docker + Flask         │   │
│  │  Port: 5000             │   │
│  └─────────────────────────┘   │
└─────────────────────────────────┘
```

### Key Architecture Features

**Frontend**

- React (Vite) hosted on S3 Static Website Hosting
- Direct asset delivery to users

**Backend**

- Flask API in Docker containers
- EC2 instances in **private subnets** (no direct internet access)
- Health check endpoint: `/api/health`

**Load Balancing**

- ALB routes traffic to healthy instances only
- Health checks every 30 seconds

**Auto Scaling & Self-Healing**

- Desired capacity: 2 instances (Min: 1, Max: 4)
- Automatic replacement of failed instances
- Zero-downtime during failures

**Security**

- Private subnets for backend
- Security groups: ALB accepts HTTP from internet, EC2 only from ALB

---

## Self-Healing Demonstration



https://github.com/user-attachments/assets/bc31521f-6cbf-4718-a684-6f2f97b9f688

**Live reliability test:**

1. Terminate a backend EC2 instance
2. Auto Scaling detects failure (< 30s)
3. New instance launches automatically
4. Load Balancer routes only to healthy targets
5. **Zero user-visible downtime**

---

## CI/CD Pipeline

Fully automated deployment with **GitHub Actions**:

```
Frontend: Push → Build React → Deploy to S3
Backend:  Push → Build Docker → Push to Docker Hub → EC2 Auto-Pulls
```

Zero manual intervention required.

---

## Tech Stack

### Frontend

- ![React](https://img.shields.io/badge/-React-61DAFB?logo=react&logoColor=white) React (Vite)
- ![TypeScript](https://img.shields.io/badge/-TypeScript-3178C6?logo=typescript&logoColor=white) TypeScript
- CodeMirror, Axios

### Backend

- ![Python](https://img.shields.io/badge/-Python-3776AB?logo=python&logoColor=white) Python 3.9 + Flask
- ![Docker](https://img.shields.io/badge/-Docker-2496ED?logo=docker&logoColor=white) Docker
- Pylint

### Cloud & DevOps

- ![AWS](https://img.shields.io/badge/-AWS-FF9900?logo=amazon-aws&logoColor=white) S3, EC2, ALB, Auto Scaling, VPC
- ![GitHub Actions](https://img.shields.io/badge/-GitHub%20Actions-2088FF?logo=github-actions&logoColor=white) GitHub Actions
- ![Docker Hub](https://img.shields.io/badge/-Docker%20Hub-2496ED?logo=docker&logoColor=white) Docker Hub

---

## Local Development

### Backend

```bash
git clone https://github.com/agavakole/pythonbuddy-cloud.git
cd pythonbuddy-cloud
python3 -m venv venv
source venv/bin/activate
pip install -r backend/requirements.txt
python -m PythonBuddy.app
# Visit: http://localhost:5000
```

### Frontend

```bash
cd frontend
pnpm install
echo "VITE_API_BASE_URL=http://localhost:5000" > .env
pnpm dev
# Visit: http://localhost:5173
```

### Docker

```bash
cd backend
docker build -t pythonbuddy-backend .
docker run -d -p 5000:5000 pythonbuddy-backend
curl http://localhost:5000/api/health
```

---

## Key Highlights

- ✅ **Fault-tolerant AWS architecture** with S3, ALB, and Auto Scaling
- ✅ **Dockerized backend** in private subnets for security
- ✅ **Self-healing system** with automatic instance replacement
- ✅ **Zero-downtime deployments** via CI/CD pipeline
- ✅ **Production cloud patterns** (multi-AZ, health checks, security groups)
- ✅ **99.9%+ uptime** through load balancing and auto-scaling

---

## Academic Context

Semester project focusing on:

- Cloud architecture design
- High availability systems
- Infrastructure automation
- Security best practices

---

## Future Improvements

- [ ] CloudFront CDN for global performance
- [ ] HTTPS with AWS Certificate Manager
- [ ] CloudWatch logging and monitoring
- [ ] ElastiCache for session management
- [ ] Multi-region disaster recovery

---

## Credits

- **Original PythonBuddy**: [Ethan Chiu](https://github.com/ethanchewy)
- **Cloud Architecture & Deployment**: Kole Agava

---

## 📄 License

Educational purposes. Original PythonBuddy by Ethan Chiu.

---

**Key Takeaway**: This project demonstrates production-style cloud engineering—building resilient, self-healing systems that handle real-world failures, not just applications.
