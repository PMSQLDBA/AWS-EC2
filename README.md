# ☁️ AWS EC2 — 10-Lesson Course

### Lesson 1 — EC2 Fundamentals

* What EC2 is
* Why EC2 exists
* EC2 vs a physical server
* Instances, AMIs, and instance types
* Regions and Availability Zones
* The basic EC2 architecture

### Lesson 2 — Launching Your First EC2 Instance

* Choosing an AMI
* Choosing an instance type
* Key pairs
* Network settings
* Storage
* Security groups
* Launching an instance

### Lesson 3 — Connecting to EC2

* Linux EC2
* SSH
* Windows EC2 / RDP
* Public vs private IP
* Elastic IP
* Troubleshooting connection problems

### Lesson 4 — EC2 Storage

* EBS
* EBS volumes
* Root volume
* Additional volumes
* Snapshots
* EBS volume types
* Instance Store

### Lesson 5 — Security Groups & EC2 Security

* Inbound rules
* Outbound rules
* Ports
* SSH (22)
* HTTP (80)
* HTTPS (443)
* Security-group best practices
* Common security mistakes

### Lesson 6 — EC2 Networking

* VPC
* Subnets
* Public/private subnets
* Route tables
* Internet Gateway
* NAT Gateway
* Public/private EC2 architecture

### Lesson 7 — EC2 Pricing & Scaling

* On-Demand
* Reserved Instances
* Savings Plans
* Spot Instances
* Instance sizing
* Cost optimization
* Auto Scaling basics

### Lesson 8 — Load Balancing with EC2

* Why load balancing is needed
* Application Load Balancer
* Target Groups
* Health checks
* Multiple EC2 instances
* High availability architecture

### Lesson 9 — Monitoring & Operations

* CloudWatch
* Metrics
* Logs
* Alarms
* Status checks
* Systems Manager
* Backups
* Troubleshooting EC2

### Lesson 10 — Real-World EC2 Architecture

We'll put everything together:

```text
                    Internet
                       │
                       ▼
                ┌─────────────┐
                │     ALB     │
                └──────┬──────┘
                       │
              ┌────────┴────────┐
              ▼                 ▼
        ┌──────────┐       ┌──────────┐
        │ EC2 #1   │       │ EC2 #2   │
        └────┬─────┘       └────┬─────┘
             │                  │
             └────────┬─────────┘
                      ▼
                 ┌─────────┐
                 │   EBS   │
                 └─────────┘

              VPC
     ┌───────────────────────┐
     │ Public + Private      │
     │ Subnets + Routing     │
     │ Security Groups       │
     └───────────────────────┘
```

By the end, you should be able to explain **how an EC2 instance is launched, secured, connected, stored, monitored, scaled, and placed into a production architecture**.

**Stop here.** Once these five are solid, Lesson 2 will be **launching your first EC2 instance**.
