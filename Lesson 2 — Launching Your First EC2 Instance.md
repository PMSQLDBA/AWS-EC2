# ☁️ AWS EC2 — Lesson 2/10

## Launching Your First EC2 Instance

In Lesson 1, we learned:

> **AMI → EC2 Instance → Virtual Server**

Now we're going to actually understand **how an EC2 instance is launched**.

---

# 1. The EC2 Launch Process

When you click **Launch instance** in AWS, you're essentially answering several questions:

```text
What OS?
   ↓
What size?
   ↓
How will I connect?
   ↓
What network?
   ↓
What storage?
   ↓
What traffic is allowed?
   ↓
🚀 Launch EC2
```

The major decisions are:

1. **AMI**
2. **Instance type**
3. **Key pair**
4. **Network settings**
5. **Security group**
6. **Storage**

Let's understand each one.

---

# 2. Step 1 — Choose an AMI

Remember:

> **AMI = template used to create your EC2 instance.**

When launching an instance, AWS asks:

**Which operating system do you want?**

For example:

```text
AMI
├── Amazon Linux
├── Ubuntu
├── Windows Server
└── Other supported images
```

Suppose you select:

```text
Ubuntu
```

AWS uses that image as the starting point for your EC2 server.

Conceptually:

```text
Ubuntu AMI
     │
     │ Launch
     ▼
┌───────────────┐
│ EC2 Instance  │
│               │
│ Ubuntu Linux  │
└───────────────┘
```

### Important distinction

**AMI ≠ EC2 instance**

Think:

> **AMI = blueprint**
> **EC2 instance = running machine created from that blueprint**

---

# 3. Step 2 — Choose an Instance Type

Now AWS asks:

> **How powerful should this virtual computer be?**

This is where you choose the **instance type**.

An instance type determines resources such as:

* CPU
* Memory
* Network performance
* Other hardware characteristics

For example, conceptually:

```text
Small instance
───────────────
CPU:  small
RAM:  small
Cost: lower


Large instance
───────────────
CPU:  larger
RAM:  larger
Cost: higher
```

So choosing an instance type is basically choosing the **hardware profile** of your virtual machine.

### General-purpose example

A general-purpose instance is often a good starting point for ordinary workloads.

Later, we'll study the different EC2 families in more detail.

**Don't memorize instance-type names yet.**

For Lesson 2, understand the principle:

> **Instance type = the compute resources allocated to your EC2 instance.**

---

# 4. Step 3 — Key Pair

This is extremely important.

Suppose you launch a Linux EC2 instance.

You need a secure way to prove:

> "I am authorized to access this server."

AWS can use a **key pair** for this.

Conceptually:

```text
Key Pair
│
├── Public Key
│      ↓
│   EC2 server
│
└── Private Key
       ↓
    You keep it
```

The private key is used when connecting to the instance.

For example, with SSH:

```text
Your computer
     │
     │ SSH + private key
     ▼
EC2 Instance
```

### 🔐 Critical rule

**Never casually share your private key.**

Your private key is sensitive.

If someone obtains it and the server is configured to accept it, they may be able to authenticate as you.

---

# 5. Step 4 — Network Settings

Your EC2 instance needs to exist inside an AWS network.

That network is called a:

> **VPC — Virtual Private Cloud**

For now, think of a VPC as:

> **Your logically isolated network inside AWS.**

Inside the VPC are **subnets**.

```text
VPC
│
├── Subnet A
│     └── EC2
│
└── Subnet B
      └── EC2
```

When launching EC2, AWS asks you which network/subnet the instance should use.

We'll study VPCs deeply in **Lesson 6**.

For now:

> **EC2 lives inside a VPC and subnet.**

---

# 6. Step 5 — Security Group

Now we reach one of the most important EC2 concepts.

A **Security Group** controls network traffic to and from your EC2 instance.

Imagine your EC2 server is a house:

