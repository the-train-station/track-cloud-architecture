# AWS Well-Architected vs GCP Cloud Architecture Framework: Comparison Matrix

## Overview

AWS Well-Architected Framework defines **6 pillars** while GCP Cloud Architecture Framework defines **5 pillars**. The frameworks overlap significantly but differ in structure, emphasis, and tooling. This matrix provides a detailed pillar-by-pillar comparison for architects working across both clouds.

### Structural Difference

| Framework | Pillar Count | Review Cadence | Primary Output |
|-----------|-------------|----------------|----------------|
| AWS Well-Architected | 6 pillars | On-demand via Tool | Improvement plan with HRIs/MRIs |
| GCP Cloud Architecture | 5 pillars | Continuous via Architecture Framework checklists | Maturity assessment across phases |

AWS separates **Sustainability** into its own pillar. GCP folds environmental efficiency into **Cost Optimization** and operational guidance. AWS also treats **Operational Excellence** as a distinct pillar, while GCP distributes operational concerns across **Operational Excellence** and **Reliability**.

---

## Pillar-by-Pillar Comparison

### 1. Operational Excellence

| Dimension | AWS (Operational Excellence Pillar) | GCP (Operational Excellence Pillar) |
|-----------|-------------------------------------|-------------------------------------|
| **Scope** | Organization, prepare, operate, evolve. Emphasis on runbooks, playbooks, and operational events. | System design, instrumentation, process automation, and rapid release cycles. |
| **Key Services** | CloudFormation, AWS Config, Systems Manager, CloudWatch, X-Ray, Organizations | Cloud Deployment Manager, Terraform on GCP, Cloud Build, Cloud Monitoring, Cloud Trace, Resource Manager |
| **Review Tooling** | AWS Well-Architected Tool (console), custom lenses | Architecture Framework checklists, Active Assist recommendations |
| **Unique Insights** | Strong emphasis on "operations as code" and organizational learning loops. Defines explicit maturity phases (manual → automated → proactive). | Emphasizes "design for change" with feature flags, canary deployments, and SRE-driven operational models. Tighter integration with SRE principles (error budgets, SLOs). |

### 2. Security

| Dimension | AWS (Security Pillar) | GCP (Security, Privacy & Compliance Pillar) |
|-----------|----------------------|----------------------------------------------|
| **Scope** | Identity, detective controls, infrastructure protection, data protection, incident response. | Identity, access management, data protection, network security, compliance automation. |
| **Key Services** | IAM, Organizations, GuardDuty, Security Hub, KMS, CloudTrail, WAF, Shield, Macie | Cloud IAM, Organization Policies, Security Command Center, Cloud KMS/HSM, Cloud Audit Logs, Cloud Armor, VPC Service Controls, BeyondCorp Enterprise |
| **Review Tooling** | Security Hub (automated checks against CIS/PCI), Trusted Advisor | Security Command Center (Standard/Premium), Security Health Analytics, Forseti (OSS, deprecated in favor of SCC) |
| **Unique Insights** | Mature shared responsibility model documentation. Strong SCPs (Service Control Policies) for preventive guardrails. Well-defined blast radius isolation via account boundaries. | Zero-trust architecture (BeyondCorp) as a first-class product. Confidential Computing (encrypted-in-use). VPC Service Controls create security perimeters without network-level changes. Organization Policy constraints are more granular than AWS SCPs for certain use cases. |

### 3. Reliability

