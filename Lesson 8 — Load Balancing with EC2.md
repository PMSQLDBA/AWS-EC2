# ☁️ AWS EC2 — Lesson 8/10

## Load Balancing with EC2

We've reached:

```text
✅ Lesson 1 — EC2 Fundamentals
✅ Lesson 2 — Launching EC2
✅ Lesson 3 — Connecting to EC2
✅ Lesson 4 — EC2 Storage
✅ Lesson 5 — Security Groups
✅ Lesson 6 — Networking
✅ Lesson 7 — Pricing & Scaling
➡️ Lesson 8 — Load Balancing
```

Today we're going to connect **multiple EC2 instances** together so they can serve users as one application.

---

# 1. The Problem

Imagine you have one EC2 instance:

```text
             USERS
                │
                ▼
             ┌─────┐
             │ EC2 │
             └─────┘
```

This works.

But what happens if:

* Traffic increases?
* The EC2 instance crashes?
* You need to perform maintenance?
* You have thousands of users?

One server becomes a problem.

So you add more:

```text
             USERS
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
     EC2 #1   EC2 #2   EC2 #3
```

Now we have a new question:

> **How do users know which EC2 instance to connect to?**

That's where a **Load Balancer** comes in.

---

# 2. What Is a Load Balancer?

A load balancer receives incoming traffic and distributes it across multiple backend servers.

Think:

```text
                  USERS
                    │
                    ▼
             ┌─────────────┐
             │    LOAD     │
             │  BALANCER   │
             └──────┬──────┘
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
        EC2 #1    EC2 #2    EC2 #3
```

Instead of users directly choosing an EC2 instance:

```text
User → EC2 #1
User → EC2 #2
User → EC2 #3
```

they connect to the load balancer:

```text
User → Load Balancer → EC2
```

---

# 3. AWS Application Load Balancer

For HTTP/HTTPS applications, AWS provides:

> **Application Load Balancer (ALB)**

ALB operates at the application layer and is designed for HTTP/HTTPS traffic.

Typical architecture:

```text
              INTERNET
                  │
                  ▼
          ┌───────────────┐
          │      ALB      │
          └───────┬───────┘
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
    EC2 #1      EC2 #2      EC2 #3
```

---

# 4. Why Do We Need an ALB?

Suppose you have:

```text
EC2 #1
EC2 #2
EC2 #3
```

Without a load balancer, users need some way to reach the correct server.

With an ALB:

```text
Users
  │
  ▼
 ALB
  │
  ├──► EC2 #1
  ├──► EC2 #2
  └──► EC2 #3
```

The ALB handles the distribution.

This provides:

### Scalability

More EC2 instances can serve traffic.

### Availability

If one instance fails, healthy instances can continue serving.

### Abstraction

Users don't need to know which individual EC2 instance is serving their request.

---

# 5. Load Balancer vs Auto Scaling

These are related but **not the same thing**.

This is extremely important.

### Load Balancer

> **Distributes traffic.**

```text
Users
 ↓
ALB
 ↓
EC2 EC2 EC2
```

### Auto Scaling

> **Changes the number of EC2 instances.**

```text
2 EC2
 ↓
High demand
 ↓
4 EC2
```

Together:

```text
               USERS
                 │
                 ▼
                ALB
                 │
        ┌────────┼────────┐
        ▼        ▼        ▼
       EC2      EC2      EC2
        └────────┼────────┘
                 ▲
                 │
           Auto Scaling
```

Think:

> **ALB decides where traffic goes.**

> **Auto Scaling decides how many instances should exist.**

---

# 6. Target Groups

Now we need to tell the ALB:

> **Which EC2 instances should receive traffic?**

That's where a:

# Target Group

comes in.

Conceptually:

```text
ALB
 │
 ▼
Target Group
 │
 ├── EC2 #1
 ├── EC2 #2
 └── EC2 #3
```

The target group contains the backend targets.

For our EC2 example:

```text
Target Group
     ↓
EC2 instances
```

---

# 7. ALB + Target Group

The complete relationship is:

```text
Users
  │
  ▼
ALB
  │
  ▼
Listener
  │
  ▼
Target Group
  │
  ├── EC2 #1
  ├── EC2 #2
  └── EC2 #3
```

There are several concepts here.

We'll take them one at a time.

---

# 8. Listener

A **listener** waits for incoming connection requests on a specified protocol and port.

For example:

```text
HTTPS
Port 443
```

Conceptually:

```text
Internet
   │
   │ HTTPS :443
   ▼
 ALB
   │
 Listener
   │
   ▼
Target Group
```

You can think of a listener as:

> **“ALB, listen for this type of traffic on this port.”**

---

# 9. Listener Rule

The ALB can use rules to decide where traffic should go.

For example:

```text
IF
Host = api.example.com

THEN
send to API Target Group
```

Or:

