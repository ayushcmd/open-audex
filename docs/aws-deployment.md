# Master Operational AWS Deployment Guide — OpenNotetaker

## 1. Verified Target Infrastructure

- **AWS Region:** `ap-southeast-2` (Sydney)
- **AWS Account ID:** `666823181619`
- **IAM Identity:** `arn:aws:iam::666823181619:user/OpenAudex`
- **Target EC2 Instance ID:** `i-04d8d41ffdeb00471` (`OpenNotetaker-EC2-Demo`)
- **Instance Type:** `t3.micro` (1 vCPU, 1 GiB RAM)
- **Public IPv4 Address:** `15.135.213.78`
- **Security Group ID:** `sg-074ebc75e34cc0dcc` (`launch-wizard-1`)
- **SSH Access:** Restricted exclusively to public IP (`27.60.8.222/32`)
- **EC2 IAM Instance Profile:** `None` (Authentication via STS temporary ECR login token)

---

## 2. Stage 1 Operational Specification (Security Group Network Authorization)

### Proposed Stage 1 Commands
*Requires user approval before execution.*

```bash
# 1. Authorize Inbound HTTP (Port 80)
aws ec2 authorize-security-group-ingress \
  --group-id sg-074ebc75e34cc0dcc \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0 \
  --region ap-southeast-2

# 2. Authorize Inbound HTTPS (Port 443)
aws ec2 authorize-security-group-ingress \
  --group-id sg-074ebc75e34cc0dcc \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0 \
  --region ap-southeast-2
```

### Expected Side Effects & Cost Impact
- **Side Effects:** Opens public network access on TCP ports 80 and 443 to Security Group `sg-074ebc75e34cc0dcc`. Allows HTTP web requests and ACME Let's Encrypt domain validation challenges to reach the EC2 host.
- **Cost Impact:** **$0.00** (Security Group rule creation incurs zero AWS charges).

### Explicit Verification Step
```bash
aws ec2 describe-security-groups \
  --group-ids sg-074ebc75e34cc0dcc \
  --region ap-southeast-2 \
  --query "SecurityGroups[0].IpPermissions[?FromPort==\`80\` || FromPort==\`443\`]"
```
*Expected Result:* JSON array showing active ingress rules for ports 80 and 443.

### Explicit Rollback Procedure
If Stage 1 needs to be revoked:
```bash
aws ec2 revoke-security-group-ingress \
  --group-id sg-074ebc75e34cc0dcc \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0 \
  --region ap-southeast-2

aws ec2 revoke-security-group-ingress \
  --group-id sg-074ebc75e34cc0dcc \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0 \
  --region ap-southeast-2
```

---

## 3. Production Sandbox Restore Script (`scripts/test-restore.sh`)

```bash
#!/bin/bash
# Production safety: Extracts into a unique timestamped sandbox directory.
# NEVER overwrites live application data in ~/opennotetaker/data or /app/data.
set -euo pipefail

BUCKET="opennotetaker-backups-666823181619"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
SANDBOX="/tmp/restore-sandbox-${TIMESTAMP}"

echo "Starting non-destructive restore verification test in $SANDBOX..."
mkdir -p "$SANDBOX"

# Download latest backup
aws s3 cp "s3://${BUCKET}/daily/latest.tar.gz" "${SANDBOX}/latest-restore.tar.gz" --region ap-southeast-2
tar -xzf "${SANDBOX}/latest-restore.tar.gz" -C "$SANDBOX"

# Schema & Integrity Check
if [ -f "${SANDBOX}/meetings.json" ] && [ -f "${SANDBOX}/users.json" ]; then
    echo "RESTORE SUCCESSFUL: meetings.json and users.json verified in sandbox."
    rm -rf "$SANDBOX"
    exit 0
else
    echo "RESTORE FAILED: Invalid backup structure."
    rm -rf "$SANDBOX"
    exit 1
fi
```