| Dimension | AWS (Reliability Pillar) | GCP (Reliability Pillar) |
|-----------|--------------------------|--------------------------|
| **Scope** | Foundations, workload architecture, change management, failure management. | Designing for high availability, scalability, disaster recovery, and operational resilience. |
| **Key Services** | Route 53, ELB, Auto Scaling, RDS Multi-AZ, S3 cross-region replication, Backup, Resilience Hub | Cloud DNS, Cloud Load Balancing (global), Managed Instance Groups, Cloud Spanner (global), Cloud Storage multi-region, Backup and DR Service |
| **Review Tooling** | AWS Resilience Hub (assesses RTO/RPO), FIS (Fault Injection Simulator) | Chaos engineering via Gremlin integration, GCP-native regional failover testing, Active Assist reliability recommendations |
| **Unique Insights** | Explicit RTO/RPO classification per workload tier. Service quotas management as a reliability concern. Detailed failure isolation using AZs and Regions with explicit guidance on "cell-based architecture." | Global load balancing as default (not regional). Spanner's global strong consistency changes DR calculus. Live Migration of VMs reduces maintenance window impact. Premium Tier networking provides higher reliability at the network layer. |

### 4. Performance Efficiency

| Dimension | AWS (Performance Efficiency Pillar) | GCP (Performance Optimization Pillar) |
|-----------|-------------------------------------|---------------------------------------|
| **Scope** | Selection (compute, storage, database, network), review, monitoring, tradeoffs. | Compute, storage, database, and network resource optimization; leveraging managed services and ML-driven scaling. |
| **Key Services** | EC2 (Graviton), Lambda, EBS/EFS/S3, DynamoDB/Aurora, CloudFront, Global Accelerator | Compute Engine (Tau/C3), Cloud Functions, Persistent Disk/Filestore/Cloud Storage, Bigtable/AlloyDB/Spanner, Cloud CDN, Premium Tier Network |
| **Review Tooling** | Compute Optimizer, Cost Explorer (right-sizing), CloudWatch dashboards | Recommender (Active Assist), Cloud Monitoring dashboards, Committed Use Discount analysis |
| **Unique Insights** | Graviton (ARM) workloads for price-performance. Extensive instance type selection (500+ types). Nitro system for near-bare-metal performance. | Custom machine types (fine-grained vCPU/RAM selection, not limited to predefined sizes). Per-second billing enables tighter right-sizing. Live Migration avoids performance impact during host maintenance. Sole-tenant nodes for compliance without performance penalty. |

### 5. Cost Optimization

| Dimension | AWS (Cost Optimization Pillar) | GCP (Cost Optimization Pillar) |
|-----------|-------------------------------|-------------------------------|
| **Scope** | Cloud financial management, expenditure awareness, cost-effective resources, managing demand/supply, optimizing over time. | Resource optimization, pricing model selection, cost governance, and sustainable efficiency. |
| **Key Services** | Cost Explorer, Budgets, Savings Plans, Reserved Instances, Spot Instances, S3 Intelligent-Tiering | Billing Console, Budget Alerts, Committed Use Discounts (CUDs), Sustained Use Discounts (automatic), Preemptible/Spot VMs, Cloud Storage lifecycle |
| **Review Tooling** | Cost Explorer, Trusted Advisor (cost checks), Compute Optimizer | Billing Reports, Recommender (Active Assist cost recommendations), Cloud Billing export to BigQuery |
| **Unique Insights** | Mature RI/SP marketplace. Granular tagging and cost allocation. Well-defined FinOps partnership model. Organization-level consolidated billing with volume discounts. | Sustained Use Discounts are automatic (no commitment required). CUDs are more flexible (spend-based, not instance-type locked). Per-second billing vs. per-hour minimums. BigQuery on-demand pricing model is uniquely suited for analytics cost control. Flat-rate network egress pricing on Premium Tier simplifies cost modeling. |

### 6. Sustainability (AWS Only)

