# Well-Architected Framework Review Worksheet

Use this worksheet to review the `sample-stack.yaml` CloudFormation template against the AWS Well-Architected Framework pillars. For each question, document the current state of the architecture and assign a risk level.

---

## Operational Excellence

### Question 1: How do you understand the health of your workload?

**Pillar:** Operational Excellence

**Current State:**
_[Document what monitoring, logging, and alerting exists in the stack]_

**Risk Level:** [ ] High  [ ] Medium  [ ] Low

**Hint:** Look for CloudWatch alarms, dashboards, health check configurations, and log aggregation. A well-architected workload has automated alerts for key metrics (CPU, memory, error rates, latency) and centralized logging.

---

### Question 2: How do you reduce defects, ease remediation, and improve flow into production?

**Pillar:** Operational Excellence

**Current State:**
_[Document how changes would be deployed and rolled back]_

**Risk Level:** [ ] High  [ ] Medium  [ ] Low

**Hint:** Look for deployment configuration (rolling updates, blue/green), rollback mechanisms in the ASG, and whether infrastructure is defined as code with version control. The stack should support safe, incremental deployments.

---

## Security

### Question 3: How do you manage identities and permissions for people and machines?

**Pillar:** Security

**Current State:**
_[Document IAM roles, instance profiles, and permission boundaries]_

**Risk Level:** [ ] High  [ ] Medium  [ ] Low

**Hint:** Every compute resource should have an IAM role with least-privilege permissions. Look for instance profiles on EC2, and check whether the database credentials are managed through Secrets Manager rather than plain-text parameters.

---

### Question 4: How do you protect your network resources?

**Pillar:** Security

**Current State:**
_[Document security group rules, NACLs, and network segmentation]_

**Risk Level:** [ ] High  [ ] Medium  [ ] Low

**Hint:** Security groups should follow least-privilege. SSH should never be open to 0.0.0.0/0 in production. Web servers should only accept traffic from the load balancer. Check for overly permissive ingress rules.

---

## Reliability

### Question 5: How does your workload withstand component failures?

**Pillar:** Reliability

**Current State:**
_[Document redundancy, failover, and fault isolation mechanisms]_

**Risk Level:** [ ] High  [ ] Medium  [ ] Low

**Hint:** Check whether critical components (database, compute) are deployed across multiple Availability Zones. A single-AZ database is a single point of failure. The ASG should span at least 2 AZs with a minimum capacity greater than 1 for production workloads.

---

### Question 6: How do you back up data?

**Pillar:** Reliability

**Current State:**
_[Document backup strategy, retention periods, and recovery testing]_

**Risk Level:** [ ] High  [ ] Medium  [ ] Low

**Hint:** RDS should have automated backups enabled with an appropriate retention period (7+ days for production). Check `BackupRetentionPeriod`. Consider whether point-in-time recovery is available and whether backup restoration has been tested.

---

## Performance Efficiency

### Question 7: How do you select the best performing architecture?

**Pillar:** Performance Efficiency

**Current State:**
_[Document instance sizing rationale and scaling policies]_

**Risk Level:** [ ] High  [ ] Medium  [ ] Low

**Hint:** The ASG should have scaling policies based on metrics (CPU, request count). Without scaling policies, the group won't respond to load changes. Also check whether instance types are appropriate and whether the ALB has connection draining configured.

---

### Question 8: How do you monitor your resources to ensure they are performing as expected?

**Pillar:** Performance Efficiency

**Current State:**
_[Document performance monitoring and benchmarking]_

**Risk Level:** [ ] High  [ ] Medium  [ ] Low

**Hint:** Look for CloudWatch alarms on latency, target response time, and unhealthy host count on the ALB. The architecture should have defined performance baselines and alerts when thresholds are breached.

---

## Cost Optimization

### Question 9: How do you manage demand and supply of resources?

**Pillar:** Cost Optimization

**Current State:**
_[Document how the architecture matches capacity to demand]_

**Risk Level:** [ ] High  [ ] Medium  [ ] Low

**Hint:** The ASG should scale in during low traffic, not just scale out. Check whether there are scale-in policies and whether the minimum capacity is set appropriately for the environment. Dev environments shouldn't run at production scale.

---

### Question 10: How do you evaluate new services and features?

**Pillar:** Cost Optimization

**Current State:**
_[Document whether the architecture uses cost-effective service choices]_

**Risk Level:** [ ] High  [ ] Medium  [ ] Low

**Hint:** Consider whether the workload could benefit from newer options: Graviton instances (better price/performance), Aurora Serverless (pay-per-query for variable loads), or Application Load Balancer features like weighted target groups. Also check for resources that could use Reserved Instances or Savings Plans.

---

## Summary

| Pillar | Questions Reviewed | High Risk | Medium Risk | Low Risk |
|--------|-------------------|-----------|-------------|----------|
| Operational Excellence | 2 | | | |
| Security | 2 | | | |
| Reliability | 2 | | | |
| Performance Efficiency | 2 | | | |
| Cost Optimization | 2 | | | |

**Overall Assessment:**
_[Summarize the top 3 risks and recommended priority actions]_

1. _[Risk 1]_
2. _[Risk 2]_
3. _[Risk 3]_
