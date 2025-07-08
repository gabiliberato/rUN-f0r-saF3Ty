## AWS Incident Response Playbook – Ransomware

This playbook outlines the structured response for handling ransomware incidents affecting AWS resources. It is based on NIST 800-61 and AWS security guidance. Each phase ensures containment, evidence preservation, recovery, and lessons learned.

---

## 📌 Phase Overview Table

| 🔄 Phase         | 🛠️ Action                                                                                                     | 👤 Primary Responsible | 🧪 Tools / Commands                                       | ✅ Completion Criteria                                                    |
|------------------|-----------------------------------------------------------------------------------------------------------------|--------------------------|--------------------------------------------------------------|----------------------------------------------------------------------------------|
| **Containment**  | Coordinate with Cloud Ops to immediately isolate affected resources. Apply restrictive NACLs.                   | Incident Lead            | AWS VPC Console, EC2 Console                                     | Compromised resources are isolated with no network connectivity.               |
| **Identification** | Preserve evidence. Snapshot all EBS volumes. Export logs (CloudTrail, Flow Logs, GuardDuty) to secure account. | Incident Responder       | AWS EC2 Console, AWS CLI, CloudTrail, GuardDuty                  | Snapshots and logs are stored securely and immutably.                          |
| **Identification** | Investigate attack vector: credential abuse, vulnerability, or other.                                          | Incident Responder       | CloudTrail, IAM, GuardDuty, VPC Flow Logs                        | Initial hypothesis of attack vector documented.                                |
| **Eradication**   | Close the entry point: revoke credentials, patch systems, scan for persistence.                               | Incident Responder       | IAM Console, Systems Manager Patch Manager, CloudTrail          | Attack vector neutralized, no persistence remains.                             |
| **Recovery**      | Restore services into a clean room VPC using backups and snapshots.                                           | Incident Lead            | AWS RDS Console, AWS EC2 Console, AWS Backup                    | Systems restored safely into isolated environment.                             |
| **Recovery**      | Scan restored data for malware and verify integrity before reconnecting.                                      | Incident Responder       | AWS Inspector, CloudWatch, EDR integration                      | Restored data is clean and verified.                                           |
| **Lessons Learned** | Draft post-incident report. Recommend DR improvements (e.g., S3 Object Lock, segmentation).                    | Incident Lead            | AWS Backup, S3, GuardDuty                                       | Recommendations proposed and tracked.                                         |

---

## 🔍 1. Identification Phase

### Objective:
Preserve evidence and begin investigation.

### AWS Commands:
```bash
# Take snapshots of all volumes
aws ec2 create-snapshot --volume-id <volume-id> --description "Ransomware evidence"

# Copy logs to secure S3
aws s3 cp s3://<log-bucket>/ s3://secure-incident-logs/ --recursive

# Get GuardDuty findings
aws guardduty list-findings --detector-id <detector-id>
```

---

## 🛡️ 2. Containment Phase

### Objective:
Stop spread of ransomware and isolate systems.

### AWS Commands:
```bash
# Block network traffic via NACL
aws ec2 replace-network-acl-entry --network-acl-id <acl-id> --rule-number 100 --protocol -1 \
  --rule-action deny --cidr-block 0.0.0.0/0 --egress

# Detach network interface to isolate EC2
aws ec2 detach-network-interface --attachment-id <eni-attachment-id>
```

---

## 🧹 3. Eradication Phase

### Objective:
Remove attacker's access and persistence.

### AWS Commands:
```bash
# List and delete suspicious IAM users
aws iam list-users
aws iam delete-user --user-name <malicious-user>

# Patch EC2 using Systems Manager
aws ssm send-command --document-name "AWS-RunPatchBaseline" --targets Key=instanceIds,Values=<instance-id>
```

---

## ♻️ 4. Recovery Phase

### Objective:
Restore services into a safe, isolated state.

### AWS Commands:
```bash
# Restore from EBS snapshot
aws ec2 create-volume --snapshot-id <snapshot-id> --availability-zone <az>

# Create new EC2 instance from AMI or snapshot
aws ec2 run-instances --image-id <ami-id> --subnet-id <clean-subnet>
```

---

## 📊 Post-Incident Activity

### Actions:
- Draft a post-mortem and share with key stakeholders.
- Recommend S3 Object Lock, network segmentation, improved DR tests.
- Update detection rules (GuardDuty, Config Rules).

---
