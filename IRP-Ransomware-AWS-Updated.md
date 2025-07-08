## AWS Incident Response Playbook – Ransomware on AWS Resources

This playbook outlines a structured response for ransomware attacks affecting AWS infrastructure. It supports containment, forensics, recovery, and strategic improvements.

---

## 📌 Phase Overview Table

| 🔄 Phase         | 🛠️ Action                                                                                                        | 👤 Primary Responsible | 🧪 Tools / Commands                                          | ✅ Completion Criteria                                                  |
|------------------|-------------------------------------------------------------------------------------------------------------------|------------------------|--------------------------------------------------------------|------------------------------------------------------------------------------|
| **Containment**  | Coordinate with Cloud Ops to immediately isolate affected resources. Apply restrictive NACLs.                    | Incident Lead          | AWS VPC Console                                             | Compromised resources are in a network "bubble" with no connectivity.     |
| **Identification** | Preserve evidence. Take forensic snapshots of all affected EBS volumes. Export logs securely.                  | Incident Responder     | AWS EC2 Console, AWS CLI                                    | Snapshots and logs are collected and stored securely.                      |
| **Identification** | Investigate the attack vector (IAM abuse, vulnerability, etc.).                                                | Incident Responder     | CloudTrail, GuardDuty, Application Logs                     | Initial hypothesis of the attack vector is formulated.                     |
| **Eradication**   | Close the entry point. Revoke credentials, patch, and scan for persistence.                                    | Incident Responder     | AWS IAM Console, Patch Manager                              | Attack vector is confirmed and neutralized.                                |
| **Recovery**      | Restore services to a clean room VPC from known-good backups.                                                  | Incident Lead          | AWS RDS Console, AWS EC2 Console                            | Data is restored in a secure, isolated environment.                         |
| **Recovery**      | Validate integrity and scan restored data for malware.                                                         | Incident Responder     | AWS Inspector, EDR Tools                                    | Data is verified and deemed clean.                                         |
| **Lessons Learned** | Recommend improvements to DR plans, S3 Object Lock, and network segmentation.                                  | Incident Lead          | AWS Backup, S3 Object Lock                                  | Resilience improvement plan is proposed.                                   |

---

## 🔍 1. Identification Phase

### Objective:
Preserve evidence and identify how the ransomware gained access.

### AWS Commands:
```bash
# Snapshot affected EBS volumes
aws ec2 create-snapshot --volume-id <vol-id> --description "Forensic snapshot"

# Copy logs to secure S3 bucket
aws s3 cp s3://<source-bucket>/ s3://secure-forensics-logs/ --recursive

# Investigate with GuardDuty
aws guardduty list-findings --detector-id <id>
```

---

## 🛡️ 2. Containment Phase

### Objective:
Isolate infected resources to prevent further damage.

### AWS Commands:
```bash
# Block all egress traffic in NACL
aws ec2 replace-network-acl-entry --network-acl-id <acl-id> --rule-number 100 --protocol -1   --rule-action deny --cidr-block 0.0.0.0/0 --egress
```

---

## 🧹 3. Eradication Phase

### Objective:
Eliminate the infection and attacker persistence.

### AWS Commands:
```bash
# Revoke IAM keys
aws iam list-access-keys --user-name <compromised-user>
aws iam delete-access-key --access-key-id <key>

# Patch instances
aws ssm send-command --document-name "AWS-RunPatchBaseline"   --targets Key=InstanceIds,Values=<instance-id>
```

---

## ♻️ 4. Recovery Phase

### Objective:
Safely restore clean workloads.

### AWS Commands:
```bash
# Restore EBS volume from snapshot
aws ec2 create-volume --snapshot-id <snapshot-id> --availability-zone <az>

# Launch new instance in clean subnet
aws ec2 run-instances --image-id <ami-id> --subnet-id <subnet-id>
```

---

## 📊 Post-Incident Activity

### Actions:
- Debrief on root causes and delays.
- Enhance DR plans with immutable backups.
- Enforce segmentation and automation of recovery workflows.

---

References:
- [AWS Security Incident Response Guide](https://docs.aws.amazon.com/security/?id=docs_gateway)
- [Amazon Inspector](https://docs.aws.amazon.com/inspector/)
- [S3 Object Lock](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock-overview.html)
