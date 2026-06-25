# Bill Analysis Questions

Work through the `sample-bill-data.csv` and answer the following questions. Each question builds your ability to read AWS bills critically and identify waste.

---

## Question 1 (Easy)

**What is the single most expensive line item on the bill, and what service does it belong to?**

Consider why this particular resource costs so much relative to other items.

---

## Question 2 (Easy)

**What is the total monthly bill? Break it down by the top 5 services by cost.**

Group all line items by service and rank them.

---

## Question 3 (Medium)

**Identify at least 3 line items that strongly suggest waste or unnecessary spending.**

For each, explain what makes it wasteful (hint: look for over-provisioning, resources with no active purpose, and mismatched storage tiers).

---

## Question 4 (Medium)

**What is the ratio of compute costs (EC2 + Lambda) to storage costs (S3 + EBS + EBS Snapshots)?**

What does this ratio tell you about the workload profile? Is it compute-heavy, storage-heavy, or balanced?

---

## Question 5 (Medium)

**The NAT Gateway data processing charge is $341.82 for 7,596 GB. What architectural patterns could be driving this cost, and what alternatives exist?**

Think about which services might be generating this egress through the NAT Gateway.

---

## Question 6 (Hard)

**Compare the dev database (RDS Multi-AZ db.r5.large) to the prod database (RDS Single-AZ db.t3.medium). What is wrong with this picture, and what does it suggest about the team's infrastructure management?**

Consider both cost and reliability implications.

---

## Question 7 (Hard)

**The ElastiCache node is a cache.r6g.xlarge (26 GB memory capacity). If the actual cache utilization is approximately 2 GB, calculate:**
- What percentage of the provisioned capacity is being used?
- What instance size would be appropriate?
- What would the monthly savings be?

---

## Question 8 (Medium)

**The S3 app-logs-archive bucket stores 8 TB in Standard storage at $184/month. If this data is accessed less than once per quarter, what storage class transitions would you recommend and what would the new cost be?**

Reference the S3 storage class pricing tiers in your answer.

---

## Question 9 (Hard)

**If you had to cut this bill by 30% in 30 days without any application changes, which items would you target and in what order?**

Prioritize by: savings magnitude, implementation risk, and speed of implementation.

---

## Question 10 (Medium)

**Which costs on this bill represent good value — money well spent relative to what the service delivers?**

Not everything is waste. Identify 2-3 items that appear appropriately sized and reasonably priced for their purpose.
