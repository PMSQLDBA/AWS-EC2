# ☁️ AWS EC2 — Lesson 4/10

## EC2 Storage: EBS, Volumes & Snapshots

In Lesson 3, you learned how to **connect to an EC2 instance**.

Now we need to answer:

> **Where does the EC2 instance store its operating system, applications, and data?**

The main answer is:

# 💾 EBS — Elastic Block Store

---

# 1. EC2 Needs Storage

Think about your laptop.

It has:

```text
CPU
RAM
SSD
```

EC2 is similar:

```text
EC2 Instance
│
├── vCPU
├── Memory
├── Network
└── Storage
```

For persistent storage, AWS commonly uses:

> **EBS — Elastic Block Store**

Think of EBS as a **virtual hard drive attached to your EC2 instance**.

```text
        EC2 Instance
       ┌──────────────┐
       │              │
       │   Ubuntu     │
       │              │
       └──────┬───────┘
              │
              │ attached
              ▼
       ┌──────────────┐
       │ EBS Volume   │
       │              │
       │    20 GB     │
       └──────────────┘
```

---

# 2. What Is an EBS Volume?

An **EBS volume** is a block-storage device that you can attach to an EC2 instance.

For example:

```text id="xkw1uy"
EBS Volume
──────────
Size: 20 GB
Type: SSD-based
```

You can use it for:

* Operating system files
* Applications
* Databases
* Logs
* User files
* Application data

---

# 3. The Root Volume

When you launch an EC2 instance, it normally has a **root volume**.

The root volume contains the operating system.

For example:

```text id="1w1j1n"
EC2
│
└── Root EBS Volume
       │
       ├── Linux OS
       ├── System files
       ├── Installed software
       └── Configuration
```

For a Linux machine, the root filesystem is commonly mounted at:

```text id="3hr3fk"
/ 
```

So if you connect to your EC2 instance and run:

```bash id="f1b8gc"
df -h
```

you can see filesystem/storage information.

---

# 4. Why Is EBS Called "Block Storage"?

Don't let the terminology scare you.

Very simply:

> **Block storage presents storage to the operating system as a block device.**

The operating system can then create a filesystem on it.

Think:

```text id="s9f4vl"
EBS
 ↓
Block device
 ↓
Filesystem
 ↓
Files & directories
```

For example:

```text id="n72p2j"
EBS Volume
     ↓
Filesystem
     ↓
/home
/etc
/var
/app
...
```

You don't normally interact with EBS by thinking about individual AWS "files."

You interact with the filesystem and files **inside the operating system**.

---

# 5. One EC2 Can Have Multiple EBS Volumes

An EC2 instance doesn't have to use only one volume.

You can have:

```text id="w9f1u2"
             EC2
              │
       ┌──────┼──────┐
       │      │      │
       ▼      ▼      ▼
     EBS 1  EBS 2  EBS 3
      20GB   100GB   50GB
```

For example:

```text id="7n1f8a"
EBS 1
→ Operating system

EBS 2
→ Application data

EBS 3
→ Database data
```

This allows you to organize storage according to the workload.

---

# 6. Root Volume vs Additional Volume

Here's the distinction:

### Root volume

Contains the operating system.

```text id="v1h7ub"
Root
 ↓
Linux
```

### Additional EBS volume

Used for additional data.

```text id="g8h8v5"
Additional EBS
 ↓
Application data
```

So:

```text id="q9x9d0"
             EC2
              │
       ┌──────┴──────┐
       ▼             ▼
   Root EBS      Data EBS
       │             │
       ▼             ▼
       OS          App data
```

---

# 7. EBS Is Persistent Storage

This is one of the most important concepts.

Suppose your EC2 instance is running:

```text id="2m9l9k"
EC2
 │
 └── EBS
      │
      └── application-data
```

You stop the EC2 instance.

The EBS volume generally remains.

```text id="1hyj5p"
EC2
 │
 ▼
Stopped

EBS
 │
 ▼
Still exists
```

Then you start the EC2 instance again.

The EBS volume can be attached and used again.

That's why EBS is considered **persistent block storage**.

---

# 8. EBS vs RAM

Don't confuse EBS with RAM.

### RAM

```text
Fast
Temporary
Used by running programs
```

### EBS

```text
Persistent
Storage
Used for files/data
```

Conceptually:

```text id="xw5rbd"
             EC2
              │
       ┌──────┴──────┐
       ▼             ▼
     RAM            EBS
       │             │
    Temporary     Persistent
```

---

# 9. EBS Volume Types

AWS provides different EBS volume types designed for different workloads.

At a high level:

```text id="qqe4a4"
EBS
│
├── General-purpose SSD
│
├── Provisioned IOPS SSD
│
└── HDD-oriented options
```

You don't need to memorize every EBS type yet.

The important principle is:

> **Different volume types provide different performance and cost characteristics.**

For many ordinary workloads, general-purpose SSD storage is a common choice.

For demanding databases or workloads requiring very high I/O performance, you may choose a more specialized type.

---

# 10. Size Matters

When creating an EBS volume, you specify its size.

For example:

```text id="c2l5wm"
20 GiB
100 GiB
500 GiB
```

Larger storage generally means higher cost.

You can also increase an EBS volume's size in many situations without recreating the EC2 instance.

Conceptually:

```text id="kw3h0f"
20 GiB
  ↓
50 GiB
  ↓
100 GiB
```

After changing the EBS volume size, the operating system/filesystem may also need to be expanded so it can use the additional space.

---

# 11. EBS Snapshots

Now we reach another very important concept.

