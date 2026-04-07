# LIPILY — Local Development Setup Guide

> **Purpose:** Get all three services running locally from zero. Follow every step in order.  
> **OS:** macOS / Linux / WSL2 on Windows. PowerShell notes included where different.  
> **Last updated:** 2026-04-07

---

## Prerequisites

Install these tools before anything else:

| Tool | Version | Install |
|---|---|---|
| Rust | 1.76+ | `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs \| sh` |
| Node.js | 20 LTS | `nvm install 20` |
| Flutter | 3.x stable | https://flutter.dev/docs/get-started/install |
| Docker Desktop | Latest | https://www.docker.com/products/docker-desktop |
| Supabase CLI | Latest | `npm install -g supabase` |
| `sqlx-cli` | 0.7+ | `cargo install sqlx-cli --no-default-features --features rustls,postgres` |

---

## Step 1 — Clone & Structure

```
lipily/
├── api/          ← Rust + Axum backend
├── mcp/          ← TypeScript MCP service
├── app/          ← Flutter app
├── supabase/     ← Supabase migrations & config
├── Makefile      ← All common commands
└── docker-compose.yml
```

---

## Step 2 — Start Local Infrastructure (Docker)

The `docker-compose.yml` starts: Supabase (PostgreSQL + Auth + Realtime + Storage), Redis (Upstash-compatible), MinIO (S3-compatible local), and Supabase Studio.

```yaml
# docker-compose.yml
version: '3.8'
services:
  supabase:
    image: supabase/postgres:15.1
    ports:
      - "54322:5432"
    environment:
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: postgres

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  minio:
    image: minio/minio
    ports:
      - "9000:9000"
      - "9001:9001"
    environment:
      MINIO_ROOT_USER: test
      MINIO_ROOT_PASSWORD: testpassword
    command: server /data --console-address ":9001"
```

```bash
# Start all infrastructure
docker compose up -d

# Verify all containers are running
docker compose ps
```

---

## Step 3 — Run Supabase Migrations

```bash
cd supabase/

# Initialize Supabase CLI (first time only)
supabase init

# Link to local stack
supabase start

# Run all migrations in order
supabase db push

# Verify migrations ran correctly
supabase db diff   # Should show no diff
```

Migrations live in `supabase/migrations/` and must follow the naming convention:  
`YYYYMMDDHHMMSS_description.sql`

Run them in the order defined in `docs/data-model.md` § Migration Order.

---

## Step 4 — Set Up the Rust API

```bash
cd api/

# Copy env file
cp .env.example .env
# Edit .env — fill in any missing values

# Prepare sqlx (compile-time query checking against local DB)
DATABASE_URL="postgresql://postgres:postgres@localhost:54322/postgres" \
  cargo sqlx prepare --workspace

# Build (first build takes ~3-5 minutes)
cargo build

# Run
cargo run

# Verify API is healthy
curl http://localhost:8080/health
# Expected: { "status": "ok", "version": "0.1.0" }
```

### Rust project structure

```
api/
├── src/
│   ├── main.rs              ← Axum app setup, router
│   ├── config.rs            ← AppConfig from env vars
│   ├── db.rs                ← sqlx pool setup
│   ├── auth.rs              ← JWT extraction middleware
│   ├── errors.rs            ← AppError type + Into<Response>
│   ├── handlers/
│   │   ├── scripts.rs
│   │   ├── blocks.rs
│   │   ├── collaboration.rs
│   │   ├── ai.rs
│   │   ├── export.rs
│   │   ├── social.rs
│   │   ├── subscriptions.rs
│   │   ├── notifications.rs
│   │   ├── admin.rs
│   │   └── internal.rs      ← Service-to-service endpoints
│   ├── ws/
│   │   └── collaboration.rs ← Yjs WebSocket handler
│   └── services/
│       ├── s3.rs
│       ├── stripe.rs
│       ├── resend.rs
│       └── redis.rs
├── Cargo.toml
└── .env.example
```

---

## Step 5 — Set Up the MCP Service

```bash
cd mcp/

# Install dependencies
npm install

# Copy env file
cp .env.example .env
# Edit .env — add your OpenRouter API key

# Build TypeScript
npm run build

# Run in development (with hot reload)
npm run dev

# Verify health
curl http://localhost:8081/health
# Expected: { "status": "ok" }
```

### MCP project structure

```
mcp/
├── src/
│   ├── index.ts             ← Express app setup
│   ├── config.ts            ← Config from env vars
│   ├── queues/
│   │   ├── scene-intelligence.queue.ts
│   │   ├── listen-pipeline.queue.ts
│   │   ├── continuity-check.queue.ts
│   │   └── ai-features.queue.ts
│   ├── workers/
│   │   ├── scene-intelligence.worker.ts
│   │   ├── listen-pipeline.worker.ts
│   │   └── ai-features.worker.ts
│   ├── prompts/
│   │   ├── scene-intelligence.prompt.ts
│   │   ├── dialogue-suggest.prompt.ts
│   │   ├── continue-scene.prompt.ts
│   │   └── character-voice.prompt.ts
│   └── lib/
│       ├── anthropic.ts     ← OpenRouter Anthropic client
│       ├── s3.ts
│       └── internal-api.ts  ← Rust API caller
├── package.json
├── tsconfig.json
└── .env.example
```

