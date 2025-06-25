# RDS Cross-Region Snapshot Replication (Production-Grade)

This lab automates the process of creating manual RDS snapshots and replicating them across AWS regions using **AWS Lambda**, **CloudWatch Events**, and **CloudFormation (IaC)**. It includes encrypted snapshot support, dynamic naming with timestamps, and monitoring readiness for production environments.

## Architecture Overview

- **RDS Instance (Source)** in `eu-west-1`
- **Manual Snapshots** created via Lambda
- **Waiter** ensures snapshot reaches `available` state
- **Encrypted Copy** sent to `eu-central-1` using a KMS key
- **Lambda Function** triggered daily using **CloudWatch Event Rule**
- **CloudFormation Template** provisions IAM roles, Lambda, schedule, and permissions

## 🛠️ Technologies Used

- AWS Lambda (Python 3.9)
- AWS CloudFormation
- AWS CloudWatch Events
- AWS Identity and Access Management (IAM)
- AWS Key Management Service (KMS)
- Amazon RDS

## Encryption Support

The solution uses a **customer-managed KMS key** in the destination region (`eu-central-1`) to support cross-region snapshot replication of encrypted RDS instances.

### Required KMS Permissions:
```json
{
  "Effect": "Allow",
  "Action": [
    "kms:Encrypt",
    "kms:Decrypt",
    "kms:DescribeKey",
    "kms:GenerateDataKey"
  ],
  "Resource": "arn:aws:kms:eu-central-1:298334370146:key/221ec784-e42e-4faa-87c5-fde51f9b3fd4"
}
```

## Deployment Instructions

### 1. Deploy CloudFormation Stack

Upload and deploy the provided `template.yaml` with the required parameter:

```bash
aws cloudformation deploy \
  --template-file template.yaml \
  --stack-name RdsSnapshotReplicationStack \
  --parameter-overrides RDSInstanceIdentifier=gtp-paul-mysql-lab-db \
  --capabilities CAPABILITY_NAMED_IAM
```

### 2. Confirm Snapshot Creation and Copy

Go to:
- **RDS → Snapshots** in `eu-west-1` for source snapshot
- **RDS → Snapshots** in `eu-central-1` for copied snapshot

Snapshot names follow format:

```
rds-auto-snapshot-YYYYMMDDHHMMSS
rds-auto-copy-YYYYMMDDHHMMSS
```

## Schedule

Snapshots are created **once every day** using:

```yaml
ScheduleExpression: rate(1 day)
```

## Optional Cleanup

To delete the stack and all associated resources:

```bash
aws cloudformation delete-stack --stack-name RdsSnapshotReplicationStack
```

To delete copied snapshots manually:

```bash
aws rds delete-db-snapshot --db-snapshot-identifier rds-auto-copy-<timestamp> --region eu-central-1
```

## Future Improvements

- Add SNS alerting for failures
- Auto-delete snapshots older than X days
- Extend support for other RDS engines or regions

## Status

✔️ Production-grade  
✔️ KMS-encrypted  
✔️ Time-based snapshot naming  
✔️ Fully serverless  
✔️ Region-to-region replication  
