# ☁️ AWS EC2 — Lesson 9/10

## Monitoring & Operations

We've reached:

```text
✅ Lesson 1 — EC2 Fundamentals
✅ Lesson 2 — Launching EC2
✅ Lesson 3 — Connecting to EC2
✅ Lesson 4 — EC2 Storage
✅ Lesson 5 — Security Groups
✅ Lesson 6 — Networking
✅ Lesson 7 — Pricing & Scaling
✅ Lesson 8 — Load Balancing
➡️ Lesson 9 — Monitoring & Operations
```

Today we answer a very practical question:

> **How do you know whether your EC2 instances are healthy, what they're doing, and when something goes wrong?**

---

# 1. Why Monitoring Matters

Imagine you have:

```text
                USERS
                  │
                  ▼
                 ALB
                  │
          ┌───────┼───────┐
          ▼       ▼       ▼
        EC2 #1  EC2 #2  EC2 #3
```

Everything looks fine.

But then:

```text
EC2 #2
CPU → 95%
Memory → ???
Disk → ???
Application → ???
```

You need visibility.

Without monitoring:

```text
Something breaks
      ↓
You find out later
      ↓
Users complain
```

With monitoring:

```text
Metric changes
      ↓
Alarm triggers
      ↓
You investigate
      ↓
Fix problem
```

---

# 2. AWS CloudWatch

The most important AWS monitoring service for this lesson is:

> **Amazon CloudWatch**

CloudWatch lets you monitor AWS resources and applications using things such as:

* Metrics
* Logs
* Alarms
* Dashboards
* Events / automated responses

For EC2, think:

```text
EC2
 │
 ├── Metrics
 ├── Logs
 └── Alarms
       ↓
   CloudWatch
```

---

# 3. CloudWatch Metrics

A **metric** is a numerical measurement over time.

For example:

```text
CPU Utilization
```

might look conceptually like:

```text
CPU %
100 |                 ╭───╮
 80 |        ╭────────╯   ╰─
 60 |───────╯
 40 |
 20 |
    └────────────────────────
              Time
```

This allows you to see how the EC2 instance is behaving.

---

# 4. Important EC2 Metrics

Common EC2 metrics include:

### CPU Utilization

```text
How much CPU is being used?
```

For example:

```text
CPU = 85%
```

could indicate heavy CPU usage.

---

### Network In

```text
How much network traffic is entering?
```

---

### Network Out

```text
How much network traffic is leaving?
```

---

### Disk-related metrics

This requires an important distinction.

CloudWatch provides certain instance metrics by default, but **guest OS-level memory and filesystem utilization aren't automatically available as standard EC2 metrics in the same way CPU/network metrics are**.

For deeper OS-level visibility, you can install/configure the CloudWatch Agent.

---

# 5. CPU Utilization

Suppose your EC2 instance normally runs at:

```text
CPU
30%
35%
32%
38%
```

Then suddenly:

```text
95%
98%
99%
```

Something may be happening.

Possibilities include:

```text
Heavy traffic
Application problem
CPU-intensive process
Unexpected workload
```

Monitoring doesn't automatically tell you *why*.

It tells you:

> **Something changed.**

You then investigate.

---

# 6. CloudWatch Logs

Metrics answer:

> **What is happening numerically?**

Logs can help answer:

> **What actually happened?**

For example, your application might produce:

```text
2026-08-11 10:30:02 INFO Request received
2026-08-11 10:30:03 INFO User authenticated
2026-08-11 10:30:05 ERROR Database connection failed
```

These are logs.

CloudWatch Logs can collect and centralize logs from applications and systems when configured appropriately.

---

# 7. Metrics vs Logs

Memorize this distinction:

```text
METRIC
  ↓
Number / measurement
  ↓
"CPU is 92%"


LOG
  ↓
Record / event
  ↓
"Database connection failed"
```

Think:

```text
Metrics → How much? How often?
Logs    → What happened?
```

---

# 8. CloudWatch Alarms

Now we make monitoring useful.

Suppose:

```text
CPU > 80%
```

You don't want to stare at the CloudWatch dashboard all day.

You can create a:

> **CloudWatch Alarm**

Conceptually:

```text
CPU
 │
 │ 80%
 ▼
Threshold
 │
 ▼
Alarm
 │
 ├── Notification
 └── Automated action
```

For example:

```text
IF CPU > 80%
FOR a defined period
THEN trigger alarm
```

---

# 9. Alarm States

A CloudWatch alarm can have states such as:

```text
OK
ALARM
INSUFFICIENT_DATA
```

### OK

The metric is within the configured condition.

```text
CPU = 40%
     ↓
OK
```

### ALARM

The configured threshold condition has been met.

```text
CPU = 95%
     ↓
ALARM
```

### INSUFFICIENT_DATA

There isn't enough data to determine the state.

---

# 10. Alarm Notifications

An alarm can trigger notifications through AWS services such as SNS.

Conceptually:

```text
EC2
 │
 ▼
CloudWatch Metric
 │
 ▼
CloudWatch Alarm
 │
 ▼
SNS
 │
 ▼
Notification
```

For example:

```text
CPU too high
     ↓
Alarm
     ↓
Notification
     ↓
Administrator investigates
```

---

# 11. Monitoring + Auto Scaling

Now connect Lesson 7.

Suppose:

```text
CPU > threshold
```

Instead of simply notifying someone, the scaling system can use appropriate metrics/policies to increase capacity.

Conceptually:

```text
High demand
    ↓
Metric
    ↓
Scaling policy
    ↓
Auto Scaling
    ↓
More EC2 instances
```

So:

```text
Monitoring
    +
Scaling
    ↓
Responsive infrastructure
```

---

# 12. EC2 Status Checks

CloudWatch isn't the only thing you need to know.

EC2 provides **status checks**.

These help determine whether an instance and its underlying system are functioning as expected.

There are two important categories to understand:

```text
System status checks
Instance status checks
```

---

# 13. System Status Check

A **system status check** generally checks the underlying AWS infrastructure supporting the instance.

For example, the problem could involve:

```text
Underlying host
Networking
AWS infrastructure
```

Conceptually:

```text
AWS Infrastructure
       │
       ▼
System Status Check
       │
       ▼
EC2
```

If there's an underlying infrastructure issue, the system status check may fail.

---

# 14. Instance Status Check

An **instance status check** focuses on the instance itself.

For example:

```text
Operating system
Networking configuration
Instance-level problems
```

Conceptually:

```text
EC2 Instance
     │
     ▼
Instance Status Check
```

So remember:

```text
System check
→ Underlying AWS system

Instance check
→ EC2 instance itself
```

---

# 15. Why Status Checks Matter

Imagine:

```text
EC2
 │
 ├── Application?
 ├── OS?
 ├── Network?
 └── Infrastructure?
```

A status check gives you another signal.

For example:

```text
Application problem
→ Application logs

CPU problem
→ CloudWatch metric

Underlying infrastructure issue
→ System status check
```

Different tools answer different questions.

---

# 16. EC2 Console Monitoring

The EC2 console provides useful information about an instance.

You can inspect things such as:

```text
Instance state
Status checks
Monitoring
Networking
Storage
Security
```

For example:

```text
Instance
   │
   ├── Running
   ├── Status checks
   ├── Monitoring
   ├── Network
   └── Storage
```

The console is useful for quickly understanding the state of an instance.

---

# 17. Systems Manager

Another important AWS operations service is:

> **AWS Systems Manager**

It helps you manage AWS resources and applications.

One particularly useful capability is:

> **Session Manager**

Session Manager can provide shell access to supported EC2 instances without requiring you to expose SSH to the internet.

Conceptually:

```text
Administrator
      │
      ▼
Systems Manager
      │
      ▼
    EC2
```

This can reduce the need for:

```text
Internet
   │
   ▼
SSH :22
   │
   ▼
EC2
```

for administrative access.

---

# 18. Session Manager vs SSH

Traditional approach:

```text
Your Computer
     │
     │ SSH :22
     ▼
Security Group
     │
     ▼
EC2
```

Session Manager approach:

```text
Administrator
     │
     ▼
Systems Manager
     │
     ▼
EC2
```

This doesn't mean SSH is bad.

It means AWS provides another operational mechanism that can be useful for reducing exposed management access.

---

# 19. CloudWatch Agent

Here's another important distinction.

Basic EC2 monitoring doesn't automatically provide every operating-system metric you'd want.

Suppose you want:

```text
Memory utilization
Disk space
Detailed OS metrics
Application-specific metrics
```

You can use the:

> **CloudWatch Agent**

Conceptually:

```text
EC2
 │
 │ CloudWatch Agent
 ▼
CloudWatch
 │
 ├── Metrics
 └── Logs
```

The agent runs on the operating system and can collect additional information.

---

# 20. Monitoring a Web Server

Imagine:

```text
                  ALB
                   │
          ┌────────┼────────┐
          ▼        ▼        ▼
        EC2 #1   EC2 #2   EC2 #3
```

You could monitor:

```text
ALB
├── Request count
├── Response behavior
└── Target health

EC2
├── CPU
├── Network
├── Status checks
└── OS/application metrics
```

Then:

```text
CloudWatch
     │
     ├── Metrics
     ├── Logs
     └── Alarms
```

This gives you a much better picture of the whole application.

---

# 21. Troubleshooting Method

Now let's say your application is unavailable.

Don't randomly change things.

Use a structured approach.

### Step 1 — Is EC2 running?

```text
Instance state
     ↓
Running?
```

---

### Step 2 — Are status checks healthy?

```text
Status checks
     ↓
Passed?
```

---

### Step 3 — Is networking working?

Check:

```text
VPC
Subnet
Route table
Security Group
```

---

### Step 4 — Is the application running?

On the instance:

```text
Web server?
Application process?
Service?
```

---

### Step 5 — Check logs

Look at:

```text
Application logs
System logs
CloudWatch Logs
```

