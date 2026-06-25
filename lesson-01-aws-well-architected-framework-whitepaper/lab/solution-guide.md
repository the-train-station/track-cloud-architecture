# Well-Architected Review: Solution Guide

This guide walks through the issues in `sample-stack.yaml` and provides corrected CloudFormation snippets for each finding.

---

## Issue 1: No Multi-AZ for RDS Database

**Pillar Violated:** Reliability

**Problem:** The RDS instance is configured with `MultiAZ: false`, creating a single point of failure. If the AZ hosting the database experiences an outage, the entire application loses its data tier with no automatic failover.

**Risk Level:** High

**Recommended Fix:** Enable Multi-AZ deployment. AWS automatically provisions a synchronous standby replica in a different AZ and handles failover transparently.

```yaml
Database:
  Type: AWS::RDS::DBInstance
  Properties:
    Engine: mysql
    EngineVersion: '8.0'
    DBInstanceClass: !Ref DBInstanceClass
    AllocatedStorage: 20
    MasterUsername: !Ref DBMasterUsername
    MasterUserPassword: !Ref DBMasterPassword
    DBSubnetGroupName: !Ref DBSubnetGroup
    VPCSecurityGroups:
      - !Ref DBSecurityGroup
    MultiAZ: true
    StorageEncrypted: true
    BackupRetentionPeriod: 7
    DeletionProtection: true
    PubliclyAccessible: false
```

---

## Issue 2: No Encryption at Rest for Database Storage

**Pillar Violated:** Security

**Problem:** `StorageEncrypted: false` means database contents are stored unencrypted on the underlying EBS volumes. If the physical storage is compromised or an EBS snapshot is shared incorrectly, data is exposed in plaintext.

**Risk Level:** High

**Recommended Fix:** Enable storage encryption with the default AWS-managed KMS key (or a customer-managed key for greater control over key rotation and access policies).

```yaml
Database:
  Type: AWS::RDS::DBInstance
  Properties:
    # ... other properties ...
    StorageEncrypted: true
    KmsKeyId: !Ref DatabaseKMSKey  # Optional: use CMK for fine-grained control

# Optional: Customer-managed KMS key
DatabaseKMSKey:
  Type: AWS::KMS::Key
  Properties:
    Description: Encryption key for RDS database
    EnableKeyRotation: true
    KeyPolicy:
      Version: '2012-10-17'
      Statement:
        - Sid: AllowRootAccountAccess
          Effect: Allow
          Principal:
            AWS: !Sub arn:aws:iam::${AWS::AccountId}:root
          Action: kms:*
          Resource: '*'
```

---

## Issue 3: SSH Open to the World (0.0.0.0/0)

**Pillar Violated:** Security

**Problem:** The web server security group allows SSH (port 22) from any IP address. This exposes instances to brute-force attacks and is a common vector for unauthorized access.

**Recommended Fix:** Remove direct SSH access entirely. Use AWS Systems Manager Session Manager for shell access, which requires no open inbound ports and provides full audit logging.

```yaml
WebServerSecurityGroup:
  Type: AWS::EC2::SecurityGroup
  Properties:
    GroupDescription: Allow traffic to web servers
    VpcId: !Ref VPC
    SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 80
        ToPort: 80
        SourceSecurityGroupId: !Ref ALBSecurityGroup
    # SSH rule removed - use SSM Session Manager instead

# Add an IAM instance profile for SSM access
WebServerInstanceProfile:
  Type: AWS::IAM::InstanceProfile
  Properties:
    Roles:
      - !Ref WebServerRole

WebServerRole:
  Type: AWS::IAM::Role
  Properties:
    AssumeRolePolicyDocument:
      Version: '2012-10-17'
      Statement:
        - Effect: Allow
          Principal:
            Service: ec2.amazonaws.com
          Action: sts:AssumeRole
    ManagedPolicyArns:
      - arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
```

---

## Issue 4: No Backup Retention (BackupRetentionPeriod: 0)

**Pillar Violated:** Reliability

**Problem:** With `BackupRetentionPeriod: 0`, automated backups are disabled entirely. There is no way to perform point-in-time recovery if data is corrupted or accidentally deleted.

**Risk Level:** High

**Recommended Fix:** Set backup retention to at least 7 days for production workloads. This enables point-in-time recovery to any second within the retention window.

```yaml
Database:
  Type: AWS::RDS::DBInstance
  Properties:
    # ... other properties ...
    BackupRetentionPeriod: 7
    PreferredBackupWindow: '03:00-04:00'  # Off-peak hours
    DeletionProtection: true  # Prevent accidental deletion
```

---

## Issue 5: No IAM Instance Profile (Least Privilege Violation)

**Pillar Violated:** Security

