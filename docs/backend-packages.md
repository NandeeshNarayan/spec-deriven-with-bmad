# LIPILY — Rust & TypeScript Packages Reference

> **Purpose:** Exact crate and npm package names for the Rust API and TypeScript MCP services.  
> **Agent rule:** Never guess crate or npm package names. Use exactly what is listed here.  
> **Last updated:** 2026-04-07

---

## Rust API — `api/Cargo.toml`

```toml
[package]
name = "lipily-api"
version = "0.1.0"
edition = "2021"

[[bin]]
name = "lipily-api"
path = "src/main.rs"

[dependencies]
# ─── Async Runtime ────────────────────────────────────────────
tokio = { version = "1.37", features = ["full"] }

# ─── Web Framework ────────────────────────────────────────────
axum = { version = "0.7", features = ["ws", "multipart", "macros"] }
axum-extra = { version = "0.9", features = ["typed-header"] }
tower = { version = "0.4", features = ["full"] }
tower-http = { version = "0.5", features = ["cors", "trace", "compression-br", "limit"] }
hyper = { version = "1.0", features = ["full"] }

# ─── Database ─────────────────────────────────────────────────
# IMPORTANT: Always connect via Supavisor port 6543, never 5432
sqlx = { version = "0.7", features = ["runtime-tokio-rustls", "postgres", "uuid", "chrono", "json"] }

# ─── Yjs / CRDT ───────────────────────────────────────────────
yrs = "0.18"
yrs-tokio = "0.2"

# ─── Redis ────────────────────────────────────────────────────
fred = { version = "9.0", features = ["enable-rustls", "subscriber-client"] }

# ─── S3 / AWS ─────────────────────────────────────────────────
aws-config = { version = "1.1", features = ["behavior-version-latest"] }
aws-sdk-s3 = "1.21"
aws-credential-types = "1.1"

# ─── Stripe ───────────────────────────────────────────────────
stripe-rust = "0.26"

# ─── Auth / JWT ───────────────────────────────────────────────
jsonwebtoken = "9.3"
hmac = "0.12"
sha2 = "0.10"

# ─── Email ────────────────────────────────────────────────────
reqwest = { version = "0.12", features = ["json", "rustls-tls"], default-features = false }

# ─── Serialization ────────────────────────────────────────────
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"

# ─── Validation ───────────────────────────────────────────────
validator = { version = "0.18", features = ["derive"] }

# ─── IDs ──────────────────────────────────────────────────────
uuid = { version = "1.8", features = ["v4", "serde"] }

# ─── Time ─────────────────────────────────────────────────────
chrono = { version = "0.4", features = ["serde"] }

# ─── Error Handling ───────────────────────────────────────────
thiserror = "1.0"
anyhow = "1.0"

# ─── Logging / Tracing ────────────────────────────────────────
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter", "json"] }
tracing-sentry = "0.31"

# ─── Sentry ───────────────────────────────────────────────────
sentry = { version = "0.32", features = ["tracing"] }

# ─── Misc ─────────────────────────────────────────────────────
bytes = "1.6"
tokio-stream = "0.1"
futures = "0.3"
once_cell = "1.19"
dotenvy = "0.15"

[dev-dependencies]
sqlx = { version = "0.7", features = ["test"] }
axum-test = "14.2"
mockall = "0.12"
tokio-tungstenite = "0.21"
```

---

## Key Rust Crate Notes

### `sqlx = "0.7"` with compile-time checking
Queries are verified at compile time against a live DB snapshot.
- Run `cargo sqlx prepare` after changing SQL queries
- The `.sqlx/` folder must be committed to git
- **CRITICAL**: Always use `SQLX_OFFLINE=true` in CI to avoid needing a live DB at build time

### `yrs = "0.18"` + `yrs-tokio = "0.2"`
- `yrs` = Rust port of Yjs (Y.Doc, Y.Text, Y.Map)
- `yrs-tokio` = async broadcast channel for Y.Doc updates over WebSocket
- Compatible with `y_crdt` Flutter client on the wire format

### `fred = "9.0"` (Redis)
- Async Redis client for Upstash
- Used for: WebSocket presence (TTL 10s), AI job queues, rate limiting, feed caching
- Enable `subscriber-client` feature for Supabase Realtime bridge