---

### Step 6 — Check metrics

Look for:

```text
CPU
Network
Disk-related metrics
Memory, if configured
```

---

### Step 7 — Check the ALB

If you're using one:

```text
ALB
 ↓
Target Group
 ↓
Health Check
```

Is the target healthy?

---

# 22. A Troubleshooting Tree

Memorize this:

```text
                APPLICATION DOWN
                       │
                       ▼
                Is EC2 running?
                  /          \
                NO            YES
                │              │
             Fix state         ▼
                         Status checks OK?
                           /          \
                         NO            YES
                         │              │
                  Investigate      ▼
                              Network OK?
                                /      \
                              NO        YES
                              │          │
                           Network      ▼
                           config    App running?
                                      /      \
                                    NO        YES
                                    │          │
                                  Fix app    Check logs/
                                             ALB/health
```

This is far better than guessing.

---

# 23. Backups Are Part of Operations

Monitoring tells you:

> **Something is wrong.**

But what if data is lost?

You also need:

> **Backup and recovery strategy.**

For EC2 environments, this can involve services and mechanisms such as:

```text
EBS Snapshots
AMI-based recovery
AWS Backup
Application/database backups
```

Don't confuse:

```text
Monitoring
≠
Backup
```

Monitoring detects problems.

Backups help you recover from them.

---

# 24. Monitoring vs Logging vs Backup

Memorize:

```text
Monitoring
→ Is something wrong?

Logging
→ What happened?

Backup
→ Can I recover?
```

And:

```text
Alarm
→ Tell me when something crosses a condition.
```

---

# 25. Operations Mindset

Running EC2 isn't just:

```text
Launch instance
     ↓
Done
```

Real operations look like:

```text
Deploy
  ↓
Monitor
  ↓
Detect
  ↓
Investigate
  ↓
Fix
  ↓
Recover
  ↓
Improve
```

That's the difference between simply **creating infrastructure** and **operating infrastructure**.

---

# 26. Complete EC2 Operations Picture

Let's combine everything we've learned.

```text
                         USERS
                           │
                           ▼
                          ALB
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
            EC2 #1       EC2 #2       EC2 #3
              │            │            │
              └────────────┼────────────┘
                           │
                    Auto Scaling
                           │
                           ▼
                       CloudWatch
                      /     |      \
                 Metrics   Logs    Alarms
```

And for administration:

```text
Administrator
      │
      ▼
Systems Manager
      │
      ▼
EC2
```

For recovery:

```text
EC2 / EBS
   │
   ▼
Backup / Snapshot
   │
   ▼
Recovery
```

---

# 🧠 Lesson 9 — What You Must Remember

### CloudWatch

> AWS monitoring and observability service.

### Metrics

> Numerical measurements over time.

```text
CPU = 72%
```

### Logs

> Records of events and activity.

```text
ERROR: Database connection failed
```

### Alarms

> Evaluate metrics against configured conditions and can trigger actions/notifications.

### Status Checks

```text
System status
→ Underlying AWS infrastructure

Instance status
→ Instance-level health
```

### Systems Manager

> AWS service for managing resources; Session Manager can provide administrative access without requiring traditional inbound SSH access.

### CloudWatch Agent

> Collects additional OS/application metrics and logs from the instance when configured.

### Backups

> Help you recover data/resources after failures or mistakes.

---

# 🧪 Lesson 9 Checkpoint

Answer these before Lesson 10.

### Q1.

What is Amazon CloudWatch?

### Q2.

What's the difference between a **metric** and a **log**?

### Q3.

What is a CloudWatch Alarm?

### Q4.

What is the difference between:

```text
System status check
vs
Instance status check
```

### Q5.

What is the CloudWatch Agent used for?

### Q6.

What is AWS Systems Manager?

### Q7.

What is Session Manager useful for?

### Q8.

Your EC2 instance is running, but your application isn't responding.

Name **at least five things** you would investigate.

### Q9.

Your CPU suddenly reaches:

```text
95%
```

What AWS service could monitor this and potentially alert you?

### Q10 — Architecture

Explain the difference between:

```text
CloudWatch
Auto Scaling
ALB
Systems Manager
```

using **one sentence for each**.

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
✅ Lesson 8 — Load Balancing
✅ Lesson 9 — Monitoring & Operations
```

# 🏁 Next: Lesson 10 — Real-World EC2 Architecture

This is the **final lesson**.

We'll bring everything together:

```text
                         USERS
                           │
                           ▼
                         ALB
                           │
                  ┌────────┴────────┐
                  ▼                 ▼
             Public Layer      Private Layer
                  │                 │
                  ▼                 ▼
                 EC2              EC2
                  │                 │
                  └────────┬────────┘
                           │
                     Auto Scaling
                           │
                      CloudWatch
                           │
                    Systems Manager
                           │
                       Backups
```

You'll learn how **VPC + Subnets + Security Groups + EC2 + ALB + Auto Scaling + CloudWatch + Operations** fit together into a realistic AWS architecture.