> **EBS Snapshot = point-in-time backup of an EBS volume.**

Imagine:

```text id="x04u0p"
EBS Volume
──────────
Application data
Database files
Configuration
```

You create a snapshot:

```text id="8k3v7z"
       EBS Volume
            │
            │ Snapshot
            ▼
      ┌─────────────┐
      │  Snapshot   │
      │   Backup    │
      └─────────────┘
```

If something goes wrong later, snapshots can help you recover or create new volumes from saved data.

---

# 12. Snapshot → New EBS Volume

A snapshot can be used to create a new EBS volume.

Conceptually:

```text id="2j8n0a"
Original EBS
     │
     ▼
  Snapshot
     │
     ▼
New EBS Volume
```

This is useful for:

* Backups
* Recovery
* Creating copies
* Building environments from a known state

---

# 13. Snapshot vs AMI

This is a common beginner confusion.

### Snapshot

Generally represents a point-in-time backup of an **EBS volume**.

```text
EBS
 ↓
Snapshot
```

### AMI

A template used to launch an **EC2 instance**.

```text
AMI
 ↓
EC2
```

So:

```text id="x2c7eg"
EBS Volume
    │
    └──► Snapshot


AMI
    │
    └──► EC2 Instance
```

An AMI can include information about the volumes needed for the instance, but don't think of **AMI and Snapshot as interchangeable terms**.

---

# 14. What Is Instance Store?

Now we need to distinguish EBS from **instance store**.

Instance store is storage physically associated with the host running the EC2 instance.

Conceptually:

```text id="j4p0h2"
EC2 Host
│
├── Compute
│
└── Instance Store
```

The key characteristic is:

> **Instance store is temporary/ephemeral storage.**

If the instance undergoes certain lifecycle events, data on instance store can be lost.

Therefore, you should **not treat instance store as your primary persistent data store**.

---

# 15. EBS vs Instance Store

This is important enough to memorize.

|                              | EBS                               | Instance Store                                  |
| ---------------------------- | --------------------------------- | ----------------------------------------------- |
| Storage type                 | Persistent block storage          | Temporary/ephemeral                             |
| Independent storage resource | Yes                               | No                                              |
| Common use                   | OS, applications, persistent data | Temporary/high-speed data                       |
| Data durability              | Designed for persistence          | Data can be lost with instance lifecycle events |

Think:

```text id="qz1kn6"
EBS
 ↓
"I want this data to persist."

Instance Store
 ↓
"I can recreate/lose this data."
```

---

# 16. A Real Example

Imagine you're running a web application.

```text id="c1d1qw"
                 EC2
                  │
          ┌───────┴────────┐
          │                │
          ▼                ▼
      Root EBS          Data EBS
       20 GB             100 GB
          │                │
          ▼                ▼
       Ubuntu          App Data
```

You could create snapshots of the important EBS data:

```text id="bq5wjj"
Data EBS
   │
   ├── Snapshot 1
   ├── Snapshot 2
   └── Snapshot 3
```

This gives you a recovery mechanism.

---

# 17. The Complete Storage Mental Model

You should now see:

```text id="1ik8ga"
                       EC2
                        │
             ┌──────────┴──────────┐
             │                     │
             ▼                     ▼
         Root EBS              Data EBS
             │                     │
             ▼                     ▼
        Operating System       Application Data
                                   │
                                   ▼
                              Snapshots
```

And separately:

```text id="4d7n9p"
EC2
 │
 └── Instance Store
        │
        └── Temporary data
```

---

# 🧠 Lesson 4 — What You Must Remember

### 1️⃣ EBS

> Persistent block storage for EC2.

### 2️⃣ Root EBS volume

> Usually contains the operating system.

### 3️⃣ Additional EBS volume

> Can hold application/data files.

### 4️⃣ Snapshot

> Point-in-time backup of an EBS volume.

### 5️⃣ Instance Store

> Temporary/ephemeral storage associated with the EC2 host.

### 6️⃣ RAM ≠ EBS

```text
RAM → temporary working memory

EBS → persistent storage
```

---

# 🧪 Lesson 4 Checkpoint

Answer these before Lesson 5.

### Q1.

What does EBS stand for?

### Q2.

What is an EBS volume?

### Q3.

What normally lives on the root EBS volume?

### Q4.

Can one EC2 instance have multiple EBS volumes?

### Q5.

What is an EBS snapshot?

### Q6.

What's the difference between:

```text
Snapshot
vs
AMI
```

### Q7.

What's the major difference between:

```text
EBS
vs
Instance Store
```

### Q8 — Scenario

You have:

```text
EC2
 │
 └── EBS
      │
      └── Important customer data
```

You accidentally delete important data.

Why might an **EBS snapshot** help?

### Q9 — Think Carefully

Your EC2 instance is stopped.

What happens conceptually to:

**A)** RAM contents
**B)** EBS data
**C)** Instance-store data

Explain each one.

---

# 🔒 Strict Progression

We've now covered:

```text
Lesson 1 → EC2 Fundamentals
Lesson 2 → Launching EC2
Lesson 3 → Connecting to EC2
Lesson 4 → EC2 Storage       ← YOU ARE HERE
```

Next:

# Lesson 5 — EC2 Security Groups & Security

We'll go strictly into:

```text
Security Group
      │
      ├── Inbound
      ├── Outbound
      ├── Ports
      ├── IP ranges
      ├── SSH
      ├── HTTP
      ├── HTTPS
      └── Security best practices
```

This is one of the **most important EC2 lessons**, because a running server that is improperly exposed can become a serious security problem.
