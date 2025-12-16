# 🧵 Spooled Cloud

**Open-source webhook queue & background job infrastructure.**

Stop losing webhooks. Stop building retry logic. Start shipping.

---

## What is Spooled?

Spooled is a **job queue as a service** — accept webhooks, process background jobs, run cron schedules, and orchestrate workflows. All with automatic retries, idempotency, and dead-letter queues built in.

```
Stripe webhook → Spooled → Your worker → ✓ Done (or retried automatically)
```

## Real-World Examples

### 🛒 E-commerce
```
Order placed → Spooled queue → [Charge payment] → [Update inventory] → [Send confirmation]
                                     ↓ (if fails)
                              Retry with backoff → DLQ for manual review
```

### 📧 SaaS Notifications
```bash
# Bulk enqueue 100 welcome emails in one API call
POST /api/v1/jobs/bulk
{ "queue_name": "emails", "jobs": [...100 users...] }
```

### 🔄 Webhook Reliability
Your Stripe webhooks fail at 3am? Spooled retries them automatically with exponential backoff. Check batch status, inspect failures, replay from DLQ — all via API.

### ⏰ Cron with Second Precision
```json
{ "cron_expression": "0 */5 * * * *", "timezone": "America/New_York" }
```
6-field cron expressions. Timezone-aware. Execution history included.

---

## Why Spooled?

| Feature | Spooled | DIY Redis/SQS |
|---------|---------|---------------|
| Automatic retries | ✅ Built-in | 🔧 Build it |
| Idempotency | ✅ Built-in | 🔧 Build it |
| Dead-letter queue | ✅ Built-in | 🔧 Build it |
| Real-time streaming | ✅ SSE + WebSocket | 🔧 Build it |
| Workflows with dependencies | ✅ Built-in | 🔧 Build it |
| Dashboard | ✅ Included | ❌ |

## Repositories

| Repo | Description |
|------|-------------|
| [spooled-backend](https://github.com/Spooled-Cloud/spooled-backend) | 🦀 Rust API server (REST + gRPC) |
| [spooled-frontend](https://github.com/Spooled-Cloud/spooled-frontend) | 🌐 Marketing site & docs |
| [spooled-dashboard](https://github.com/Spooled-Cloud/spooled-dashboard) | 📊 Admin dashboard |
| [spooled-sdk-nodejs](https://github.com/Spooled-Cloud/spooled-sdk-nodejs) | 📦 TypeScript/Node.js SDK |

## Quick Start

```bash
# Enqueue a job
curl -X POST https://api.spooled.cloud/api/v1/jobs \
  -H "Authorization: Bearer sk_live_..." \
  -d '{"queue_name": "emails", "payload": {"to": "user@example.com"}}'

# It's queued. Your worker processes it. If it fails, Spooled retries.
```

---

## 💖 Sponsor Us

Building reliable infrastructure is hard. We're a small team making webhooks and background jobs **just work** for everyone.

**Your sponsorship helps us:**
- Keep the open-source version free forever
- Add more SDKs (Python, Go, PHP)
- Improve documentation and examples
- Build features the community requests

👉 **[Become a Sponsor](https://github.com/sponsors/Spooled-Cloud)**

Every contribution — code, docs, or sponsorship — helps developers ship faster without reinventing the wheel.

---

## Links

- 🌐 **Website:** [spooled.cloud](https://spooled.cloud)
- 📖 **Docs:** [spooled.cloud/docs](https://spooled.cloud/docs)
- 💬 **Discord:** [Join our community](https://discord.gg/spooled)
- 📧 **Email:** support@spooled.cloud

---

<p align="center">
  <sub>Apache 2.0 License • Built with 🦀 Rust + ⚡ PostgreSQL + 🔴 Redis</sub>
</p>
