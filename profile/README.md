<p align="center">
  <img alt="Spooled Cloud" src="https://spooled.cloud/brand/primary/logo-horizontal-emerald-dark.png#gh-dark-mode-only" width="360">
  <img alt="Spooled Cloud" src="https://spooled.cloud/brand/primary/logo-horizontal-emerald-light.png#gh-light-mode-only" width="360">
</p>

<h3 align="center">Open-source job queue on PostgreSQL</h3>

<p align="center">
  Durable jobs, automatic retries, dead-letter queues, cron schedules, DAG workflows
  <br>and real-time streaming — over REST and gRPC.
</p>

<p align="center">
  <a href="https://spooled.cloud">Website</a> ·
  <a href="https://spooled.cloud/docs">Docs</a> ·
  <a href="https://example.spooled.cloud"><b>Live demo</b></a> ·
  <a href="https://dashboard.spooled.cloud">Dashboard</a>
</p>

<p align="center">
  <a href="LICENSE"><img alt="License" src="https://img.shields.io/badge/license-Apache%202.0-blue.svg"></a>
  <a href="https://www.npmjs.com/package/@spooled/sdk"><img alt="Node SDK" src="https://img.shields.io/npm/v/@spooled/sdk?label=node&logo=npm"></a>
  <a href="https://pypi.org/project/spooled/"><img alt="Python SDK" src="https://img.shields.io/pypi/v/spooled?label=python&logo=python&logoColor=white"></a>
  <a href="https://github.com/Spooled-Cloud/spooled-sdk-go"><img alt="Go SDK" src="https://img.shields.io/github/v/release/Spooled-Cloud/spooled-sdk-go?label=go&logo=go&color=00ADD8"></a>
  <a href="https://packagist.org/packages/spooled-cloud/spooled"><img alt="PHP SDK" src="https://img.shields.io/packagist/v/spooled-cloud/spooled?label=php&logo=php&logoColor=white"></a>
</p>

---

## What it is

Your app has work that shouldn't happen during a request: sending the welcome
email, fulfilling a Stripe order, resizing an upload, generating a report. That
work fails in ways a request cannot recover from — the mail provider is down,
the payment API times out, the box restarts mid-job.

Spooled is where that work waits. You enqueue a job over REST or gRPC, your own
workers claim it and run it, and Spooled handles what happens when it doesn't
go cleanly: retry with exponential backoff, park it in a dead-letter queue after
the last attempt, and stream every state change so you can watch it happen.

Jobs live in PostgreSQL, so they survive a restart and you can query them with
SQL you already know. **Your code runs on your infrastructure** — Spooled stores
and schedules the work, it never executes it.

```
   your app                    spooled                      your workers
   ────────                    ───────                      ────────────

  enqueue  ──────────▶  ┌──────────────────┐  ◀──────────  claim
                        │   queue (Postgres)│
  webhook  ──────────▶  │  ● ● ● ● ●        │  ──────────▶  run your code
  (Stripe, GitHub)      │                   │
                        │  retries · priority│  ◀──────────  complete / fail
  cron ──────────────▶  │  schedules · DLQ   │
                        └─────────┬─────────┘
                                  │
                          state streams back
                          (WebSocket / SSE)
```

