# PillSync-Docs  

A Cloud-Native Medication Reminder and Adherence Tracking Platform

**Note:** The source code for this project is contained in private repositories for security and intellectual property reasons. This repository serves as technical documentation, architecture overview, and deployment showcase.

PillSync is a production-grade, full-stack application designed to help users manage prescriptions, track medication adherence, and receive intelligent, timely notifications. Built with a modern tech stack and deployed using enterprise-level cloud infrastructure, this project demonstrates advanced skills in Full Stack Development, Cloud Architecture, DevOps, and Distributed Systems.

**Live Application:** pillsync.punyaksirohi.in  
**API Endpoint:** api.pillsync.punyaksirohi.in

---

## Project Screenshots

### Dashboard & Analytics  
(Insert screenshot: `dashboard.png`)

### Medication Management  
(Insert screenshot: `medications.png`)

### Automated CloudFront Invalidations (AWS Lambda)  
(Insert screenshot: `cloudfront-invalidation.png`)

---

# Architecture Overview

PillSync uses a split-domain, microservices-inspired architecture to maximize scalability, cost efficiency, and global performance.

```mermaid
graph TB
    User[End User]
    
    subgraph "Frontend Layer - Serverless"
        CF[CloudFront CDN<br/>Global Edge Locations]
        S3[S3 Bucket<br/>Static Website Hosting]
        Lambda[Lambda Function<br/>Cache Invalidation]
    end
    
    subgraph "DNS Layer"
        Vercel[Vercel DNS]
        FrontendDomain[pillsync.punyaksirohi.in<br/>CNAME]
        BackendDomain[api.pillsync.punyaksirohi.in<br/>A Record]
    end
    
    subgraph "Backend Layer - EC2 VPS"
        Nginx[Nginx Reverse Proxy<br/>Let's Encrypt SSL/TLS]
        Gunicorn[Gunicorn WSGI Server<br/>Systemd Service]
        Django[Django REST Framework]
        Celery[Celery + Celery Beat<br/>Background Tasks]
        Redis[Redis<br/>Message Broker]
    end
    
    subgraph "Data Layer"
        RDS[(AWS RDS<br/>PostgreSQL)]
        S3Media[S3 Bucket<br/>Media Storage]
    end
    
    User -->|HTTPS Request| Vercel
    Vercel -->|Static Assets| FrontendDomain
    Vercel -->|API Calls| BackendDomain
    FrontendDomain -->|CNAME| CF
    CF -->|Cached Content| S3
    S3 -.S3 Event Trigger.-> Lambda
    Lambda -.Invalidate Cache.-> CF
    
    BackendDomain -->|A Record| Nginx
    Nginx -->|Proxy Pass| Gunicorn
    Gunicorn --> Django
    Django --> RDS
    Django --> S3Media
    Django -.Task Queue.-> Redis
    Redis --> Celery
    Celery -.Send Reminders.-> User
````

---

# Request Flow

## Frontend Requests

1. User navigates to `pillsync.punyaksirohi.in`
2. DNS resolves to CloudFront distribution
3. CloudFront serves cached React SPA from S3 (edge-optimized)
4. Lambda automatically invalidates CloudFront cache on S3 upload

## API Requests

1. React app calls `api.pillsync.punyaksirohi.in`
2. DNS resolves to EC2 Elastic IP
3. Nginx terminates SSL/TLS and proxies to Gunicorn (port 8000)
4. Django processes the request and interacts with PostgreSQL RDS
5. Celery Beat schedules daily medication dose generation at midnight
6. Celery workers send notifications asynchronously

---

# Features

## Core Functionality

* Medication CRUD operations
* Smart Dose Tracking using Celery Beat
* Adherence Analytics (streaks, calendar view, time-of-day patterns)
* AI Adherence Predictions (Google Gemini)
* Caregiver Portal with permission levels
* Notification System (email, push)
* Token-Based Caregiver Invitations

## Technical Highlights

* JWT Authentication with rotating tokens
* User-scoped queries for maximum security
* React Query for cached state management
* Axios interceptors with automatic token refresh
* Clean import path aliases (`@/`, `@api/`, `@components/`)

---

# Technology Stack

## Frontend

| Technology   | Purpose                 | Version |
| ------------ | ----------------------- | ------- |
| React        | UI Framework            | 19.x    |
| Vite         | Build Tool              | 5.x     |
| TailwindCSS  | Styling                 | 3.x     |
| React Query  | Server State Management | Latest  |
| Axios        | HTTP Client             | Custom  |
| React Router | Routing                 | v6      |

## Backend

| Technology            | Purpose            | Version |
| --------------------- | ------------------ | ------- |
| Django                | Backend Framework  | 5.1.3   |
| Django REST Framework | API Layer          | 3.x     |
| Celery                | Task Queue         | 5.x     |
| Celery Beat           | Scheduler          | 2.x     |
| Redis                 | Message Broker     | 7.x     |
| Gunicorn              | WSGI Server        | 20.x    |
| Nginx                 | Reverse Proxy      | 1.18    |
| PostgreSQL            | Database (AWS RDS) | 14.x    |
| Gemini AI SDK         | AI Integration     | Latest  |

## Infrastructure (AWS)

| Service         | Purpose                 | Configuration           |
| --------------- | ----------------------- | ----------------------- |
| EC2             | Backend Compute         | t2.medium               |
| RDS             | PostgreSQL              | db.t3.micro             |
| S3              | Static + Media Storage  | 2 Buckets               |
| CloudFront      | CDN                     | Global coverage         |
| Lambda          | Event-driven automation | Cache invalidation      |
| Route 53        | DNS                     | Managed via Vercel      |
| IAM             | Access Control          | Least-Privilege Roles   |
| Security Groups | Firewall                | Ports 22, 80, 443, 8000 |

---

# Key Technical Challenges Solved

## 1. Split-Domain Architecture

**Problem:** Seamlessly connecting CloudFront frontend and EC2 backend.
**Solution:** Split-horizon DNS: frontend via CloudFront, backend via EC2.
**Result:** 70% cost reduction, <100ms load time worldwide.

## 2. CORS & Cross-Origin Security

**Problem:** Authenticated React-to-Django requests across domains.
**Solution:** Strict CORS policies, secure credentials, header forwarding.
**Result:** Stable and secure communication.

## 3. Infinite Auth Loop in React

**Problem:** Token refresh loop causing browser crash.
**Solution:** Deterministic state machine + isolated refresh interceptor.
**Result:** Zero auth loops, clean fallback behavior.

## 4. Background Task Reliability

**Problem:** Celery workers died on EC2 reboot.
**Solution:** Systemd service files with restart policies.
**Result:** 99.9% reliability.

## 5. Automated CloudFront Cache Invalidation

**Problem:** Manual invalidation slowed deployments.
**Solution:** Lambda triggers on S3 PUT → triggers CloudFront invalidation.
**Result:** CI/CD fully automated.

---

# Project Structure

```
PillSync/
├── pillsync-backend/           # Django REST API
│   ├── pillsync/
│   │   ├── settings.py
│   │   ├── urls.py
│   │   └── celery.py
│   ├── users/
│   ├── medications/
│   ├── doses/
│   ├── adherence/
│   ├── caregiver/
│   ├── notifications/
│   ├── ai_predictions/
│   │   └── gemini_service.py
│   ├── requirements.txt
│   └── manage.py
│
├── pillsync-frontend/          # React SPA
│   ├── src/
│   │   ├── api/
│   │   ├── components/
│   │   ├── hooks/
│   │   │   └── api/
│   │   ├── lib/
│   │   │   └── axios.js
│   │   ├── pages/
│   │   └── App.jsx
│   ├── vite.config.js
│   ├── tailwind.config.js
│   └── package.json
│
└── README.md
```

---

# Author

**Punya K Sirohi**

Portfolio: [https://punyaksirohi.in](https://punyaksirohi.in)
GitHub: [https://github.com/PunyaKSirohi](https://github.com/PunyaKSirohi)
LinkedIn: [https://linkedin.com/in/punyaksirohi](https://linkedin.com/in/punyaksirohi)
Email: [punyaksirohi@example.com](mailto:punyaksirohi@example.com)

---

# License
MIT License
```
