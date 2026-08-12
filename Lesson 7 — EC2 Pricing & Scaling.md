# ☁️ AWS EC2 — Lesson 7/10

## EC2 Pricing & Scaling

So far:

```text
✅ Lesson 1 — EC2 Fundamentals
✅ Lesson 2 — Launching EC2
✅ Lesson 3 — Connecting to EC2
✅ Lesson 4 — EC2 Storage
✅ Lesson 5 — Security Groups
✅ Lesson 6 — Networking
➡️ Lesson 7 — Pricing & Scaling
```

This lesson has **two goals**:

1. Understand **how EC2 costs money**
2. Understand **how EC2 can scale when demand changes**

---

# Part A — EC2 Pricing 💰

## 1. Why Does EC2 Cost Money?

When you launch EC2, you're consuming AWS resources.

At a high level, your bill can involve:

```text
EC2
├── Compute
├── EBS storage
├── Data transfer
└── Other related resources
```

The most important beginner concept is:

> **Running an EC2 instance generally incurs compute charges based on the instance and pricing model you choose.**

And the total AWS bill isn't necessarily just "EC2 instance price."

For example:

```text
EC2 Instance
     +
EBS
     +
Data Transfer
     +
Other AWS Services
     ↓
Total Cost
```

---

# 2. On-Demand Pricing

The simplest pricing model is:

> **On-Demand**

You pay for compute capacity without making a long-term commitment.

Conceptually:

```text
Launch EC2
    ↓
Use EC2
    ↓
Pay for usage
```

This is useful when:

* You're learning AWS
* You have unpredictable workloads
* You don't want a long-term commitment
* You need flexibility

### Mental model

```text
On-Demand
     ↓
Maximum flexibility
     ↓
Usually higher effective cost
```

---

# 3. Reserved Instances

Reserved Instances are designed for workloads where you expect to use certain EC2 capacity consistently for a longer period.

Conceptually:

```text
Long-term commitment
        ↓
Potential discount
        ↓
Less flexibility than On-Demand
```

For example, imagine:

```text
Production application
      ↓
Runs continuously
      ↓
Predictable workload
      ↓
Long-term commitment may make sense
```

The important lesson isn't memorizing discount percentages.

It's understanding:

> **Commitment can reduce cost when your usage is predictable.**

---

# 4. Savings Plans

Savings Plans are another way to reduce compute costs through a commitment.

The basic idea:

```text
Commit to a certain level of usage/spend
              ↓
Receive discounted rates
```

Compared with simply buying a particular EC2 configuration, Savings Plans can provide flexibility across eligible compute usage depending on the plan.

For now, remember:

```text
On-Demand
→ Flexible, no long-term commitment

Reserved Instances
→ Commitment around defined capacity/configuration

Savings Plans
→ Commitment to eligible compute usage
```

---

# 5. Spot Instances

Now we have a very different model.

**Spot Instances** let you use spare AWS compute capacity at potentially large discounts compared with On-Demand.

But there's a tradeoff:

> **The capacity can be interrupted by AWS.**

Therefore, Spot is best suited for workloads that can tolerate interruptions.

Examples:

```text
Good candidates
├── Batch processing
├── Large-scale data processing
├── Fault-tolerant workloads
└── Other interruptible jobs
```

Poor candidate:

```text
Critical single server
that must stay available continuously
```

Mental model:

```text
Spot
 ↓
Cheap
 +
Interruptible
```

---

# 6. Pricing Models — Comparison

| Model                  | Flexibility        | Commitment                 | Interruption risk          |
| ---------------------- | ------------------ | -------------------------- | -------------------------- |
| **On-Demand**          | High               | None                       | No Spot-style interruption |
| **Reserved Instances** | Lower              | Long-term                  | No Spot-style interruption |
| **Savings Plans**      | Relatively high    | Long-term usage commitment | No Spot-style interruption |
| **Spot**               | Workload-dependent | None in the same sense     | **Yes**                    |

The exact pricing and terms depend on the AWS service, region, instance configuration, and pricing option.

---

# Part B — Cost Optimization 💰

Now let's talk about controlling EC2 costs.

## 7. Right-Sizing

Suppose your application uses:

```text
CPU: 5%
RAM: 10%
```

but you've selected a very large instance.

You're paying for resources you're barely using.

That's called **overprovisioning**.

Instead:

```text
Too large
    ↓
Analyze utilization
    ↓
Choose appropriate size
    ↓
Lower cost
```

This is called:

> **Right-sizing**

---

# 8. Don't Leave Unused Resources Running

A very common beginner mistake:

```text
Launch EC2
    ↓
Experiment
    ↓
Forget about it
    ↓
Instance keeps running
    ↓
💰 Charges continue
```

