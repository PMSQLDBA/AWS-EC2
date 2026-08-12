# ☁️ AWS EC2 — Lesson 10/10

## Real-World EC2 Architecture

🎉 **Final Lesson!**

We've built the pieces individually. Now we're going to put **all 10 lessons together** into one realistic EC2 architecture.

```text
✅ Lesson 1 — EC2 Fundamentals
✅ Lesson 2 — Launching EC2
✅ Lesson 3 — Connecting to EC2
✅ Lesson 4 — EC2 Storage
✅ Lesson 5 — Security Groups
✅ Lesson 6 — Networking
✅ Lesson 7 — Pricing & Scaling
✅ Lesson 8 — Load Balancing
✅ Lesson 9 — Monitoring & Operations
➡️ Lesson 10 — Real-World Architecture
```

---

# 1. The Scenario

Imagine you're building an online shopping application.

Users should be able to:

```text
Browse website
     ↓
Log in
     ↓
View products
     ↓
Place orders
```

We want the architecture to be:

* Secure
* Highly available
* Scalable
* Monitorable
* Recoverable

Instead of putting everything onto one EC2 instance, we'll separate the architecture into layers.

---

# 2. The Big Picture

Here's the architecture we'll build conceptually:

```text
                           INTERNET
                               │
                               ▼
                        ┌─────────────┐
                        │     ALB     │
                        │Load Balancer│
                        └──────┬──────┘
                               │
                    ┌──────────┴──────────┐
                    │                     │
                    ▼                     ▼
                 AZ-A                  AZ-B
             ┌──────────┐          ┌──────────┐
             │ EC2 #1   │          │ EC2 #2   │
             │ Web/App  │          │ Web/App  │
             └────┬─────┘          └────┬─────┘
                  │                     │
                  └──────────┬──────────┘
                             │
                             ▼
                       Private Layer
                             │
                             ▼
                         Database
```

Around all of this:

```text
Security Groups
Auto Scaling
CloudWatch
Systems Manager
Backups
```

---

# 3. Start With the Region

Everything starts with an AWS Region.

For example:

```text id="b4z9o1"
AWS Region
│
├── Availability Zone A
│
└── Availability Zone B
```

Why multiple Availability Zones?

Because we don't want the entire application dependent on one location.

---

# 4. Create a VPC

Inside the Region:

```text id="x3l4e5"
AWS Region
    │
    ▼
   VPC
```

The VPC is our private networking environment.

For example:

```text id="z0n3jv"
VPC
10.0.0.0/16
```

Inside it, we'll create multiple subnets.

---

# 5. Create Public Subnets

We need public subnets for components that need to receive internet traffic.

For example:

```text id="9g3o2n"
VPC
│
├── Public Subnet A
│      AZ-A
│
└── Public Subnet B
       AZ-B
```

The public subnets have routes toward an Internet Gateway.

Conceptually:

```text id="8r9p8d"
Internet
    │
    ▼
Internet Gateway
    │
    ▼
Public Subnets
```

---

# 6. Create Private Subnets

Now create private subnets.

```text id="1x7v1d"
VPC
│
├── Public Subnet A
├── Public Subnet B
│
├── Private Subnet A
└── Private Subnet B
```

Our application servers can live in private subnets.

For example:

```text id="9x0o1q"
Private Subnet A
    │
    └── EC2 #1

Private Subnet B
    │
    └── EC2 #2
```

The EC2 instances don't need to be directly exposed to the public internet.

---

# 7. Internet Gateway

The Internet Gateway connects the VPC to the internet.

Our public architecture becomes:

```text id="7x1z4h"
                  INTERNET
                      │
                      ▼
              Internet Gateway
                      │
              ┌───────┴───────┐
              ▼               ▼
        Public Subnet A   Public Subnet B
```

The route tables determine where traffic goes.

---

# 8. Route Tables

Public subnet:

```text id="l9i4gq"
Destination       Target
────────────────────────────
VPC CIDR          local
0.0.0.0/0         Internet Gateway
```

Conceptually:

```text id="84i2bk"
Public Subnet
     │
     ▼
Route Table
     │
     ▼
Internet Gateway
     │
     ▼
Internet
```

Private subnet:

```text id="0sjq7r"
Destination       Target
────────────────────────────
VPC CIDR          local
0.0.0.0/0         NAT Gateway
```

This provides outbound internet connectivity without making the private instances directly internet-facing.

---

# 9. NAT Gateway

Why might private EC2 instances need internet access?