```text
                 INTERNET
                    │
                    ▼
             ┌─────────────┐
             │  SECURITY   │
             │    GROUP    │
             └──────┬──────┘
                    │
                    ▼
               ┌────────┐
               │  EC2   │
               │ Server │
               └────────┘
```

The security group decides:

> **Which network traffic is allowed?**

For example:

```text
Port 22  → SSH
Port 80  → HTTP
Port 443 → HTTPS
```

You might configure:

```text
Inbound
──────────────
SSH    22   → allowed from your IP
HTTP   80   → allowed
HTTPS  443  → allowed
```

We'll dedicate much more attention to security groups in **Lesson 5**.

For now, remember:

> **Security Group = virtual firewall for your EC2 network traffic.**

---

# 7. Step 6 — Storage

Your EC2 instance needs storage.

For example:

```text
EC2
│
├── CPU
├── RAM
├── Network
└── Storage
```

AWS commonly provides storage through **EBS — Elastic Block Store**.

Think of EBS like a virtual hard drive:

```text
EC2
 │
 └──── EBS Volume
          │
          ├── Operating system
          ├── Applications
          └── Data
```

For example:

```text
Root volume
     │
     └── 20 GB
```

The exact storage configuration depends on the workload.

We'll study EBS properly in **Lesson 4**.

---

# 8. Putting Everything Together

When you launch an EC2 instance, you're essentially creating this:

```text
                    AWS
                     │
                 Region
                     │
              Availability Zone
                     │
                     ▼
                    VPC
                     │
                   Subnet
                     │
                     ▼
             ┌──────────────┐
             │     EC2      │
             │              │
             │   Ubuntu     │
             │              │
             │ CPU + RAM    │
             └──────┬───────┘
                    │
                  EBS
                    │
                 Storage
```

And controlling access:

```text
Internet
    │
    ▼
Security Group
    │
    ▼
   EC2
```

---

# 9. A Real Example

Imagine you're creating a small web server.

You might choose:

```text
AMI
→ Ubuntu

Instance Type
→ General-purpose instance

Key Pair
→ Your SSH key

VPC
→ Your VPC

Subnet
→ A subnet in the selected AZ

Security Group
→ Allow SSH + HTTP

Storage
→ EBS root volume
```

Then:

```text
             Internet
                 │
          ┌──────▼──────┐
          │  Security   │
          │    Group    │
          └──────┬──────┘
                 │
                 ▼
          ┌─────────────┐
          │     EC2     │
          │   Ubuntu    │
          └──────┬──────┘
                 │
                 ▼
               EBS
```

Your EC2 instance is now ready to run your application.

---

# 10. The Most Important Mental Model

Don't memorize the AWS console screens.

Understand the **decisions**:

```text
AMI
 ↓
"What operating system/template?"

Instance Type
 ↓
"How much computing power?"

Key Pair
 ↓
"How will I authenticate?"

VPC/Subnet
 ↓
"Where will this server live?"

Security Group
 ↓
"What traffic is allowed?"

EBS
 ↓
"Where will my data/storage live?"
```

That is the core of launching an EC2 instance.

---

# 🧠 Lesson 2 Checkpoint

Try answering these **without looking back**:

### Q1.

What is the difference between an **AMI** and an **EC2 instance**?

### Q2.

What does an **instance type** determine?

### Q3.

Why do we use an **EC2 key pair**?

### Q4.

What is a **Security Group**?

### Q5.

What is **EBS**?

### Q6.

Where does an EC2 instance live?

A) Only inside an Availability Zone
B) Inside a VPC/subnet within an Availability Zone
C) Directly on the public internet
D) Inside an AMI

### Q7 — Architecture

Complete this:

```text
AMI
 ↓
   ?
 ↓
EC2 Instance
 ↓
   ?
 ↓
Storage
```

Lesson 3 will be **Connecting to EC2 — SSH, public/private IPs, and actually accessing your server.**