| Dimension | AWS (Sustainability Pillar) | GCP Equivalent |
|-----------|----------------------------|----------------|
| **Scope** | Region selection, user behavior patterns, software/architecture patterns, data patterns, hardware patterns, development/deployment process. | No dedicated pillar. GCP addresses sustainability via Carbon Footprint dashboard and claims of carbon-neutral operations since 2007. Guidance is embedded in Cost Optimization and general best practices. |
| **Key Services** | Customer Carbon Footprint Tool, Graviton (energy-efficient), S3 Intelligent-Tiering | Carbon Footprint dashboard, low-carbon regions, efficient ML (TPUs vs GPUs) |
| **Review Tooling** | Customer Carbon Footprint dashboard | Carbon Footprint reporting in Billing Console |
| **Unique Insights** | Explicit framework for measuring and reducing workload carbon impact. Provides proxy metrics (compute hours, storage volume) when direct carbon measurement is unavailable. | GCP positions itself as running on carbon-free energy (goal: 24/7 by 2030). Region picker includes carbon intensity data. TPU efficiency arguments for ML workloads. Less prescriptive framework but stronger claims on current carbon neutrality. |

---

## Summary: Framework Strengths

| Area | AWS Wins | GCP Wins |
|------|----------|----------|
| **Maturity of tooling** | Well-Architected Tool provides structured reviews with custom lenses per industry/workload | Active Assist provides continuous, ML-driven recommendations without manual review triggers |
| **Operational guidance** | Deeper prescriptive runbook/playbook patterns | Tighter SRE integration (error budgets, SLOs as first-class concepts) |
| **Security model** | More mature multi-account isolation strategy (Landing Zone) | Zero-trust (BeyondCorp) and VPC Service Controls are more advanced than AWS equivalents |
| **Reliability at scale** | Cell-based architecture guidance, Resilience Hub for RTO/RPO | Global load balancing and Spanner global consistency reduce DR complexity |
| **Cost governance** | More granular tagging, mature RI marketplace, deeper FinOps tooling | Automatic sustained use discounts, per-second billing, spend-based CUDs |
| **Sustainability** | Dedicated pillar with measurement framework | Stronger carbon-neutral claims and carbon-free energy commitment |
| **Community & ecosystem** | Larger partner ecosystem, more third-party lenses | Closer alignment with open-source (Kubernetes, Istio, Knative originated at Google) |

---

## Translation Guide: AWS Concepts to GCP Equivalents

### Infrastructure & Compute

| AWS Concept | GCP Equivalent | Key Behavioral Difference |
|-------------|----------------|---------------------------|
| CloudFormation | Deployment Manager / Terraform | Deployment Manager is YAML-based and less feature-rich than CFN. Terraform is the de facto standard on GCP. |
| EC2 | Compute Engine | GCE offers custom machine types and per-second billing. Live Migration is default. |
| Auto Scaling Groups | Managed Instance Groups (MIGs) | MIGs support rolling updates, canary deployments, and auto-healing natively. |
| ECS/EKS | Cloud Run / GKE | GKE is the original managed Kubernetes. Cloud Run is serverless containers (no direct AWS equivalent — closest is Fargate + App Runner). |
| Lambda | Cloud Functions / Cloud Run | Cloud Functions for event-driven; Cloud Run for container-based serverless with longer timeouts and more runtime flexibility. |
| Elastic Beanstalk | App Engine | App Engine Standard has faster cold starts but more constraints. App Engine Flexible is closer to Beanstalk. |

### Networking

| AWS Concept | GCP Equivalent | Key Behavioral Difference |
|-------------|----------------|---------------------------|
| VPC (regional) | VPC (global) | GCP VPCs are global with regional subnets. No need for VPC peering across regions within the same VPC. |
| ALB/NLB | Cloud Load Balancing | GCP LB is global by default (single anycast IP). No need for Route 53 latency routing to achieve global distribution. |
| Route 53 | Cloud DNS | Cloud DNS is simpler; complex routing policies require Cloud Load Balancing or Traffic Director. |
| Transit Gateway | Cloud Interconnect / VPC Peering | GCP's global VPC reduces the need for transit constructs. Shared VPC is the primary multi-project networking model. |
| PrivateLink | Private Service Connect | Similar functionality; PSC uses Consumer/Producer model with forwarding rules. |
| CloudFront | Cloud CDN | Cloud CDN integrates directly with Cloud Load Balancing (no separate distribution concept). |
| WAF | Cloud Armor | Cloud Armor attaches to Load Balancing backends. Supports Adaptive Protection (ML-based). |