For example:

```text id="e7x7x4"
Private EC2
     │
     │ Download updates
     ▼
Internet
```

We don't want to give the EC2 a public-facing role just for outbound connectivity.

So:

```text id="w6p2zj"
Private EC2
     │
     ▼
Private Route Table
     │
     ▼
NAT Gateway
     │
     ▼
Internet Gateway
     │
     ▼
Internet
```

---

# 10. Application Load Balancer

Now we need to expose our application.

Users should not directly connect to:

```text id="9r3m6x"
EC2 #1
EC2 #2
```

Instead:

```text id="d4w6am"
Users
  │
  ▼
ALB
```

The ALB is placed across appropriate public subnets.

Conceptually:

```text id="8o6z1s"
                    INTERNET
                        │
                        ▼
                 ┌─────────────┐
                 │     ALB     │
                 └──────┬──────┘
                        │
               ┌────────┴────────┐
               ▼                 ▼
          Private AZ-A      Private AZ-B
               │                 │
               ▼                 ▼
             EC2 #1            EC2 #2
```

---

# 11. Target Group

The ALB needs to know where to send requests.

So we create:

```text id="g6i2xy"
Target Group
│
├── EC2 #1
└── EC2 #2
```

The relationship:

```text id="3v5h2d"
User
 ↓
ALB
 ↓
Listener
 ↓
Target Group
 ↓
EC2
```

---

# 12. Health Checks

The ALB periodically checks the EC2 instances.

For example:

```text id="0i1y8f"
GET /health
```

Imagine:

```text id="2v2o3x"
EC2 #1 → Healthy
EC2 #2 → Healthy
```

Both receive traffic.

But:

```text id="b5t4r9"
EC2 #2 → Unhealthy
```

The ALB can stop routing new traffic to EC2 #2.

```text id="0q5j0h"
ALB
 │
 ├── EC2 #1 ✅
 └── EC2 #2 ❌
```

---

# 13. Security Groups

Now we secure the architecture.

We'll use separate Security Groups.

### ALB Security Group

Allow:

```text id="2i4o0v"
HTTPS :443
Source: Internet
```

Conceptually:

```text id="8z6n1m"
Internet
    │
    │ 443
    ▼
ALB Security Group
```

---

# 14. EC2 Security Group

The EC2 instances don't need to accept application traffic from everyone on the internet.

Instead:

```text id="e7z6ap"
EC2 Security Group
       ↑
       │
Traffic from ALB Security Group
```

Conceptually:

```text id="2f2z1m"
Internet
   │
   ▼
  ALB
   │
   │ Application traffic
   ▼
 EC2
```

This creates a security boundary.

---

# 15. Administrative Access

We also need administrators to manage EC2.

Instead of exposing SSH broadly:

```text id="4q3r0y"
Internet
   │
   ▼
SSH :22
   │
   ▼
EC2
```

we can use Systems Manager Session Manager where appropriate:

```text id="r1y4cj"
Administrator
     │
     ▼
Systems Manager
     │
     ▼
EC2
```

This can reduce the need for public administrative access.

---

# 16. Auto Scaling Group

Now imagine our application gets popular.

Initially:

```text id="4c8f3p"
EC2 #1
EC2 #2
```

Traffic increases.

Auto Scaling can add:

```text id="0q0g8b"
EC2 #3
EC2 #4
```

Now:

```text id="t6f5b4"
              ALB
               │
       ┌───────┼────────┐
       ▼       ▼        ▼
     EC2 #1  EC2 #2   EC2 #3
                        │
                      EC2 #4
```

The target group can receive the new instances according to the Auto Scaling configuration.

---

# 17. Multi-AZ Scaling

This is where our architecture becomes highly available.

Instead of:

```text id="9qf4pp"
AZ-A
 ├── EC2 #1
 ├── EC2 #2
 └── EC2 #3
```

we distribute instances:

```text id="6k5y5b"
             Region
          ┌─────┴─────┐
          ▼           ▼
        AZ-A         AZ-B
          │           │
       EC2 #1      EC2 #2
       EC2 #3      EC2 #4
```

Now a failure in one AZ doesn't necessarily take down the entire application.

---

# 18. Minimum, Desired, Maximum

Suppose the Auto Scaling Group has:

```text id="1x2k5z"
Minimum = 2
Desired = 2
Maximum = 6
```

Normal state:

```text id="e7t8uk"
2 EC2 instances
```

High traffic:

```text id="2v0x6w"
2
↓
3
↓
4
↓
5
↓
6
```

Low traffic:

```text id="1b9jkl"
6
↓
5
↓
4
↓
3
↓
2
```

The system adjusts capacity according to your scaling configuration.

---

# 19. CloudWatch

Now we need visibility.

CloudWatch monitors things such as:

```text id="j8x4dr"
EC2
├── CPU
├── Network
└── Other configured metrics

ALB
├── Request metrics
├── Target health-related metrics
└── Other load-balancer metrics
```

We can create alarms.

For example:

```text id="5k4g8a"
CPU
 │
 ▼
CloudWatch Alarm
 │
 ▼
Notification / Scaling-related action
```

---

# 20. Logs

Our application generates logs:

```text id="4w0y0q"
INFO User logged in
INFO Order created
ERROR Database unavailable
```

We can configure appropriate log collection.

Then:

```text id="v3x4me"
EC2
 │
 ▼
CloudWatch Logs
```

Now operations teams can investigate application behavior.

---

# 21. Database Layer

Our application probably needs a database.

For a real AWS architecture, you would often consider a managed database service such as Amazon RDS rather than installing the database directly on an EC2 instance.

Conceptually:

```text id="a7c9v0"
                ALB
                 │
                 ▼
              EC2s
                 │
                 ▼
          Private Database
```

The database should not be publicly accessible.

Security should be layered:

```text id="u1s9g3"
Internet
   │
   X
Database
```

Only the required application layer should be able to communicate with it.

---

# 22. Complete Security Architecture

Think in layers:

```text id="1x0w8b"
                 INTERNET
                     │
                     ▼
                ┌────────┐
                │  ALB   │
                └───┬────┘
                    │
              ALB Security Group
                    │
                    ▼
              ┌──────────┐
              │   EC2    │
              └────┬─────┘
                   │
             EC2 Security Group
                   │
                   ▼
              ┌──────────┐
              │ Database │
              └──────────┘
```

The idea is:

> **Each layer only accepts the traffic it actually needs.**

---

# 23. Complete Networking Architecture

Now combine networking:

```text id="0q3q9r"
                         INTERNET
                            │
                            ▼
                    Internet Gateway
                            │
                            ▼
                    ┌──────────────┐
                    │Public Subnets│
                    │    ALB       │
                    └──────┬───────┘
                           │
                           ▼
                    Private Subnets
                    ┌──────┴──────┐
                    ▼             ▼
                  EC2 #1        EC2 #2
                    │             │
                    └──────┬──────┘
                           │
                           ▼
                       Database
```

And outbound access from private instances:

```text id="i5c7a1"
Private EC2
     │
     ▼
NAT Gateway
     │
     ▼
Internet Gateway
     │
     ▼
Internet
```

---

# 24. Complete Operations Architecture

Now add operations:

```text id="yd4i5e"
                       CloudWatch
                      /    │     \
                 Metrics  Logs   Alarms
                    │
                    │
                    ▼
                   EC2
                    │
                    ▼
             Systems Manager
```

And recovery:

```text id="3o2w2p"
EC2 / EBS
   │
   ▼
Snapshots / Backups
   │
   ▼
Recovery
```

---

# 25. The Final Architecture

Here is the complete mental picture:

```text id="1p7n4j"
                              USERS
                                │
                                ▼
                         ┌─────────────┐
                         │     ALB     │
                         │ HTTPS :443  │
                         └──────┬──────┘
                                │
                 ┌──────────────┴──────────────┐
                 │                             │
                 ▼                             ▼
              AZ-A                           AZ-B
        ┌────────────────┐             ┌────────────────┐
        │ Private Subnet │             │ Private Subnet │
        │                │             │                │
        │    EC2 #1      │             │    EC2 #2      │
        │    EC2 #3      │             │    EC2 #4      │
        └───────┬────────┘             └───────┬────────┘
                │                              │
                └──────────────┬───────────────┘
                               │
                               ▼
                        Database Layer
                         Private Only

        ┌──────────────────────────────────────────┐
        │              OPERATIONS                  │
        │                                          │
        │  CloudWatch → Metrics / Logs / Alarms    │
        │  Systems Manager → Administration        │
        │  Backups → Recovery                      │
        │  Auto Scaling → Capacity                 │
        └──────────────────────────────────────────┘
```

---

# 26. How a User Request Travels

Let's follow **one HTTPS request**.

### Step 1 — User

