# 🧵 Spooled Cloud

**Open-source webhook queue & background job infrastructure.**

Stop losing webhooks. Stop rebuilding retry logic. Start shipping.

---

## What is Spooled?

Spooled is a **reliable queue + worker coordination system** for:

- **Webhook ingestion** (accept webhooks, deduplicate, persist, process async)
- **Background jobs** (at-least-once delivery, leases, retries, DLQ)
- **Cron schedules** (timezone-aware, supports 6-field cron with seconds)
- **Workflows** (job dependencies / DAG execution)
- **Real-time visibility** (WebSocket + SSE; gRPC streaming for high-throughput workers)

```
Stripe webhook → Spooled → Your worker → ✓ Done (or retried automatically)
```

Spooled Cloud is available as a managed service, and the backend is open-source for self-hosting.

---

## Real-World Examples

### 🛒 E-commerce

```
Order placed → Spooled queue → [Charge payment] → [Update inventory] → [Send confirmation]
                                     ↓ (if fails)
                              Retry with backoff → DLQ for manual review
```

### 📧 SaaS Notifications (Bulk enqueue)

```bash
# Bulk enqueue (up to 100 jobs per request)
curl -X POST https://api.spooled.cloud/api/v1/jobs/bulk \
  -H "Authorization: Bearer sk_live_..." \
  -H "Content-Type: application/json" \
  -d '{
    "jobs": [
      { "queue_name": "emails", "payload": { "to": "a@example.com" } },
      { "queue_name": "emails", "payload": { "to": "b@example.com" } }
    ]
  }'
```

### 🔄 Webhook Reliability

If Stripe (or any provider) retries webhooks at 3am, Spooled keeps them durable and makes failures visible:
- automatic retries (exponential backoff)
- dead-letter queue (DLQ) for manual review/replay
- delivery history and real-time events

### ⏰ Cron (Timezone-aware, Second Precision)

```json
{ "cron_expression": "0 */5 * * * *", "timezone": "America/New_York" }
```

---

## Why Spooled?

| Feature | Spooled | DIY Redis/SQS |
|---------|---------|---------------|
| Automatic retries | ✅ Built-in | 🔧 Build it |
| Idempotency | ✅ Built-in | 🔧 Build it |
| Dead-letter queue | ✅ Built-in | 🔧 Build it |
| Real-time streaming | ✅ SSE + WebSocket | 🔧 Build it |
| High-throughput workers | ✅ gRPC + streaming | 🔧 Build it |
| Workflows with dependencies | ✅ Built-in | 🔧 Build it |
| Dashboard | ✅ Included | ❌ |

---

## SDKs

- **Node.js / TypeScript**: `@spooled/sdk` ([repo](https://github.com/Spooled-Cloud/spooled-sdk-nodejs))
- **Python**: `spooled` ([repo](https://github.com/Spooled-Cloud/spooled-sdk-python))
- **Go**: coming soon (use REST/gRPC directly for now)

---

## Repositories

| Repo | Description |
|------|-------------|
| [spooled-backend](https://github.com/Spooled-Cloud/spooled-backend) | 🦀 Rust API server (REST + gRPC) |
| [spooled-frontend](https://github.com/Spooled-Cloud/spooled-frontend) | 🌐 Marketing site & docs |
| [spooled-dashboard](https://github.com/Spooled-Cloud/spooled-dashboard) | 📊 Admin dashboard |
| [spooled-sdk-nodejs](https://github.com/Spooled-Cloud/spooled-sdk-nodejs) | 📦 TypeScript/Node.js SDK |
| [spooled-sdk-python](https://github.com/Spooled-Cloud/spooled-sdk-python) | 🐍 Python SDK |

---

## Quick Start

### REST (cURL)

```bash
curl -X POST https://api.spooled.cloud/api/v1/jobs \
  -H "Authorization: Bearer sk_live_..." \
  -H "Content-Type: application/json" \
  -d '{
    "queue_name": "emails",
    "payload": { "to": "user@example.com", "template": "welcome" },
    "idempotency_key": "welcome-user-123"
  }'
```

### Node.js SDK

```ts
import { SpooledClient } from '@spooled/sdk';

const client = new SpooledClient({ apiKey: process.env.SPOOLED_API_KEY! });
const { id } = await client.jobs.create({
  queueName: 'emails',
  payload: { to: 'user@example.com', template: 'welcome' },
  idempotencyKey: 'welcome-user-123',
});
console.log('Job created:', id);
```

### Python SDK

```python
from spooled import SpooledClient

client = SpooledClient(api_key="sk_live_...")
res = client.jobs.create({
    "queue_name": "emails",
    "payload": {"to": "user@example.com", "template": "welcome"},
    "idempotency_key": "welcome-user-123",
})
print("Job created:", res.id)
client.close()
```

---

## 💖 Sponsor Us

Building reliable infrastructure is hard. We’re a small team making webhooks and background jobs **just work** for everyone.

**Your sponsorship helps us:**
- Keep the open-source version free forever
- Add more SDKs (Go, PHP, …)
- Improve documentation and examples
- Build features the community requests

👉 **[Become a Sponsor](https://github.com/sponsors/Spooled-Cloud)**

---

## Links

- **Website**: [spooled.cloud](https://spooled.cloud)
- **Docs**: [spooled.cloud/docs](https://spooled.cloud/docs)

---

<p align="center">
  <sub>Apache 2.0 License • Built with 🦀 Rust + ⚡ PostgreSQL + 🔴 Redis</sub>
</p>

