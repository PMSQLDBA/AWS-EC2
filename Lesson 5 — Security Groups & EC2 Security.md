# ☁️ AWS EC2 — Lesson 5/10

## EC2 Security Groups & Security

So far:

```text
Lesson 1 → EC2 Fundamentals
Lesson 2 → Launching EC2
Lesson 3 → Connecting to EC2
Lesson 4 → EC2 Storage
Lesson 5 → Security Groups & Security  ← YOU ARE HERE
```

Today we're focusing **strictly on securing EC2 network access**.

---

# 1. What Is a Security Group?

A **Security Group (SG)** is a virtual firewall associated with resources such as EC2 instances.

Its job is to control:

> **What network traffic is allowed to reach the instance and what traffic the instance can send.**

Think:

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
                ┌─────────┐
                │   EC2   │
                └─────────┘
```

The Security Group doesn't contain your application.

It controls **network access** to it.

---

# 2. Firewall Analogy

Imagine your EC2 server is a house.

```text
                 HOUSE
                  │
        ┌─────────┴─────────┐
        │                   │
       Door               Windows
        │                   │
      Access              Access
```

The Security Group is like the security system deciding:

```text
Who can enter?
Which entrance?
Under what conditions?
```

In networking terms:

```text
Who?
 ↓
Source IP / Security Group

Which entrance?
 ↓
Port

What protocol?
 ↓
TCP / UDP / etc.
```

---

# 3. Inbound Traffic

**Inbound = traffic coming toward your EC2 instance.**

For example:

```text
Your Computer
     │
     │ SSH
     ▼
    EC2
```

That's inbound traffic from the EC2 instance's perspective.

A Security Group might contain:

```text
Inbound Rules

SSH
TCP
22
Source: Your IP
```

Meaning:

> Allow SSH traffic on TCP port 22 from the specified source.

---

# 4. Outbound Traffic

**Outbound = traffic leaving your EC2 instance.**

For example:

```text
EC2
 │
 │ HTTPS
 ▼
External service
```

That's outbound traffic.

Security Groups therefore have:

```text
Inbound Rules
     +
Outbound Rules
```

You need to understand both.

---

# 5. Ports

A **port** identifies a network service endpoint.

Think of an IP address as identifying a building:

```text
IP Address
    ↓
Building
```

The port identifies a particular door/service:

```text
IP + Port
     ↓
Specific service
```

Common ports you'll encounter:

|     Port | Common use |
| -------: | ---------- |
|   **22** | SSH        |
|   **80** | HTTP       |
|  **443** | HTTPS      |
| **3389** | RDP        |

---

# 6. SSH — Port 22

Linux EC2 instances commonly use SSH.

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

A rule might be:

```text
Protocol: TCP
Port: 22
Source: Your trusted IP
```

This allows SSH access from that source.

---

# 7. HTTP — Port 80

Suppose your EC2 instance is running a website.

HTTP normally uses:

```text
TCP 80
```

Traffic:

```text
Browser
   │
   │ HTTP
   │ Port 80
   ▼
Security Group
   │
   ▼
EC2 Web Server
```

So the Security Group could allow:

```text
Inbound
TCP
80
Source: Internet
```

---

# 8. HTTPS — Port 443

HTTPS normally uses:

```text
TCP 443
```

A secure website might look like:

```text
Browser
   │
   │ HTTPS
   │ TCP 443
   ▼
Security Group
   │
   ▼
EC2
```

For a public web server, HTTPS traffic commonly needs to be reachable from the internet.

---

# 9. RDP — Port 3389

Windows servers commonly use:

> **RDP — Remote Desktop Protocol**

Default port:

```text
TCP 3389
```

Conceptually:

```text
Your Computer
      │
      │ RDP : 3389
      ▼
Security Group
      │
      ▼
