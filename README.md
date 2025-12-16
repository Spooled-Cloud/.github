## 🧵 Spooled Cloud

**Open-source webhook queue & background job infrastructure.**

Spooled handles the hard parts of async processing — retries, idempotency, dead-letter queues, cron schedules, and workflows — so you can focus on your business logic.

### The Problem

```
Stripe sends webhook → Your server is down → Event lost forever 😱
```

### The Solution

```
Stripe sends webhook → Spooled queues it → Retries automatically → Your worker processes it ✓
```

### What You Get

- **Bulk operations** — Enqueue 100 jobs in one API call
- **Automatic retries** — Exponential backoff, configurable per job
- **Idempotency** — No duplicate processing, guaranteed
- **Dead-letter queue** — Inspect failures, replay with one click
- **Cron schedules** — Second-precision, timezone-aware
- **Workflows** — Chain jobs with dependencies
- **Real-time** — SSE + WebSocket streaming
- **REST + gRPC** — Use what fits your stack

### Quick Example

```bash
curl -X POST https://api.spooled.cloud/api/v1/jobs \
  -H "Authorization: Bearer sk_live_..." \
  -d '{"queue_name": "emails", "payload": {"to": "user@example.com"}, "idempotency_key": "welcome-123"}'
```

### Links

🌐 [spooled.cloud](https://spooled.cloud) · 📖 [Documentation](https://spooled.cloud/docs)

---

<sub>Apache 2.0 · Built with Rust, PostgreSQL, Redis</sub>
