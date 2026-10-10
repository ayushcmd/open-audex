# Security Review & Privacy Compliance — OpenNotetaker

## 1. Security Architecture & Controls

- **Transport Security:** Enforcement of TLS 1.3 via reverse proxy (Caddy / Nginx) with automatic HTTPS redirection.
- **Credential Storage:** Secrets managed strictly via system environment variables (`.env`). No API keys or AWS credentials committed to source code.
- **Network Isolation:** Docker container ports bound exclusively to `127.0.0.1`. No direct public access to database or backend port 5173.
- **Session Security:** `HttpOnly` and `SameSite=Lax` cookies; `Secure` flag automatically set when base URL uses `https://`.
- **Worker Authentication:** Bearer token (`RUNNER_TOKEN`) enforced on all internal `/api/runner/*` API routes.

---

## 2. Privacy & Recording Consent

- Meeting recordings and transcripts contain sensitive information.
- Video recording is disabled by default (`VIDEO_RECORDING_ENABLED=false`).
- Public share links use token hashing so raw links are rendered once and cannot be enumerated or re-derived.