---

## Step 6 — Set Up the Flutter App

```bash
cd app/

# Get all dependencies
flutter pub get

# Run on web (recommended for development)
flutter run -d chrome \
  --dart-define=SUPABASE_URL=http://localhost:54321 \
  --dart-define=SUPABASE_ANON_KEY=your-local-anon-key \
  --dart-define=API_BASE_URL=http://localhost:8080 \
  --dart-define=ENVIRONMENT=development

# Or use the Makefile shortcut:
make dev-flutter
```

### Flutter project structure

```
app/
├── lib/
│   ├── main.dart
│   ├── config/
│   │   └── app_config.dart
│   ├── features/
│   │   ├── auth/
│   │   ├── dashboard/
│   │   ├── editor/
│   │   │   ├── widgets/
│   │   │   │   ├── block_widget.dart
│   │   │   │   ├── scene_navigator.dart
│   │   │   │   └── ai_panel.dart
│   │   │   ├── providers/
│   │   │   └── models/
│   │   ├── collaboration/
│   │   ├── social/
│   │   ├── listen/
│   │   ├── subscription/
│   │   └── profile/
│   ├── shared/
│   │   ├── providers/
│   │   │   ├── auth_provider.dart
│   │   │   └── subscription_provider.dart
│   │   ├── widgets/
│   │   └── theme/
│   └── routing/
│       └── app_router.dart
├── test/
├── pubspec.yaml
└── .dart_defines/
    ├── development.env
    └── production.env
```

---

## Step 7 — Makefile Commands

```makefile
# Makefile

.PHONY: dev dev-api dev-mcp dev-flutter db-migrate db-reset test

dev: ## Start all services in parallel
	make -j3 dev-api dev-mcp dev-flutter

dev-api: ## Start Rust API
	cd api && cargo run

dev-mcp: ## Start MCP service
	cd mcp && npm run dev

dev-flutter: ## Start Flutter web
	cd app && flutter run -d chrome \
		--dart-define-from-file=.dart_defines/development.env

db-migrate: ## Run all pending migrations
	cd supabase && supabase db push

db-reset: ## Reset local DB and re-run all migrations
	cd supabase && supabase db reset

db-prepare: ## Re-generate sqlx offline query cache
	cd api && DATABASE_URL=$$DATABASE_URL cargo sqlx prepare --workspace

test: ## Run all tests
	make test-api test-mcp test-flutter

test-api: ## Run Rust tests
	cd api && cargo test

test-mcp: ## Run TypeScript tests
	cd mcp && npm test

test-flutter: ## Run Flutter tests
	cd app && flutter test

lint: ## Lint all services
	cd api && cargo clippy -- -D warnings
	cd mcp && npm run lint
	cd app && flutter analyze

format: ## Format all code
	cd api && cargo fmt
	cd mcp && npm run format
	cd app && dart format .

build-api: ## Build Rust API Docker image
	docker build -t lipily-api:local ./api

build-mcp: ## Build MCP Docker image
	docker build -t lipily-mcp:local ./mcp

health: ## Check all services are running
	curl -s http://localhost:8080/health | jq
	curl -s http://localhost:8081/health | jq
```

---

## Step 8 — Verify Everything Works

Run this checklist after setup:

- [ ] `docker compose ps` — all 3 containers (supabase, redis, minio) are Up
- [ ] `curl http://localhost:8080/health` → `{ "status": "ok" }`
- [ ] `curl http://localhost:8081/health` → `{ "status": "ok" }`
- [ ] Flutter app loads in browser at `http://localhost:PORT`
- [ ] Magic Link auth email arrives in Supabase Studio → Inbucket (http://localhost:54324)
- [ ] Create a script → blocks save to local DB (check in Supabase Studio)
- [ ] Run `make test` → all tests pass

---

## Common Issues

### `DATABASE_URL connection refused`
Make sure `docker compose up -d` has fully started before running Rust. Supabase PostgreSQL can take 10–20 seconds to be ready.

### `sqlx offline mode error`
Run `make db-prepare` to regenerate the sqlx offline query cache after changing SQL queries.

### `Flutter web CORS error hitting localhost:8080`
Axum CORS config in `api/src/main.rs` must include `http://localhost:*` in `allow_origins` for development.

### `OpenRouter key not working`
Check that `ANTHROPIC_BASE_URL` is set to `https://openrouter.ai/api` (not `https://api.anthropic.com`).

### `Supabase auth not working locally`
Get the local anon key from `supabase start` output or `supabase status`. It changes on `supabase db reset`.

---

## CI/CD Pipeline (GitHub Actions)

```yaml
# .github/workflows/ci.yml (summary)
on: [push, pull_request]

jobs:
  api:
    - cargo fmt --check
    - cargo clippy -- -D warnings
    - cargo test

  mcp:
    - npm run lint
    - npm test

  flutter:
    - flutter analyze
    - flutter test

  deploy-staging:
    needs: [api, mcp, flutter]
    if: github.ref == 'refs/heads/main'
    - docker build + push to Artifact Registry
    - gcloud run deploy --image ... (staging)

  deploy-production:
    needs: [deploy-staging]
    if: github.ref == 'refs/heads/production'
    - gcloud run deploy --image ... (production)
```