For resources you don't need, consider appropriate lifecycle actions.

For example:

```text
Development instance
        ↓
Finished working
        ↓
Stop it when appropriate
```

Remember that stopping an EC2 instance does **not necessarily eliminate every charge**.

For example, attached EBS storage can continue to incur charges.

---

# 9. Scaling

Now let's move from **cost** to **capacity**.

Imagine your website normally gets:

```text
100 users
```

One EC2 instance might be enough.

Then traffic increases:

```text
100
 ↓
1,000
 ↓
10,000
 ↓
100,000 users
```

One server may no longer be enough.

You need to **scale**.

There are two fundamental approaches.

---

# 10. Vertical Scaling

Vertical scaling means:

> **Make the existing EC2 instance bigger.**

For example:

```text
Before:

EC2
2 vCPU
4 GB RAM


After:

EC2
8 vCPU
32 GB RAM
```

Conceptually:

```text
Small EC2
    ↓
Larger EC2
```

This is called:

> **Scale Up**

or

> **Scale Down** when reducing capacity.

---

# 11. Horizontal Scaling

Horizontal scaling means:

> **Add more EC2 instances.**

Instead of:

```text
        EC2
```

you have:

```text
     ┌───────┐
     │ EC2 1 │
     └───────┘

     ┌───────┐
     │ EC2 2 │
     └───────┘

     ┌───────┐
     │ EC2 3 │
     └───────┘
```

This is called:

> **Scale Out**

And reducing the number is:

> **Scale In**

---

# 12. Vertical vs Horizontal

|                 | Vertical         | Horizontal                                 |
| --------------- | ---------------- | ------------------------------------------ |
| Meaning         | Bigger instance  | More instances                             |
| Also called     | Scale up         | Scale out                                  |
| Example         | 2 → 8 vCPU       | 2 → 5 EC2s                                 |
| Main advantage  | Simple           | Better distributed capacity                |
| Main limitation | Hardware ceiling | Application must handle multiple instances |

---

# 13. Why Horizontal Scaling Is Powerful

Suppose:

```text
               Users
                 │
                 ▼
              EC2 #1
```

If EC2 #1 fails:

```text
Users
  │
  X
EC2 #1
```

Your application can become unavailable.

But with multiple instances:

```text
              Users
                │
        ┌───────┴───────┐
        ▼               ▼
     EC2 #1           EC2 #2
```

one instance can fail while another continues serving traffic.

This introduces **availability** as well as capacity.

We'll use this architecture in Lesson 8.

---

# 14. What Is Auto Scaling?

Now imagine traffic changes automatically:

```text
9 AM
↓
2 EC2 instances

12 PM
↓
5 EC2 instances

6 PM
↓
3 EC2 instances

2 AM
↓
1 EC2 instance
```

You don't want a human manually changing this all day.

That's where **EC2 Auto Scaling** comes in.

> **Auto Scaling can automatically adjust the number of EC2 instances according to defined policies and conditions.**

Conceptually:

```text
                Auto Scaling
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
       Scale Out              Scale In
          │                     │
          ▼                     ▼
     Add instances         Remove instances
```

---

# 15. Auto Scaling Group

The main concept is an:

> **Auto Scaling Group (ASG)**

An ASG manages a group of EC2 instances.

For example:

```text
Auto Scaling Group
│
├── EC2 #1
├── EC2 #2
└── EC2 #3
```

You can define desired capacity.

For example:

```text
Minimum = 2
Desired = 2
Maximum = 5
```

Meaning conceptually:

```text
Never below 2
      │
      ▼
Normal target: 2
      │
      ▼
Can grow up to 5
```

---

# 16. Minimum, Desired, Maximum

This is extremely important.

Suppose:

```text id="j6dbr6"
Minimum = 2
Desired = 3
Maximum = 6
```

The Auto Scaling Group tries to maintain:

```text
At least:     2
Desired:      3
Maximum:      6
```

So under normal circumstances:

```text
3 EC2 instances
```

If demand increases:

```text
3 → 4 → 5 → 6
```

If demand decreases:

```text
6 → 5 → 4 → 3
```

But it won't intentionally go below the configured minimum through normal scaling behavior.

---

# 17. Scaling Policies

How does Auto Scaling know when to add/remove instances?

You can use **scaling policies** based on metrics or other conditions.

For example:

```text
CPU utilization
       ↓
Too high?
       ↓
Scale out
```

And:

```text
CPU utilization
       ↓
Low for a sustained period?
       ↓
Scale in
```

Conceptually:

```text
CPU > threshold
       │
       ▼
   Add EC2
```

or:

```text
CPU < threshold
       │
       ▼
   Remove EC2
```

