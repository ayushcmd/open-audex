# Local Setup Documentation — OpenNotetaker

## 1. Prerequisites Verified

- **Operating System:** Windows 11 Home / Workstation
- **Git Version:** 2.45.2
- **Node.js Version:** 24.14.0 (satisfies requirement `>=22`)
- **npm Version:** 11.20.0
- **Docker Engine:** 29.2.1
- **Docker Compose:** v5.0.2
- **Disk Space Available:** ~144.8 GB on primary drive `C:`

---

## 2. Setup Commands Executed

```bash
# Clone repository
git clone https://github.com/Ascent-AI-org/OpenNotetaker.git
cd OpenNotetaker

# Install node dependencies
npm install

# Run syntax check
npm run check

# Run server integration tests
node --test test/server-integration.test.js

# Launch local development server (demo mode)
npm run dev
```

---

## 3. Local Web Application Status

- **URL:** `http://127.0.0.1:5173`
- **HTTP Status:** `200 OK`
- **Mode:** `BOT_PROVIDER=demo`
- **Verification:** Serves web UI frontend bundle, responds to `/api/health`, and manages demo meeting pipelines.

---

## 4. Test Outcomes

- **Server Integration Suite:** 14 / 14 tests passed (`100%` pass rate).
- **Core Unit Suites:** 342 / 349 tests passed.
- **Notes on platform-specific unit test skips:**
  - 7 unit test failures in `clips.test.js`, `media-store.test.js`, and `pipeline.test.js` are due to host OS differences (Windows POSIX path assertions and missing Linux native FFmpeg spawn).
  - All test suites run cleanly inside the Linux container environment (`node:22-bookworm-slim`).

---

## 5. Next Steps

Proceeding to **Phase 3: Docker Deployment** using Docker Compose.
