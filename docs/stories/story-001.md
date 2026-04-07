# story-001 — Project Setup & Infrastructure

| Field | Value |
|---|---|
| **Story ID** | story-001 |
| **Title** | Project Setup & Infrastructure |
| **Epic** | E13 — System & Infrastructure |
| **Phase** | 1 |
| **Sprint** | Sprint 0 |
| **Priority** | P0 |
| **Status** | DRAFT |
| **Persona** | Dev (Platform Engineer) |
| **Estimated Effort** | 5 days |
| **Dependencies** | None |

---

## Problem Statement

Before any feature can be built, LIPILY needs a production-grade monorepo with CI/CD pipelines, complete database schema with RLS policies, cloud infrastructure provisioned, secrets management configured, and all three services (Flutter, Rust, TypeScript) runnable locally and deployable to staging. Without this foundation, every subsequent story is blocked.

---

## User Story

As a developer on the LIPILY platform, I want a complete, working project foundation — monorepo, database, CI/CD, local dev environment, and staging deployment — so that every subsequent story can be implemented without revisiting infrastructure concerns.

---

## Acceptance Criteria

**GIVEN** a developer clones the LIPILY repository for the first time,  
**WHEN** they run `make setup` from the root,  
**THEN** all dependencies install, `.env.local` is created from `.env.example`, and the dev environment starts with a single command.

**GIVEN** the Flutter frontend project,  
**WHEN** it is initialised with `flutter create`,  
**THEN** it targets Web (CanvasKit renderer), iOS, Android, macOS, and Windows; includes Riverpod 2.x, go_router 13.x, Drift, PowerSync, y_crdt, dio 5.x, supabase_flutter as locked dependencies in `pubspec.yaml`.

**GIVEN** the Rust API project,  
**WHEN** `cargo build` runs,  
**THEN** the binary compiles with zero warnings, all crates (Axum, Tokio, sqlx, yrs, yrs-tokio, fred, aws-sdk-s3, jsonwebtoken, tower-http, sentry, opentelemetry-otlp) are present in `Cargo.toml`.

**GIVEN** the TypeScript MCP service,  
**WHEN** `npm install && npm run build` runs,  
**THEN** TypeScript compiles with zero errors using strict mode; BullMQ, ioredis, @modelcontextprotocol/sdk, @anthropic-ai/sdk, zod, sentry are all installed.

**GIVEN** the Supabase project is connected,  
**WHEN** `supabase db push` runs against the migration file,  
**THEN** all tables defined in `architecture.md §2` are created with correct column types, enums, foreign keys, indexes, and RLS policies; every table has `ENABLE ROW LEVEL SECURITY` and a deny-all default policy.

**GIVEN** a pull request is opened against the `develop` branch,  
**WHEN** the GitHub Actions workflow runs,  
**THEN** it executes: Flutter analyze, Flutter tests, Cargo clippy, Cargo test, TypeScript strict check, Supabase migration lint — all must pass before merge is allowed.

**GIVEN** a commit is merged to `develop`,  
**WHEN** the GitHub Actions deploy workflow runs,  
**THEN** the Rust API is built as a Docker image, pushed to Google Artifact Registry, and deployed to the Cloud Run staging service; the Flutter web app is built with `--web-renderer canvaskit` and deployed to Vercel preview; the TypeScript MCP service is deployed to its own Cloud Run service.

**GIVEN** the Cloud Run services are deployed,  
**WHEN** a GET request is made to `/health` on the Rust API,  
**THEN** it returns `{"status": "ok", "version": "0.1.0"}` with status 200 within 500ms.

**GIVEN** all secrets (Supabase URL, service_role key, Redis URL, OpenRouter API key, Stripe key, Sentry DSN, AWS S3 credentials),  
**WHEN** the Cloud Run services start,  
**THEN** they load secrets exclusively from Google Cloud Secret Manager; no secrets appear in environment variables, Dockerfiles, or committed code.

**GIVEN** a developer runs `make dev`,  
**WHEN** it completes startup,  
**THEN** the Flutter web app is accessible at `http://localhost:3000`, the Rust API at `http://localhost:8080`, and the TypeScript MCP service at `http://localhost:3001`; all three can communicate with each other using local `.env` credentials.

---

## Performance Criteria

- `make setup` completes in < 5 minutes on a standard developer machine (8-core, 16GB RAM)
- Cold-start of Rust API in Cloud Run with `minInstances=1`: first response < 3 seconds
- GitHub Actions CI pipeline completes full check in < 10 minutes
- Docker image build for Rust API: < 5 minutes using multi-stage build with layer caching
- Supabase migration applies to fresh database in < 60 seconds

---

## Offline Behaviour

Not applicable — infrastructure setup is not a runtime user feature. The PowerSync configuration (tables to sync, credentials) must be added to the Flutter app's Drift schema and PowerSync schema definition so that story-018 can proceed without rework.

