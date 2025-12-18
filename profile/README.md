# Spooled Cloud

Open-source webhook queue & background job infrastructure — with a hosted cloud and a full self-host option.

Spooled is a **reliable queue + worker coordination system** for:

- Webhook ingestion (custom incoming endpoints)
- Background jobs (leases, retries, DLQ)
- Workflows (DAG dependencies)
- Schedules (timezone-aware, **6-field cron** with seconds)
- Real-time visibility (WebSocket + SSE; gRPC streaming for high-throughput workers)

```
Stripe webhook → Spooled → Your worker → ✅ done (or retried automatically)
```

---

## What’s in this repo (monorepo)

This repository contains the full product:

- `spooled-backend/` — Rust backend (REST + WebSocket/SSE + gRPC), Postgres migrations, infra/docker, k8s, OpenAPI
- `spooled-dashboard/` — Web dashboard (queue/job/workflow visibility + account/billing)
- `spooled-frontend/` — Marketing site + docs (Astro)
- `spooled-sdk-nodejs/` — Node.js/TypeScript SDK (`@spooled/sdk`)
- `spooled-sdk-python/` — Python SDK (`spooled`)
- `spooled-sdk-go/` — Go SDK (`github.com/spooled-cloud/spooled-sdk-go`)
- `spooled-sdk-php/` — PHP SDK (Composer package)

For “golden” real-world examples, the SDK test scripts are the best reference:

- Node: `spooled-sdk-nodejs/scripts/test-local.ts`
- Python: `spooled-sdk-python/scripts/test_local.py`
- Go: `spooled-sdk-go/scripts/test-local/main.go`
- PHP: `spooled-sdk-php/scripts/test-local.php`

---

## Production endpoints

- Marketing + docs: `https://spooled.cloud`
- Dashboard: `https://dashboard.spooled.cloud`
- REST API (also WebSocket + SSE): `https://api.spooled.cloud`
- gRPC API: `grpc.spooled.cloud:443` (TLS)

---

## Authentication / API keys

- **Current prefixes**: `sp_live_...` (production) and `sp_test_...` (test/dev)
- **Legacy**: some tooling still accepts `sk_live_...` / `sk_test_...` (older keys)

REST auth is via:

- `Authorization: Bearer <API_KEY>`

gRPC auth is via metadata:

- `x-api-key: <API_KEY>`

---

## Quick start: enqueue a job

### REST (cURL)

```bash
curl -X POST https://api.spooled.cloud/api/v1/jobs \
  -H "Authorization: Bearer sp_live_YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "queue_name": "emails",
    "payload": { "to": "user@example.com", "template": "welcome" },
    "idempotency_key": "welcome-user-123",
    "priority": 10
  }'
```

### Node.js / TypeScript SDK

```ts
import { SpooledClient } from "@spooled/sdk";

const client = new SpooledClient({ apiKey: process.env.SPOOLED_API_KEY! });

const { id } = await client.jobs.create({
  queueName: "emails",
  payload: { to: "user@example.com", template: "welcome" },
  idempotencyKey: "welcome-user-123",
  priority: 10,
});

console.log("Job created:", id);
```

### Python SDK

```python
from spooled import SpooledClient
import os

client = SpooledClient(api_key=os.environ["SPOOLED_API_KEY"])

res = client.jobs.create({
    "queue_name": "emails",
    "payload": {"to": "user@example.com", "template": "welcome"},
    "idempotency_key": "welcome-user-123",
    "priority": 10,
})

print("Job created:", res.id)
client.close()
```

### Go SDK

```go
package main

import (
	"context"
	"fmt"
	"os"

	"github.com/spooled-cloud/spooled-sdk-go/spooled"
	"github.com/spooled-cloud/spooled-sdk-go/spooled/resources"
)

func ptr[T any](v T) *T { return &v }

func main() {
	client := spooled.NewClient(spooled.WithAPIKey(os.Getenv("SPOOLED_API_KEY")))

	resp, err := client.Jobs().Create(context.Background(), &resources.CreateJobRequest{
		QueueName:       "emails",
		Payload:         map[string]interface{}{"to": "user@example.com", "template": "welcome"},
		IdempotencyKey:  ptr("welcome-user-123"),
		Priority:        ptr(10),
	})
	if err != nil {
		panic(err)
	}

	fmt.Println("Job created:", resp.ID)
}
```

### PHP SDK

