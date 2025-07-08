## AWS Incident Response Playbook – Root Account Compromise

This playbook provides a structured response to incidents involving compromise of the AWS root account. The root account has unrestricted access and must be quickly secured to prevent catastrophic consequences.

---

## 📌 Phase Overview Table

| 🔄 Phase         | 🛠️ Action                                                                                                     | 👤 Primary Responsible | 🧪 Tools / Commands                                            | ✅ Completion Criteria                                                    |
|------------------|------------------------------------------------------------------------------------------------------------------|--------------------------|------------------------------------------------------------------|----------------------------------------------------------------------------------|
| **Identification** | Attempt an immediate login to the Root account to change the password and re-enable MFA.                      | Incident Lead            | AWS Console                                                     | Access is either successful or failure is confirmed.                            |
| **Containment**  | If login fails, follow the established protocol to engage AWS Enterprise Support by phone immediately.         | Incident Lead            | Phone                                                           | AWS support case is opened and reference number is obtained.                    |
| **Containment**  | Using a separate Administrator IAM account, begin real-time CloudTrail monitoring of the root user's activities. | Incident Responder       | AWS CloudTrail, CloudWatch Alarms                               | Real-time visibility of attacker actions is established.                        |
| **Containment**  | Immediately attempt to sign in and reset the root password. Revoke unauthorized MFA devices.                    | Security Lead            | AWS Console, `aws iam`, email recovery flow                     | Root account is recovered and access is secured.                                |
| **Containment**  | Disable active sessions and revoke all access keys linked to the root account.                                 | Security Lead            | AWS CloudTrail, AWS Console, `aws iam delete-access-key`        | Root session tokens and keys are rendered unusable.                             |
| **Identification** | Review CloudTrail logs for all root activity and confirm unauthorized use.                                    | Incident Responder       | AWS CloudTrail, `aws cloudtrail lookup-events`                  | Timeline of root account actions is documented.                                 |
| **Eradication**   | After regaining control, invalidate all active sessions. Rotate ALL credentials for ALL IAM users.             | Incident Lead            | AWS IAM Console, Automation Scripts                             | All access keys and passwords in the account have been rotated.                 |
| **Eradication**   | Audit all attacker actions. Hunt for new IAM users, roles, modified policies, rogue resources, and SCPs.       | Incident Responder       | CloudTrail, AWS Config, Trusted Advisor                         | A complete damage report of malicious changes is created.                       |
| **Eradication**   | Rotate all sensitive credentials and remove any backdoors (users, roles, policies).                            | Incident Responder       | AWS IAM, AWS Secrets Manager                                    | Credentials rotated and environment verified clean.                             |
| **Recovery**      | Request and validate that Cloud Ops restores security configurations and removes malicious resources.         | Incident Lead            | AWS Console, IaC (Terraform/CloudFormation)                     | Environment is restored to its security baseline.                               |
| **Recovery**      | Reinforce least privilege by removing root access from workflows. Enable strong MFA and alerts.               | Cloud Security Architect | AWS IAM, AWS Organizations, Config Rules                        | Root usage locked down and monitored.                                           |
| **Lessons Learned** | Draft a post-incident report recommending strengthening root account process and SCP implementation.          | Incident Lead            | AWS Organizations                                               | Governance improvements for root account are proposed.                          |
| **Lessons Learned** | Document root account risks and update organizational response processes.                                    | Security Lead            | Security Wiki, Compliance Tracker                               | Root compromise procedures and detection coverage are improved.                 |

---

## 🔍 1. Identification Phase

### Objective:
Validate if the root account was used maliciously and understand scope.

### AWS Commands:
```bash
# Query all events made by root user
aws cloudtrail lookup-events --lookup-attributes AttributeKey=Username,AttributeValue=root

# Identify active sessions (CloudTrail + logs)
aws cloudtrail lookup-events --lookup-attributes AttributeKey=EventName,AttributeValue=ConsoleLogin
```

---

## 🛡️ 2. Containment Phase

### Objective:
Stop ongoing misuse of the root account.

### AWS Commands:
```bash
# Delete all root access keys (if exist)
aws iam list-access-keys --user-name root
aws iam delete-access-key --user-name root --access-key-id <key>

# Disable root MFA (only through account recovery)
# Recovery must be done via AWS support and email verification
```

---

## 🧹 3. Eradication Phase

### Objective:
Remove attacker persistence and audit for additional compromise.

### AWS Commands:
```bash
# List all IAM users and roles
aws iam list-users
aws iam list-roles

# Look for anomalies: unfamiliar users, permissions, roles
# Remove unauthorized IAM users or attached policies
aws iam delete-user --user-name <suspicious>
```

---

## ♻️ 4. Recovery Phase

### Objective:
Harden the root account and restore trust.

### AWS Commands:
```bash
# Ensure root account uses a strong password + MFA
# Set up CloudTrail alerting
aws sns create-topic --name RootAccountAlerts

# Enable AWS Config rule for root account usage
# Manual setup or AWS Config Console
```

---

## 📊 Post-Incident Activity

### Actions:
- Conduct RCA and improve preventive controls.
- Train personnel on root account risk.
- Add root use detection to Security Hub/GuardDuty.

---
