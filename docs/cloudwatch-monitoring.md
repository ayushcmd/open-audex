# AWS CloudWatch Monitoring & Alerting — OpenNotetaker

## 1. CloudWatch Agent & Log Configuration

The Amazon CloudWatch Agent collects system-level metrics and container logs from EC2.

### Log Streams Configured
- `/opennotetaker/web`: `docker logs open-notetaker`
- `/opennotetaker/worker`: `docker logs open-notetaker-worker`
- `/opennotetaker/caddy`: Caddy access & error logs

---

## 2. Key Operational Metrics

| Metric | Target Threshold | Description |
| :--- | :--- | :--- |
| **CPUUtilization** | < 80% sustained | Detects container CPU starvation during video encoding |
| **DiskSpaceUtilization** | < 85% | Prevents data loss when media files accumulate |
| **MemoryUtilization** | < 80% | Monitors RAM usage of Chromium headless instances |
| **HTTP 5xx Error Rate** | < 1% | Triggers alarm on server-side exceptions |

---

## 3. Alarm Definitions & SNS Notifications

- **Alarm 1: Application Health Failure**
  - **Condition:** 2 consecutive `/api/health` 5xx status codes or timeouts over 2 minutes.
  - **Action:** Send SNS notification to DevOps email / Pager.

- **Alarm 2: Low Free Disk Space**
  - **Condition:** `DiskSpaceUtilization` > 85% for 5 minutes.
  - **Action:** Trigger automated cleanup script (`VIDEO_DISK_BUDGET_GB` eviction).
