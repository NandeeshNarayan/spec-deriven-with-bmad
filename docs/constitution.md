# LIPILY — Constitution

> This document is the supreme governing law for every decision, implementation,
> and review in the LIPILY product. Every agent, contributor, and reviewer MUST
> read and internalize this constitution before writing a single line of code.

---

## §0 — THE HUMAN AUTHORSHIP PRINCIPLE (Supreme Law)

This section is the unconditional governing philosophy above all other rules.
No rule in any other section may override §0. Any ambiguity in any other
section is resolved by deferring to §0.

1. **The writer owns every word in their script. Always.** LIPILY claims zero ownership of any content created by any user. Scripts are the user's intellectual property. Zero training on user scripts without explicit opt-in.

2. **AI never writes directly into the script — always into a suggestion layer first.** No AI action, under any circumstance, modifies the canonical script content without first passing through a visible suggestion layer that the human writer must explicitly act upon.

3. **Every AI output is EDITABLE by the human at any time, including after acceptance.** This applies to: dialogue suggestions, scene continuations, extracted metadata (5-pillar), pitch documents, voice assignments, generated imagery, structural analysis results, continuity flags, pacing analysis, logline/synopsis/treatment, character voice guardian flags, and any future AI feature. "Accepted" AI content is NOT permanent — the writer can change it at any time, forever.

4. **AI-generated content must always be visually distinguishable from human-written content until the human explicitly edits it.** The `ContentSource` enum (`aiGenerated`, `humanConfirmed`, `humanEdited`, `humanWritten`) governs this. Only `aiGenerated` state displays the AI badge in the UI. Once the human edits or confirms, the badge is removed — but the content remains fully editable.

5. **Every AI feature must have:**
   - (a) Manual trigger — no AI runs without explicit user action (except debounced 5-pillar extraction, which the user can globally disable)
   - (b) One-click dismiss — every AI suggestion can be dismissed instantly
   - (c) Full-edit mode — every AI output has an edit button
   - (d) Regenerate option — every AI output can be regenerated
   - (e) "Regenerate with instructions" custom prompt field — every AI output can be regenerated with writer-specified instructions

6. **5-Pillar sidebar items are AI-suggested.** Every item is editable, deletable, and manually addable at any time. Confirmed items remain editable forever. Dismissed items are never re-extracted by AI for that scene.

7. **AI-assigned TTS voices are editable from a voice picker at any time** before and during playback generation. The writer has full control over which voice represents each character.

8. **AI-generated scene imagery can be regenerated with a custom prompt or replaced with the writer's own uploaded image at any time.** Writer-uploaded images always take precedence in "Listen the Script" presentations.

9. **AI-selected background music can be changed from a music picker at any time.** The writer approves music selection before final assembly.

10. **Every agent write goes through the approval queue.** Writer reviews and approves before any change is committed to the script. The `agent_approvals` table mediates all AI-to-script writes. No bypass. No auto-commit.

**CHECK BEFORE IMPLEMENTING ANY AI FEATURE:** Does this feature comply with all 10 points above? If not, redesign it until it does.

---

## §1 — Locked Tech Stack

> Every technology below is **LOCKED**. No agent may substitute, upgrade, or
> replace any item without a written approval comment in this section. No exceptions.

### Frontend

| Technology | Version | Purpose | Service |
|---|---|---|---|
| Flutter | 3.x | Single codebase: Web, iOS, Android, macOS, Windows | Flutter |
| CanvasKit | (bundled) | Web renderer (`--web-renderer canvaskit`) | Flutter |
| Vercel | — | Static CDN hosting (GitHub Actions → Vercel CLI deploy) | Flutter Web |
| Riverpod | 2.x | State management (AsyncNotifier pattern + code generation via `riverpod_generator`) | Flutter |
| go_router | 13.x | Navigation and routing | Flutter |
| Drift | (latest) | Local SQLite DB, offline queue | Flutter |
| PowerSync SDK | (Flutter) | Offline sync with Supabase | Flutter |
| y_crdt | (latest) | Flutter Yjs CRDT client (syncs with yrs on Rust backend) | Flutter |
| dio | 5.x | HTTP client | Flutter |
| supabase_flutter | (latest) | Auth client (anon key only — no service role ever in Flutter) | Flutter |

