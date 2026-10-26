# Backup and Recovery Strategy — OpenNotetaker

## 1. Storage Architecture

OpenNotetaker uses JSON file storage and local media directories mounted at `/app/data`:
- `meetings.json`: Primary metadata, transcripts, and action items.
- `users.json`: User accounts and sessions.
- `media/`: Recorded MP4 video/audio clips.

---

## 2. Automated S3 Backup Process

A automated daily backup script compresses `/app/data` and uploads it to an encrypted S3 bucket.

### Backup Script (`scripts/backup-to-s3.sh`)
```bash
#!/bin/bash
TIMESTAMP=$(date +%Y%m%m_%H%M%S)
BACKUP_FILE="/tmp/opennotetaker-backup-${TIMESTAMP}.tar.gz"
S3_BUCKET="s3://opennotetaker-backups"

# Create archive
tar -czf "$BACKUP_FILE" -C ~/opennotetaker/data .

# Sync to S3 with SSE-KMS encryption
aws s3 cp "$BACKUP_FILE" "${S3_BUCKET}/backup-${TIMESTAMP}.tar.gz" --sse aws:kms

# Cleanup local temporary archive
rm -f "$BACKUP_FILE"
```

---

## 3. Disaster Recovery & Verification

### Disaster Recovery Steps
1. Provision fresh EC2 instance or target directory.
2. Download latest backup archive from S3:
   ```bash
   aws s3 cp s3://opennotetaker-backups/latest.tar.gz ./backup.tar.gz
   ```
3. Extract backup into `./data` volume directory:
   ```bash
   mkdir -p data && tar -xzf backup.tar.gz -C ./data
   ```
4. Start Docker Compose stack: `docker compose up -d`.

### Target Recovery Metrics
- **Recovery Time Objective (RTO):** < 15 minutes.
- **Recovery Point Objective (RPO):** 24 hours (daily backup) / < 1 hour with EBS snapshots.