```text
IF
Path = /images/*

THEN
send to Image Target Group
```

Conceptually:

```text
                ALB
                 │
          ┌──────┴──────┐
          │             │
       /api/*        /images/*
          │             │
          ▼             ▼
      API TG         Image TG
```

This is one reason ALBs are useful for modern web applications.

---

# 10. Health Checks

Now imagine:

```text
ALB
 │
 ├── EC2 #1 → healthy
 ├── EC2 #2 → healthy
 └── EC2 #3 → broken
```

You don't want the ALB sending users to the broken server.

That's why target groups perform **health checks**.

The ALB asks:

> **“Is this target healthy enough to receive traffic?”**

For example, the application might expose:

```text
/health
```

The load balancer can periodically check it.

Conceptually:

```text
ALB
 │
 ├──► EC2 #1
 │     /health → OK
 │
 ├──► EC2 #2
 │     /health → OK
 │
 └──► EC2 #3
       /health → FAIL
```

Then the ALB can stop routing traffic to the unhealthy target.

---

# 11. Why Health Checks Matter

Without health checks:

```text
ALB
 │
 ├── EC2 #1 ✅
 ├── EC2 #2 ❌
 └── EC2 #3 ✅
```

The ALB might continue sending requests to the broken instance.

With health checks:

```text
ALB
 │
 ├── EC2 #1 ✅
 ├── EC2 #2 ❌ → removed from traffic
 └── EC2 #3 ✅
```

This improves availability.

---

# 12. Load Distribution

Suppose you have three healthy EC2 instances:

```text
       ALB
        │
   ┌────┼────┐
   ▼    ▼    ▼
  EC1  EC2  EC3
```

The ALB distributes requests among eligible targets according to its routing behavior and configuration.

Conceptually:

```text
Request 1 → EC2 #1
Request 2 → EC2 #2
Request 3 → EC2 #3
Request 4 → EC2 #1
...
```

Don't think of it as simply:

> "Exactly one request to each server forever."

The actual distribution depends on the load balancer's behavior and configuration.

---

# 13. Availability Zones

Now connect this to Lesson 6.

A production application shouldn't ideally depend on one Availability Zone.

Instead:

```text
                 Region
          ┌────────┴────────┐
          ▼                 ▼
        AZ-A               AZ-B
          │                 │
        EC2 #1             EC2 #2
        EC2 #3             EC2 #4
          │                 │
          └────────┬────────┘
                   ▼
                  ALB
```

The ALB can operate across Availability Zones.

This helps protect your application against an Availability Zone-level failure.

---

# 14. Security Groups With ALB

Now let's combine Lesson 5.

You can have:

```text
Internet
   │
   ▼
ALB Security Group
   │
   ▼
ALB
   │
   ▼
EC2 Security Group
   │
   ▼
EC2
```

A strong architecture is to separate the security responsibilities.

For example:

### ALB Security Group

Allow:

```text
HTTPS :443
from the internet
```

### EC2 Security Group

Allow application traffic:

```text
from the ALB's Security Group
```

Conceptually:

```text
Internet
   │
   │ 443
   ▼
[ ALB-SG ]
   │
   ▼
   ALB
   │
   │ App traffic
   ▼
[ EC2-SG ]
   │
   ▼
  EC2
```

This is much better than blindly allowing the entire internet to reach every EC2 instance.

---

# 15. A Very Important Security Improvement

Suppose your EC2 application listens on port 8080.

You don't necessarily want:

```text
Internet
   │
   │ :8080
   ▼
EC2
```

Instead:

```text
Internet
   │
   │ HTTPS :443
   ▼
ALB
   │
   │ :8080
   ▼
EC2
```

And the EC2 Security Group can allow port 8080 **only from the ALB's Security Group**.

This creates a layered architecture:

```text
Public Internet
      ↓
     ALB
      ↓
    EC2
```

---

# 16. ALB and HTTPS

A common production setup is:

```text
User
 │
 │ HTTPS
 │ 443
 ▼
ALB
 │
 ▼
EC2
```

The ALB can handle HTTPS connections using an appropriate certificate.

This means users communicate securely with the load balancer.

The exact encryption architecture between the ALB and backend instances depends on your configuration.

---

# 17. What Happens When an EC2 Instance Dies?

Suppose:

```text
ALB
 │
 ├── EC2 #1 ✅
 ├── EC2 #2 ❌
 └── EC2 #3 ✅
```

The health check detects:

```text
EC2 #2
   ↓
Unhealthy
```

The ALB stops sending new traffic to it.

Now:

```text
ALB
 │
 ├── EC2 #1 ✅
 └── EC2 #3 ✅
```

If you're using an Auto Scaling Group, the Auto Scaling system can replace the unhealthy instance depending on the configured behavior.

Then:

```text
Old EC2 #2
     ↓
Removed
     ↓
New EC2 #4
     ↓
Health check passes
     ↓
Traffic begins
```