### Backend API (Primary)

| Technology | Version | Purpose | Service |
|---|---|---|---|
| Rust | stable | Primary backend language | Rust API |
| Axum | (latest) | HTTP framework | Rust API |
| Tokio | (latest) | Async runtime | Rust API |
| tokio-tungstenite | (latest) | WebSocket server (for Yjs CRDT sync) | Rust API |
| yrs + yrs-tokio | (latest) | CRDT engine (official Rust port of Yjs, embedded directly) | Rust API |
| jsonwebtoken | (latest) | JWT verification (RS256, verifies Supabase JWKS) | Rust API |
| sqlx | (latest) | DB client (compile-time SQL checking, async) | Rust API |
| fred.rs | (latest) | Cache client (Upstash Redis) | Rust API |
| aws-sdk-s3 | (latest) | Storage client (S3-compatible for Supabase Storage and Cloudflare R2) | Rust API |
| Docker | — | Containerization | Rust API |
| Google Cloud Run | — | Deployment (sessionAffinity: true, timeoutSeconds: 3600, minInstances: 1) | Rust API |

### Agent / MCP Service (Separate)

| Technology | Version | Purpose | Service |
|---|---|---|---|
| TypeScript | 5.x | Agent service language | TypeScript MCP |
| @modelcontextprotocol/sdk | (latest) | MCP framework | TypeScript MCP |
| BullMQ | (latest) | Async background job queue | TypeScript MCP |
| ioredis | (latest) | Queue storage (required for BullMQ) with Upstash Redis | TypeScript MCP |
| Anthropic SDK | (latest) | AI calls via OpenRouter (`ANTHROPIC_BASE_URL=https://openrouter.ai/api`) | TypeScript MCP |
| claude-sonnet-4-5 | — | Primary model (complex agents) | TypeScript MCP |
| claude-haiku-3-5 | — | Fast model (quick tasks, autocomplete) | TypeScript MCP |
| GPT-4o | — | Fallback model (if Claude rate-limits) | TypeScript MCP |
| Gemini 1.5 Pro | — | Fallback model (if GPT-4o rate-limits) | TypeScript MCP |
| OpenRouter TTS | — | TTS voice generation for "Listen the Script" | TypeScript MCP |
| DALL-E 3 via OpenRouter | — | Image generation for scene imagery | TypeScript MCP |
| Google Cloud Run | — | Deployment (separate service from Rust API) | TypeScript MCP |

### Database

| Technology | Version | Purpose | Service |
|---|---|---|---|
| Supabase PostgreSQL | — | Primary database | All |
| Supavisor | — | Connection pooler (port 6543 only — NEVER direct connection) | Rust API |
| RLS | — | Row Level Security (enabled on every table, no exceptions) | Supabase |

### Auth

| Technology | Version | Purpose | Service |
|---|---|---|---|
| Supabase Auth | — | Authentication provider | All |
| Magic Link (Resend) | — | Email-based passwordless auth | Flutter / Rust API |
| Google OAuth | — | Social auth | Flutter / Rust API |
| JWT RS256 | — | Token format | All |

### Storage

| Technology | Version | Purpose | Service |
|---|---|---|---|
| Supabase Storage | — | Phase 1: PDFs, FDX exports, audio, images | Rust API |
| Cloudflare R2 | — | Phase 3+: Migration target (same aws-sdk-s3 code, one URL change) | Rust API |

### Cache & Queues

| Technology | Version | Purpose | Service |
|---|---|---|---|
| Upstash Redis | — | Serverless Redis (no connection pool issues on Cloud Run) | Rust API + TypeScript MCP |
| fred.rs | — | Rust Redis client | Rust API |
| ioredis | — | TypeScript Redis client (required for BullMQ) | TypeScript MCP |

### Email