```text id="4y2l3e"
Browser
   │
   │ HTTPS
   ▼
Internet
```

### Step 2 — Internet Gateway

```text id="a8s5g7"
Internet
   │
   ▼
Internet Gateway
```

### Step 3 — ALB

```text id="2n6m4k"
Internet Gateway
       │
       ▼
      ALB
```

### Step 4 — Listener

```text id="0c6q7w"
ALB
 │
 ▼
HTTPS Listener :443
```

### Step 5 — Target Group

```text id="4j7f3q"
Listener
   │
   ▼
Target Group
```

### Step 6 — Healthy EC2

```text id="5m8h2p"
Target Group
    │
    ▼
EC2 #2
```

### Step 7 — Application

```text id="7v3j1s"
EC2
 │
 ▼
Application
 │
 ▼
Database
```

### Step 8 — Response

The response travels back through the established connection.

Conceptually:

```text id="5j8z6m"
Database
   ↓
Application
   ↓
EC2
   ↓
ALB
   ↓
Internet
   ↓
User
```

---

# 27. What Happens When Traffic Increases?

Suppose:

```text id="9u2r5w"
Normal traffic
     ↓
2 EC2 instances
```

Then traffic increases:

```text id="6e1x8k"
High traffic
     ↓
CloudWatch metrics
     ↓
Scaling policy
     ↓
Auto Scaling Group
     ↓
Launch additional EC2
```

Now:

```text id="h7r4z1"
EC2 #1
EC2 #2
EC2 #3
EC2 #4
```

The ALB distributes traffic across healthy targets.

---

# 28. What Happens When an EC2 Fails?

Suppose:

```text id="g9s2q7"
EC2 #2
   ↓
FAILS
```

The target health check detects the problem.

```text id="3v5x8b"
ALB
 │
 ├── EC2 #1 ✅
 ├── EC2 #2 ❌
 └── EC2 #3 ✅
```

The ALB stops sending new traffic to the unhealthy instance.

Then Auto Scaling can replace the failed capacity according to its configuration.

```text id="f2n7s4"
Failed EC2
    ↓
Removed
    ↓
Replacement EC2
    ↓
Health check passes
    ↓
Receives traffic
```

This is **self-healing infrastructure** at a basic conceptual level.

---

# 29. What Happens When an AZ Has a Problem?

Suppose:

```text id="k7w3n9"
AZ-A ❌
AZ-B ✅
```

Because our application is distributed:

```text id="s4p8y2"
AZ-A
 EC2 #1 ❌
 EC2 #3 ❌

AZ-B
 EC2 #2 ✅
 EC2 #4 ✅
```

The application may continue operating from AZ-B, depending on the architecture and capacity available there.

This is why:

> **Multi-AZ architecture is important for high availability.**

---

# 30. What Happens When You Need to Scale Down?

Traffic decreases:

```text id="u3d6w9"
High traffic
    ↓
6 EC2
    ↓
Traffic decreases
    ↓
Auto Scaling
    ↓
4 EC2
    ↓
2 EC2
```

The goal is to maintain the desired level of capacity without permanently running unnecessary instances.

This connects directly back to **Lesson 7 — Pricing & Scaling**.

---

# 31. What Happens During an Investigation?

Suppose users report:

> "The website is slow."

You don't immediately reboot everything.

You investigate:

```text id="0o7n2p"
ALB
 ↓
Target health
 ↓
CloudWatch metrics
 ↓
EC2 CPU
 ↓
Network
 ↓
Application logs
 ↓
Database
```

Maybe you discover:

```text id="w4r8z0"
EC2 CPU = 98%
```

Then:

```text id="f7v3k2"
Why?
 ↓
Traffic spike?
Application bug?
Unexpected process?
Instance too small?
```

Monitoring gives you evidence.

---

# 32. The 10-Lesson EC2 Map

You've now learned the complete progression:

```text id="z5t8x2"
LESSON 1
EC2 Fundamentals
      ↓
LESSON 2
Launch EC2
      ↓
LESSON 3
Connect to EC2
      ↓
LESSON 4
Storage
      ↓
LESSON 5
Security Groups
      ↓
LESSON 6
Networking
      ↓
LESSON 7
Pricing & Scaling
      ↓
LESSON 8
Load Balancing
      ↓
LESSON 9
Monitoring & Operations
      ↓
LESSON 10
Real-World Architecture
```

And all the pieces connect:

```text id="a2g7p1"
EC2
│
├── Storage
│
├── Security Groups
│
├── VPC / Subnets
│
├── Route Tables
│
├── Internet Gateway / NAT
│
├── ALB
│
├── Target Groups
│
├── Auto Scaling
│
├── CloudWatch
│
├── Systems Manager
│
└── Backups
```

---

# 🧠 Final EC2 Cheat Sheet

## EC2

> Virtual servers in AWS.

## VPC

> Your logical network environment in AWS.

## Subnet

> A network segment inside a VPC and tied to an Availability Zone.

## Route Table

> Determines where network traffic goes.

## Internet Gateway

> Provides a path between a VPC and the internet.

## NAT Gateway

> Allows resources in private subnets to initiate outbound internet connections without being directly internet-facing.

## Security Group

> Stateful, instance/network-interface-level firewall controlling allowed traffic.

## EBS

> Persistent block storage commonly used with EC2.

## ALB

> Distributes HTTP/HTTPS traffic across healthy targets.

## Target Group

> Group of backend targets receiving traffic from the load balancer.

## Health Check

> Determines whether a target is healthy.

## Auto Scaling

> Automatically adjusts EC2 capacity according to configured policies.

## CloudWatch

> Monitors metrics, logs, and alarms.

## Systems Manager

> Helps manage and operate AWS resources and EC2 instances.

---

# 🏆 The One Diagram to Remember

If you remember **only one EC2 architecture diagram**, remember this:

```text
                           🌎 USERS
                              │
                              ▼
                     ┌────────────────┐
                     │      ALB       │
                     │   HTTPS :443   │
                     └───────┬────────┘
                             │
                 ┌───────────┴───────────┐
                 │                       │
                 ▼                       ▼
              AZ-A                     AZ-B
        ┌──────────────┐        ┌──────────────┐
        │ Private      │        │ Private      │
        │ Subnet       │        │ Subnet       │
        │              │        │              │
        │ EC2 #1       │        │ EC2 #2       │
        │ EC2 #3       │        │ EC2 #4       │
        └──────┬───────┘        └──────┬───────┘
               │                       │
               └───────────┬───────────┘
                           ▼
                    DATABASE LAYER
                       PRIVATE
                           │
                           ▼
                    ┌──────────────┐
                    │   Backups    │
                    └──────────────┘


        ┌──────────────────────────────────────┐
        │              OPERATIONS              │
        │                                      │
        │ CloudWatch → Metrics / Logs / Alarms │
        │ Auto Scaling → Capacity              │
        │ Systems Manager → Administration     │
        │ Security Groups → Network Security   │
        └──────────────────────────────────────┘
```

---

# 🎓 Final Checkpoint — EC2 Mastery

Before considering EC2 fully understood, make sure you can explain these **without looking at your notes**:

### 1.

What is an EC2 instance?

### 2.

What is the difference between an AMI and an instance type?

### 3.

What is the difference between private and public IP addresses?

### 4.

What does a Security Group do?

### 5.

What is a VPC?

### 6.

What is a subnet?

### 7.

What makes a subnet public?

### 8.

What does a route table do?

### 9.

What does an Internet Gateway do?

### 10.

Why would a private subnet use a NAT Gateway?

### 11.

What's the difference between vertical and horizontal scaling?

### 12.

What's the difference between an ALB and an Auto Scaling Group?

### 13.

What is a Target Group?

### 14.

Why are health checks important?

### 15.

What's the difference between a CloudWatch metric and a log?

### 16.

What's the difference between monitoring and backup?

### 17.

Why use multiple Availability Zones?

### 18.

Why shouldn't a database normally be directly exposed to the public internet?

### 19.

Explain this flow:

```text
User
 ↓
ALB
 ↓
Target Group
 ↓
EC2
 ↓
Database
```

### 20. 🔥 Final Challenge

Explain this entire architecture in your own words:

```text
                         INTERNET
                            │
                            ▼
                           ALB
                            │
               ┌────────────┴────────────┐
               ▼                         ▼
             AZ-A                      AZ-B
               │                         │
            EC2 #1                    EC2 #2
               │                         │
               └────────────┬────────────┘
                            │
                       Auto Scaling
                            │
                            ▼
                        Database
```

Then explain where these fit:

```text
VPC
Subnet
Route Table
Internet Gateway
NAT Gateway
Security Group
Target Group
CloudWatch
Systems Manager
Backups
```

If you can explain that diagram and the role of each component, you've moved beyond simply knowing **what EC2 is**—you understand the **core architecture around EC2**.