### `stripe-rust = "0.26"`
- Webhook signature verification: `stripe_rust::Webhook::construct_event()`
- All Stripe operations in `api/src/services/stripe.rs`

### `tower-http`
- `CorsLayer`: configure `allow_origins` from env var `ALLOWED_ORIGINS`
- `TraceLayer`: structured request logging
- `CompressionLayer`: br compression for all JSON responses
- `RequestBodyLimitLayer`: 10MB limit on import/upload endpoints

---

## TypeScript MCP Service — `mcp/package.json`

```json
{
  "name": "lipily-mcp",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "test": "vitest run",
    "test:coverage": "vitest run --coverage",
    "lint": "eslint src --ext .ts",
    "format": "prettier --write src"
  },
  "dependencies": {
    "@anthropic-ai/sdk": "^0.24.3",
    "@modelcontextprotocol/sdk": "^1.0.3",
    "bullmq": "^5.12.0",
    "ioredis": "^5.3.2",
    "express": "^4.19.2",
    "@aws-sdk/client-s3": "^3.600.0",
    "@aws-sdk/s3-request-presigner": "^3.600.0",
    "axios": "^1.7.2",
    "zod": "^3.23.8",
    "@sentry/node": "^8.10.0",
    "winston": "^3.13.0",
    "dotenv": "^16.4.5",
    "uuid": "^10.0.0"
  },
  "devDependencies": {
    "typescript": "^5.5.3",
    "tsx": "^4.16.0",
    "vitest": "^1.6.0",
    "@vitest/coverage-v8": "^1.6.0",
    "eslint": "^8.57.0",
    "@typescript-eslint/eslint-plugin": "^7.14.1",
    "@typescript-eslint/parser": "^7.14.1",
    "prettier": "^3.3.2",
    "@types/express": "^4.17.21",
    "@types/ioredis": "^4.28.10",
    "@types/uuid": "^10.0.0"
  }
}
```

---

## Key MCP Package Notes

### `@anthropic-ai/sdk: "^0.24.3"`
- **CRITICAL**: Set `baseURL` to `https://openrouter.ai/api` — never the default Anthropic URL
- Model IDs: `claude-sonnet-4-5` (primary), `claude-haiku-3-5` (fast)
- Always pass `X-Title: LIPILY` header to OpenRouter for attribution

```typescript
// mcp/src/lib/anthropic.ts — CORRECT pattern
import Anthropic from '@anthropic-ai/sdk'

export const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY!,
  baseURL: process.env.ANTHROPIC_BASE_URL!, // https://openrouter.ai/api
  defaultHeaders: {
    'HTTP-Referer': 'https://lipily.com',
    'X-Title': 'LIPILY',
  },
})
```

### `bullmq: "^5.12.0"`
- Job queues for all async AI work (Scene Intelligence, Listen pipeline, etc.)
- Worker concurrency: 3 per Cloud Run instance
- Job retry: 3 attempts with exponential backoff (2^attempt * 1000ms)
- Dead letter queue: jobs moved to `lipily:failed` after 3 retries

### `zod: "^3.23.8"`
- Runtime validation of all incoming job payloads
- Used to validate Rust → MCP internal API calls
- All MCP endpoints validate input with Zod schemas before processing

### `mcp/tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "tests"]
}
```

---

## Rust Dockerfile

```dockerfile
# api/Dockerfile
FROM rust:1.76-slim AS builder
WORKDIR /app
COPY Cargo.toml Cargo.lock ./
COPY src ./src
COPY .sqlx ./.sqlx
ENV SQLX_OFFLINE=true
RUN cargo build --release

FROM debian:bookworm-slim
RUN apt-get update && apt-get install -y ca-certificates && rm -rf /var/lib/apt/lists/*
COPY --from=builder /app/target/release/lipily-api /usr/local/bin/lipily-api
EXPOSE 8080
CMD ["lipily-api"]
```

## MCP Dockerfile

```dockerfile
# mcp/Dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package.json ./
EXPOSE 8081
CMD ["node", "dist/index.js"]
```

---

## Cloud Run Config Summary

| Service | `minInstances` | `maxInstances` | `sessionAffinity` | Memory | CPU |
|---|---|---|---|---|---|
| `lipily-api` | 1 | 20 | `true` | 512Mi | 1 |
| `lipily-mcp` | 0 | 10 | `false` | 1Gi | 2 |