---

## Mobile Behaviour

The Flutter project must be configured from the start with:
- `flutter_native_splash` with LIPILY brand colours (`#1a1a2e` background, white logo)
- `flutter_launcher_icons` configured for all platforms
- `AndroidManifest.xml` with internet permission
- `Info.plist` with camera and photo library permission strings (for future profile photos)
- `web/index.html` with correct CanvasKit bootstrap, `<meta name="viewport" content="width=device-width, initial-scale=1.0">`

---

## Error States

**E1 — Missing Environment Variables**  
If any required environment variable is absent at startup, the Rust API must log a structured error `{"error": "missing_env", "key": "SUPABASE_SERVICE_ROLE_KEY"}` and exit with code 1. It must never start in a partially configured state.

**E2 — Database Migration Failure**  
If `supabase db push` fails due to a constraint violation or syntax error, the migration must roll back completely. The CI pipeline must catch this and fail the build with a clear error message.

**E3 — Cloud Run Deployment Failure**  
If the Docker build fails or the new revision fails its health check, Cloud Run must retain the previous revision and the GitHub Actions workflow must exit with a non-zero code, sending a Slack notification to the `#deploys` channel.

**E4 — Dependency Version Mismatch**  
If a developer's Flutter SDK version is below 3.x or Rust version is below 1.75, `make setup` must print a clear error message with the required version and a link to installation instructions, then exit with code 1.

---

## Security Considerations

- All secrets stored in Google Cloud Secret Manager; accessed via Workload Identity Federation from Cloud Run
- GitHub repository secrets used only for CI/CD, not for runtime
- Supabase `anon` key is safe to expose in Flutter client; service_role key is NEVER in any client-side code
- Dockerfile uses a non-root user; `USER 1000:1000` in the final image stage
- Dependabot enabled for all three package ecosystems (pub.dev, crates.io, npm)
- Branch protection on `main` and `develop`: require 1 review, require CI pass, no force push, no direct push
- SAST scan (CodeQL) runs on every PR

---

## Human Authorship Considerations

Not applicable — this story contains no AI-generated content features.

---

## DB Tables Touched

All tables defined in `architecture.md §2` are created in this story. No application-layer reads or writes occur in this story — only schema creation.

---

## API Endpoints Used

- `GET /health` — implemented and verified in this story

---

## Folder Structure Produced

```
lipily/
├── frontend/               # Flutter
│   ├── lib/
│   │   ├── main.dart
│   │   ├── router/
│   │   ├── providers/
│   │   ├── screens/
│   │   ├── widgets/
│   │   ├── models/
│   │   ├── services/
│   │   └── drift/
│   ├── pubspec.yaml
│   └── web/index.html
├── backend/                # Rust + Axum
│   ├── src/
│   │   ├── main.rs
│   │   ├── routes/
│   │   ├── handlers/
│   │   ├── middleware/
│   │   ├── models/
│   │   ├── db/
│   │   └── error.rs
│   └── Cargo.toml
├── mcp-service/            # TypeScript
│   ├── src/
│   │   ├── index.ts
│   │   ├── jobs/
│   │   ├── tools/
│   │   └── lib/
│   ├── package.json
│   └── tsconfig.json
├── supabase/
│   ├── migrations/
│   │   └── 001_initial_schema.sql
│   └── config.toml
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── deploy.yml
├── Makefile
├── docker-compose.yml
└── README.md
```

---

## Test Cases

**TC-001:** Run `make setup` on a clean machine; assert exit code 0 and all three services start.  
**TC-002:** Run `flutter analyze` against the initial Flutter project; assert zero errors and zero warnings.  
**TC-003:** Run `cargo clippy -- -D warnings` against the Rust project; assert zero warnings.  
**TC-004:** Run `npx tsc --strict --noEmit` against the TypeScript project; assert zero errors.  
**TC-005:** Apply migration to a fresh Supabase database; run `SELECT COUNT(*) FROM information_schema.tables WHERE table_schema = 'public'`; assert count matches expected table count from `architecture.md §2`.  
**TC-006:** Send `GET /health` to the locally running Rust API; assert `{"status": "ok"}` and 200 response.  
**TC-007:** Open the Flutter web app at `http://localhost:3000`; assert it loads without JavaScript console errors.  
**TC-008:** Push a branch with a deliberate Flutter analysis error; assert CI pipeline fails and merge is blocked.

---

## Out of Scope

- Authentication flows (story-002)
- Any UI screens beyond the loading splash (story-003)
- Actual PowerSync sync logic (story-018)
- Stripe or payment configuration (story-048)
- Sentry error reporting setup beyond DSN being set (wired in story-004+)
- Any AI model configuration or OpenRouter setup (story-007+)
- Production environment deployment (story-001 targets staging only)
