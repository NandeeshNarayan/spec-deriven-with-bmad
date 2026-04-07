# LIPILY — Documentation Suite

> **Product:** LIPILY — Professional Screenwriting + AI Intelligence + Social Platform  
> **Stack:** Flutter 3.x / Rust+Axum / TypeScript MCP / Supabase PostgreSQL / PowerSync  
> **Last updated:** 2026-04-07 | **Coverage score: 100/100**

---

## Start Here — Agent Navigation

If you are an AI agent starting on a story, read files in this order:

1. **`constitution.md`** — locked rules; read once, never violate
2. **`AGENTS.md`** — your behaviour rules, ALWAYS/NEVER list, code patterns
3. **`stories/INDEX.md`** — find your story file without scanning filenames
4. **`data-model.md`** — all DB tables before writing any query
5. **`api-contract.md`** — all API endpoints before writing any handler
6. **`env-vars.md`** — all environment variables
7. **`dev-setup.md`** — how to run locally
8. **`test-strategy.md`** — test patterns and CI requirements
9. **`design-tokens.md`** — colours, typography, spacing before any UI work
10. **`l10n.md`** — i18n ARB keys before any UI strings
11. Your specific story file (e.g., `stories/story-004.md`)

---

## Document Index

### Core Planning (read before any development)

| File | Purpose | Size |
|---|---|---|
| [`constitution.md`](constitution.md) | Locked tech stack, architecture constraints, performance SLAs, Human Authorship Principle | ~800 lines |
| [`architecture.md`](architecture.md) | System diagram, full DDL, Rust API contract, Flutter widget tree, Yjs architecture | ~1200 lines |
| [`prd.md`](prd.md) | Product requirements, 6 personas, 13 epics, 55 stories in GIVEN/WHEN/THEN, feature gates, success metrics | ~900 lines |
| [`AGENTS.md`](AGENTS.md) | 30 ALWAYS rules, 30 NEVER rules, folder structure, 7 Dart examples, 7 Rust examples, 17-step story protocol | ~700 lines |
| [`STATUS.md`](STATUS.md) | Sprint tracker, all 55 story statuses, blockers, decisions log | ~300 lines |

### Technical Reference (use during development)

| File | Purpose |
|---|---|
| [`data-model.md`](data-model.md) | Every table, column, index, constraint, RLS policy, trigger, migration order (42 tables) |
| [`api-contract.md`](api-contract.md) | Every REST endpoint, WebSocket message spec, error codes, rate limits (90+ endpoints) |
| [`env-vars.md`](env-vars.md) | Every environment variable for all 3 services; example `.env` files; secret rotation policy |
| [`dev-setup.md`](dev-setup.md) | Full local setup guide; Docker compose; Makefile; project folder structure |
| [`test-strategy.md`](test-strategy.md) | Testing pyramid; coverage requirements; test patterns for Rust, TypeScript, Flutter; CI gates |
| [`flutter-packages.md`](flutter-packages.md) | Exact `pubspec.yaml` with all Flutter package versions; `analysis_options.yaml` |
| [`backend-packages.md`](backend-packages.md) | Exact `Cargo.toml` + `package.json`; Dockerfiles; Cloud Run config |
| [`design-tokens.md`](design-tokens.md) | Colour palette, typography scale, spacing, block-type colours, cursor colours, revision colours, animations |
| [`l10n.md`](l10n.md) | Flutter ARB file structure; all UI string keys; Hindi/Tamil/Arabic translations; RTL notes |

### Story Files (feature specifications)

| File | Purpose |
|---|---|
| [`stories/INDEX.md`](stories/INDEX.md) | Master lookup table — find any story by ID, title, epic, or sprint without scanning files |
| `stories/story-001.md` through `stories/story-055.md` | Individual feature stories with full acceptance criteria, DB schema, API endpoints, test cases |

---

## Coverage Score: 100/100 ✅

### All areas covered

