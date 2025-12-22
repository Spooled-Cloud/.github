<picture>
  <img alt="Spooled" src="https://spooled.cloud/logo-horizontal.webp" width="380">
</picture>

# Spooled Cloud

### Open-source webhook queue & background job infrastructure

[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)
[![Go SDK](https://img.shields.io/badge/Go_SDK-v1.0.5-00ADD8?logo=go)](https://github.com/Spooled-Cloud/spooled-sdk-go)
[![Node SDK](https://img.shields.io/npm/v/@spooled/sdk?label=Node%20SDK&logo=npm)](https://www.npmjs.com/package/@spooled/sdk)
[![Python SDK](https://img.shields.io/pypi/v/spooled?label=Python%20SDK&logo=python)](https://pypi.org/project/spooled/)
[![PHP SDK](https://img.shields.io/packagist/v/spooled-cloud/spooled?label=PHP%20SDK&logo=php)](https://packagist.org/packages/spooled-cloud/spooled)

---

## 🤔 What is Spooled?

**Spooled is like a super-reliable to-do list for your servers.**

Imagine you have a website that needs to:
- Send welcome emails when users sign up
- Process payments from Stripe
- Generate PDF reports
- Resize uploaded images

These tasks can fail (email server down, payment timeout, etc.). Spooled makes sure they **always get done** — with automatic retries, scheduling, and real-time monitoring.

```
┌──────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│   YOUR APP             SPOOLED                       YOUR WORKERS        │
│   ────────             ───────                       ────────────        │
│                                                                          │
│  ┌──────────┐        ┌──────────────────┐           ┌─────────────┐      │
│  │   User   │───────▶│    Job Queue     │──────────▶│   Worker    │      │
│  │   signs  │ Enqueue│                  │   Claim   │   sends     │      │
│  │    up    │   job  │  ┌───┐┌───┐┌───┐ │    job    │   welcome   │      │
│  └──────────┘        │  │ J ││ J ││ J │ │           │    email    │      │
│                      │  └───┘└───┘└───┘ │           └──────┬──────┘      │
│                      │                  │                  │             │
│  ┌──────────┐        │    * Retries     │           ┌──────▼──────┐      │
│  │  Stripe  │───────▶│    * Priority    │           │    DONE!    │      │
│  │  webhook │        │    * Schedules   │           │     or      │      │
│  └──────────┘        │    * Dead-letter │           │    Retry    │      │
│                      └──────────────────┘           └─────────────┘      │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## ✨ Key Features

| Feature | Description |
|---------|-------------|
| 📬 **Job Queues** | Reliable FIFO queues with priority support (0-100) |
| 🔄 **Automatic Retries** | Configurable retry policies with exponential backoff |
| ⏰ **Schedules** | Cron-based scheduling with timezone support (6-field with seconds!) |
| 🔀 **Workflows** | DAG-based job dependencies (run job B after job A completes) |
| 🪝 **Webhooks** | Receive webhooks from Stripe, GitHub, etc. → auto-queue as jobs |
| 📊 **Real-time Dashboard** | See all your jobs, queues, and workers live |
| 🚀 **High Performance** | gRPC streaming for workers processing 1000s of jobs/sec |
| ☠️ **Dead Letter Queue** | Failed jobs are saved for inspection and manual retry |

---

## 🚀 Quick Start (5 minutes)

### Step 1: Get an API Key

1. Go to [dashboard.spooled.cloud](https://dashboard.spooled.cloud)
2. Sign up / Log in
3. Copy your API key (starts with `sk_live_...` or `sk_test_...`)

### Step 2: Create Your First Job

Choose your language:

<details>
<summary><b>🟢 Node.js / TypeScript</b></summary>

```bash
npm install @spooled/sdk
```

```typescript
import { SpooledClient } from "@spooled/sdk";

const client = new SpooledClient({ 
  apiKey: "sk_live_YOUR_API_KEY" 
});

// Create a job
const job = await client.jobs.create({
  queueName: "emails",
  payload: { 
    to: "user@example.com", 
    subject: "Welcome!" 
  }
});

console.log("✅ Job created:", job.id);
```
</details>

<details>
<summary><b>🐍 Python</b></summary>

```bash
pip install spooled
```

```python
from spooled import SpooledClient

client = SpooledClient(api_key="sk_live_YOUR_API_KEY")

# Create a job
job = client.jobs.create({
    "queue_name": "emails",
    "payload": {
        "to": "user@example.com",
        "subject": "Welcome!"
    }
})

print(f"✅ Job created: {job.id}")
client.close()
```
</details>

<details>
<summary><b>🔵 Go</b></summary>

```bash
go get github.com/spooled-cloud/spooled-sdk-go
```

```go
package main

import (
    "context"
    "fmt"
    "github.com/spooled-cloud/spooled-sdk-go/spooled"
    "github.com/spooled-cloud/spooled-sdk-go/spooled/resources"
)

func main() {
    client := spooled.NewClient(
        spooled.WithAPIKey("sk_live_YOUR_API_KEY"),
    )

    job, _ := client.Jobs().Create(context.Background(), &resources.CreateJobRequest{
        QueueName: "emails",
        Payload: map[string]any{
            "to":      "user@example.com",
            "subject": "Welcome!",
        },
    })

    fmt.Println("✅ Job created:", job.ID)
}
```
</details>

<details>
<summary><b>🐘 PHP</b></summary>

```bash
composer require spooled-cloud/spooled
```

```php
<?php
use Spooled\SpooledClient;
use Spooled\Config\ClientOptions;

$client = new SpooledClient(new ClientOptions(
    apiKey: 'sk_live_YOUR_API_KEY'
));

$job = $client->jobs->create([
    'queue' => 'emails',
    'payload' => [
        'to' => 'user@example.com',
        'subject' => 'Welcome!'
    ]
]);

echo "✅ Job created: {$job->id}\n";
```
</details>

<details>
<summary><b>🌐 cURL (REST API)</b></summary>

```bash
curl -X POST https://api.spooled.cloud/api/v1/jobs \
  -H "Authorization: Bearer sk_live_YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "queue_name": "emails",
    "payload": {
      "to": "user@example.com",
      "subject": "Welcome!"
    }
  }'
```
</details>

### Step 3: Process Jobs (Worker)

Your worker claims jobs, processes them, and marks them complete:

<details>
<summary><b>🟢 Node.js Worker</b></summary>

```typescript
import { SpooledWorker } from "@spooled/sdk";

const worker = new SpooledWorker({
  apiKey: "sk_live_YOUR_API_KEY",
  queueName: "emails",
  handler: async (job) => {
    // Your business logic here
    console.log("Processing:", job.payload);
    
    await sendEmail(job.payload.to, job.payload.subject);
    
    return { success: true };
  }
});

worker.start();
console.log("🚀 Worker started, waiting for jobs...");
```
</details>

<details>
<summary><b>🐍 Python Worker</b></summary>

```python
from spooled import SpooledWorker

def handle_job(job):
    print(f"Processing: {job.payload}")
    send_email(job.payload["to"], job.payload["subject"])
    return {"success": True}

worker = SpooledWorker(
    api_key="sk_live_YOUR_API_KEY",
    queue_name="emails",
    handler=handle_job
)

print("🚀 Worker started, waiting for jobs...")
worker.start()
```
</details>

<details>
<summary><b>🔵 Go Worker</b></summary>

```go
package main

import (
    "context"
    "fmt"
    "github.com/spooled-cloud/spooled-sdk-go/spooled"
    "github.com/spooled-cloud/spooled-sdk-go/spooled/worker"
)

func main() {
    w := spooled.NewSpooledWorker(
        "sk_live_YOUR_API_KEY",
        "emails",
        func(ctx context.Context, job *worker.Job) (map[string]any, error) {
            fmt.Println("Processing:", job.Payload)
            // Your business logic here
            return map[string]any{"success": true}, nil
        },
    )

    fmt.Println("🚀 Worker started, waiting for jobs...")
    w.Start(context.Background())
}
```
</details>

---

## 🏗️ Architecture Overview

```
                     ┌──────────────────────────────────────────┐
                     │              SPOOLED CLOUD               │
                     │                                          │
 ┌──────────┐        │    ┌────────────────────────────────┐    │        ┌──────────┐
 │   Your   │  REST  │    │           API Server           │    │  REST  │   Your   │
 │   App    │───────▶│    │    (Rust, high-performance)    │◀───┼────────│  Workers │
 └──────────┘        │    │                                │    │        └──────────┘
                     │    │    * Job management            │    │
 ┌──────────┐        │    │    * Queue operations          │    │        ┌──────────┐
 │  Stripe  │  HTTP  │    │    * Webhook ingestion         │    │  gRPC  │ High-vol │
 │  GitHub  │───────▶│    │    * Schedule execution        │    │◀───────│  Workers │
 │   etc.   │        │    └───────────────┬────────────────┘    │        └──────────┘
 └──────────┘        │                    │                     │
                     │                    ▼                     │
                     │    ┌────────────────────────────────┐    │
                     │    │       PostgreSQL + Redis       │    │
                     │    │  (Durable storage + caching)   │    │
                     │    └────────────────────────────────┘    │
                     │                                          │
                     │    ┌────────────────────────────────┐    │        ┌──────────┐
                     │    │           Dashboard            │    │  WS/   │   You    │
                     │    │   (Real-time job monitoring)   │◀───┼──SSE───│ watching │
                     │    └────────────────────────────────┘    │        └──────────┘
                     └──────────────────────────────────────────┘
```

---

## 📦 What's in This Repository

This is a **monorepo** containing the entire Spooled platform:

```
spooled-cloud/
├── spooled-backend/       # 🦀 Rust API server (REST + gRPC + WebSocket)
├── spooled-dashboard/     # ⚛️  React dashboard (job monitoring, billing)
├── spooled-frontend/      # 🚀 Astro marketing site + documentation
├── spooled-sdk-nodejs/    # 🟢 Node.js/TypeScript SDK
├── spooled-sdk-python/    # 🐍 Python SDK
├── spooled-sdk-go/        # 🔵 Go SDK
└── spooled-sdk-php/       # 🐘 PHP SDK
```

### SDK Status

| SDK | Package | Status | Tests |
|-----|---------|--------|-------|
| Node.js | `@spooled/sdk` | ✅ Production Ready | 175 tests |
| Python | `spooled` | ✅ Production Ready | 173 tests |
| Go | `github.com/spooled-cloud/spooled-sdk-go` | ✅ Production Ready | 194 tests |
| PHP | `spooled-cloud/spooled` | ✅ Production Ready | 175 tests |

---

## 🔑 Authentication

### API Key Formats

| Prefix | Environment | Usage |
|--------|-------------|-------|
| `sk_live_...` | Production | Real data, billing enabled |
| `sk_test_...` | Test/Dev | Safe for testing, no billing |

> **Note**: Documentation uses `sp_live_`/`sp_test_` prefixes to avoid GitHub secret scanning. All SDKs accept both formats.

### How to Authenticate

**REST API** — Use Bearer token in header:
```http
Authorization: Bearer sk_live_YOUR_API_KEY
```

**gRPC API** — Use metadata:
```
x-api-key: sk_live_YOUR_API_KEY
```

---

## 🌐 Production Endpoints

| Service | URL | Protocol |
|---------|-----|----------|
| 🌍 Website & Docs | https://spooled.cloud | HTTPS |
| 📊 Dashboard | https://dashboard.spooled.cloud | HTTPS |
| 🔌 REST API | https://api.spooled.cloud | HTTPS |
| ⚡ gRPC API | grpc.spooled.cloud:443 | gRPC + TLS |

---

## 💻 Local Development

### Prerequisites

| Tool | Version | Purpose |
|------|---------|---------|
| Docker | Latest | Database & cache containers |
| Rust | 1.70+ | Backend server |
| Node.js | 20+ | Dashboard & frontend |
| Go | 1.21+ | Go SDK |
| Python | 3.11+ | Python SDK |
| PHP | 8.2+ | PHP SDK |

### Start the Backend

```bash
# 1. Clone and enter the repo
git clone https://github.com/Spooled-Cloud/spooled-cloud.git
cd spooled-cloud/spooled-backend

# 2. Start infrastructure (Postgres, Redis)
docker compose up -d postgres redis pgbouncer

# 3. Run the API server
cargo run

# ✅ REST API available at http://localhost:8080
# ✅ gRPC API available at localhost:50051
```

### Start the Dashboard

```bash
cd spooled-dashboard

PUBLIC_API_URL=http://localhost:8080 \
PUBLIC_WS_URL=ws://localhost:8080 \
docker compose up -d

# ✅ Dashboard available at http://localhost:4321
```

### Start the Docs Site

```bash
cd spooled-frontend
npm install
npm run dev -- --port 4322

# ✅ Docs available at http://localhost:4322
```

### Local Ports Reference

| Service | Port | Environment Variable |
|---------|------|---------------------|
| Backend REST/WS/SSE | 8080 | `PORT` |
| Backend gRPC | 50051 | `GRPC_PORT` |
| Dashboard | 4321 | `PORT` |
| Docs (Astro) | 4322 | Use `--port` flag |
| PostgreSQL | 5433 | Docker mapped |
| PgBouncer | 6432 | Docker mapped |
| Redis | 6379 | Docker mapped |

---

## 🧪 Running Tests

```bash
# Backend tests (Rust)
cd spooled-backend && cargo test

# Frontend tests
cd spooled-frontend && npm test && npm run lint

# SDK integration tests (the "golden" reference tests)
cd spooled-sdk-nodejs && npm test
cd spooled-sdk-python && python -m pytest
cd spooled-sdk-go && go test ./...
cd spooled-sdk-php && ./vendor/bin/phpunit
```

---

## 📚 Common Use Cases

### 1. Process Stripe Webhooks

```typescript
// Stripe sends webhook → Spooled queues it → Your worker processes it
const job = await client.jobs.create({
  queueName: "stripe-webhooks",
  payload: stripeEvent,
  idempotencyKey: stripeEvent.id, // Prevents duplicates!
});
```

### 2. Send Emails in Background

```typescript
// Don't make users wait for email sending
const job = await client.jobs.create({
  queueName: "emails",
  payload: { to: user.email, template: "welcome" },
  priority: 10, // Higher = processed first
});
```

### 3. Schedule Daily Reports

```typescript
// Run every day at 9 AM EST
const schedule = await client.schedules.create({
  name: "daily-report",
  queueName: "reports",
  cronExpression: "0 0 9 * * *", // 6-field cron (with seconds!)
  timezone: "America/New_York",
  payload: { type: "daily-summary" },
});
```

### 4. Workflow Dependencies

```typescript
// Job B runs only after Job A completes
const workflow = await client.workflows.create({
  name: "user-onboarding",
  jobs: [
    { id: "create-account", queueName: "accounts", payload: {...} },
    { id: "send-welcome", queueName: "emails", payload: {...}, dependsOn: ["create-account"] },
    { id: "setup-billing", queueName: "billing", payload: {...}, dependsOn: ["create-account"] },
  ]
});
```

---

## 🤝 Contributing

We love contributions! See the contribution guides:

- [Backend Contributing Guide](spooled-backend/CONTRIBUTING.md)
- [Go SDK Contributing Guide](spooled-sdk-go/CONTRIBUTING.md)

### Key Files to Update

When changing behavior, remember to update:
- SDK `test-local.*` scripts (canonical usage examples)
- `spooled-frontend/src/lib/snippets.ts` (docs code snippets)

---

## 📖 Documentation

| Resource | Location |
|----------|----------|
| 📚 Product Docs | `spooled-frontend/src/pages/docs/*` |
| 📋 OpenAPI Spec | `spooled-backend/docs/openapi.yaml` |
| 📝 Backend Guides | `spooled-backend/docs/guides/*` |
| 💳 Stripe Setup | `STRIPE_SETUP.md` |

---

## 📜 License

Apache 2.0 — See [LICENSE](LICENSE) for details.

---

## 💖 Support the Project

If Spooled saves you time or prevents webhook outages:

- ⭐ **Star this repo** — It helps others find us!
- 💝 **[Sponsor on GitHub](https://github.com/sponsors/spooled-cloud)** — Support development

---

<p align="center">
  <b>Built with ❤️ for developers who hate losing webhooks</b>
  <br><br>
  <a href="https://spooled.cloud">Website</a> •
  <a href="https://dashboard.spooled.cloud">Dashboard</a> •
  <a href="https://spooled.cloud/docs">Documentation</a>
</p>
