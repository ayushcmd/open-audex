# SLA, RTO, RPO, and TTR Definitions — OpenNotetaker

## 1. Service Level Agreements (SLA) & Targets

- **Target Uptime Availability:** `99.5%` uptime per calendar month.
- **Web Interface Response Time:** P95 latency < `500 ms` for standard REST requests.
- **Meeting Processing SLA:** Meeting summary & action item generation completed within `3 minutes` of meeting end.

---

## 2. Recovery Time & Point Objectives

### Recovery Time Objective (RTO)
- **Target:** `15 Minutes`
- **Definition:** Time elapsed between total instance/host failure and full restoration of the web interface and recording workers on a replacement EC2 instance.

### Recovery Point Objective (RPO)
- **Target:** `1 Hour` (with EBS snapshotting) / `24 Hours` (with daily S3 backups).
- **Definition:** Maximum acceptable data loss window for meeting transcripts and notes in a catastrophic event.

### Time To Resolve (TTR)
- **Target:** `< 5 Minutes` for single container restart failures (automated via Docker `--restart unless-stopped`).
- **Calculation:** $\text{TTR} = T_{\text{restoration}} - T_{\text{incident\_detection}}$
