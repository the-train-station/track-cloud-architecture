# Solution Guide

## Bill Summary

Total monthly spend: approximately **$2,698** across 38 line items.

Top services by cost:
1. Amazon EC2: $875
2. Amazon RDS: $464
3. NAT Gateway: $374
4. ElastiCache: $263
5. Amazon S3: $269

---

## Top 5 Cost Improvements

### 1. Right-size the order-api EC2 instance (r5.2xlarge to m5.large)

**The Waste:**
An r5.2xlarge ($487/mo) is a memory-optimized instance with 64 GB RAM, used for a simple order API. The "r" family is designed for in-memory databases and caches, not stateless API services. A typical REST API serving moderate traffic needs 2-4 vCPUs and 8 GB RAM at most.

**Recommended Fix:**
Migrate to m5.large (2 vCPU, 8 GB RAM) or m5.xlarge (4 vCPU, 16 GB RAM) depending on load testing results. Start with m5.xlarge to be conservative.

**Estimated Savings:**
- Current: $487.20/mo (r5.2xlarge)
- Proposed: $140.16/mo (m5.xlarge)
- Savings: **$347.04/mo (71% reduction on this line item)**

**Implementation Steps:**
1. Enable detailed CloudWatch metrics (CPU, memory via CloudWatch Agent) for 1 week
2. Review peak utilization — confirm memory usage is under 16 GB
3. Launch new m5.xlarge with identical AMI/config
4. Shift traffic via target group, monitor for 24 hours
5. Terminate old instance

**Quick Win or Strategic:** Quick win (1-2 days)

**Frugal Architect Principle:** *Law 2: Systems that Last Align Cost to Business* — a simple API does not require memory-optimized hardware. Cost should reflect the actual resource demands of the workload.

---

### 2. Downgrade dev database from Multi-AZ r5.large to Single-AZ t3.medium

**The Waste:**
The dev database runs Multi-AZ db.r5.large at $380/mo. Multi-AZ provides automatic failover for high availability — completely unnecessary for a development environment that likely has zero real users. Meanwhile, the actual production database is a smaller Single-AZ t3.medium ($49/mo), which is an inverted risk posture.

**Recommended Fix:**
Convert the dev database to Single-AZ db.t3.small (or t3.micro if it is lightly used). Separately, evaluate whether prod should be Multi-AZ (but that is a separate discussion about reliability, not cost waste).

**Estimated Savings:**
- Current: $380.16/mo (Multi-AZ db.r5.large) + $23 storage = $403/mo
- Proposed: ~$25/mo (Single-AZ db.t3.small) + $11.50 storage (reduce to 100GB) = ~$37/mo
- Savings: **$366/mo (91% reduction)**

**Implementation Steps:**
1. Confirm this is truly a dev environment (check connections, query volume)
2. Take a manual snapshot as backup
3. Modify instance: disable Multi-AZ, change class to db.t3.small
4. Reduce allocated storage if possible (note: RDS storage can only scale up, so may need to recreate)
5. Update connection strings if endpoint changes

**Quick Win or Strategic:** Quick win (hours, with a brief maintenance window)

**Frugal Architect Principle:** *Law 1: Cost is a Non-Functional Requirement* — dev environments should be explicitly budgeted. Treating them as "just another deployment" of prod architecture leads to paying production prices for zero production value.

---

### 3. Move S3 app-logs-archive to Intelligent-Tiering or Glacier

**The Waste:**
8 TB of log archives in S3 Standard at $184/mo. Log archives are by definition rarely accessed — they exist for compliance and incident investigation. S3 Standard is designed for frequently accessed data.

**Recommended Fix:**
Implement an S3 Lifecycle Policy:
- Objects 0-30 days: Standard (for recent log queries)
- Objects 30-90 days: Standard-IA ($0.0125/GB)
- Objects 90-365 days: Glacier Instant Retrieval ($0.004/GB)
- Objects >365 days: Glacier Deep Archive ($0.00099/GB)

Alternatively, use S3 Intelligent-Tiering ($0.0025/GB monitoring fee) if access patterns are unpredictable.

**Estimated Savings:**
Assuming 80% of the 8 TB is older than 90 days and qualifies for Glacier Instant Retrieval:
- Current: 8 TB Standard = $184/mo
- Proposed: 1.6 TB Standard ($36.80) + 6.4 TB Glacier Instant ($26.21) = ~$63/mo
- Savings: **$121/mo (66% reduction)**

**Implementation Steps:**
1. Analyze object age distribution with S3 Storage Lens or inventory report
2. Create lifecycle policy on the bucket
3. Wait for transitions to complete (happens automatically over 24-48 hours)
4. Verify access patterns are not disrupted (test retrieval of older objects)