Windows EC2
```

Again, don't expose administrative ports more broadly than necessary.

---

# 10. Source Matters

Consider this rule:

```text
SSH
Port 22
Source: 203.0.113.10/32
```

This means:

> Allow SSH from that specific IPv4 address.

The `/32` represents a single IPv4 address in CIDR notation.

Compare:

```text
203.0.113.10/32
```

with:

```text
0.0.0.0/0
```

The second represents:

> All IPv4 addresses.

So:

```text
Your IP only
      ↓
More restrictive


0.0.0.0/0
      ↓
Anywhere on IPv4 internet
```

---

# 11. The Dangerous Beginner Mistake

Suppose you create:

```text
SSH
Port 22
Source: 0.0.0.0/0
```

You've effectively said:

> Allow SSH connections from anywhere on the IPv4 internet.

That exposes the SSH service broadly.

It's generally better to restrict administrative access to trusted sources where practical.

For example:

```text
SSH
22
Source: Your trusted IP
```

---

# 12. Web Traffic Is Different

A public website is supposed to be accessible by many users.

So you might intentionally allow:

```text
HTTP
80
0.0.0.0/0
```

and/or:

```text
HTTPS
443
0.0.0.0/0
```

The key distinction is:

```text
Public application traffic
        ↓
May need broad access

Administrative traffic
        ↓
Should generally be restricted
```

---

# 13. Security Groups Are Stateful

This is a very important AWS concept.

Security Groups are **stateful**.

Suppose your EC2 instance receives an allowed request:

```text
Client
  │
  │ Request
  ▼
 EC2
```

The response traffic is automatically allowed back for that established connection.

Conceptually:

```text
Client
  │
  │ Request ───────►
  │
  │ ◄──────── Response
  │
```

You don't normally need to create a separate inbound rule just to allow the response to an allowed inbound connection.

---

# 14. Security Groups Are Allow-Based

Another important point:

Security Groups use **allow rules**.

Think:

```text
Allow SSH
Allow HTTPS
Allow HTTP
```

You don't create a Security Group rule saying:

```text
"Deny this IP"
```

as the normal Security Group mechanism.

If traffic doesn't match an allowed rule, it isn't allowed.

Conceptually:

```text
Traffic
   │
   ▼
Matches allow rule?
   │
 ┌─┴───┐
Yes    No
 │      │
 ▼      ▼
Allow  Block
```

---

# 15. Security Group vs Network ACL

Don't confuse these.

For now:

### Security Group

```text
Associated with
EC2/network interfaces
```

### Network ACL

```text
Associated with
subnets
```

Conceptually:

```text
VPC
│
├── Network ACL
│       ↓
│     Subnet
│       ↓
│      EC2
│       ↓
│ Security Group
```

We'll study Network ACLs later when we cover VPC networking.

For this lesson:

> **Security Group = your primary EC2-level network firewall concept.**

---

# 16. Multiple Security Groups

An EC2 instance can have multiple Security Groups associated with it.

Conceptually:

```text
             EC2
              │
       ┌──────┼──────┐
       ▼      ▼      ▼
      SG1    SG2    SG3
```

The effective access is based on the allowed rules across the associated Security Groups.

For example:

```text
SG-Web
 └── Allow HTTPS

SG-Admin
 └── Allow SSH from trusted IP
```

This can help organize access logically.

---

# 17. Security Group Example

Imagine you have a public web server.

You might want:

```text
                INTERNET
                   │
          ┌────────┴────────┐
          │                 │
       HTTPS              SSH
       :443               :22
          │                 │
          ▼                 ▼
       Public           Trusted IP
          │                 │
          └────────┬────────┘
                   ▼
                  EC2
```

Rules:

```text
Inbound:

HTTPS
TCP 443
Source: 0.0.0.0/0

SSH
TCP 22
Source: Trusted IP
```

That's much more sensible than:

```text
SSH
TCP 22
Source: 0.0.0.0/0
```

for a normal administrative setup.

---

# 18. Outbound Example

Suppose your EC2 application needs to call an external API:

```text
EC2
 │
 │ HTTPS :443
 ▼
