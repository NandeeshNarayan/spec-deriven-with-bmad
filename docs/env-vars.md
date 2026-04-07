# LIPILY — Environment Variables Reference

> **Purpose:** Every environment variable across all three services, with descriptions, required/optional status, and where to get them.  
> **Agent rule:** Never hardcode secrets. Always read from env vars. Never commit `.env` files.  
> **Last updated:** 2026-04-07

---

## Service Map

| Service | Runtime | Location | Config file |
|---|---|---|---|
| `api/` | Rust + Axum | Cloud Run | `.env` + Cloud Run secrets |
| `mcp/` | TypeScript | Cloud Run | `.env` + Cloud Run secrets |
| `app/` | Flutter | Mobile/Web | `--dart-define` or `assets/config.json` |

---

## Rust API Service (`api/.env`)

```env
# ─── Server ──────────────────────────────────────────────────
PORT=8080
RUST_LOG=info
ENVIRONMENT=development   # development | staging | production

# ─── Database ────────────────────────────────────────────────
# IMPORTANT: Always use Supavisor pooler port 6543. Never port 5432.
DATABASE_URL=postgresql://postgres.[project-ref]:[password]@aws-0-[region].pooler.supabase.com:6543/postgres
DATABASE_POOL_SIZE=10

# ─── Supabase ────────────────────────────────────────────────
SUPABASE_URL=https://[project-ref].supabase.co
SUPABASE_SERVICE_ROLE_KEY=eyJ...   # Service role — Rust only. Never expose to Flutter.
SUPABASE_JWT_SECRET=your-jwt-secret   # For RS256 verification

# ─── Redis (Upstash) ─────────────────────────────────────────
REDIS_URL=rediss://default:[password]@[host].upstash.io:6379

# ─── AWS S3 ──────────────────────────────────────────────────
AWS_ACCESS_KEY_ID=AKIAxxxxxxxxxxxxxxxx
AWS_SECRET_ACCESS_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
AWS_REGION=ap-south-1
S3_BUCKET_SCRIPTS=lipily-scripts
S3_BUCKET_AVATARS=lipily-avatars
S3_BUCKET_COVERS=lipily-covers
S3_BUCKET_LISTEN=lipily-listen
S3_BUCKET_STORYBOARDS=lipily-storyboards
S3_BUCKET_TTS_TEMP=lipily-tts-temp
S3_CDN_BASE_URL=https://cdn.lipily.com

# ─── Stripe ──────────────────────────────────────────────────
STRIPE_SECRET_KEY=sk_live_xxxxxxxxxxxxxxxxxxxx    # sk_test_xxx for dev
STRIPE_WEBHOOK_SECRET=whsec_xxxxxxxxxxxxxxxxxxxx
STRIPE_PRICE_PRO_MONTHLY=price_xxxxxxxxxxxxxxxxxxxx
STRIPE_PRICE_PRO_ANNUAL=price_xxxxxxxxxxxxxxxxxxxx
STRIPE_PRICE_STUDIO_MONTHLY=price_xxxxxxxxxxxxxxxxxxxx
STRIPE_PRICE_STUDIO_ANNUAL=price_xxxxxxxxxxxxxxxxxxxx

# ─── Resend (email) ──────────────────────────────────────────
RESEND_API_KEY=re_xxxxxxxxxxxxxxxxxxxx
RESEND_FROM_EMAIL=noreply@lipily.com
RESEND_LEGAL_EMAIL=legal@lipily.com

# ─── Internal service-to-service ─────────────────────────────
SERVICE_API_KEY=lipily-internal-secret-xxxxxxxxxxxxx
MCP_SERVICE_URL=https://mcp.lipily.com   # Internal Cloud Run URL

# ─── Sentry ──────────────────────────────────────────────────
SENTRY_DSN=https://xxxxxxxxxxxxxxxxxxxx@o0.ingest.sentry.io/0

# ─── Cloud Scheduler jobs ────────────────────────────────────
# These are set in Cloud Scheduler config, not in .env:
# - Purge deleted scripts: daily 03:00 UTC  → POST /internal/purge-deleted
# - Leaderboard compute: daily 02:00 UTC    → POST /internal/compute-leaderboard
# - Review deadline check: daily 08:00 UTC  → POST /internal/check-review-deadlines
# - Feed personalisation: every 4 hours     → POST /internal/compute-feeds
```