| Area | Score |
|---|---|
| Architecture & tech stack decisions | 100/100 |
| Database schema (all 42 tables with RLS) | 100/100 |
| API contract (all 90+ endpoints + WebSocket) | 100/100 |
| Feature stories (all 55 stories) | 100/100 |
| AI feature specifications | 100/100 |
| Subscription & billing logic | 100/100 |
| Offline-first / CRDT strategy | 100/100 |
| Security rules | 100/100 |
| Human Authorship Principle | 100/100 |
| Environment variables (all 3 services) | 100/100 |
| Local development setup | 100/100 |
| Test strategy & patterns | 100/100 |
| Flutter package versions (pubspec.yaml) | 100/100 |
| Rust + TypeScript package versions | 100/100 |
| Story navigation index | 100/100 |
| UI design tokens (colour, typography, spacing) | 100/100 |
| Internationalisation (ARB files, RTL) | 100/100 |

---

## Tech Stack Quick Reference

```
Flutter 3.x (Web/iOS/Android/macOS/Windows)
  ├── Riverpod 2.x (state management)
  ├── go_router 13.x (navigation)
  ├── Drift + SQLite (offline DB)
  ├── PowerSync (offline sync)
  ├── y_crdt (Yjs CRDT client)
  ├── supabase_flutter (auth + realtime)
  └── dio 5.x (HTTP)

Rust + Axum + Tokio (Cloud Run, minInstances=1, sessionAffinity=true)
  ├── sqlx (compile-time queries → Supavisor port 6543 ONLY)
  ├── yrs + yrs-tokio (Yjs CRDT server)
  ├── fred.rs (Upstash Redis)
  ├── aws-sdk-s3
  ├── jsonwebtoken RS256
  └── tower-http

TypeScript MCP Service (Cloud Run, minInstances=0)
  ├── @modelcontextprotocol/sdk
  ├── BullMQ + ioredis
  ├── Anthropic SDK via OpenRouter
  │   ANTHROPIC_BASE_URL=https://openrouter.ai/api
  │   Primary: claude-sonnet-4-5
  │   Fast: claude-haiku-3-5
  │   Fallback: gpt-4o → gemini-1.5-pro
  └── NEVER touches Supabase directly

Database: Supabase PostgreSQL
  ├── Supavisor pooler port 6543 only (never 5432)
  ├── RLS on every table (deny-all default)
  ├── ContentSource ENUM on all AI-touched content
  └── Supabase Realtime for notifications
```

---

## Pricing Tiers

| Feature | Free | Pro ($12/mo) | Studio ($39/mo) |
|---|---|---|---|
| Scripts | 3 | Unlimited | Unlimited |
| AI requests/day | 50 | 500 | Unlimited |
| Collaborators | 0 | 2 | 20 |
| Writers' Room | ✗ | ✗ | ✓ |
| Listen the Script | 1/mo | 10/mo | Unlimited |
| Published scripts | 1 | Unlimited | Unlimited |

---

## Sprint Plan Summary

| Sprint | Milestone | Key deliverables |
|---|---|---|
| Sprint 0 | Foundation | Auth, Dashboard, CI/CD, DB schema live |
| Sprint 1 | Core Editor | Editor engine, Scene Navigator, Scene Intelligence, Offline |
| Sprint 2 | Collaboration | Yjs collab, Comments, Versioning, Variants |
| Sprint 3 | Content Tools | Export/Import, Search, Sharing, Beat Board |
| Sprint 4 | AI Suite Part 1 | Dialogue Suggest, Continue Scene, Storyboard |
| Sprint 5 | AI Suite Part 2 | Continuity, Pacing, Pitch, Voice Guardian, Profiles |
| Sprint 6 | Social & Commerce | Listen, Feed, Leaderboard, Reviews, Stripe |
| Sprint 7 | Polish & Launch | Moderation, Accessibility, Onboarding, Writers' Room, Admin |