**[See it running →](https://example.spooled.cloud)** — SpriteForge builds an
animated sprite from parallel frame jobs, with retries and live events you can
watch. It is a real workload against production, not a mock.

---

## Quick start

Get a key at [dashboard.spooled.cloud](https://dashboard.spooled.cloud), then:

<details open>
<summary><b>Node.js / TypeScript</b></summary>

```bash
npm install @spooled/sdk
```

```typescript
import { SpooledClient, SpooledWorker } from "@spooled/sdk";

const client = new SpooledClient({ apiKey: process.env.SPOOLED_API_KEY! });

// Enqueue — returns as soon as the job is durable.
await client.jobs.create({
  queueName: "emails",
  payload: { to: "user@example.com", subject: "Welcome!" },
  idempotencyKey: `welcome:${user.id}`, // retrying this call won't double-send
});

// Process — runs wherever you run it.
new SpooledWorker({
  apiKey: process.env.SPOOLED_API_KEY!,
  queueName: "emails",
  handler: async (job) => {
    await sendEmail(job.payload.to, job.payload.subject);
    return { sent: true };
  },
}).start();
```

</details>

<details>
<summary><b>Python</b></summary>

```bash
pip install spooled
```

```python
import os
from spooled import SpooledClient, SpooledWorker

client = SpooledClient(api_key=os.environ["SPOOLED_API_KEY"])

client.jobs.create({
    "queue_name": "emails",
    "payload": {"to": "user@example.com", "subject": "Welcome!"},
    "idempotency_key": f"welcome:{user.id}",
})

def handle(job):
    send_email(job.payload["to"], job.payload["subject"])
    return {"sent": True}

SpooledWorker(
    api_key=os.environ["SPOOLED_API_KEY"],
    queue_name="emails",
    handler=handle,
).start()
```

</details>

<details>
<summary><b>Go</b></summary>

```bash
go get github.com/spooled-cloud/spooled-sdk-go
```

```go
client := spooled.NewClient(spooled.WithAPIKey(os.Getenv("SPOOLED_API_KEY")))

client.Jobs().Create(ctx, &resources.CreateJobRequest{
    QueueName:      "emails",
    Payload:        map[string]any{"to": "user@example.com", "subject": "Welcome!"},
    IdempotencyKey: "welcome:" + user.ID,
})

w := spooled.NewSpooledWorker(
    os.Getenv("SPOOLED_API_KEY"), "emails",
    func(ctx context.Context, job *worker.Job) (map[string]any, error) {
        return map[string]any{"sent": true}, sendEmail(job.Payload)
    },
)
w.Start(ctx)
```

</details>

<details>
<summary><b>PHP</b></summary>

```bash
composer require spooled-cloud/spooled
```

```php
use Spooled\SpooledClient;
use Spooled\Config\ClientOptions;

$client = new SpooledClient(new ClientOptions(apiKey: getenv('SPOOLED_API_KEY')));

$client->jobs->create([
    'queue_name'      => 'emails',
    'payload'         => ['to' => 'user@example.com', 'subject' => 'Welcome!'],
    'idempotency_key' => "welcome:{$user->id}",
]);
```

</details>

<details>
<summary><b>REST (any language)</b></summary>

```bash
curl -X POST https://api.spooled.cloud/api/v1/jobs \
  -H "Authorization: Bearer $SPOOLED_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "queue_name": "emails",
    "payload": { "to": "user@example.com", "subject": "Welcome!" },
    "idempotency_key": "welcome:123"
  }'
```

</details>

---

## What you get

| | |
|---|---|
| **Durable queues** | Jobs in PostgreSQL, with priority (0–100) and FIFO within a priority |
| **Automatic retries** | Exponential backoff, configurable max attempts |
| **Dead-letter queue** | Exhausted jobs are kept for inspection and replay, not dropped |
| **Idempotency keys** | The same key enqueues one job, however many times you call |
| **Cron schedules** | 5-field cron, or 6-field with a leading seconds column, with timezones |
| **Workflows** | DAG dependencies — run B and C after A completes |
| **Incoming webhooks** | Endpoints that turn a Stripe or GitHub delivery into a job |
| **Outgoing webhooks** | Delivery on job events, HMAC-signed when you set a secret |
| **Real-time** | WebSocket and SSE streams of job, queue and worker state |
| **gRPC streaming** | Bidirectional `ProcessJobs` for workers that pull continuously |

---

## Repositories

Each of these is its **own repository** with its own releases — they are
developed side by side but versioned independently. (The marketing site and
documentation live in a private repository; the docs themselves are public at
[spooled.cloud/docs](https://spooled.cloud/docs).)

| Repository | What it is |
|---|---|
| [spooled-backend](https://github.com/Spooled-Cloud/spooled-backend) | Rust API server — REST, gRPC, WebSocket, scheduler, workers |
| [spooled-dashboard](https://github.com/Spooled-Cloud/spooled-dashboard) | Job, queue and worker monitoring UI |
| [spooled-sdk-nodejs](https://github.com/Spooled-Cloud/spooled-sdk-nodejs) | Node.js / TypeScript client |
| [spooled-sdk-python](https://github.com/Spooled-Cloud/spooled-sdk-python) | Python client |
| [spooled-sdk-go](https://github.com/Spooled-Cloud/spooled-sdk-go) | Go client |
| [spooled-sdk-php](https://github.com/Spooled-Cloud/spooled-sdk-php) | PHP client |
| [spooled-example-spriteforge](https://github.com/Spooled-Cloud/spooled-example-spriteforge) | The live demo — a full app built on Spooled |

---

## Authentication

API keys are issued with an `sp_live_` or `sp_test_` prefix. Keys with the older
`sk_` prefix are still accepted, so existing integrations keep working.

```http
Authorization: Bearer sp_live_...      # REST
```

```
x-api-key: sp_live_...                 # gRPC metadata
```

A key is shown **once**, at creation. Keys can be scoped to specific queues, and
a scoped key cannot mint a broader one.

> Job payloads are stored as plain JSONB and are **not** encrypted at the
> application layer. Keep secrets out of payloads — pass a reference and resolve
> it in your worker. See [Security](https://spooled.cloud/security).

---

## Endpoints

| | |
|---|---|
| REST API | `https://api.spooled.cloud` |
| gRPC | `grpc.spooled.cloud:443` (TLS) |
| Dashboard | `https://dashboard.spooled.cloud` |
| Docs | `https://spooled.cloud/docs` |
| Status | `https://spooled.cloud/status` |

---

## Self-hosting

The core is Apache-2.0 and runs on your own PostgreSQL. You need Postgres and
Redis; everything else is the one Rust binary.

```bash
git clone https://github.com/Spooled-Cloud/spooled-backend.git
cd spooled-backend
docker compose up -d postgres redis
cargo run
# REST on :8080, gRPC on :50051
```

Full guide: **[spooled.cloud/docs/deployment](https://spooled.cloud/docs/deployment)**.
There is no feature difference between self-hosted and hosted — the hosted
service is the same code, operated for you.

---

## Contributing

Issues and pull requests are welcome on any of the repositories above. Each
carries its own `CONTRIBUTING.md` with its test and release process.

When you change a public contract, the SDK `test-local.*` scripts and
`spooled-frontend/src/lib/snippets.ts` are the places docs and examples are
generated from — keeping them in step is what stops the docs drifting.

---

## License and ownership

The open-source code is Apache-2.0, copyright Yevhen Salitrynskyi — see
[LICENSE](LICENSE).

**Spooled Cloud**, the hosted service, is operated by **YS Progress Inc.**
(Ontario, Canada) under its own [Terms](https://spooled.cloud/terms) and
[Privacy Policy](https://spooled.cloud/privacy). You can self-host the
open-source core with no relationship to the hosted service.

---

<p align="center">
  <sub>
    ⭐ Star a repo if this saved you a queue you didn't want to build ·
    <a href="https://github.com/sponsors/ysalitrynskyi">Sponsor</a>
  </sub>
</p>