| Technology | Version | Purpose | Service |
|---|---|---|---|
| Resend SDK | — | Magic Link auth, collaboration invites, notification digests, review assignments | Rust API |

### Payments

| Technology | Version | Purpose | Service |
|---|---|---|---|
| Stripe | — | Subscriptions (Checkout), reviewer payouts (Connect), self-service (Customer Portal) | Rust API |

### Real-Time Collaboration

| Technology | Version | Purpose | Service |
|---|---|---|---|
| yrs + yrs-tokio | — | Yjs CRDT, embedded in Rust API | Rust API |
| y_crdt | — | Flutter Yjs client | Flutter |
| Upstash Redis | — | Cursor presence storage + broadcast | Rust API |

### Observability

| Technology | Version | Purpose | Service |
|---|---|---|---|
| OpenTelemetry | — | opentelemetry-rust + tracing-opentelemetry (Rust), @opentelemetry/sdk-node (TypeScript) | All |
| Google Cloud Trace | — | Trace export (free on Cloud Run) | All |
| Sentry | — | Error tracking (Rust + Flutter + TypeScript) | All |
| PostHog | — | Product analytics (Flutter Web + mobile) | Flutter |

### CI/CD

| Technology | Version | Purpose | Service |
|---|---|---|---|
| GitHub Actions | — | Build → deploy pipeline | All |
| Google Secret Manager | — | Secrets management (never hardcoded, never in .env, never in git) | All |

**LOCK NOTICE:** No substitution of any technology in this section is permitted without a written approval comment appended directly below this notice, signed with date, author, and justification.

_Approval comments (append below):_

_(none)_

---

## §2 — Hard Architecture Rules

1. **Never expose `service_role` key in Flutter client.** Flutter uses only the Supabase `anon` key. The `service_role` key exists only in the Rust API environment, injected via Google Secret Manager.

2. **Flutter client uses anon key only for auth; all writes go via Rust API.** Read-only RLS-protected queries from Flutter directly to Supabase are the only exception.

3. **Agent/MCP service never touches Supabase directly.** All writes from the TypeScript MCP service go through the Rust API via internal HTTP endpoints, authenticated with a service-to-service API key (rotated monthly, stored in Secret Manager).

4. **Every JWT is verified on every authenticated request in Rust** using the `jsonwebtoken` crate with RS256 signature verification against Supabase JWKS endpoint. No caching of JWKS beyond 1 hour. No bypass for "internal" endpoints.

5. **sqlx compile-time checked queries only.** Zero raw string SQL concatenation. All queries use `query!` or `query_as!` macros. This makes SQL injection impossible by design.

6. **Riverpod AsyncNotifier pattern only in Flutter.** No `setState()`. No `ChangeNotifier`. No `StateNotifier`. All state management uses `@riverpod` annotation with `AsyncNotifier` for async operations and `Notifier` for sync state.

7. **Sealed `Result<T, AppError>` types for all Dart operations that can fail.** Every repository method, every service method, every provider method that can fail returns a `Result` sealed class, never throws raw exceptions.

8. **Zod schemas for all TypeScript handler inputs.** Every MCP tool, every BullMQ job payload, every API client response is validated with a Zod schema before processing.

9. **Serde structs with validators for all Rust handler inputs.** Every Axum handler uses typed structs with `#[derive(Deserialize)]` and custom validators (via `validator` crate or manual validation) on every field.

10. **PowerSync for all offline sync.** No manual sync logic. No custom WebSocket-based sync for data that PowerSync handles. PowerSync is the single source of truth for offline data synchronization.

11. **yrs + yrs-tokio for all real-time collaboration.** No other CRDT library. No Operational Transform. No custom conflict resolution for collaborative editing. Yjs handles it.

12. **Supavisor pooler only for DB connections (port 6543).** Never use direct connection strings from Cloud Run. Direct connections are only for local development and migrations.

13. **All secrets in Google Secret Manager.** Injected as environment variables at Cloud Run startup. Never in code, never in Docker image layers, never in git, never in `.env` files committed to repository.