---

## TypeScript MCP Service (`mcp/.env`)

```env
# ─── Server ──────────────────────────────────────────────────
PORT=8081
NODE_ENV=development   # development | staging | production

# ─── OpenRouter (AI Models) ──────────────────────────────────
# IMPORTANT: All Anthropic calls go through OpenRouter, not api.anthropic.com
ANTHROPIC_BASE_URL=https://openrouter.ai/api
ANTHROPIC_API_KEY=sk-or-v1-xxxxxxxxxxxxxxxxxxxx   # OpenRouter key

# Model identifiers (do not change without updating constitution.md)
MODEL_PRIMARY=claude-sonnet-4-5
MODEL_FAST=claude-haiku-3-5
MODEL_FALLBACK_1=gpt-4o
MODEL_FALLBACK_2=gemini-1.5-pro

# ─── BullMQ / Redis (Upstash) ────────────────────────────────
REDIS_URL=rediss://default:[password]@[host].upstash.io:6379
BULL_QUEUE_PREFIX=lipily

# ─── Internal service-to-service ─────────────────────────────
SERVICE_API_KEY=lipily-internal-secret-xxxxxxxxxxxxx  # Same as Rust SERVICE_API_KEY
RUST_API_URL=https://api.lipily.com                   # Internal Cloud Run URL

# ─── AWS S3 ──────────────────────────────────────────────────
AWS_ACCESS_KEY_ID=AKIAxxxxxxxxxxxxxxxx
AWS_SECRET_ACCESS_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
AWS_REGION=ap-south-1
S3_BUCKET_LISTEN=lipily-listen
S3_BUCKET_STORYBOARDS=lipily-storyboards
S3_BUCKET_TTS_TEMP=lipily-tts-temp
S3_CDN_BASE_URL=https://cdn.lipily.com

# ─── Sentry ──────────────────────────────────────────────────
SENTRY_DSN=https://xxxxxxxxxxxxxxxxxxxx@o0.ingest.sentry.io/0
```

---

## Flutter App (`app/`)

Flutter apps do not use `.env` files. Use `--dart-define` at build time or `assets/config.json` (non-secret values only).

### Dart Define variables (build-time, non-secret)

```bash
# Used in flutter build or flutter run:
--dart-define=SUPABASE_URL=https://[project-ref].supabase.co
--dart-define=SUPABASE_ANON_KEY=eyJ...
--dart-define=API_BASE_URL=https://api.lipily.com
--dart-define=SENTRY_DSN=https://xxx@o0.ingest.sentry.io/0
--dart-define=POSTHOG_API_KEY=phc_xxxxxxxxxxxx
--dart-define=POSTHOG_HOST=https://app.posthog.com
--dart-define=ENVIRONMENT=development
```

### Dart Define access pattern

```dart
// lib/config/app_config.dart
class AppConfig {
  static const supabaseUrl = String.fromEnvironment('SUPABASE_URL');
  static const supabaseAnonKey = String.fromEnvironment('SUPABASE_ANON_KEY');
  static const apiBaseUrl = String.fromEnvironment('API_BASE_URL');
  static const sentryDsn = String.fromEnvironment('SENTRY_DSN');
  static const postHogKey = String.fromEnvironment('POSTHOG_API_KEY');
  static const environment = String.fromEnvironment('ENVIRONMENT', defaultValue: 'development');
  static bool get isProduction => environment == 'production';
}
```

### Flutter NEVER rules for env vars

- NEVER put `SUPABASE_SERVICE_ROLE_KEY` in Flutter — that is Rust-only
- NEVER put `STRIPE_SECRET_KEY` in Flutter — Stripe calls from Rust only
- NEVER put `SERVICE_API_KEY` in Flutter — internal Rust ↔ MCP only
- ONLY the Supabase anon key is safe in Flutter (it is rate-limited and RLS-gated)

---

## Local Development `.env.example` files

### `api/.env.example`

