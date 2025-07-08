## AWS Incident Response Playbook – Supply Chain Compromise

This playbook outlines the structured response for handling incidents involving supply chain compromise within AWS environments. This includes the injection of malicious dependencies, tampered build artifacts, or unauthorized changes to pipelines. It is aligned with AWS and industry best practices.

---

## 📌 Phase Overview Table

| 🔄 Phase       | 🛠️ Action                                                                                                      | 👤 Primary Responsible | 🧪 Tools / Commands                                     | ✅ Completion Criteria                                                       |
|----------------|------------------------------------------------------------------------------------------------------------------|------------------------|----------------------------------------------------------|-----------------------------------------------------------------------------------|
| **Containment** | Block the destination IP or domain of the malicious traffic at the network layer.                              | Incident Responder     | AWS VPC Console, AWS Network Firewall                        | Exfiltration traffic is blocked, confirmed via VPC Flow Logs.                    |
| **Containment** | Coordinate with DevOps/Development to identify all applications using the compromised library.                  | Incident Lead          | AWS CodeBuild, AWS CodePipeline, AWS Inspector, SBOM tools   | A complete list of all affected assets is created.                               |
| **Containment** | Request and validate that the SRE/DevOps team rolls back all affected applications to the last known-good version. | Incident Lead        | AWS CodeDeploy, AWS CodePipeline                             | The malicious version is no longer running in the production environment.        |
| **Eradication** | Request and validate that the DevOps team purges the malicious library version from internal artifact repositories. | Incident Lead      | AWS CodeArtifact, S3 CLI commands                           | The malicious artifact is permanently removed from internal repositories.        |
| **Recovery**    | Lead the mass rotation of all credentials and secrets accessible to the compromised applications.              | Incident Lead          | AWS Secrets Manager                                          | All potentially exposed secrets have been rotated.                               |
| **Recovery**    | Oversee the re-deployment of applications with the safe library version and monitor for anomalies.              | Incident Lead          | AWS CodeDeploy, CloudWatch, GuardDuty                        | Services are restored and operating normally under observation.                  |

---

## 🔍 1. Identification Phase

### Objective:
Detect and confirm indicators of supply chain compromise.

### Steps:
- Analyze alerts from AWS Security Hub, GuardDuty.
- Validate changes in CodePipeline/CodeBuild.
- Check artifact integrity using hashes or Inspector reports.
- Identify initial point of compromise.

### AWS Commands:
```bash
# Look up suspicious activities in CloudTrail
aws cloudtrail lookup-events --lookup-attributes AttributeKey=EventName,AttributeValue=PutObject

# Review CodePipeline execution history
aws codepipeline list-pipeline-executions --pipeline-name <pipeline-name>

# Check contents in S3 artifact bucket
aws s3 ls s3://<artifact-bucket>/ --recursive
```

---

## 🛡️ 2. Containment Phase

### Objective:
Prevent the spread and usage of the compromised components.

### AWS Commands:
```bash
# Block malicious IP or domain via NACL or security group
aws ec2 describe-security-groups
aws ec2 revoke-security-group-egress --group-id <sg-id> --cidr <malicious-ip>/32 --protocol all

# Review all deployed CodeBuild images or dependencies
aws codebuild list-projects
aws codebuild batch-get-projects --names <project-name>

# Trigger rollback via CodeDeploy
aws deploy create-deployment --application-name <AppName> --deployment-group-name <Group>   --revision revisionType=S3,s3Location={bucket=<bucket>,key=<path>,bundleType=zip}
```

---

## 🧹 3. Eradication Phase

### Objective:
Remove all malicious or unauthorized components.

### AWS Commands:
```bash
# Delete malicious artifacts from S3
aws s3 rm s3://<bucket-name>/<artifact-path>

# Remove compromised package version from CodeArtifact (if used)
aws codeartifact delete-package-versions --domain <domain> --repository <repo>   --package <package-name> --versions <malicious-version>

# Review CodePipeline and rollback changes
aws codepipeline get-pipeline --name <pipeline-name>
```

---

## ♻️ 4. Recovery Phase

### Objective:
Restore services safely and rebuild trust.

### AWS Commands:
```bash
# Rotate secrets using AWS Secrets Manager
aws secretsmanager list-secrets
aws secretsmanager rotate-secret --secret-id <secret-id>

# Re-deploy application using clean build artifact
aws deploy create-deployment --application-name <AppName> --deployment-group-name <Group>   --revision revisionType=S3,s3Location={bucket=<bucket>,key=<path>,bundleType=zip}

# Enable alarms and logs for monitoring
aws cloudwatch describe-alarms
aws logs describe-log-groups
```

---

## 📊 Post-Incident Activity

### Recommended Actions:
- Debrief with DevOps, Security, and stakeholders.
- Update playbooks and internal documentation.
- Improve CI/CD security checks and SBOM tooling.
- Add detection rules in Security Hub and GuardDuty.

---

For updates and integrations, refer to:
- [AWS CodePipeline Security](https://docs.aws.amazon.com/codepipeline/latest/userguide/security.html)
- [AWS Security Incident Response Guide](https://docs.aws.amazon.com/security/?id=docs_gateway)
- [SLSA Framework](https://slsa.dev/) for secure software supply chains.