### Data & Storage

| AWS Concept | GCP Equivalent | Key Behavioral Difference |
|-------------|----------------|---------------------------|
| S3 | Cloud Storage | Nearly equivalent. GCP uses flat namespace with "/" convention. Uniform bucket-level access replaces ACLs. |
| RDS | Cloud SQL | Cloud SQL supports MySQL, PostgreSQL, SQL Server. AlloyDB is PostgreSQL-compatible with Spanner-like performance. |
| DynamoDB | Firestore / Bigtable | Firestore for document model (smaller scale). Bigtable for wide-column (massive scale, no secondary indexes). |
| Aurora | AlloyDB / Cloud Spanner | AlloyDB for PostgreSQL compatibility. Spanner for global strong consistency with horizontal scaling. |
| ElastiCache | Memorystore | Memorystore supports Redis and Memcached. Fully managed, similar feature parity. |
| Redshift | BigQuery | BigQuery is serverless (no cluster management). Pricing is per-query or flat-rate. Separation of storage and compute is native. |
| Kinesis | Pub/Sub + Dataflow | Pub/Sub for ingestion (no shard management). Dataflow (Apache Beam) for stream processing. |

### Security & Identity

| AWS Concept | GCP Equivalent | Key Behavioral Difference |
|-------------|----------------|---------------------------|
| IAM Roles (assumed) | IAM Roles (bound to identities) | GCP uses binding model: roles are granted to members on resources. No "assume role" — use service account impersonation. |
| SCPs (Service Control Policies) | Organization Policy Constraints | Org Policies constrain resource configurations (e.g., "no public IPs"). Different mechanism than SCPs which deny API calls. |
| AWS Organizations | Resource Manager (Org/Folders/Projects) | GCP hierarchy: Organization → Folders → Projects. Projects are the isolation boundary (not accounts). |
| GuardDuty | Security Command Center + Event Threat Detection | SCC Premium includes threat detection, vulnerability scanning, and compliance reporting in one product. |
| KMS | Cloud KMS / Cloud HSM | Similar. GCP KMS integrates with CMEK for most services. Cloud HSM for FIPS 140-2 Level 3. |
| CloudTrail | Cloud Audit Logs | Admin Activity logs are always on and free. Data Access logs must be enabled per-service. |
| Secrets Manager | Secret Manager | GCP Secret Manager is simpler and cheaper. Automatic rotation requires Cloud Functions (less native than AWS). |

### Observability

| AWS Concept | GCP Equivalent | Key Behavioral Difference |
|-------------|----------------|---------------------------|
| CloudWatch Metrics | Cloud Monitoring | Cloud Monitoring uses Monarch (same system Google uses internally). MQL or PromQL query languages. |
| CloudWatch Logs | Cloud Logging | Structured logging with Log Router for routing to multiple sinks. Logs Explorer has powerful filtering. |
| X-Ray | Cloud Trace | Cloud Trace integrates with OpenTelemetry. Automatic trace sampling for App Engine and Cloud Run. |
| CloudWatch Alarms | Alerting Policies | Alerting policies support multi-condition triggers and notification channels (PagerDuty, Slack, etc.). |
| AWS Config | Cloud Asset Inventory + Policy Analyzer | CAI provides historical asset state. Policy Analyzer answers "who can access what" queries. |

---

## When to Use Which Framework

- **AWS-primary shop moving to GCP**: Start with this comparison matrix, then deep-dive into GCP Architecture Framework pillars where your AWS architecture deviates most (typically Networking and IAM due to structural differences).
- **Multi-cloud environment**: Use both frameworks but normalize on a common set of design principles. The GCP framework's SRE alignment and AWS's operational excellence runbooks complement each other.
- **Greenfield on GCP**: Start directly with GCP Cloud Architecture Framework and reference AWS Well-Architected only for concepts GCP doesn't cover as deeply (sustainability measurement, cell-based architecture patterns).