External API
```

Your outbound rules need to permit the required traffic.

The exact configuration depends on the Security Group's outbound rules and your network setup.

---

# 19. Security Group Mental Model

Memorize this:

```text
                    EC2
                     │
            ┌────────┴────────┐
            │                 │
       INBOUND            OUTBOUND
            │                 │
            ▼                 ▼
      Who can reach?     Where can it go?
            │                 │
            ▼                 ▼
       IP / SG / Port     IP / Port / Protocol
```

---

# 20. A Practical Web Server

Let's imagine an EC2 web server:

```text
                    INTERNET
                       │
              ┌────────┴────────┐
              │                 │
             :443              :22
              │                 │
           Everyone         Trusted IP
              │                 │
              └────────┬────────┘
                       ▼
                  ┌─────────┐
                  │   EC2   │
                  │ Web App │
                  └─────────┘
```

Security Group:

```text
Inbound
──────────────────────────
HTTPS | TCP | 443 | Public
SSH   | TCP | 22  | Trusted IP
```

This follows the basic security principle:

> **Expose only what needs to be exposed.**

---

# 21. Principle of Least Privilege

This is one of the most important security concepts in AWS.

> **Give only the access that is necessary.**

Bad:

```text
SSH
22
Anywhere
```

Better:

```text
SSH
22
Trusted IP only
```

Bad:

```text
Open every port
```

Better:

```text
Only required ports
```

Think:

```text
Minimum access
      ↓
Minimum attack surface
      ↓
Better security
```

---

# 🧠 Lesson 5 — What You Must Remember

### Security Group

> Virtual firewall controlling network traffic associated with an EC2 instance.

### Inbound

```text
Traffic → EC2
```

### Outbound

```text
EC2 → destination
```

### Common ports

```text
22   → SSH
80   → HTTP
443  → HTTPS
3389 → RDP
```

### `0.0.0.0/0`

```text
All IPv4 addresses
```

### Stateful

> Responses to allowed connections are automatically permitted.

### Allow-based

> Security Groups define allowed traffic; unmatched traffic isn't allowed.

### Least privilege

> Allow only the access that is actually needed.

---

# 🧪 Lesson 5 Checkpoint

Answer these **before Lesson 6**.

### Q1.

What is a Security Group?

### Q2.

What's the difference between inbound and outbound traffic?

### Q3.

What is port 22 commonly used for?

### Q4.

What are ports 80 and 443 commonly used for?

### Q5.

What does this mean?

```text
SSH
TCP
22
0.0.0.0/0
```

### Q6.

Why is allowing SSH from `0.0.0.0/0` generally less secure than allowing it only from a trusted IP?

### Q7.

Are Security Groups **stateful or stateless**?

### Q8.

Are Security Group rules primarily **allow rules or deny rules**?

### Q9 — Scenario

Your web server needs:

```text
Public website → HTTPS
Administrator → SSH
```

Which source would you generally want for each?

```text
HTTPS :443 → ?
SSH   :22  → ?
```

### Q10 — Troubleshooting

Your EC2 server is running, but:

```text
SSH → works
HTTPS → doesn't work
```

Give **at least three things** you would investigate.

---

# 🔒 Strict Progression

You've now completed:

```text
✅ Lesson 1 — EC2 Fundamentals
✅ Lesson 2 — Launching Your First EC2 Instance
✅ Lesson 3 — Connecting to EC2
✅ Lesson 4 — EC2 Storage
✅ Lesson 5 — Security Groups & Security
```

Next is:

# 🌐 Lesson 6 — EC2 Networking

We'll build the networking picture from the ground up:

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
Internet Gateway / NAT
    ↓
EC2
```

And you'll finally see **exactly how an EC2 instance communicates with the internet and with other AWS resources.**