This is the power of combining:

```text
ALB + Target Group + Auto Scaling
```

---

# 18. Complete Production Flow

Now let's put everything together.

```text
                         USERS
                           │
                           ▼
                    ┌─────────────┐
                    │     ALB     │
                    └──────┬──────┘
                           │
                     Listener
                           │
                           ▼
                    Target Group
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
            EC2 #1       EC2 #2       EC2 #3
              │            │            │
              └────────────┼────────────┘
                           ▲
                           │
                    Auto Scaling
```

With networking:

```text
Internet
   │
   ▼
Internet Gateway
   │
   ▼
Public Subnets
   │
   ▼
ALB
   │
   ▼
Private Subnets
   │
   ▼
EC2 Instances
```

That's a very common cloud architecture pattern.

---

# 19. Load Balancer vs Target Group vs Listener

This is a common interview question.

### Load Balancer

> Receives and distributes traffic.

```text
ALB
```

### Listener

> Listens for incoming traffic on a port/protocol and applies rules.

```text
HTTPS :443
```

### Target Group

> Defines the backend targets that can receive traffic.

```text
EC2 #1
EC2 #2
EC2 #3
```

Memorize:

```text
ALB
 ↓
Listener
 ↓
Target Group
 ↓
EC2
```

---

# 20. ALB vs EC2

Don't confuse the roles.

```text
ALB
→ Traffic management

EC2
→ Application execution
```

The ALB isn't normally where your application code runs.

Your application runs on the backend EC2 instances.

---

# 21. ALB + Auto Scaling

This is the key architecture from today's lesson:

```text
                    USERS
                      │
                      ▼
                    ALB
                      │
              ┌───────┴───────┐
              ▼               ▼
            EC2 #1          EC2 #2
              │               │
              └───────┬───────┘
                      ▲
                      │
                Auto Scaling
```

Traffic increases:

```text
2 EC2
 ↓
3 EC2
 ↓
4 EC2
```

Traffic decreases:

```text
4 EC2
 ↓
3 EC2
 ↓
2 EC2
```

The ALB automatically routes traffic to the healthy available targets.

---

# 🧠 Lesson 8 — What You Must Remember

### Load Balancer

> Distributes incoming traffic across backend targets.

### ALB

> AWS Application Load Balancer for HTTP/HTTPS applications.

### Listener

> Receives traffic on a specified port/protocol and applies routing rules.

### Target Group

> Group of backend targets such as EC2 instances.

### Health Check

> Determines whether a target is healthy enough to receive traffic.

### Auto Scaling

> Changes the number of EC2 instances.

The key distinction:

```text
ALB
 ↓
Where should traffic go?

Auto Scaling
 ↓
How many instances should exist?
```

---

# 🧪 Lesson 8 Checkpoint

Answer these before Lesson 9.

### Q1.

Why do we use a Load Balancer with multiple EC2 instances?

### Q2.

What is an Application Load Balancer?

### Q3.

What is a Target Group?

### Q4.

What is a Listener?

### Q5.

What is a Health Check?

### Q6.

What's the difference between an ALB and Auto Scaling?

### Q7.

Complete this:

```text
Users
  ↓
______
  ↓
Listener
  ↓
______
  ↓
EC2 Instances
```

### Q8 — Scenario

You have:

```text
ALB
 │
 ├── EC2 #1 ✅
 ├── EC2 #2 ❌
 └── EC2 #3 ✅
```

EC2 #2 fails its health checks.

What should happen to traffic destined for EC2 #2?

### Q9 — Security Scenario

You have:

```text
Internet
   ↓
ALB
   ↓
EC2
```

Would you generally want the EC2 Security Group to allow the application port from **anywhere on the internet**, or restrict it to traffic from the ALB?

Explain why.

### Q10 — Architecture

Why is this architecture stronger than a single public EC2?

```text
                 Internet
                    ↓
                   ALB
                ↙   ↓   ↘
             EC2   EC2   EC2
```

Give **at least two advantages**.

---

# 🔒 Strict Progression

You've now completed:

```text
✅ Lesson 1 — EC2 Fundamentals
✅ Lesson 2 — Launching Your First EC2 Instance
✅ Lesson 3 — Connecting to EC2
✅ Lesson 4 — EC2 Storage
✅ Lesson 5 — Security Groups & Security
✅ Lesson 6 — EC2 Networking
✅ Lesson 7 — Pricing & Scaling
✅ Lesson 8 — Load Balancing with EC2
```

### Lesson 9 — Monitoring & Operations

```text
CloudWatch
Metrics
Logs
Alarms
Status Checks
Systems Manager
Backups
Troubleshooting
```

### Lesson 10 — Real-World EC2 Architecture

We'll combine **all 10 lessons** into one production-style architecture and walk through how the pieces interact end-to-end.