**Problem:** The LaunchTemplate does not specify an `IamInstanceProfile`. Without it, EC2 instances have no AWS permissions through IAM roles, which means:
- Applications can't interact with AWS services securely
- Developers may resort to embedding access keys in code or environment variables
- There's no audit trail of what permissions the instances use

**Risk Level:** Medium

**Recommended Fix:** Create a role with only the permissions the application needs, and attach it via an instance profile.

```yaml
LaunchTemplate:
  Type: AWS::EC2::LaunchTemplate
  Properties:
    LaunchTemplateData:
      ImageId: ami-0c02fb55956c7d316
      InstanceType: !Ref InstanceType
      KeyName: !Ref KeyPairName
      IamInstanceProfile:
        Arn: !GetAtt WebServerInstanceProfile.Arn
      SecurityGroupIds:
        - !Ref WebServerSecurityGroup
      UserData:
        Fn::Base64: |
          #!/bin/bash
          yum update -y
          yum install -y httpd amazon-cloudwatch-agent
          systemctl start httpd
          systemctl enable httpd

WebServerRole:
  Type: AWS::IAM::Role
  Properties:
    AssumeRolePolicyDocument:
      Version: '2012-10-17'
      Statement:
        - Effect: Allow
          Principal:
            Service: ec2.amazonaws.com
          Action: sts:AssumeRole
    ManagedPolicyArns:
      - arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
      - arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy

WebServerInstanceProfile:
  Type: AWS::IAM::InstanceProfile
  Properties:
    Roles:
      - !Ref WebServerRole
```

---

## Issue 6: No CloudWatch Alarms or Monitoring

**Pillar Violated:** Operational Excellence / Performance Efficiency

**Problem:** The stack has zero CloudWatch alarms. There is no automated detection of:
- High CPU utilization on instances
- Unhealthy targets behind the ALB
- Database connection exhaustion
- Elevated error rates

The team won't know something is broken until users report it.

**Risk Level:** Medium

**Recommended Fix:** Add alarms for critical operational metrics and wire them to an SNS topic for notification.

```yaml
Parameters:
  AlertEmail:
    Type: String
    Description: Email address for operational alerts

Resources:
  AlertTopic:
    Type: AWS::SNS::Topic
    Properties:
      Subscription:
        - Protocol: email
          Endpoint: !Ref AlertEmail

  HighCPUAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmDescription: CPU utilization exceeds 80% for 5 minutes
      Namespace: AWS/EC2
      MetricName: CPUUtilization
      Dimensions:
        - Name: AutoScalingGroupName
          Value: !Ref AutoScalingGroup
      Statistic: Average
      Period: 300
      EvaluationPeriods: 2
      Threshold: 80
      ComparisonOperator: GreaterThanThreshold
      AlarmActions:
        - !Ref AlertTopic

  UnhealthyHostAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmDescription: One or more ALB targets are unhealthy
      Namespace: AWS/ApplicationELB
      MetricName: UnHealthyHostCount
      Dimensions:
        - Name: TargetGroup
          Value: !GetAtt ALBTargetGroup.TargetGroupFullName
        - Name: LoadBalancer
          Value: !GetAtt ApplicationLoadBalancer.LoadBalancerFullName
      Statistic: Sum
      Period: 60
      EvaluationPeriods: 3
      Threshold: 1
      ComparisonOperator: GreaterThanOrEqualToThreshold
      AlarmActions:
        - !Ref AlertTopic

  DatabaseCPUAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmDescription: RDS CPU utilization exceeds 80%
      Namespace: AWS/RDS
      MetricName: CPUUtilization
      Dimensions:
        - Name: DBInstanceIdentifier
          Value: !Ref Database
      Statistic: Average
      Period: 300
      EvaluationPeriods: 2
      Threshold: 80
      ComparisonOperator: GreaterThanThreshold
      AlarmActions:
        - !Ref AlertTopic
```

---

## Summary of Findings

| # | Issue | Pillar | Risk | Fix Complexity |
|---|-------|--------|------|----------------|
| 1 | Single-AZ database | Reliability | High | Low (one property change) |
| 2 | No storage encryption | Security | High | Low (one property change) |
| 3 | SSH open to 0.0.0.0/0 | Security | High | Medium (add SSM role) |
| 4 | No backup retention | Reliability | High | Low (one property change) |
| 5 | No IAM instance profile | Security | Medium | Medium (create role + profile) |
| 6 | No CloudWatch alarms | Operational Excellence | Medium | Medium (add alarm resources) |

## Priority Remediation Order

1. **Immediate (Day 1):** Issues 2, 3, 4 -- these are security and data-loss risks with simple fixes
2. **Short-term (Week 1):** Issues 1, 5 -- reliability and least-privilege improvements
3. **Ongoing:** Issue 6 -- build out monitoring iteratively as you understand traffic patterns