14. **Never hardcode any credential, API key, or secret anywhere.** This includes Stripe keys, Supabase keys, OpenRouter keys, Resend keys, and internal service-to-service keys.

15. **All Drift queries use typed DAOs.** No raw SQL in Drift. Every table has a corresponding DAO class with typed methods.

16. **All file URLs are signed.** Time-limited signed URLs for private files. Never permanent public URLs for user-generated content.

17. **Feature-level separation in Flutter.** Each feature follows `data/`, `domain/`, `presentation/` structure. No cross-feature imports except through shared `core/` modules.

18. **WebSocket connections require JWT verification on upgrade handshake.** Document ID is validated against user's access rights before a Yjs session is established.

19. **All API responses use consistent error format.** Every error response follows the `AppError` enum structure with `status_code`, `error_code`, `message`, and optional `details` fields.

20. **No in-memory caching in Cloud Run containers.** Cloud Run instances are stateless and can be replaced at any time. All caching goes through Upstash Redis (fred.rs in Rust, ioredis in TypeScript).

21. **Cloud Run Rust API settings are fixed:** `minInstances=1`, `sessionAffinity=true`, `timeoutSeconds=3600`. Required for WebSocket stability.

22. **Cloud Run TypeScript MCP settings:** `minInstances=0` (scale to zero when idle). Jobs are picked up from BullMQ queue on scale-up.

23. **All real-time cursor presence data flows through Upstash Redis.** Key pattern: `presence:{scriptId}`. TTL: 5 seconds. Refreshed every 3 seconds by client. Broadcast to all WebSocket connections via Tokio broadcast channel.

24. **Fountain format is the canonical internal representation.** All script content is stored and manipulated as typed Fountain elements. The Rust backend owns the Fountain parser.

25. **Every new database table must have RLS enabled with deny-all default policy.** Explicit allow policies must be written for every operation (SELECT, INSERT, UPDATE, DELETE) that should be permitted.

26. **All timestamps are TIMESTAMPTZ**, not TIMESTAMP. No timezone ambiguity.

27. **All IDs are UUIDs** generated by `gen_random_uuid()`. No serial integers for primary keys.

28. **Position ordering uses gap strategy** (1000, 2000, 3000). Rebalance triggered when gap falls below 10.

---

## §3 — Performance Non-Negotiables

Every target below is a hard requirement. If a feature cannot meet its target, the implementation must be redesigned — not the target lowered.

| Metric | Target | Measurement |
|---|---|---|
| Cold start (Flutter Web on 4G) | < 3 seconds | Time from navigation to interactive (LCP) |
| Editor input latency | < 16ms | Single keypress to screen render (single frame at 60fps) |
| Yjs cursor sync | < 200ms | Time from one user's keystroke to other user seeing cursor move |
| Offline queue flush | < 5 seconds | Time to drain Drift offline_write_queue on reconnect |
| PDF export (120 pages) | < 30 seconds | Full script PDF via Cloud Run Puppeteer |
| AI suggestions p95 | < 3 seconds | Dialogue suggest, continue scene (from trigger to visible) |
| "Listen the Script" generation | < 60 seconds | For a 10-page scene (background job — user not blocked) |
| FTS search (Cmd+K) | < 200ms | From keypress to results rendered |
| Flutter Web bundle | < 4MB | Initial load (CanvasKit) |
| Editor scroll | 60fps minimum (120fps target) | Smooth scroll in block editor view |
| Supabase query p95 | < 100ms | All database queries via Supavisor |
| Cloud Run Rust API p95 | < 200ms | All authenticated API endpoints |
| WebSocket reconnect | < 2 seconds | Time from disconnect detection to reconnected state |
| Yjs state reconciliation | < 1 second | On reconnect, full state merge |
| PostHog event flush | < 500ms | Non-blocking analytics event dispatch |

---

## §4 — Offline-First Rules

### PowerSync Configuration

Tables that sync offline via PowerSync:

- `scripts` — full row for scripts the user owns or collaborates on
- `scenes` — all scenes for synced scripts
- `blocks` — all blocks for synced scripts
- `scene_intelligence` — 5-pillar data for synced scripts
- `story_bible_entries` — story bible for synced scripts
- `beat_notes` — beat board data for synced scripts
- `characters` — character records for synced scripts
- `locations` — location records for synced scripts
- `comments` — comments for synced scripts
- `drafts` — draft metadata (not full snapshots — those are fetched on demand)
- `formatting_preferences` — per-script formatting settings
- `script_tags` — inline tags for synced scripts
- `research_notes` — research notes for synced scripts
- `shots` — storyboard shots for synced scripts

### Drift offline_write_queue Schema

```sql
CREATE TABLE offline_write_queue (
  id TEXT PRIMARY KEY,
  operation_type TEXT NOT NULL,        -- 'create', 'update', 'delete'
  table_name TEXT NOT NULL,
  record_id TEXT NOT NULL,
  payload TEXT NOT NULL,               -- JSON serialized
  retry_count INTEGER NOT NULL DEFAULT 0,
  last_error TEXT,
  created_at INTEGER NOT NULL,         -- Unix milliseconds
  status TEXT NOT NULL DEFAULT 'pending' -- 'pending', 'processing', 'failed', 'dead_letter'
);
```

### Retry Strategy

- Exponential backoff: 1s → 2s → 4s → 8s → 16s
- Maximum 5 retries per operation
- After 5 failures: move to `dead_letter` status
- Dead letter queue reviewed on next app foreground (show subtle notification to user)

### Conflict Resolution

- **Collaborative scripts (2+ collaborators):** Server wins. Yjs CRDT handles merge automatically on reconnect. No manual conflict resolution needed for content.
- **Solo scripts (1 collaborator):** Client wins. Drift offline queue is flushed to server, overwriting server state for the affected records.

### Focus Mode Integration

- When Focus Mode is active, PowerSync sync is disabled
- All writes queue to Drift offline_write_queue
- On Focus Mode exit, sync resumes and queue is drained

### Offline Indicator

- Subtle banner at bottom of screen: "Working offline — changes saved locally"
- Not intrusive — does not cover any editor content
- Dismiss after 5 seconds, re-shows on reconnect attempt failure

---

## §5 — Editor Rules

### Fountain Format

Fountain is the canonical internal representation. All blocks are stored as typed elements with a `type` enum:

| Type | Description |
|---|---|
| `SCENE_HEADING` | Scene header: `INT. LOCATION - TIME` |
| `ACTION` | Action/description lines |
| `CHARACTER` | Character name (all caps) |
| `DIALOGUE` | Dialogue text |
| `PARENTHETICAL` | Parenthetical direction `(softly)` |
| `TRANSITION` | Scene transition: `CUT TO:`, `FADE IN:` |
| `SHOT` | Camera shot: `CLOSE ON`, `ANGLE ON` |

### Smart Flow Logic Matrix

The Enter key and Tab key behavior depends on the current block type. This matrix defines all 14 transitions:

| Current Block Type | Enter → Creates | Tab → Changes To |
|---|---|---|
| `SCENE_HEADING` | `ACTION` | `TRANSITION` |
| `ACTION` | `ACTION` | `CHARACTER` |
| `CHARACTER` | `DIALOGUE` | `ACTION` |
| `DIALOGUE` | `CHARACTER` | `PARENTHETICAL` |
| `PARENTHETICAL` | `DIALOGUE` | `ACTION` |
| `TRANSITION` | `SCENE_HEADING` | `ACTION` |
| `SHOT` | `ACTION` | `CHARACTER` |

**Empty block behavior:** If the current block is empty and the user presses Enter, the block type cycles to the next logical type (same as Tab). Double Enter on an empty block always creates an `ACTION` block.

### SmartType Autocomplete

- **Character names:** Trigger after minimum 1 character typed in a `CHARACTER` block. Source: all character names from `characters` table for the current script. Case-insensitive prefix match.
- **Location names:** Trigger after `INT.` or `EXT.` typed in a `SCENE_HEADING` block. Source: all location names from `locations` table for the current script.
- **Extensions:** After character name in `CHARACTER` block, show `(V.O.)`, `(O.S.)`, `(CONT'D)` suggestions.

