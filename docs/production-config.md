# Production Configuration & Reverse Proxy Plan — OpenNotetaker

## 1. Traffic Architecture

```
Internet (HTTPS Port 443)
       │
       ▼
┌────────────────────────────────────────┐
│ Caddy / Nginx Reverse Proxy (TLS Cert) │
└────────────────────────────────────────┘
       │
       ▼ (127.0.0.1:5173)
┌────────────────────────────────────────┐
│ OpenNotetaker Container (Web App)       │
└────────────────────────────────────────┘
```

---

## 2. Caddy Reverse Proxy Setup

Caddy provides automatic TLS certificate issuance via Let's Encrypt / ZeroSSL.

### Example `Caddyfile`
```caddy
notes.example.com {
    reverse_proxy 127.0.0.1:5173 {
        header_up X-Forwarded-Proto {scheme}
        header_up X-Forwarded-For {remote_host}
    }
}
```

### Environment Settings in `.env`
```env
OPENNOTETAKER_BASE_URL=https://notes.example.com
TRUST_PROXY_HOPS=1
AUTH_SECURE_COOKIES=true
```

---

## 3. Security Hardening Checklist

- **Public Bindings:** Application container binds ONLY to `127.0.0.1:5173`, keeping internal ports hidden from the public internet.
- **WebSocket & Header Proxying:** Standard reverse proxy headers (`X-Forwarded-For`, `X-Forwarded-Proto`) properly forwarded.
- **Secure Cookies:** `AUTH_SECURE_COOKIES=true` enforced when HTTPS is enabled.
- **Rate Limiting:** `TRUST_PROXY_HOPS=1` ensures client IP rate limiting works correctly through Caddy.
