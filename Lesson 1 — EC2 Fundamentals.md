# 🚀 Lesson 1 — EC2 Fundamentals

**EC2 = Elastic Compute Cloud.**

In simple terms:

> **EC2 lets you rent virtual computers from AWS and run your applications on them.**

Imagine you want to run a website.

You could buy a physical server:

```text
You
 │
 └── Buy physical server
       │
       ├── CPU
       ├── RAM
       ├── Disk
       ├── Network
       └── Electricity
```

That's expensive and inconvenient.

With EC2:

```text
You
 │
 └── AWS
      │
      └── EC2
           │
           └── Virtual Server
                 │
                 ├── CPU
                 ├── RAM
                 ├── Storage
                 └── Network
```

AWS owns and operates the underlying physical infrastructure.

You operate the **virtual server**.

### 1. What is an EC2 Instance?

An **EC2 instance** is essentially a virtual machine running in AWS.

For example:

```text
EC2 Instance
────────────
CPU:     2 vCPU
RAM:     4 GB
Disk:    30 GB
OS:      Linux
Network: Private/Public networking
```

You can install software on it just like you would on a normal computer.

For example:

```bash
sudo apt update
sudo apt install nginx
```

Now your EC2 instance can run a web server.

---

### 2. What does "Elastic" mean?

**Elastic** means you can adjust your computing capacity according to your needs.

Suppose your website normally needs:

```text
2 EC2 instances
```

But during a huge traffic spike:

```text
2 → 10 EC2 instances
```

And later:

```text
10 → 2 EC2 instances
```

That's the basic idea behind **elasticity**.

---

### 3. EC2 Isn't Just One Type of Computer

AWS offers many instance types.

Conceptually:

```text
General Purpose
      │
      ├── Balanced CPU + RAM
      │
      ▼
Compute Optimized
      │
      └── More CPU-focused

Memory Optimized
      │
      └── More RAM-focused

Storage Optimized
      │
      └── High storage performance

Accelerated Computing
      │
      └── GPUs / specialized hardware
```

So you choose an instance based on what your workload needs.

---

### 4. What is an AMI?

When you launch an EC2 instance, you need an operating-system image.

AWS calls this an **AMI — Amazon Machine Image**.

Think:

```text
AMI
 │
 ├── Operating System
 ├── Configuration
 └── Software/image information
```

For example:

```text
Amazon Linux AMI
Ubuntu AMI
Windows Server AMI
```

You can think of an AMI as a **template used to create an EC2 instance**.

So:

```text
AMI
 │
 │ Launch
 ▼
EC2 Instance
```

One AMI can be used to launch multiple instances.

---

### 5. Region and Availability Zone

Your EC2 instance doesn't exist in some mysterious "AWS cloud."

It exists in a specific **AWS Region**, and within that region, in an **Availability Zone (AZ)**.

Conceptually:

```text
AWS Region
│
├── Availability Zone A
│      ├── EC2
│      └── EC2
│
├── Availability Zone B
│      ├── EC2
│      └── EC2
│
└── Availability Zone C
       └── EC2
```

A **Region** is a geographic AWS location.

An **Availability Zone** is an isolated location within an AWS Region.

For example, a production application might run EC2 instances across multiple AZs to improve availability.

---

## 🧠 The 5 Things You Must Know From Lesson 1

Memorize this relationship:

```text
                 AWS Region
                     │
                     ▼
              Availability Zone
                     │
                     ▼
                EC2 Instance
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
         CPU        RAM       Storage
```

And:

```text
AMI ──────► Template
              │
              ▼
        EC2 Instance
              │
              ▼
       Running virtual server
```

### Lesson 1 Checkpoint

Answer these **without looking back**:

1. What does EC2 stand for?
2. What is an EC2 instance?
3. What is an AMI?
4. What's the difference between a Region and an Availability Zone?
5. Why is EC2 called "Elastic"?
