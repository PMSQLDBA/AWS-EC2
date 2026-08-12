# ☁️ AWS EC2 — Lesson 6/10

## EC2 Networking

We've covered:

```text
✅ Lesson 1 — EC2 Fundamentals
✅ Lesson 2 — Launching EC2
✅ Lesson 3 — Connecting to EC2
✅ Lesson 4 — EC2 Storage
✅ Lesson 5 — Security Groups
➡️ Lesson 6 — EC2 Networking
```

This is a **very important lesson** because EC2 networking connects almost everything you've learned so far.

---

# 1. The Big Picture

An EC2 instance doesn't simply "connect to the internet."

It exists inside an AWS network.

The basic hierarchy is:

```text
AWS Region
    │
    ▼
Availability Zone
    │
    ▼
VPC
    │
    ▼
Subnet
    │
    ▼
EC2
```

And traffic flows through networking components such as:

```text
Route Table
Internet Gateway
NAT Gateway
Security Group
```

Our goal is to understand how they fit together.

---

# 2. What Is a VPC?

**VPC = Virtual Private Cloud**

A VPC is your logically isolated network inside AWS.

Think of it as:

> **Your own network environment inside AWS.**

For example:

```text
AWS
│
└── VPC
     │
     ├── Subnet A
     │    └── EC2
     │
     └── Subnet B
          └── EC2
```

You control important networking aspects such as:

* IP address ranges
* Subnets
* Routing
* Internet connectivity
* Network security

---

# 3. VPC CIDR

When creating a VPC, you define an IP address range.

For example:

```text
10.0.0.0/16
```

This is written using **CIDR notation**.

You can think of:

```text
10.0.0.0/16
```

as defining the overall address space available to your VPC.

For this lesson, don't worry about calculating CIDR ranges manually.

Just understand:

> **VPC CIDR = the IP address range allocated to your VPC.**

---

# 4. What Is a Subnet?

A **subnet** is a smaller network inside your VPC.

Imagine:

```text
VPC
10.0.0.0/16
│
├── Subnet A
│   10.0.1.0/24
│
└── Subnet B
    10.0.2.0/24
```

So:

```text
VPC
 ↓
Subnet
 ↓
EC2
```

An EC2 instance is launched into a subnet.

---

# 5. Public vs Private Subnet

This is one of the most important networking concepts.

### Public subnet

A subnet is considered **public** when its route table has a route to an Internet Gateway.

Conceptually:

```text
Public Subnet
     │
     ▼
Internet Gateway
     │
     ▼
Internet
```

### Private subnet

A private subnet does not have a direct route to an Internet Gateway for internet access.

Conceptually:

```text
Private Subnet
     │
     └── No direct internet route
```

Remember:

> **Public/private is primarily about routing, not simply whether an EC2 instance has a public IP.**

---

# 6. Public EC2 Architecture

Suppose you want a web server accessible from the internet.

A simplified architecture:

```text
                INTERNET
                    │
                    ▼
            Internet Gateway
                    │
                    ▼
              Public Subnet
                    │
                    ▼
                   EC2
```

The EC2 instance can have:

```text
Private IP
+
Public IPv4 address
```

The route table provides the path toward the Internet Gateway.

---

# 7. Private EC2 Architecture

Now suppose you have a database server.

You don't want the database directly exposed to the internet.

You could place it in a private subnet:

```text
             INTERNET
                 │
                 ▼
           Public Subnet
                 │
              Web EC2
                 │
                 ▼
           Private Subnet
                 │
                 ▼
             Database
```

The database has no direct internet-facing route.

This is a fundamental cloud architecture pattern.

---

# 8. Route Tables

Now we reach one of the most important components.

A **route table** tells AWS:

> **Where should network traffic go?**

Think of it like a road map.

For example:

```text
Destination       Target
────────────────────────────
10.0.0.0/16       local
0.0.0.0/0         Internet Gateway
```

The meaning is roughly:

```text
10.0.0.0/16
     ↓
Keep traffic inside the VPC

0.0.0.0/0
     ↓
Send other IPv4 traffic toward the Internet Gateway
```

---

# 9. The `0.0.0.0/0` Route

You've already seen:

```text
0.0.0.0/0
```

in Security Groups.

Here it has a different meaning.

In a route table:

```text
0.0.0.0/0 → Internet Gateway
```

means:

> For destinations not matching a more specific IPv4 route, send traffic to the Internet Gateway.

So don't confuse:

```text
Security Group:
0.0.0.0/0
→ source from anywhere

Route Table:
0.0.0.0/0
→ default IPv4 destination route
```

