## AWS Incident Response Playbook – Credential Compromise

This playbook outlines the structured response for handling incidents involving potentially compromised AWS IAM access keys. It is designed to ensure rapid identification, containment, eradication, and recovery from credential-based threats, and is aligned with AWS and industry best practices.

---

## 📌 Phase Overview Table

| 🔄 Phase        | 🛠️ Action                                                                                             | 👤 Primary Responsible | 🧪 Tools / Commands                                             | ✅ Completion Criteria                                                        |
|----------------|--------------------------------------------------------------------------------------------------------|------------------------|----------------------------------------------------------------|--------------------------------------------------------------------------------|
| **Identification** | Validate the alert. Confirm the key belongs to the organization and verify its IAM permissions.         | Incident Responder     | `AWS IAM Console`, `aws iam get-access-key-info`               | Key ownership and permissions are documented.                                 |
| **Containment**    | Revoke (delete) the compromised access key. DO NOT just disable it. Apply a `DenyAll` policy to the IAM user. | Incident Responder     | `AWS IAM Console`, `aws iam delete-access-key`                 | Key can no longer be used. IAM user is locked down.                           |
| **Containment**    | Analyze CloudTrail for all activity from the compromised key. Focus on IPs, UserAgents, suspicious API calls. | Incident Responder     | AWS CloudTrail, Amazon Athena, `jq`                            | Timeline of attacker activities is created and documented.                    |
| **Eradication**    | Deeply analyze attacker activity. Hunt for persistence (e.g., creation of IAM users, roles, resources).     | Incident Responder     | CloudTrail, AWS Config                                         | No backdoors or unauthorized persistence detected.                            |
| **Eradication**    | Coordinate with DevOps to identify root cause (e.g., `.gitignore` failure) and purge secret from Git history. | Incident Lead          | `git`, [BFG Repo-Cleaner](https://rtyley.github.io/bfg-repo-cleaner/) | Secret is permanently removed from all repositories.                         |
| **Recovery**       | Recommend credential rotation for other possibly exposed secrets if the dev environment was compromised.    | Incident Lead          | AWS Secrets Manager, Manual                                    | All related credentials rotated and confirmed by their owners.                |

---

## 🔍 1. Identification Phase

### Objective:
Confirm the legitimacy of the alert and determine whether the access key is owned by your organization.

### Steps:
- Review alert details from monitoring tools or AWS GuardDuty.
- Use the AWS CLI to verify the access key:

```bash
aws iam get-access-key-info --access-key-id <ACCESS_KEY_ID>
```

- Identify the IAM user associated with the key.
- Review permissions using IAM policies and role attachments.

### Outputs:
- Identity and scope of the key confirmed.
- Permissions documented for impact assessment.

---

## 🛡️ 2. Containment Phase

### Objective:
Prevent further unauthorized use of the compromised key.

### Steps:
1. **Delete the access key:**

```bash
aws iam delete-access-key --access-key-id <ACCESS_KEY_ID> --user-name <IAM_USER>
```

2. **Apply a DenyAll IAM policy to the user:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*"
    }
  ]
}
```

3. **Audit CloudTrail logs:**
Use Athena, CloudTrail Lookup, or `jq` to filter activity:

```bash
aws cloudtrail lookup-events --lookup-attributes AttributeKey=AccessKeyId,AttributeValue=<ACCESS_KEY_ID>
```

Focus on:
- IP addresses
- UserAgent strings
- Sensitive API calls (`List*`, `GetObject`, `PutObject`, `CreateUser`, `CreateRole`, etc.)

### Outputs:
- Key rendered unusable.
- IAM user blocked.
- Attacker activity timeline generated.

---

## 🧹 3. Eradication Phase

### Objective:
Ensure no persistence or follow-up actions remain from the attacker.

### Steps:
1. Search CloudTrail for resource creation (IAM, EC2, Lambda, etc.).
2. Use AWS Config to identify unauthorized changes to users, roles, or resources.
3. Collaborate with DevOps to trace the root cause.
   - Examples: exposed secrets in GitHub, `.gitignore` misconfiguration.
4. Use `BFG Repo-Cleaner` to scrub secrets:

```bash
bfg --delete-files "*.env"
bfg --delete-text "AWS_SECRET_ACCESS_KEY"
```

### Outputs:
- Persistence removed.
- Secret completely removed from version control.

---

## 🔁 4. Recovery Phase

### Objective:
Restore trust in the environment and prevent future recurrence.

### Steps:
1. Identify other credentials potentially compromised.
2. Rotate all credentials using:
   - AWS Secrets Manager
   - Manual rotation for IAM users, third-party tokens, etc.
3. Notify affected teams and confirm credential updates.
4. Update `.gitignore` and CI/CD security measures.

### Outputs:
- All exposed credentials rotated.
- Dev environment cleaned and hardened.

---

## 📊 Post-Incident Activity

### Recommended Actions:
- Conduct internal debrief with stakeholders.
- Update playbook based on findings.
- Add indicators to detection rules.
- Perform red team simulation to validate future readiness.

---