### Autosave Rules

- Every 30 seconds minimum while editor is active
- On every navigation away from editor (route change)
- On Focus Mode exit
- Autosave creates a draft record with `is_autosave: true`
- Autosave drafts are retained for 30 days then purged by scheduled job

### Yjs Document Structure

```
Y.Doc
└── blocks: Y.Array<Y.Map>
    └── Y.Map keys:
        ├── id: string (UUID)
        ├── type: string (block type enum)
        ├── content: string (text content)
        ├── position: number (gap-ordered position)
        ├── metadata: string (JSON — variants, tags, etc.)
        ├── content_source: string (ContentSource enum)
        └── deleted_at: string | null (ISO timestamp)
```

Each block is a `Y.Map` inside a `Y.Array`. Yjs handles concurrent edits to the same block (character-level CRDT). Position changes (reordering) are handled by updating the `position` value in the `Y.Map`.

---

## §6 — AI Rules

1. **All AI calls originate from the TypeScript MCP service.** Never from Flutter client directly. Never from Rust API directly. The Rust API acts as a gateway: receives the request, validates auth, checks rate limits, then delegates to the TypeScript MCP service via BullMQ job or direct HTTP call.

2. **All AI calls use OpenRouter via the Anthropic SDK.** `ANTHROPIC_BASE_URL=https://openrouter.ai/api`. This is transparent to application code — the Anthropic SDK is used, but the actual routing is through OpenRouter.

3. **5-pillar extraction is debounced 2 seconds on block save.** When a block is saved, a 2-second timer starts. If another block save occurs within 2 seconds, the timer resets. When the timer fires, the extraction job is queued.

4. **Rate limits are enforced server-side in Rust (Upstash Redis):**
   - Free: 50 AI requests per day
   - Pro: 500 AI requests per day
   - Studio: unlimited
   - Rate limit check occurs BEFORE the job is queued

5. **Every AI action is opt-in.** No AI feature runs automatically without explicit user trigger. The sole exception is debounced 5-pillar extraction, which the user can globally disable in settings.

6. **ContentSource enum must be attached to every AI-generated item:**
   - `aiGenerated` — AI created this item, shows AI badge
   - `humanConfirmed` — Writer confirmed AI suggestion, badge removed
   - `humanEdited` — Writer edited AI suggestion, badge removed
   - `humanWritten` — Writer created this item manually, no badge

7. **Only `aiGenerated` state shows the AI badge in UI.** `humanConfirmed`, `humanEdited`, and `humanWritten` are all treated identically in UI — no badge, no visual difference.

8. **Approval queue:** Every agent write creates an `agent_approvals` record. Flutter polls the approval queue. Writer sees a diff of the proposed change. Writer approves or rejects. Only on approval does the Rust API commit the change to the script content tables.

9. **Model selection:**
   - `claude-sonnet-4-5`: Complex agents (continuity check, pacing analysis, pitch generation, "Listen the Script" generation)
   - `claude-haiku-3-5`: Fast tasks (dialogue suggest, 5-pillar extraction, SmartType suggestions, character voice guardian)

10. **Fallback chain:** Claude → GPT-4o → Gemini 1.5 Pro. Handled by OpenRouter configuration, transparent to application code. If Claude is rate-limited, OpenRouter automatically routes to the next available model.

---

## §7 — Security Rules

Every rule below is non-negotiable. Every layer of LIPILY must enforce these. Violations are treated as critical bugs.

1. **NEVER expose `service_role` key to Flutter client.** Flutter uses only the `anon` key.

2. **NEVER accept unvalidated input.** All Rust handlers use typed structs with `serde` + custom validators. All TypeScript handlers use Zod schemas.

3. **NEVER allow direct Supabase writes from Flutter client for any sensitive operation.** All writes go: Flutter → Rust API (JWT verified) → Supabase (`service_role`). Read-only RLS-protected queries are the only exception.