Same CIDR notation, different context.

---

# 10. Internet Gateway

An **Internet Gateway (IGW)** provides a path between a VPC and the internet.

Conceptually:

```text
Internet
    │
    ▼
Internet Gateway
    │
    ▼
VPC
```

For an EC2 instance to have direct internet connectivity, the VPC needs an Internet Gateway and appropriate routing.

A typical public subnet looks like:

```text
Internet
    │
    ▼
Internet Gateway
    │
    ▼
Route Table
    │
    ▼
Public Subnet
    │
    ▼
EC2
```

---

# 11. Important: IGW Alone Isn't Enough

This is a common beginner mistake.

Having an Internet Gateway attached to your VPC does **not automatically make every EC2 instance internet-accessible**.

You also need appropriate:

* Subnet routing
* IP addressing
* Security Group rules
* Network configuration

Think:

```text
Internet Gateway
       +
Route
       +
Appropriate IP configuration
       +
Security Group
       ↓
Internet connectivity
```

---

# 12. Public IP vs Private IP

Let's connect this to Lesson 3.

Your EC2 might have:

```text
Private IP:
10.0.1.25

Public IPv4:
18.x.x.x
```

The private IP is used inside the VPC.

The public address enables internet-facing connectivity when the network configuration permits it.

Conceptually:

```text
Internet
   │
   │ Public addressing
   ▼
EC2
   │
   │ Private addressing
   ▼
VPC
```

---

# 13. NAT Gateway

Now let's solve a common problem.

Suppose you have a private EC2 instance.

It needs to download updates from the internet.

But you don't want the instance to be directly reachable from the internet.

What do you do?

Use a **NAT Gateway**.

NAT stands for:

> **Network Address Translation**

Conceptually:

```text
Private EC2
    │
    │ Outbound request
    ▼
NAT Gateway
    │
    ▼
Internet Gateway
    │
    ▼
Internet
```

The private instance can initiate outbound internet connections without being directly exposed as an internet-facing server.

---

# 14. Public vs Private Subnet Architecture

A very common architecture is:

```text
                         INTERNET
                            │
                            ▼
                    ┌───────────────┐
                    │ Internet      │
                    │ Gateway       │
                    └───────┬───────┘
                            │
                ┌───────────┴───────────┐
                │                       │
                ▼                       ▼
         Public Subnet            NAT Gateway
                │                       │
                ▼                       ▼
           Web EC2              Private Subnet
                                        │
                                        ▼
                                   App EC2
```

The public subnet contains internet-facing components.

The private subnet contains internal components.

---

# 15. Why Use Private Subnets?

Suppose you have:

```text
Web Server
Application Server
Database
```

You generally don't want all three directly exposed to the internet.

Instead:

```text
Internet
   │
   ▼
Public Layer
   │
   ▼
Private Application Layer
   │
   ▼
Private Database Layer
```

This reduces the attack surface.

---

# 16. Route Tables + Subnets

Here's a subtle but important concept:

> **A route table is associated with a subnet.**

For example:

```text
VPC
│
├── Public Subnet
│      │
│      └── Public Route Table
│              │
│              └── 0.0.0.0/0 → IGW
│
└── Private Subnet
       │
       └── Private Route Table
               │
               └── 0.0.0.0/0 → NAT Gateway
```

Therefore:

```text
Public Subnet
→ Internet Gateway

Private Subnet
→ NAT Gateway
```

This is a simplified but extremely useful mental model.

---

# 17. Security Group Still Matters

Don't forget Lesson 5.

Even if the network path exists:

```text
Internet
 ↓
IGW
 ↓
Route Table
 ↓
Subnet
 ↓
EC2
```

the Security Group can still block traffic.

So think of networking as multiple layers:

```text
Can the network route the traffic?
          +
Does the security configuration allow it?
          ↓
        Access
```

---

# 18. Network Interface

Every EC2 instance communicates through a network interface called an:

> **ENI — Elastic Network Interface**

Conceptually:

```text
EC2
 │
 ▼
ENI
 │
 ├── Private IP
 ├── Security Groups
 └── Network connectivity
```

You don't need to manage ENIs manually for basic EC2 usage.

But you should know:

> **The network interface is the networking component attached to the EC2 instance.**

---

# 19. Availability Zones

Remember from Lesson 1:

```text
Region
│
├── AZ-A
├── AZ-B
└── AZ-C
```

Subnets belong to a specific Availability Zone.

For example:

```text
VPC
│
├── Subnet A
│    └── AZ-A
│
├── Subnet B
│    └── AZ-B
│
└── Subnet C
     └── AZ-C
```

This becomes extremely important for high availability.

For example:

```text
              Region
          ┌──────┴──────┐
          ▼             ▼
        AZ-A           AZ-B
          │             │
        EC2 #1         EC2 #2
```

If one Availability Zone has a problem, another can continue serving traffic.

We'll use this heavily in Lessons 7–10.

---

# 20. Complete EC2 Networking Picture

Now put everything together:

```text
                           INTERNET
                              │
                              ▼
                     Internet Gateway
                              │
                  ┌───────────┴───────────┐
                  │                       │
                  ▼                       │
            Public Route Table            │
                  │                       │
                  ▼                       │
            Public Subnet                 │
                  │                       │
                  ▼                       │
             Public EC2                  NAT
                                          │
                                          ▼
                                   Private Route Table
                                          │
                                          ▼
                                   Private Subnet
                                          │
                                          ▼
                                      Private EC2
```

And security:

```text
                    Security Groups
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
        Public EC2                 Private EC2
```

---

# 21. Public EC2 — Traffic Flow

Suppose someone visits your website.

```text
User Browser
     │
     │ HTTPS
     ▼
Internet
     │
     ▼
Internet Gateway
     │
     ▼
Route Table
     │
     ▼
Public Subnet
     │
     ▼
Security Group
     │
     ▼
EC2
```

That's the basic path.

---

# 22. Private EC2 — Outbound Traffic Flow

Suppose a private EC2 instance needs to download a package.

```text
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

The important point:

> The private EC2 does not need a public IPv4 address to initiate this outbound connection through a NAT Gateway.

---

# 23. Public vs Private — Memorize This

### Public subnet

```text
Route:
0.0.0.0/0 → Internet Gateway
```

### Private subnet with outbound internet access

```text
Route:
0.0.0.0/0 → NAT Gateway
```

### Private subnet without internet access

```text
No internet default route
```

---

# 🧠 Lesson 6 — The Core Mental Model

Memorize this hierarchy:

```text
AWS Region
    ↓
Availability Zone
    ↓
VPC
    ↓
Subnet
    ↓
Route Table
    ↓
Network Interface
    ↓
EC2
```

And memorize the internet paths:

### Public:

```text
EC2
 ↓
Public Subnet
 ↓
Route Table
 ↓
Internet Gateway
 ↓
Internet
```

### Private outbound:

```text
Private EC2
 ↓
Private Route Table
 ↓
NAT Gateway
 ↓
Internet Gateway
 ↓
Internet
```

---

# 🧪 Lesson 6 Checkpoint

Answer these before Lesson 7.

### Q1.

What does VPC stand for?

### Q2.

What is a subnet?

### Q3.

What's the difference between a public subnet and a private subnet?

### Q4.

What does a route table do?

### Q5.

What is an Internet Gateway?

### Q6.

What problem does a NAT Gateway solve?

### Q7.

Where does a subnet exist?

A) Across multiple Regions
B) Inside a VPC and within one Availability Zone
C) Outside the VPC
D) Inside a Security Group

### Q8.

What's the basic route for a public EC2 instance?

```text
EC2
 ↓
________
 ↓
________
 ↓
Internet
```

### Q9.

What's the basic outbound route for a private EC2 instance?

```text
Private EC2
 ↓
________
 ↓
NAT Gateway
 ↓
________
 ↓
Internet
```

### Q10 — Scenario

You have:

```text
Internet
   │
   ▼
Web EC2
   │
   ▼
Database EC2
```

You want users on the internet to reach the **Web EC2**, but you do **not** want the Database EC2 directly accessible from the internet.

Where would you place each?

```text
Web EC2       → ?
Database EC2  → ?
```

And why?

---

# 🔒 Strict Progression

You've now covered:

```text
✅ Lesson 1 — EC2 Fundamentals
✅ Lesson 2 — Launching Your First EC2 Instance
✅ Lesson 3 — Connecting to EC2
✅ Lesson 4 — EC2 Storage
✅ Lesson 5 — Security Groups & Security
✅ Lesson 6 — EC2 Networking
```

Next:

# 📈 Lesson 7 — EC2 Pricing & Scaling

We'll cover:

```text
On-Demand
Reserved Instances
Savings Plans
Spot Instances
Instance sizing
Cost optimization
Auto Scaling
```

The goal will be to understand **how EC2 costs money and how AWS can automatically increase or decrease the number of EC2 instances based on demand.**
