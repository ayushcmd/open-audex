# Performance Testing Strategy — OpenNotetaker

## 1. Load Testing Overview

Load testing is conducted using **k6** against the deployment endpoint to measure HTTP throughput, P95 latency, and error rates under concurrent user load.

---

## 2. k6 Test Script (`scripts/k6-load-test.js`)

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '30s', target: 20 }, // Ramp up to 20 users
    { duration: '1m',  target: 50 }, // Sustained load
    { duration: '30s', target: 0  }, // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'], // 95% of requests under 500ms
    http_req_failed: ['rate<0.01'],   // Error rate under 1%
  },
};

export default function () {
  const res = http.get('http://127.0.0.1:5173/api/health');
  check(res, { 'status was 200': (r) => r.status === 200 });
  sleep(1);
}
```

---

## 3. Benchmarks & Resource Limits

- **Lightweight API P95 Latency:** `< 120 ms`
- **Concurrent Recording Capacity:** 1 worker replica per vCPU / 2GB RAM.
- **Resource Constraints:** Chrome headless instance requires ~1.2 GB RAM and ~0.8 vCPU per active Google Meet recording session.