4. **Agent service (TypeScript MCP) NEVER touches Supabase directly.** All agent writes go through Rust API with service-to-service auth (internal API key, rotated monthly, stored in Secret Manager).

5. **Every Supabase table has RLS enabled.** Default policy: deny all. Explicit allow policies only.

6. **Rate limiting on every endpoint via Upstash Redis:**
   - Auth endpoints: 5 requests/minute per IP
   - Write endpoints: 60 requests/minute per user
   - AI endpoints: per subscription tier (Free: 50/day, Pro: 500/day, Studio: unlimited)
   - Public endpoints: 30 requests/minute per IP

7. **CORS: whitelist only.** Origins: `[lipily.com, *.lipily.com, localhost:3000 (dev only)]`. No wildcard `*` ever.

8. **CSP headers on all web responses.** No inline scripts in Flutter web output except CanvasKit bootstrap.

9. **Stripe webhooks: verify `Stripe-Signature` header on every webhook** using the Stripe webhook secret (from Secret Manager). Reject all unverified webhook calls with 401.

10. **File uploads: validate MIME type server-side** (not just extension). Max file size: 50MB for scripts, 500MB for audio. Virus scan via ClamAV on all uploads before storage (Phase 3+; use Supabase Storage policies until scale).

11. **SQL injection: impossible by design.** sqlx compile-time checked queries. Zero string concatenation in SQL. Zero raw SQL in Drift.

12. **Secrets: all secrets in Google Secret Manager,** injected as environment variables at Cloud Run startup. Never in code, never in Docker image, never in git.

13. **JWT expiry: 1-hour access tokens, 7-day refresh tokens.** Refresh token rotation on every use.

14. **WebSocket auth: JWT verified on WebSocket upgrade handshake.** Document ID validated against user's access rights before Yjs session established.

15. **Generated content safety: all AI-generated images and text pass through OpenRouter's moderation layer.** Additional server-side content check before storage.

16. **Dependency scanning: GitHub Actions runs `cargo audit` (Rust) and `npm audit` (TypeScript) on every PR.** Block merge on high-severity vulnerabilities.

17. **GDPR/IT Act compliance:** User PII stored in Supabase EU region. Indian users notified of data location. Right to erasure: account deletion purges all PII within 30 days. Scripts are user's intellectual property — LIPILY claims zero ownership. Zero training on user scripts without explicit opt-in.

18. **No user under 13.** Age verification checkbox on signup. Enforced at account creation.

19. **All API responses include security headers:**
    - `X-Content-Type-Options: nosniff`
    - `X-Frame-Options: DENY`
    - `Referrer-Policy: strict-origin-when-cross-origin`
    - `Permissions-Policy: camera=(), microphone=(), geolocation=()`

20. **Penetration testing checklist must be run before every major release** (OWASP Top 10 at minimum).

---

## §8 — Scope

### v1 (Phase 1–2): Core Product

Core editor + offline-first + Yjs collaboration + AI scene intelligence + full AI suite + exports (PDF, FDX, Fountain) + imports (FDX, Fountain, TXT) + story bible + beat board + versioning + branch editing + all editor utility features (search, find/replace, formatting prefs, title page, inline tagging, smart deletion, alternate dialogue, character/location manager, script statistics, revision mode, storyboard, script breakdown, research mode, keyboard shortcuts).

### v2 (Phase 3): Social Layer

Profiles + publishing + "Listen the Script" + discovery feed + leaderboard + review submission + reviewer program + producer discovery portal.

### v3 (Phase 4): Scale & Enterprise

Writers' Room mode + Dialogue Read-Aloud TTS + Student Tier discounts + production integrations + full admin dashboard + content moderation + onboarding flow + observability + scale infrastructure.

### NEVER List (Not In Scope)

The following features are explicitly **out of scope** for all versions and must never be implemented without a separate product decision:

- Integrated DMs / messaging system
- Stealth Mode
- Scene Marketplace / Forking (separate product decision)
- Virus scanning before v3 (use Supabase Storage policies until scale)

---

## §9 — Data Model Constraints

