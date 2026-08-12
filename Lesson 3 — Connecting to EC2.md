# ☁️ AWS EC2 — Lesson 3/10

## Connecting to EC2

In Lesson 2, we launched an EC2 instance.

Now we need to answer:

> **“How do I actually get inside my EC2 server and use it?”**

For a Linux EC2 instance, the most common answer is:

# 🔑 SSH

---

# 1. What Is SSH?

**SSH = Secure Shell**

SSH lets you securely connect from your computer to a remote server.

Think of it like:

```text
Your Computer
     │
     │ SSH
     ▼
┌──────────────┐
│ EC2 Instance │
│              │
│ Linux Server │
└──────────────┘
```

Once connected, you can run commands on the EC2 server.

For example:

```bash
ls
```

or:

```bash
pwd
```

or:

```bash
sudo apt update
```

The commands execute **on the EC2 machine**, not your local computer.

---

# 2. What Do You Need to Connect?

For a typical Linux EC2 connection, you need:

```text
1. Running EC2 instance
2. Network connectivity
3. Public/reachable address
4. Security Group allowing SSH
5. Correct username
6. Private key
```

Let's understand each.

---

# 3. Public IP vs Private IP

This is extremely important.

An EC2 instance can have a **private IP address**.

Example:

```text
10.0.1.25
```

That's used for communication inside the AWS network.

It may also have a **public IPv4 address**, such as:

```text
18.x.x.x
```

The public address can allow communication between the instance and the internet, when the surrounding network configuration permits it.

Conceptually:

```text
                 INTERNET
                     │
                     │
                Public IP
                     │
                     ▼
              ┌─────────────┐
              │     EC2     │
              │             │
              │ Private IP  │
              │ 10.0.1.25   │
              └─────────────┘
```

### Remember

**Public IP → external connectivity**

**Private IP → internal VPC connectivity**

We'll explore the networking details in Lesson 6.

---

# 4. Security Group Must Allow SSH

Having a public IP isn't enough.

Your Security Group must allow SSH traffic.

SSH normally uses:

```text
TCP port 22
```

So your security group might contain:

```text
Inbound Rule
──────────────────────
Type: SSH
Protocol: TCP
Port: 22
Source: Your IP
```

Then:

```text
Your Computer
      │
      │ TCP 22
      ▼
Security Group
      │
      ▼
    EC2
```

If port 22 isn't allowed, the connection can fail.

---

# 5. Why Should You Restrict SSH?

You could technically allow:

```text
SSH
22
0.0.0.0/0
```

which means:

> Allow SSH from anywhere on the IPv4 internet.

That's generally **not a good security practice** unless you have a specific reason and additional protections.

Prefer restricting SSH access to trusted sources where practical.

For example:

```text
SSH
Port 22
Source:
Your public IP / trusted network
```

We'll go deeper into security in **Lesson 5**.

---

# 6. Your Private Key

Remember the key pair from Lesson 2?

You downloaded/stored your private key.

For example:

```text
my-ec2-key.pem
```

Conceptually:

```text
Your Computer
│
├── my-ec2-key.pem
│
└─────────────── SSH ───────────────► EC2
```

The private key proves that you possess the credential corresponding to the public key configured for the instance.

### 🔐 Important

Treat the private key like a password **plus** a cryptographic credential.

Don't:

```text
❌ Email it to people
❌ Put it in GitHub
❌ Upload it publicly
❌ Paste it into random websites
```

---

# 7. The SSH Command

A typical SSH command looks like:

```bash
ssh -i my-ec2-key.pem username@PUBLIC_IP
```

For example, conceptually:

```bash
ssh -i my-ec2-key.pem ubuntu@18.x.x.x
```

Break it down:

```text
ssh
 │
 ├── -i my-ec2-key.pem
 │      │
 │      └── private key
 │
 └── ubuntu@18.x.x.x
       │       │
       │       └── EC2 public IP
       │
       └──────── username
```

---

# 8. The Username Matters

The SSH username depends on the AMI.

For example, an Ubuntu image commonly uses:

```text
ubuntu
```

An Amazon Linux image commonly uses:

```text
ec2-user
```

So:

```text
Ubuntu
   ↓
ubuntu@IP

Amazon Linux
   ↓
ec2-user@IP
```

Using the wrong username can cause authentication failure even if everything else is configured correctly.

---

# 9. What Happens During SSH?

When you run:

```bash
ssh -i my-ec2-key.pem ubuntu@PUBLIC_IP
```

Think about the journey:

```text
Your Computer
      │
      │ 1. SSH connection
      ▼
Internet
      │
      │ 2. Port 22
      ▼
Security Group
      │
      │ 3. Traffic allowed?
      ▼
EC2 Instance
      │
      │ 4. Authenticate
      ▼
Linux Shell
```

If authentication succeeds:

```text
ubuntu@ip-10-0-1-25:~$
```

🎉

You're now inside your EC2 server.

---

# 10. Local Computer vs EC2

This distinction is extremely important for beginners.

Suppose your computer has:

```text
Your Computer
C:\Users\You>
```

And you connect to EC2:

```text
ubuntu@ip-10-0-1-25:~$
```

Now:

```bash
pwd
```

runs **on EC2**.

You can verify where you are with:

```bash
hostname
```

You might see an EC2 hostname.

The mental model is:

```text
┌──────────────────┐
│ YOUR COMPUTER    │
│                  │
│ SSH Client       │
└────────┬─────────┘
         │
         │ SSH
         ▼
┌──────────────────┐
│ EC2 INSTANCE     │
│                  │
│ Linux            │
│ SSH Server       │
└──────────────────┘
```

---

# 11. What If SSH Doesn't Work?

Don't panic. 😄

EC2 connection problems are usually caused by one of several things.

Think systematically:

```text
SSH failing?
     │
     ├── Is EC2 running?
     │
     ├── Is the address correct?
     │
     ├── Is port 22 allowed?
     │
     ├── Is the source IP allowed?
     │
     ├── Is the correct private key being used?
     │
     ├── Is the username correct?
     │
     └── Is the network route configured correctly?
```

We'll eventually troubleshoot each of these.

---

# 12. Common Errors

### ❌ `Connection timed out`

Often indicates a **network reachability problem**.

Check things like:

```text
Security Group
Public IP
Subnet routing
Internet Gateway
Network ACLs
```

---

### ❌ `Permission denied (publickey)`

Often indicates an **authentication problem**.

Check:

```text
Private key
Username
Correct EC2 instance
Correct key pair
```

---

### ❌ `Could not resolve hostname`

Usually means the hostname/address you're using isn't being resolved correctly.

Check that the address is correct.

---

# 13. Public IP Can Change

This is an important EC2 concept.

A normal automatically assigned public IPv4 address can change when the instance is stopped and started.

So you shouldn't blindly assume:

```text
EC2
↓
Public IP
↓
Permanent forever
```

For situations where you need a persistent public IPv4 address, AWS provides **Elastic IP addresses**.

Conceptually:

```text
Normal public IP
      ↓
Can change

Elastic IP
      ↓
Designed to remain associated with your AWS account
until you release it
```

We'll cover Elastic IPs as part of EC2 networking.

---

# 14. Windows EC2

Everything we've discussed so far has focused on Linux.

Windows EC2 instances commonly use:

> **RDP — Remote Desktop Protocol**

Instead of:

```text
SSH → Linux
```

you commonly have:

```text
RDP → Windows
```

Conceptually:

```text
Your Computer
      │
      │ RDP
      ▼
Windows EC2
```

RDP normally uses:

```text
TCP port 3389
```

Again, the Security Group must permit the required traffic.

---

# 15. SSH vs RDP

|                   | Linux EC2     | Windows EC2         |
| ----------------- | ------------- | ------------------- |
| Common protocol   | SSH           | RDP                 |
| Common port       | 22            | 3389                |
| Typical interface | Terminal      | Graphical desktop   |
| Authentication    | Often SSH key | Windows credentials |

For this course, our primary EC2 examples will use **Linux + SSH**.

---

# 🧠 Lesson 3 Mental Model

Memorize this:

```text
                YOUR COMPUTER
                     │
                     │ SSH
                     │
                Public IP
                     │
                     ▼
              Security Group
                 Port 22
                     │
                     ▼
                EC2 Instance
                     │
                     ▼
                Linux Shell
```

And remember:

```text
Private IP
   ↓
Internal VPC communication

Public IP
   ↓
Internet-facing connectivity

Port 22
   ↓
SSH

Private Key
   ↓
Authentication credential

Security Group
   ↓
Controls allowed network traffic
```

---

# 🧪 Lesson 3 Checkpoint

Answer these before Lesson 4.

### Q1.

What does SSH stand for?

### Q2.

What port does SSH normally use?

### Q3.

What's the difference between a public IP and a private IP?

### Q4.

Why does the Security Group need an inbound rule for SSH?

### Q5.

What is the purpose of the private key?

### Q6.

Why might this fail?

```bash
ssh -i mykey.pem ubuntu@18.x.x.x
```

Give **at least three possible reasons**.

### Q7.

You're using an Ubuntu EC2 instance. Which username is commonly used?

A) `root`
B) `ubuntu`
C) `admin`
D) `ec2-user`

### Q8.

Complete the path:

```text
Your Computer
      ↓
     SSH
      ↓
    Port __
      ↓
Security Group
      ↓
     EC2
      ↓
   Linux Shell
```

### Q9 — Important Scenario

Your EC2 instance says **Running**, but SSH gives:

```text
Connection timed out
```

Would you immediately assume the private key is wrong?

**Why or why not?**

---

## 🔒 Strict Progression

**Lesson 4 — EC2 Storage** comes next:

```text
EC2
 │
 ├── Root Volume
 ├── EBS
 ├── Volumes
 ├── Snapshots
 ├── Volume Types
 └── Instance Store
```

We won't move into Load Balancers, Auto Scaling, or advanced architecture yet.