**Quick Win or Strategic:** Quick win (30 minutes to configure, automatic execution)

**Frugal Architect Principle:** *Law 5: Unobserved Systems Lead to Unknown Costs* — without lifecycle policies and storage analytics, data accumulates in the most expensive tier indefinitely. Observability of storage usage prevents silent cost growth.

---

### 4. Eliminate unattached EBS volumes

**The Waste:**
Two unattached EBS volumes totaling 700 GB ($56/mo). These are orphaned resources from decommissioned servers — paying for storage that nothing reads or writes to.

**Recommended Fix:**
Snapshot both volumes (for safety), then delete them. If the data is needed for reference, the snapshots are cheaper at $0.05/GB-month vs $0.08/GB-month for provisioned volumes.

**Estimated Savings:**
- Current: $56/mo (700 GB provisioned gp3)
- Proposed: $0/mo (delete) or $35/mo (keep as snapshots only)
- Savings: **$56/mo (100%) or $21/mo (63%)**

**Implementation Steps:**
1. Identify volumes in EC2 console (state: "available" = unattached)
2. Check volume tags/descriptions for ownership context
3. Create snapshots of both volumes
4. Notify the team with a 48-hour warning
5. Delete the volumes

**Quick Win or Strategic:** Quick win (15 minutes)

**Frugal Architect Principle:** *Law 3: Architecting is a Series of Trade-offs* — the tradeoff here is trivial: snapshot storage vs provisioned volume cost for data nobody is using. There is no architectural reason to keep unattached volumes.

---

### 5. Right-size ElastiCache from r6g.xlarge to r6g.large or t4g.medium

**The Waste:**
A cache.r6g.xlarge provides 26.32 GB of memory capacity at $259/mo. If actual cache utilization is ~2 GB, the cluster is running at 7.6% utilization — paying for 24 GB of unused memory.

**Recommended Fix:**
Migrate to cache.t4g.medium (3.09 GB, ~$47/mo) or cache.r6g.large (13.07 GB, ~$130/mo) depending on growth projections. For 2 GB current usage with reasonable headroom, t4g.medium is appropriate.

**Estimated Savings:**
- Current: $259.20/mo (cache.r6g.xlarge)
- Proposed: $47/mo (cache.t4g.medium)
- Savings: **$212/mo (82% reduction)**

**Implementation Steps:**
1. Verify current memory usage via CloudWatch `BytesUsedForCache` and `CurrConnections` metrics over 2 weeks
2. Check for burst patterns that might exceed t4g.medium capacity
3. Create a new cache cluster with the smaller node type
4. Update application connection strings
5. Run in parallel for 24 hours, then decommission old cluster

**Quick Win or Strategic:** Quick win with monitoring (2-3 days for safe migration)

**Frugal Architect Principle:** *Law 4: Uncontested Resources are Wasted Resources* — memory that is provisioned but never used is uncontested capacity providing zero value. Right-sizing ensures you pay for what you actually consume.

---

## Quick Wins vs. Strategic Changes

### Quick Wins (implement this week)

| Improvement | Savings | Effort | Risk |
|-------------|---------|--------|------|
| Delete unattached EBS volumes | $56/mo | 15 minutes | Low |
| S3 lifecycle policy for logs | $121/mo | 30 minutes | Low |
| Downgrade dev RDS | $366/mo | 1-2 hours | Low |

**Total quick wins: $543/mo**

### Short-term Changes (implement within 2 weeks)

| Improvement | Savings | Effort | Risk |
|-------------|---------|--------|------|
| Right-size ElastiCache | $212/mo | 2-3 days | Medium |
| Right-size order-api EC2 | $347/mo | 2-3 days | Medium |

**Total short-term: $559/mo**

### Combined Savings

**Total: $1,102/mo (~41% bill reduction)**

---

## Additional Observations

- **NAT Gateway ($374/mo):** Consider VPC endpoints for S3/DynamoDB traffic, or evaluate whether workloads could run in public subnets with security groups instead of private subnets with NAT. This is a strategic change requiring network architecture review.
- **CloudWatch log retention ($128/mo):** Reduce retention from 12 months to 30-90 days for non-compliance log groups. Export older logs to S3 with lifecycle policies for long-term retention at lower cost.
- **EBS Snapshots ($75/mo):** Review 90-day retention policy. If daily snapshots are needed, consider AWS Backup with lifecycle rules to transition older snapshots or reduce retention to 30 days.
- **Production RDS is under-provisioned:** The prod database (Single-AZ t3.medium) should likely be Multi-AZ for reliability. This is not a cost saving but a risk that should be surfaced to leadership.
