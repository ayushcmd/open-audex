# AWS CloudWatch Monitoring & Operational Guide — OpenAudex

## 1. Modular CloudWatch Container Logging

Container log shipping to Amazon CloudWatch is implemented via an optional Docker Compose extension file: `docker-compose.cloudwatch.yml`.

### Deployment Modes
* **Default Production Deployment (Default Logging):**
  Uses standard host Docker `json-file` logging without requiring AWS IAM log permissions.
  ```bash
  IMAGE_TAG=<commit_sha> docker compose -f docker-compose.prod.yml up -d
  ```
* **CloudWatch Enabled Deployment (Optional Override):**
  Enables the Docker `awslogs` driver to forward stdout/stderr streams to CloudWatch.
  ```bash
  IMAGE_TAG=<commit_sha> docker compose -f docker-compose.prod.yml -f docker-compose.cloudwatch.yml up -d
  ```

---

## 2. Infrastructure & Prerequisites

> [!IMPORTANT]
> **Pre-Creation Requirement:** The target log group `/opennotetaker/production` must be created in AWS region `ap-southeast-2` prior to enabling `docker-compose.cloudwatch.yml`.

### Infrastructure Setup Checklist
1. **EC2 IAM Instance Profile:** The EC2 host (`i-04d8d41ffdeb00471`) must have an attached IAM Instance Profile with permissions:
   - `logs:CreateLogStream`
   - `logs:PutLogEvents`
   - `logs:DescribeLogStreams`
2. **Pre-Created Log Group:** Create `/opennotetaker/production` in region `ap-southeast-2`.
3. **Explicit Log Retention:** Configure log group retention explicitly to **7 Days** in AWS Console to prevent storage cost accumulation under AWS Free Tier limits (5 GB ingestion/storage free).
4. **Log Streams:**
   - `web`: Access & operational logs from `open-notetaker`
   - `worker`: Processing logs from `open-notetaker-worker`

---

## 3. Key Operational Metrics & Thresholds

| Metric | Target Threshold | Description |
| :--- | :--- | :--- |
| **CPUUtilization** | < 80% sustained | Detects container CPU starvation during video encoding |
| **DiskSpaceUtilization** | < 85% | Prevents data loss when media files accumulate in `/app/data` |
| **MemoryUtilization** | < 80% | Monitors RAM usage of Chromium headless instances |
| **HTTP 5xx Error Rate** | < 1% | Triggers alarm on server-side exceptions |

---

## 4. Alarm Definitions & SNS Notifications

- **Alarm 1: Application Health Failure**
  - **Condition:** 2 consecutive `/api/health` non-200 responses or timeouts over 2 minutes.
  - **Action:** Trigger automated rollback / alert notification.

- **Alarm 2: Low Free Disk Space**
  - **Condition:** `DiskSpaceUtilization` > 85% for 5 minutes.
  - **Action:** Trigger automated media retention purge script (`VIDEO_DISK_BUDGET_GB` eviction).

---

## 5. Cost, Safety, and Content Privacy

- **Cost Management:** CloudWatch costs depend on actual AWS ingestion volume and pricing. Retaining logs for 7 days ensures ingestion remains well within the AWS 5 GB Free Tier limit ($0.00/month).
- **Content Privacy:** Server stdout/stderr outputs operational status codes (`GET /api/health 200`, `POST /api/meetings 200`). Runtime secrets (`.env` file, `RUNNER_TOKEN`), OAuth tokens, and meeting audio/video buffers are strictly excluded from container logs.
- **Verification Rule:** CloudWatch monitoring must not be claimed as operational until log streams `web` and `worker` have been verified with live ingestion evidence in the AWS CloudWatch Console.
