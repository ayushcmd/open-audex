# Failure-Recovery Test Procedures & Log Protocols — OpenNotetaker

## 1. Test Scenarios & Recovery Verification

### Test Scenario 1: Web Container Process Failure
- **Action:** Simulate web container process crash: `docker stop open-notetaker`.
- **Expected Outcome:** Docker engine detects container stop and automatically restarts container within 5 seconds due to `restart: unless-stopped`.
- **Validation:** Health endpoint `GET /api/health` returns `HTTP 200 OK`. Meeting data in `meetings.json` remains completely intact.

### Test Scenario 2: Worker Container Crash During Recording
- **Action:** Stop recording worker container while job is leased: `docker stop open-notetaker-worker`.
- **Expected Outcome:** Worker lease expires after `RUNNER_LEASE_SECONDS` (120s). The job state is automatically reclaimed by another active worker replica or marked for retry. Partial transcript chunks flushed prior to worker stop remain saved.

### Test Scenario 3: Disk Full Prevention Trigger
- **Action:** Fill volume disk beyond `VIDEO_MIN_FREE_DISK_GB` threshold.
- **Expected Outcome:** Application safety floor halts new video recording requests before `meetings.json` can be corrupted, preventing transcription outages.