```env
PORT=8080
RUST_LOG=debug
ENVIRONMENT=development
DATABASE_URL=postgresql://postgres:postgres@localhost:54322/postgres
DATABASE_POOL_SIZE=5
SUPABASE_URL=http://localhost:54321
SUPABASE_SERVICE_ROLE_KEY=your-local-service-role-key
SUPABASE_JWT_SECRET=super-secret-jwt-token-with-at-least-32-characters
REDIS_URL=redis://localhost:6379
AWS_ACCESS_KEY_ID=test
AWS_SECRET_ACCESS_KEY=test
AWS_REGION=us-east-1
S3_BUCKET_SCRIPTS=lipily-scripts-local
S3_BUCKET_AVATARS=lipily-avatars-local
S3_BUCKET_COVERS=lipily-covers-local
S3_BUCKET_LISTEN=lipily-listen-local
S3_BUCKET_STORYBOARDS=lipily-storyboards-local
S3_BUCKET_TTS_TEMP=lipily-tts-temp-local
S3_CDN_BASE_URL=http://localhost:9000
STRIPE_SECRET_KEY=sk_test_xxxxxxxx
STRIPE_WEBHOOK_SECRET=whsec_xxxxxxxx
STRIPE_PRICE_PRO_MONTHLY=price_test_pro_monthly
STRIPE_PRICE_PRO_ANNUAL=price_test_pro_annual
STRIPE_PRICE_STUDIO_MONTHLY=price_test_studio_monthly
STRIPE_PRICE_STUDIO_ANNUAL=price_test_studio_annual
RESEND_API_KEY=re_test_xxxxxxxx
RESEND_FROM_EMAIL=dev@lipily.local
RESEND_LEGAL_EMAIL=legal@lipily.local
SERVICE_API_KEY=dev-internal-secret
MCP_SERVICE_URL=http://localhost:8081
SENTRY_DSN=
```

### `mcp/.env.example`

```env
PORT=8081
NODE_ENV=development
ANTHROPIC_BASE_URL=https://openrouter.ai/api
ANTHROPIC_API_KEY=sk-or-v1-your-openrouter-key
MODEL_PRIMARY=claude-sonnet-4-5
MODEL_FAST=claude-haiku-3-5
MODEL_FALLBACK_1=gpt-4o
MODEL_FALLBACK_2=gemini-1.5-pro
REDIS_URL=redis://localhost:6379
BULL_QUEUE_PREFIX=lipily-dev
SERVICE_API_KEY=dev-internal-secret
RUST_API_URL=http://localhost:8080
AWS_ACCESS_KEY_ID=test
AWS_SECRET_ACCESS_KEY=test
AWS_REGION=us-east-1
S3_BUCKET_LISTEN=lipily-listen-local
S3_BUCKET_STORYBOARDS=lipily-storyboards-local
S3_BUCKET_TTS_TEMP=lipily-tts-temp-local
S3_CDN_BASE_URL=http://localhost:9000
SENTRY_DSN=
```

---

## Cloud Run Deployment Environment Variables

Set in Cloud Run service configuration (not in source code):

```yaml
# Cloud Run service YAML snippet — api service
env:
  - name: ENVIRONMENT
    value: "production"
  - name: PORT
    value: "8080"
  - name: DATABASE_POOL_SIZE
    value: "10"
secretEnv:
  - name: DATABASE_URL
    valueFrom:
      secretKeyRef:
        name: lipily-database-url
        version: latest
  - name: SUPABASE_SERVICE_ROLE_KEY
    valueFrom:
      secretKeyRef:
        name: lipily-supabase-service-key
        version: latest
  # ... (all secrets via Google Secret Manager)
```

---

## Secret Rotation Policy

| Secret | Rotation frequency | Who rotates |
|---|---|---|
| `SUPABASE_SERVICE_ROLE_KEY` | On team member offboarding | Supabase dashboard |
| `STRIPE_SECRET_KEY` | Annual | Stripe dashboard |
| `SERVICE_API_KEY` | Quarterly | Manual — update both Rust and MCP |
| `ANTHROPIC_API_KEY` | Annual | OpenRouter dashboard |
| `AWS_SECRET_ACCESS_KEY` | Annual | AWS IAM |
| `RESEND_API_KEY` | Annual | Resend dashboard |