In real production systems, you often choose metrics that actually reflect application demand rather than blindly relying on CPU alone.

---

# 18. Auto Scaling Doesn't Mean "Free"

Important!

Suppose Auto Scaling creates:

```text
1 EC2
 ↓
2 EC2
 ↓
5 EC2
```

You are now using more compute capacity.

Therefore:

```text
More instances
      ↓
More capacity
      ↓
Potentially higher cost
```

Auto Scaling helps optimize capacity, but it does **not** magically eliminate cost.

---

# 19. Auto Scaling + Cost Optimization

Here's where the concepts connect.

Imagine:

```text
Low traffic
   ↓
2 EC2
   ↓
Lower compute usage


High traffic
   ↓
5 EC2
   ↓
Higher capacity
```

Then:

```text
Traffic decreases
   ↓
Scale in
   ↓
2 EC2
```

This avoids permanently running five instances when you only need two.

---

# 20. The Production Mental Model

A common architecture is:

```text
                 USERS
                   │
                   ▼
              Load Balancer
                   │
          ┌────────┼────────┐
          ▼        ▼        ▼
        EC2      EC2      EC2
          └────────┼────────┘
                   │
             Auto Scaling
```

The Load Balancer distributes requests.

The Auto Scaling Group adjusts the number of EC2 instances.

We'll study the Load Balancer properly in **Lesson 8**.

---

# 21. Pricing + Scaling Together

Now combine both halves of this lesson.

### Pricing asks:

> **How much does my compute cost?**

### Scaling asks:

> **How much compute capacity do I need right now?**

Together:

```text
              APPLICATION
                   │
             Demand changes
                   │
                   ▼
              Auto Scaling
                   │
          ┌────────┴────────┐
          ▼                 ▼
      Scale Out          Scale In
          │                 │
          ▼                 ▼
     More EC2s          Fewer EC2s
          │                 │
          └────────┬────────┘
                   ▼
                  COST
```

---

# 🧠 Lesson 7 — What You Must Remember

### Pricing

```text
On-Demand
→ Flexible, no long-term commitment

Reserved Instances
→ Long-term commitment

Savings Plans
→ Long-term compute usage commitment

Spot
→ Spare capacity + potential savings + interruption risk
```

### Scaling

```text
Vertical
→ Bigger EC2

Horizontal
→ More EC2s
```

### Auto Scaling

```text
Auto Scaling Group
       │
       ├── Minimum
       ├── Desired
       └── Maximum
```

And:

```text
High demand
   ↓
Scale Out

Low demand
   ↓
Scale In
```

---

# 🧪 Lesson 7 Checkpoint

Answer these before Lesson 8.

### Q1.

What is On-Demand pricing?

### Q2.

What is the basic idea behind Reserved Instances?

### Q3.

What is a Savings Plan?

### Q4.

Why are Spot Instances usually unsuitable for workloads that cannot tolerate interruption?

### Q5.

What's the difference between:

```text
Vertical scaling
vs
Horizontal scaling
```

### Q6.

What is an Auto Scaling Group?

### Q7.

What do these mean?

```text
Minimum = 2
Desired = 3
Maximum = 6
```

### Q8.

Your application has:

```text
2 EC2 instances
```

Traffic suddenly becomes much higher.

Should you generally:

**A)** Make one instance infinitely larger
**B)** Consider scaling out by adding instances
**C)** Delete the instances
**D)** Remove the Security Group

### Q9 — Scenario

Your application has low traffic overnight and high traffic during the day.

Why might **Auto Scaling** be better than permanently running a large number of EC2 instances?

### Q10 — Think Carefully

You stop an EC2 instance to save money.

Does that necessarily mean your AWS bill for that instance becomes **$0**?

Explain why or why not.

---

# 🔒 Strict Progression

You have now completed:

```text
✅ Lesson 1 — EC2 Fundamentals
✅ Lesson 2 — Launching Your First EC2 Instance
✅ Lesson 3 — Connecting to EC2
✅ Lesson 4 — EC2 Storage
✅ Lesson 5 — Security Groups & Security
✅ Lesson 6 — EC2 Networking
✅ Lesson 7 — Pricing & Scaling
```

Next:

# ⚖️ Lesson 8 — Load Balancing with EC2

We'll build:

```text
                  USERS
                    │
                    ▼
             ┌─────────────┐
             │     ALB     │
             └──────┬──────┘
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
        EC2 #1    EC2 #2    EC2 #3
          │         │         │
          └─────────┼─────────┘
                    │
             Auto Scaling
```

You'll learn **why multiple EC2 instances need a Load Balancer, what a Target Group is, how health checks work, and how ALB + EC2 + Auto Scaling fit together.**