1. **UUIDs everywhere.** All primary keys use `gen_random_uuid()`. No serial integers.

2. **All timestamps are `TIMESTAMPTZ`** (not `TIMESTAMP`). No timezone ambiguity.

3. **Position ordering: gap strategy.** Initial positions: 1000, 2000, 3000. Insertions use midpoint. Rebalance trigger when gap falls below 10. Rebalance resets gaps to 1000 increments.

4. **Soft deletes: `deleted_at TIMESTAMPTZ` nullable column.** 30-day retention period, then permanent purge by scheduled Cloud Run job. All queries filter `WHERE deleted_at IS NULL` by default.

5. **All text fields have explicit `CHECK` length constraints.** No unbounded text fields. Every `TEXT` column has a maximum length enforced at the database level.

6. **No nullable foreign keys.** If a relationship is optional, use a separate linking table. Exception: `parent_id` in self-referential tables (e.g., `comments.parent_id`), which must have `ON DELETE CASCADE`.

7. **ContentSource enum in DB:**
   ```sql
   CREATE TYPE content_source AS ENUM (
     'ai_generated', 'human_confirmed', 'human_edited', 'human_written'
   );
   ```

8. **Username: `UNIQUE`, `CITEXT` extension** for case-insensitive uniqueness. `CREATE EXTENSION IF NOT EXISTS citext;`

---

## §10 — Deployment Rules

1. **Cloud Run Rust API:** `minInstances=1`, `sessionAffinity=true`, `timeoutSeconds=3600`. These are required for WebSocket stability and must not be changed.

2. **Cloud Run TypeScript MCP:** `minInstances=0` (scale to zero when idle). BullMQ picks up jobs on scale-up.

3. **GitHub Actions pipeline:**
   - Build and test on every PR
   - Deploy to staging on merge to `develop` branch
   - Deploy to production on merge to `main` branch only

4. **Zero-downtime deploys:** Cloud Run traffic splitting: 10% → 50% → 100% with 5-minute intervals between steps.

5. **Rollback:** Less than 5 minutes via Cloud Run traffic split revert to previous revision.

6. **Secret rotation:** Every 90 days minimum. Automated via Secret Manager versioning + Cloud Run auto-redeploy on secret version change.

---

## §11 — Multilingual Rules

1. **All text storage is UTF-8 in PostgreSQL.** Database encoding is `UTF8` by default.

2. **RTL language support:** Flutter `Directionality` widget wraps `BlockEditorView` based on `scripts.rtl` boolean value. RTL is per-script, not per-account.

3. **No Latin-only assumptions in any regex or validation.** Character name validation, location validation, search — all must handle Unicode correctly. No `[a-zA-Z]` patterns on user content.

4. **Font fallback stack must include Noto Sans** for non-Latin scripts (Hindi, Tamil, Arabic, Korean, Japanese, Chinese, Thai, etc.).

5. **AI features detect script language and respond in same language.** Language code from `scripts.language` is passed to every AI agent in the system prompt.

6. **No hardcoded language in any error message.** All user-facing strings are in the Flutter `l10n` system (`app_en.arb`, `app_hi.arb`, etc.).

---

## §12 — Accessibility Rules

1. **WCAG 2.1 AA minimum** on all screens. AA compliance is a release blocker.

2. **All interactive elements ≥ 44px tap target.** Buttons, links, input fields, checkboxes — all must meet this minimum.

3. **Color contrast ≥ 4.5:1** for all body text against its background.

4. **No information conveyed by color alone.** Every color-coded element has a secondary indicator (text label, icon, pattern).

5. **All images have meaningful alt text.** AI-generated images include auto-generated alt text from the image generation prompt.

6. **Screen reader support for editor.** `aria-live` regions for block type labels on Flutter Web. Semantic headings for scene headings.

7. **Keyboard navigable end-to-end.** No mouse required. Tab through all interactive elements in logical order. Focus rings visible at all times.

8. **`prefers-reduced-motion` respected** on all animations. When system requests reduced motion, all non-essential animations are disabled.

---

_End of Constitution._
