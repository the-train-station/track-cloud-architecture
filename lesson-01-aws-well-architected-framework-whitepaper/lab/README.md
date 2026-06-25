# Lab: Well-Architected Framework Review

## Objective

Practice conducting a Well-Architected Review against a real CloudFormation stack. You will identify architectural issues, classify them by pillar, and propose fixes.

## Prerequisites

- AWS account with CloudFormation permissions (or just review the template locally)
- Familiarity with the six Well-Architected Framework pillars
- AWS CLI configured (optional, for deployment)

## Exercise Flow

### Step 1: Review the Sample Architecture (15 min)

Open `sample-stack.yaml` and read through the template. Understand what it deploys:
- VPC with public and private subnets across 2 AZs
- Application Load Balancer in public subnets
- Auto Scaling Group with EC2 instances
- RDS MySQL database in private subnets

### Step 2: Complete the Review Worksheet (30 min)

Open `well-architected-review-worksheet.md` and answer each question based on what you observe in the stack. For each question:
1. Document the current state (what exists or is missing)
2. Assign a risk level (High / Medium / Low)
3. Note what you would recommend changing

### Step 3: (Optional) Deploy the Stack

If you have an AWS account available, deploy the stack to see it running:
```bash
aws cloudformation deploy \
  --template-file sample-stack.yaml \
  --stack-name wa-review-lab \
  --parameter-overrides KeyPairName=your-key-pair DBMasterPassword=YourPass123!
```

### Step 4: Compare Against the Solution Guide (15 min)

Open `solution-guide.md` and compare your findings. Check:
- Did you catch all 6 issues?
- Did you correctly identify which pillar each issue violates?
- Are your risk assessments aligned with the guide?

### Step 5: Clean Up

If you deployed the stack:
```bash
aws cloudformation delete-stack --stack-name wa-review-lab
```

## Key Takeaways

- A Well-Architected Review is a structured conversation, not a pass/fail audit
- Most issues map to multiple pillars (e.g., encryption is both Security and Compliance)
- Prioritize remediation by risk level and fix complexity