```php
<?php

declare(strict_types=1);

use Spooled\SpooledClient;
use Spooled\Config\ClientOptions;

$client = new SpooledClient(new ClientOptions(
    apiKey: getenv('SPOOLED_API_KEY'),
));

$job = $client->jobs->create([
    'queue' => 'emails',
    'payload' => ['to' => 'user@example.com', 'template' => 'welcome'],
    'idempotencyKey' => 'welcome-user-123',
    'priority' => 10,
]);

echo "Job created: {$job->id}\n";
```

---

## Bulk enqueue (up to 100 jobs / request)

### REST (cURL)

```bash
curl -X POST https://api.spooled.cloud/api/v1/jobs/bulk \
  -H "Authorization: Bearer sp_live_YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "queue_name": "emails",
    "jobs": [
      { "payload": { "to": "a@example.com" } },
      { "payload": { "to": "b@example.com" } }
    ],
    "default_priority": 5
  }'
```

---

## Incoming webhooks (custom endpoint → queued jobs)

Spooled can generate custom incoming webhook endpoints that enqueue jobs for you:

```text
POST /api/v1/webhooks/{org_id}/custom
X-Webhook-Token: whk_...
Content-Type: application/json

{ "queue_name": "payments", "payload": { ... } }
```

---

## Local development

### Prereqs

- Docker (for Postgres/Redis/PgBouncer)
- Rust (backend)
- Node.js 20+ (dashboard + marketing/docs)
- Go (Go SDK)
- Python 3.11+ (Python SDK)
- PHP 8.2+ + Composer (PHP SDK)

### 1) Backend (REST on `:8080`, gRPC on `:50051`)

```bash
cd spooled-backend

# Start infra deps (Postgres, Redis, PgBouncer)
docker compose up -d postgres redis pgbouncer

# Run the API server (REST defaults to :8080; gRPC defaults to :50051)
cargo run
```

### 2) Dashboard (defaults to `:4321`)

The dashboard talks to the backend via `PUBLIC_API_URL`/`PUBLIC_WS_URL`.

```bash
cd spooled-dashboard

# Run in Docker (defaults to :4321)
PUBLIC_API_URL=http://localhost:8080 \
PUBLIC_WS_URL=ws://localhost:8080 \
docker compose up -d
```

### 3) Marketing site + docs (Astro)

Note: both `spooled-dashboard` and `spooled-frontend` default to port `4321`, so run one on another port.

```bash
cd spooled-frontend
npm install
npm run dev -- --port 4322
```

---

## Ports (local defaults)

| Service | Default | Notes |
|---|---:|---|
| Backend REST + WS + SSE | `8080` | `PORT` env var |
| Backend gRPC | `50051` | `GRPC_PORT` env var |
| Dashboard | `4321` | `PORT` env var in docker compose |
| Marketing/docs (Astro dev) | `4321` | recommend `--port 4322` to avoid collision |
| Postgres (docker) | `5433` | maps to container `5432` |
| PgBouncer (docker) | `6432` | maps to container `6432` |
| Redis (docker) | `6379` |  |

---

## Testing

```bash
# Backend
cd spooled-backend && cargo test

# Frontend (marketing/docs)
cd spooled-frontend && npm run test && npm run lint

# SDK golden tests
cd spooled-sdk-nodejs && npm run test
cd spooled-sdk-python && python -m pytest
cd spooled-sdk-go && go test ./...
cd spooled-sdk-php && ./vendor/bin/phpunit
```

---

## Contributing

See per-package contribution guides:

- `spooled-backend/CONTRIBUTING.md`
- `spooled-sdk-go/CONTRIBUTING.md`

If you’re changing behavior, update:

- SDK `test-local.*` scripts (canonical usage)
- `spooled-frontend/src/lib/snippets.ts` (public docs snippets)

---

## License

Each package includes its own `LICENSE`. The backend and SDKs are Apache 2.0.

---

## Docs & useful references (in this repo)

- Product docs: `spooled-frontend/src/pages/docs/*`
- Backend OpenAPI: `spooled-backend/docs/openapi.yaml`
- Backend guides: `spooled-backend/docs/guides/*`
- Stripe/billing setup: `STRIPE_SETUP.md`
- Frontend implementation notes: `FRONTEND_TECHNICAL_TASK.md`

---

## Links

- Website + docs: `https://spooled.cloud`
- Dashboard: `https://dashboard.spooled.cloud`

---

## Sponsor

If Spooled saves you time (or prevents webhook outages), consider sponsoring development:

- GitHub Sponsors: `https://github.com/sponsors/spooled-cloud`
