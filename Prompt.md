You are a senior product engineer, systems architect, and technical documentation 
specialist. Your task is to create a complete, production-grade, from-scratch 
planning documentation suite for a product called LIPILY.

Do NOT reference or reuse any previously existing files. Build everything fresh 
from this prompt alone. Every file must be exhaustive, complete, and 
production-ready. No placeholders. No "TBD". No truncated sections. Every 
story file must be independently executable by a developer agent reading 
only that file + constitution.md + architecture.md.

Read this entire prompt before creating any file.

═══════════════════════════════════════════════════════════════
PART 1 — PRODUCT VISION
═══════════════════════════════════════════════════════════════

LIPILY is a three-layer product built to solve two real problems:
(1) The recognition gap for screenwriters in global cinema — especially 
    India — where brilliant writers have no discovery platform.
(2) The tooling gap — every existing screenwriting tool is outdated, 
    desktop-only, has no real AI, and costs $249 for a 1990s product.

LIPILY's positioning: "Where screenwriters write, get discovered, 
and get heard — literally."

─── LAYER 1: PROFESSIONAL SCREENWRITING TOOL ───────────────────

The best screenwriting editor in the market. Industry-standard 
auto-formatting (Scene Heading, Action, Character, Dialogue, 
Parenthetical, Transition, Shot). Smart Flow Logic — intelligent 
Enter/Tab behavior that predicts the next element contextually. 
SmartType autocomplete for character names, location names, and 
extensions (V.O./O.S.) learned from the script's own content. 
Alternate Dialogue — store multiple variant lines per dialogue block, 
cycle with arrow UI. Smart Deletion — Shift+Backspace auto-backs up 
content. 30-day Recently Deleted recovery panel. Offline-first, 
cross-platform (Flutter Web, iOS, Android, macOS, Windows).

Focus Mode (flight mode): full-screen writing, all UI hidden, 
non-active text faded, typewriter view (active line centered), 
notifications disabled, forces offline-save mode, Midnight Mode 
(pure black background). Exit via Escape or floating button.

Git-style versioning: named drafts, WGA revision color cycle 
(White→Blue→Pink→Yellow→Green→Goldenrod→Buff→Salmon), visual diff 
viewer (additions green, deletions red strikethrough, modified yellow), 
merge request system with owner review, scene-specific change history, 
autosave snapshot every 60 seconds.

Isolated Branch Editing: create an "Edit Branch" — full isolated 
copy of the script at that moment. Edits on branch don't affect Main. 
Submit Merge Request when ready. Owner reviews diff, can Approve & Merge, 
Request Changes (with inline comments), or Reject. Conflict resolver 
for overlapping edits.

Beat Board: visual index cards in horizontal timeline. Each card has 
scene heading, beat summary, color tag, page estimate, emotional beat. 
Drag to reorder (syncs to script order). Structural overlay toggle: 
3-Act, Save the Cat (15 beats), Sequence Method — shows beat labels 
at expected page positions.

Story Bible panel: 5 tabs — Characters, Locations, Themes, Backstory, 
Timeline. Auto-populated by AI scene intelligence. Every entry 
manually editable at any time.

Real-time collaboration: Yjs CRDT (yrs embedded in Rust backend). 
Cursor presence with colored name tags. Role-based access: 
Owner / Lead Writer / Editor / Writer / Viewer.

Comment threads on any block. Reply threads (max 2 levels). 
Resolve/unresolve. Comments panel with filter.

Read-only secure sharing links (JWT-signed, configurable expiry).

Global search (Cmd+K): scene headings, block content (FTS), 
character names, location names, script titles. < 200ms.

Find & Replace: Cmd+F, search entire script or current scene, 
case-sensitive toggle, whole word toggle, replace single/all, 
character rename across all block types.

Title Page Editor: Title, Written By, Based On, Contact Info, 
Draft Date, WGA registration number. Auto-populates from script 
metadata. Exports as page 1.

Script formatting preferences: page size, font size, margins, 
scene number display, header/footer. Industry presets: WGA Feature, 
BBC Drama, Bollywood, Stage Play. Save as named templates.

Script Statistics & Writing Goals: session timer, daily goals 
(words or scenes), progress bar, total stats dashboard, writing 
streak tracker, weekly email report.

Inline Scene Tagging: manually highlight any text and tag as PROP, 
CAST, LOCATION, VFX, STUNT, WARDROBE, MAKEUP, VEHICLE. Non-printing. 
Syncs to 5-pillar sidebar and production reports.

Revision Mode (production standard): page locking, asterisks (*) 
in right margin for changed lines, A/B page system, scene Omit 
with label preserved, WGA revision color cycle.

Keyboard shortcuts: full map, custom remapping, Cmd+/ reference panel.

Storyboard / Shot Visualization: per-scene shot panels. Shot type, 
camera movement, description, AI-generated reference image. 
Reorderable. Shot list auto-generated for production. AI images 
are always replaceable with writer's own uploads.

Script Breakdown Auto-Generation: scene breakdown sheet, cast report, 
props master list, locations report. Sources: scene_intelligence + 
inline tags. Export as PDF or CSV.

Inspiration & Research Mode: web search panel, scene-attached notes, 
character mood boards, pinned reference images. Private, never exported.

─── LAYER 2: AI SCENE INTELLIGENCE ENGINE ──────────────────────

The core differentiator. The writer writes naturally. The AI reads 
every line and automatically populates a persistent 5-Pillar Scene 
Sidebar — no manual tagging required.

Five Pillars (extracted per scene, debounced 2 seconds on block save):
🌍 Environment: location type, INT/EXT, time of day, weather, 
   atmosphere, spatial descriptors
📦 Props: physical objects mentioned or implied, confidence score, 
   scene-by-scene tracking
👥 Characters: names present, emotional state, action, 
   relationship dynamics
🎵 Music/Sound: mood, genre suggestion, ambient sounds, SFX cues
💭 Emotion: scene tone, tension level, genre beat, pacing note

Every extracted item in the sidebar is:
- Editable (tap to edit text) at any time — including after confirmation
- Deletable (remove what AI got wrong)
- Manually addable (writer can add items AI missed)
- Confirmable (locks item, removes AI badge)
- Dismissible (tells AI to never re-extract that item for this scene)

This same metadata powers: production PDF export, script breakdown, 
"Listen the Script" audio-visual presentation, storyboard image 
generation prompts.

AI Feature Suite (all Pro/Studio, all manually triggered, all editable):
- AI Dialogue Suggest: 3 suggestions per character/dialogue block, 
  shown as insertable variants. Character profile + scene context given.
- AI Continue Scene (Writer's Block Breaker): 3 explicit modes — 
  (1) Socratic Questions: 5 questions to help writer discover scene. 
  No generated content. (2) Three Directions: 3 brief directional 
  summaries, no dialogue. (3) Rough Placeholder: 3-5 block rough draft 
  the writer is expected to rewrite.
- AI Continuity & Logic Checker: dead characters reappearing, location 
  contradictions, age/timeline inconsistencies, prop continuity, weather 
  inconsistencies. Background job, non-blocking, notify on complete.
- AI Pacing & Structure Analyzer: dialogue-to-action ratio per act, 
  scene length distribution, B-story gap detection, emotional intensity 
  curve, act balance. Flags with page references. Never rewrites anything.
- AI Logline & Pitch Generator: logline, synopsis, one-page treatment, 
  comp titles list. Triggered at 60+ pages. All outputs fully editable.
- AI Character Voice Guardian: monitors dialogue against voice profile. 
  Flags "Is this [CHARACTER]?" without rewriting.
- AI Multilingual Support: write in any language. AI responds in same 
  language. RTL support. Non-Latin script support.

CRITICAL AI PRINCIPLE — runs through every AI feature:
Every AI action is a suggestion. Every AI output lives in a 
suggestion layer. Nothing enters the script without explicit 
writer approval. After approval, every AI-accepted item remains 
fully editable forever. AI never locks anything. Writer can 
regenerate any AI output with a "Regenerate with instructions" 
custom prompt at any time.

─── LAYER 3: SOCIAL DISCOVERY PLATFORM ─────────────────────────

Mission: give screenwriters the recognition they deserve. 
Writers in Indian cinema (and globally) have no discovery platform. 
Thick PDFs sit unread. LIPILY changes this.

Unique @usernames: social identity layer. 3-30 chars, alphanumeric 
+ underscore, globally unique, case-insensitive, changeable once/30 days.

Public writer profiles at lipily.com/@username: display name, bio, 
location, published scripts, follower count, following count, portfolio.

Script publishing: writer toggles Private → Public when ready. 
Choose genres, add logline, add tags, set visibility 
(Public / Unlisted / Invite-Only). Unpublish anytime.

"LISTEN THE SCRIPT" — The flagship social format:
For any published script or scene, generate an audio-visual presentation:
1. AI assigns distinct TTS voice per character (OpenRouter TTS or 
   equivalent) — persistent per character across the script.
   Writer can change any voice from a voice picker at any time.
2. Dialogue blocks narrated by character voice.
3. Action blocks narrated by neutral narrator voice.
4. Scene imagery generated by AI (DALL-E via OpenRouter) based on 
   environment + props data from scene_intelligence.
   If writer has a storyboard (story-050), storyboard images are 
   used instead of generating new ones. Writer can regenerate any 
   image with a custom prompt or replace with uploaded image.
5. Background music auto-selected from curated tracks by scene 
   emotion tag. Writer can change music from a picker at any time.
6. Dialogue shown as animated subtitles.
7. Assembled as a timed playback experience (3-8 mins per scene).
8. Published as a social post. Plays on mobile like a Reel 
   (autoplay on scroll). Works without LIPILY account.
Generation < 60s for a 10-page scene. Runs as background job via 
BullMQ in the TypeScript MCP service. Every generation step's 
output is editable by the writer before the final presentation 
is published.

Social feed: For You (algorithmic), Trending, Following, By Genre, 
New Releases. Each item is a "Listen" post or script card. 
Like, bookmark, share. Infinite scroll. Genre/format/language/
regional filters (Bollywood/Hollywood/Nollywood/K-Drama etc).

Leaderboard: top writers by total script views, Listen plays, 
follower growth, review score. Filters: Genre, Format, Region, 
Weekly/Monthly/All-time. Updated every 24 hours. Badges: 
Rising Star, Top 10%, Genre Champion.

Review Submission System: Pro writers submit completed scripts 
(min 70 pages feature / 25 pages TV). Review record lifecycle: 
Submitted → Assigned → In Review → Complete. 14-day SLA. 
Reviewed badge on published script.

Reviewer Program: (1) Verified Critics — manual approval. 
(2) Student Reviewers — .edu email or student ID, 48h approval. 
Pay per review via Stripe Connect: Critics $15, Students $5. 
Structured review form: logline clarity, character depth, structure, 
dialogue, originality, overall score 1-10, 300-word minimum notes.

Producer Discovery Portal: separate Producer Account type 
(company email verified). Browse reviewed scripts with filters. 
Request contact with writer. No account needed to view 
"Listen the Script" posts.

Content Moderation: report scripts for plagiarism, inappropriate 
content, copyright violation, spam. Auto-hide on 3+ reports in 24h. 
Admin moderation queue. Appeal system.

═══════════════════════════════════════════════════════════════
PART 2 — TECH STACK (LOCKED — NON-NEGOTIABLE)
═══════════════════════════════════════════════════════════════

Every technology below is LOCKED. No agent may substitute, 
upgrade, or replace any item without a written approval comment 
in constitution.md §1. No exceptions.

FRONTEND:
- Framework: Flutter 3.x (single codebase: Web, iOS, Android, 
  macOS, Windows)
- Web renderer: CanvasKit (--web-renderer canvaskit)
- Web hosting: Vercel (static CDN, GitHub Actions → Vercel CLI deploy)
- State management: Riverpod 2.x (AsyncNotifier pattern + 
  code generation via riverpod_generator)
- Navigation: go_router 13.x
- Local DB: Drift (SQLite, offline queue)
- Offline sync: PowerSync SDK (Flutter)
- CRDT client: y_crdt (Flutter Yjs client — syncs with yrs on Rust backend)
- HTTP client: dio 5.x
- Auth client: supabase_flutter (anon key only — no service role ever 
  in Flutter)

BACKEND API (Primary):
- Language + Framework: Rust + Axum + Tokio
- Containerized: Docker → Google Cloud Run
- WebSocket server: Tokio-tungstenite (for Yjs CRDT sync)
- CRDT engine: yrs + yrs-tokio (official Rust port of Yjs, 
  embedded directly in Rust API — no proxy layer)
- Fountain/FDX parser: Rust (custom implementation lives here)
- JWT verification: jsonwebtoken crate (RS256, verifies Supabase JWKS)
- DB client: sqlx (compile-time SQL checking, async)
- DB connection: Supabase Supavisor pooler port 6543 (NEVER direct 
  connection string from Cloud Run)
- Cache client: fred.rs (Upstash Redis)
- Storage client: aws-sdk-s3 (S3-compatible, works for both 
  Supabase Storage and Cloudflare R2 — one URL change to migrate)
- PDF export: Headless Chromium via Rust subprocess call to 
  a separate Cloud Run container running Puppeteer (TypeScript)
- Cloud Run settings: sessionAffinity: true, timeoutSeconds: 3600, 
  minInstances: 1 (required for WebSocket stability)

AGENT / MCP SERVICE (Separate):
- Language + Framework: TypeScript + @modelcontextprotocol/sdk
- Hosted: Google Cloud Run (separate service from Rust API)
- Job queue: BullMQ (async background jobs: 
  "Listen the Script" generation, AI analysis, continuity checks, 
  pitch generation, breakdown generation)
- Queue storage: Upstash Redis (ioredis — required for BullMQ)
- AI calls: OpenRouter via Anthropic SDK 
  (ANTHROPIC_BASE_URL=https://openrouter.ai/api)
- Primary model: claude-sonnet-4-5 (complex agents)
- Fast model: claude-haiku-3-5 (quick tasks, autocomplete)
- Fallback: GPT-4o → Gemini 1.5 Pro (if Claude rate-limits)
- CRITICAL: Agent service NEVER touches database directly. 
  ALL writes go through Rust API. Every agent write goes through 
  approval queue — writer reviews and approves before any change 
  is committed to the script.
- TTS: OpenRouter TTS models (for "Listen the Script" voice generation)
- Image generation: DALL-E 3 via OpenRouter (for scene imagery)

DATABASE:
- Supabase PostgreSQL
- Connection: Supavisor pooler only (port 6543)
- RLS: enabled on every single table — no exceptions
- ORM: sqlx with compile-time checked queries in Rust

AUTH:
- Provider: Supabase Auth
- Methods: Magic Link (email via Resend) + Google OAuth
- Token format: JWT RS256
- Rust backend: verify every JWT using jsonwebtoken crate 
  + Supabase JWKS endpoint on every authenticated request
- Flutter client: uses supabase_flutter anon key
- Rust backend: uses service_role key (never exposed to client)
- Token refresh: handled by supabase_flutter automatically

STORAGE:
- Phase 1: Supabase Storage (PDFs, FDX exports, audio files, 
  generated images)
- Phase 3+: Cloudflare R2 (migration — same aws-sdk-s3 code, 
  one URL change)
- All file URLs are signed (time-limited, never permanent public URLs 
  for private files)

CACHE & QUEUES:
- Provider: Upstash Redis (serverless-compatible, 
  no connection pool issues on Cloud Run)
- Rust: fred.rs crate
- TypeScript: ioredis (required for BullMQ)
- Stores: presence data, agent job status, rate limit counters, 
  script context cache (for AI), session data

EMAIL:
- Provider: Resend SDK
- Uses: Magic Link auth, collaboration invites, 
  notification digests, review assignments

PAYMENTS:
- Provider: Stripe
- Stripe Checkout for subscriptions
- Stripe Connect for reviewer payouts
- Stripe Customer Portal for self-service management
- Webhook verification on every webhook event

REAL-TIME COLLABORATION:
- Engine: yrs + yrs-tokio (Yjs CRDT, embedded in Rust API)
- Flutter client: y_crdt package
- No proxy layer. No separate Node.js service.
- Flutter client auto-reconnects on drop; Yjs reconciles state 
  on reconnect.
- Cursor presence: stored in Upstash Redis, broadcast via 
  WebSocket connection on the Rust backend.

OBSERVABILITY:
- SDK: OpenTelemetry (opentelemetry-rust + tracing-opentelemetry 
  in Rust; @opentelemetry/sdk-node in TypeScript)
- Export: Google Cloud Trace (free on Cloud Run)
- Error tracking: Sentry (Rust + Flutter + TypeScript)
- Analytics: PostHog (Flutter Web + mobile)

CI/CD:
- GitHub Actions → build → deploy to Cloud Run + Vercel
- Environments: development, staging, production
- Secrets: Google Secret Manager (never hardcoded, never in .env 
  committed to repo)

═══════════════════════════════════════════════════════════════
PART 3 — SECURITY ARCHITECTURE
═══════════════════════════════════════════════════════════════

LIPILY must have zero known security vulnerabilities. Every layer 
follows security best practices and SOLID principles. This is 
non-negotiable and must be reflected in every file.

REQUEST AUTHENTICATION FLOW (Every API call):
Flutter (anon JWT from Supabase Auth)
  → Rust Axum backend (extracts Bearer token from Authorization header)
  → jsonwebtoken crate verifies RS256 signature against Supabase JWKS
  → Extract user_id from validated JWT claims
  → All DB queries use service_role but scoped to verified user_id
  → Response

No request to the Rust API is processed without valid JWT verification.
No exceptions. Even "public" endpoints verify origin and apply rate limits.

SECURITY RULES (enforce in every file that touches security):

1. NEVER expose service_role key to Flutter client. 
   Flutter uses only the anon key.
2. NEVER accept unvalidated input. All Rust handlers use 
   typed structs with serde + custom validators. All TypeScript 
   handlers use Zod schemas.
3. NEVER allow direct Supabase writes from Flutter client for 
   any sensitive operation. All writes go: 
   Flutter → Rust API (JWT verified) → Supabase (service_role).
   Read-only RLS-protected queries are the only exception.
4. Agent service (TypeScript MCP) NEVER touches Supabase directly. 
   All agent writes go through Rust API with service-to-service auth 
   (internal API key, rotated monthly, stored in Secret Manager).
5. Every Supabase table has RLS enabled. 
   Default policy: deny all. Explicit allow policies only.
6. Rate limiting on every endpoint via Upstash Redis:
   - Auth endpoints: 5 req/min per IP
   - Write endpoints: 60 req/min per user
   - AI endpoints: per subscription tier (Free: 50/day, Pro: 500/day, 
     Studio: unlimited)
   - Public endpoints: 30 req/min per IP
7. CORS: whitelist only. Origins: [lipily.com, *.lipily.com, 
   localhost:3000 (dev only)]. No wildcard.
8. CSP headers on all web responses. No inline scripts in Flutter web 
   output except CanvasKit bootstrap.
9. Stripe webhooks: verify Stripe-Signature header on every webhook 
   using Stripe webhook secret. Reject all unverified webhook calls.
10. File uploads: validate MIME type server-side (not just extension). 
    Max file size: 50MB for scripts, 500MB for audio. 
    Virus scan via ClamAV on all uploads before storage.
11. SQL injection: impossible by design (sqlx compile-time checked 
    queries, zero string concatenation in SQL).
12. Secrets: all secrets in Google Secret Manager, injected as 
    environment variables at Cloud Run startup. Never in code, 
    never in Docker image, never in git.
13. JWT expiry: 1-hour access tokens, 7-day refresh tokens. 
    Refresh token rotation on every use.
14. WebSocket auth: JWT verified on WebSocket upgrade handshake. 
    Document ID validated against user's access rights before 
    Yjs session established.
15. Generated content safety: all AI-generated images and text 
    pass through OpenRouter's moderation layer. 
    Additional server-side content check before storage.
16. Dependency scanning: GitHub Actions runs `cargo audit` (Rust) 
    and `npm audit` (TypeScript) on every PR. Block merge on 
    high-severity vulnerabilities.
17. GDPR/IT Act compliance: user PII stored in Supabase EU region. 
    Indian users notified of data location. Right to erasure: 
    account deletion purges all PII within 30 days. 
    Scripts are user's intellectual property — LIPILY claims zero 
    ownership. Zero training on user scripts without explicit opt-in.
18. No user under 13. Age verification checkbox on signup.
19. All API responses include security headers:
    X-Content-Type-Options: nosniff
    X-Frame-Options: DENY
    Referrer-Policy: strict-origin-when-cross-origin
    Permissions-Policy: camera=(), microphone=(), geolocation=()
20. Penetration testing checklist must be run before every major 
    release (OWASP Top 10 at minimum).

═══════════════════════════════════════════════════════════════
PART 4 — PRICING TIERS
═══════════════════════════════════════════════════════════════

FREE (forever, no credit card):
- 3 scripts max (hard limit enforced server-side)
- Basic editor (all formatting features)
- Scene navigator, beat board (view only), story bible (3 entries max)
- PDF + FDX + Fountain export
- 3 named drafts
- 3 collaborators per script
- Basic AI: 50 AI requests/day (scene intelligence debounced 
  extraction counts as 1 per save)
- 1 "Listen the Script" generation per month
- Read-only sharing links (3 active at once)
- Public profile + 1 published script

PRO ($12/month or $99/year):
- Unlimited scripts
- Full AI suite (500 AI requests/day)
- Full story bible, beat board, storyboard
- Unlimited named drafts + autosave history
- Version history (30 days)
- Up to 10 collaborators per script
- Unlimited "Listen the Script" generations
- Review submission (submit scripts for formal review)
- Script breakdown generation
- Unlimited read-only sharing links
- Full public profile + unlimited published scripts
- Branch editing + merge requests

STUDIO ($39/month per team):
- Everything in Pro
- Writers' Room mode (20 simultaneous users)
- Showrunner controls + scene-level locking
- Admin dashboard
- Production export (breakdown sheets, cast reports)
- SSO + enterprise security
- Dedicated support
- Student seat discounts (verified .edu email)
- Unlimited AI requests

═══════════════════════════════════════════════════════════════
PART 5 — USER PERSONAS
═══════════════════════════════════════════════════════════════

ARJUN (25, Mumbai): Emerging writer. Phone + laptop. No money for 
Final Draft. Writes in Hindi and English. Dreams of Bollywood break. 
Uses WhatsApp to share scripts. Pain: no recognition, no feedback, 
no professional tool.

PRIYA (32, Mumbai): Professional TV writer on 3 shows simultaneously. 
Needs collaboration and version control. Frustrated by 
WhatsApp script sharing and email chains.

MARCUS (38, Hollywood): Staff writer. Writers' room veteran. 
Needs real-time multi-user editing, scene locking, showrunner 
approval workflow.

AISHA (28, Lagos): Independent filmmaker. Uses Scrivener + 
Final Draft + Notion + Google Docs — 4 tools for one job. 
Needs story bible, beat board, and editor unified.

DEV (21, Film school student): Wants to review scripts to 
build portfolio and earn. Entry point into the industry.

RAJESH (50, Film producer, Mumbai): Browses for fresh scripts. 
Wants reviewed/rated content. Cannot read 120-page PDFs cold. 
Needs "Listen the Script" to evaluate quickly.

═══════════════════════════════════════════════════════════════
PART 6 — BUILD PHASES
═══════════════════════════════════════════════════════════════

Phase 1 (Sprint 0-2): Supabase schema → Rust API → Fountain parser 
→ Flutter editor → Yjs sync. NO AI yet. Core writing tool only.

Phase 2 (Sprint 2-3): MCP service → first AI agents → 5-pillar 
scene intelligence → approval queue UI → BullMQ background jobs.
Full AI suite.

Phase 3 (Sprint 3-5): Social layer — profiles, publishing, 
"Listen the Script", discovery feed.

Phase 4 (Sprint 5-7): Review marketplace, producer portal, 
leaderboard, Writers' Room, advanced production tools, 
observability + scale.

═══════════════════════════════════════════════════════════════
PART 7 — FILES TO CREATE
═══════════════════════════════════════════════════════════════

Create ALL files listed below in this exact folder structure:

docs/
├── constitution.md
├── prd.md
├── architecture.md
├── AGENTS.md
├── STATUS.md
└── stories/
    ├── story-001-project-setup-infrastructure.md
    ├── story-002-authentication.md
    ├── story-003-dashboard.md
    ├── story-004-core-editor-engine.md
    ├── story-005-scene-navigator.md
    ├── story-006-focus-mode.md
    ├── story-007-scene-intelligence-5-pillar.md
    ├── story-008-story-bible-panel.md
    ├── story-009-collaboration-invite-roles.md
    ├── story-010-realtime-collaboration-yjs.md
    ├── story-011-comment-threads.md
    ├── story-012-git-style-versioning-drafts.md
    ├── story-013-branch-editing-merge-requests.md
    ├── story-014-smart-deletion-recovery.md
    ├── story-015-alternate-dialogue-variants.md
    ├── story-016-character-location-manager.md
    ├── story-017-beat-board.md
    ├── story-018-offline-first-powersync.md
    ├── story-019-export-production-pdf.md
    ├── story-020-export-fountain-fdx.md
    ├── story-021-script-import.md
    ├── story-022-readonly-sharing-link.md
    ├── story-023-global-search.md
    ├── story-024-inline-scene-tagging.md
    ├── story-025-title-page-editor.md
    ├── story-026-find-replace.md
    ├── story-027-script-formatting-preferences.md
    ├── story-028-script-statistics-goals.md
    ├── story-029-revision-mode-production.md
    ├── story-030-storyboard-shot-visualization.md
    ├── story-031-script-breakdown-auto-generation.md
    ├── story-032-inspiration-research-mode.md
    ├── story-033-ai-dialogue-suggest.md
    ├── story-034-ai-continue-scene-writers-block.md
    ├── story-035-ai-continuity-checker.md
    ├── story-036-ai-pacing-structure-analyzer.md
    ├── story-037-ai-logline-pitch-generator.md
    ├── story-038-ai-character-voice-guardian.md
    ├── story-039-ai-multilingual-support.md
    ├── story-040-writer-profiles-usernames.md
    ├── story-041-script-publishing.md
    ├── story-042-listen-the-script.md
    ├── story-043-social-feed-discovery.md
    ├── story-044-leaderboard.md
    ├── story-045-review-submission.md
    ├── story-046-reviewer-program-payouts.md
    ├── story-047-producer-discovery-portal.md
    ├── story-048-subscription-stripe.md
    ├── story-049-notifications.md
    ├── story-050-content-moderation-reporting.md
    ├── story-051-keyboard-shortcuts-accessibility.md
    ├── story-052-onboarding-flow.md
    ├── story-053-writers-room-mode.md
    ├── story-054-dialogue-read-aloud-tts.md
    └── story-055-admin-dashboard.md

═══════════════════════════════════════════════════════════════
PART 8 — CORE PLANNING FILE SPECIFICATIONS
═══════════════════════════════════════════════════════════════

──────────────────────────────────────────────────────────────
FILE 1: constitution.md
──────────────────────────────────────────────────────────────

§0 — THE HUMAN AUTHORSHIP PRINCIPLE (Supreme Law)
Write this as the unconditional governing philosophy above all 
other rules. Include all of:
1. The writer owns every word in their script. Always.
2. AI never writes directly into the script — always into a 
   suggestion layer first.
3. Every AI output — dialogue suggestions, continuations, 
   extracted metadata, pitch documents, voice assignments, 
   generated imagery, structural analysis, continuity flags — 
   is EDITABLE by the human at any time, including after 
   acceptance. "Accepted" AI content is NOT permanent.
4. AI-generated content must always be visually distinguishable 
   from human-written content until the human explicitly edits it.
5. Every AI feature must have: (a) manual trigger, (b) one-click 
   dismiss, (c) full-edit mode, (d) regenerate option, 
   (e) "regenerate with instructions" custom prompt field.
6. 5-Pillar sidebar items are AI-suggested. Every item is 
   editable, deletable, and manually addable at any time. 
   Confirmed items remain editable forever.
7. AI-assigned TTS voices are editable from a voice picker 
   at any time before and during playback generation.
8. AI-generated scene imagery can be regenerated with a custom 
   prompt or replaced with writer's own uploaded image at any time.
9. AI-selected background music can be changed from a music 
   picker at any time.
10. Every agent write goes through approval queue. Writer reviews 
    and approves before any change is committed to the script.
This principle must be checked before implementing any AI feature.

§1 — Locked Tech Stack
List every technology from Part 2 with: technology name, version, 
purpose, which service uses it (Flutter / Rust API / TypeScript MCP). 
Every item explicitly marked LOCKED. Include the note that no 
substitution is permitted without a written approval comment in 
this section.

§2 — Hard Architecture Rules (25+ rules)
Include:
- Never expose service_role key in Flutter client
- Flutter client uses anon key only for auth; all writes go via 
  Rust API
- Agent/MCP service never touches Supabase directly — all writes 
  via Rust API internal endpoint
- Every JWT verified on every authenticated request in Rust using 
  jsonwebtoken + Supabase JWKS
- sqlx compile-time checked queries only — no raw string SQL
- Riverpod AsyncNotifier pattern only in Flutter — no setState, 
  no ChangeNotifier
- Sealed Result types for all Dart operations that can fail
- Zod schemas for all TypeScript handler inputs
- Serde structs with validators for all Rust handler inputs
- PowerSync for all offline sync — no manual sync logic
- yrs + yrs-tokio for all real-time collaboration — no other CRDT
- Supavisor pooler only for DB connections (port 6543)
- All secrets in Google Secret Manager
- Never hardcode any credential, API key, or secret anywhere

§3 — Performance Non-Negotiables (15+ targets)
- Cold start < 3s on 4G (Flutter Web)
- Editor input latency < 16ms (single keypress to screen)
- Yjs cursor sync < 200ms
- Offline queue flush < 5s on reconnect
- PDF export < 30s for 120-page script
- AI suggestions < 3s p95 (dialogue suggest, continue scene)
- "Listen the Script" generation < 60s for 10-page scene 
  (background job — user not blocked)
- FTS search (Cmd+K) < 200ms
- Flutter Web bundle < 4MB initial load (CanvasKit)
- 60fps minimum scroll on editor (120fps target)
- Supabase query p95 < 100ms
- Cloud Run Rust API p95 < 200ms
- WebSocket reconnect < 2s
- Yjs state reconciliation on reconnect < 1s
- PostHog event flush < 500ms (non-blocking)

§4 — Offline-First Rules
- PowerSync config: list all tables that sync offline
- Drift offline_write_queue schema (full table definition)
- Retry strategy: exponential backoff starting 1s, doubling, 
  max 5 retries, then dead letter queue
- Conflict resolution: server wins for collaborative scripts, 
  client wins for solo scripts
- All Drift queries use typed DAOs
- Focus Mode: forces offline save mode, no sync attempts while 
  Focus Mode is active
- Offline indicator: subtle banner, not intrusive

§5 — Editor Rules
- Fountain format is the canonical internal representation
- All blocks stored as typed Fountain elements with a `type` enum: 
  SCENE_HEADING, ACTION, CHARACTER, DIALOGUE, PARENTHETICAL, 
  TRANSITION, SHOT
- Smart Flow Logic matrix: define Enter behavior AND Tab behavior 
  for every block type (full 7×2 matrix, all 14 transitions)
- SmartType: trigger character after minimum 1 char in CHARACTER 
  block, location after INT./EXT. in SCENE_HEADING block
- Autosave: every 30 seconds minimum, on every navigation away 
  from editor, on Focus Mode exit
- Yjs document structure: describe how blocks map to Yjs shared 
  types (Y.Array of Y.Map)

§6 — AI Rules
- All AI calls originate from TypeScript MCP service, never 
  from Flutter client directly, never from Rust API directly
- All AI calls use OpenRouter via Anthropic SDK
- 5-pillar extraction debounced 2s on block save
- Rate limits enforced server-side in Rust (Upstash Redis): 
  Free 50/day, Pro 500/day, Studio unlimited
- Every AI action is opt-in. No AI feature runs automatically 
  without explicit user trigger (except debounced 5-pillar 
  extraction which user can disable globally)
- ContentSource enum must be attached to every AI-generated item: 
  {aiGenerated, humanConfirmed, humanEdited, humanWritten}
- Only `aiGenerated` state shows AI badge in UI
- Approval queue: every agent write creates an approval record. 
  Flutter polls approval queue. Writer sees diff of proposed change. 
  Writer approves or rejects. Only on approval does Rust API 
  commit the change.
- Model selection: claude-sonnet-4-5 for complex agents 
  (continuity check, pacing analysis, pitch generation). 
  claude-haiku-3-5 for fast tasks 
  (dialogue suggest, 5-pillar extraction, SmartType)
- Fallback chain: Claude → GPT-4o → Gemini 1.5 Pro 
  (handled by OpenRouter, transparent to application code)

§7 — Security Rules (all 20 rules from Part 3 of this prompt)
Write every rule in full. This section is the security bible.

§8 — Scope
v1 (Phase 1-2): Core editor + offline + Yjs collab + 
AI scene intelligence + full AI suite + exports + imports + 
story bible + beat board + versioning + branch editing + 
all editor utility features
v2 (Phase 3): Social layer — profiles, publishing, 
"Listen the Script", feed, leaderboard, reviews, 
producer portal
v3 (Phase 4): Writers' Room mode, Dialogue Read-Aloud TTS, 
Student Tier, Production integrations, Scale infrastructure
NEVER list: Integrated DMs/messaging, Stealth Mode, 
Scene Marketplace/Forking (separate product decision), 
Virus scan before v3 (use Supabase Storage policies until scale)

§9 — Data Model Constraints
- UUIDs everywhere (gen_random_uuid())
- All timestamps TIMESTAMPTZ (not TIMESTAMP)
- Position ordering: gap strategy (1000, 2000, 3000 — 
  rebalance trigger at gap < 10)
- Soft deletes: deleted_at TIMESTAMPTZ nullable — 30-day retention 
  then permanent purge job
- All text fields have explicit CHECK length constraints
- No nullable foreign keys
- ContentSource enum in DB: 
  ('ai_generated', 'human_confirmed', 'human_edited', 'human_written')
- Username: UNIQUE, CITEXT extension for case-insensitive uniqueness

§10 — Deployment Rules
- Cloud Run Rust API: minInstances=1, sessionAffinity=true, 
  timeoutSeconds=3600 (WebSocket stability)
- Cloud Run TypeScript MCP: minInstances=0 (scale to zero when idle)
- GitHub Actions: build on every PR, deploy to staging on merge to 
  develop, deploy to production on merge to main only
- Zero-downtime deploys: Cloud Run traffic splitting 
  (10% → 50% → 100% with 5-min intervals)
- Rollback: < 5 minutes via Cloud Run traffic split revert
- Secret rotation: every 90 days minimum, automated via 
  Secret Manager + Cloud Run auto-redeploy

§11 — Multilingual Rules
- All text storage UTF-8 in PostgreSQL
- RTL language support: Flutter Directionality widget per script
- No Latin-only assumptions in any regex or validation
- Font fallback stack must include Noto Sans for non-Latin scripts
- AI features detect script language and respond in same language
- No hardcoded language in any error message — all strings 
  in Flutter l10n system

§12 — Accessibility Rules
- WCAG 2.1 AA minimum on all screens
- All interactive elements ≥ 44px tap target
- Color contrast ≥ 4.5:1 for all body text
- No information conveyed by color alone
- All images have meaningful alt text
- Screen reader support for editor (aria-live regions for 
  block type labels on Flutter Web)
- Keyboard navigable end-to-end (no mouse required)
- prefers-reduced-motion respected on all animations

──────────────────────────────────────────────────────────────
FILE 2: prd.md
──────────────────────────────────────────────────────────────

§1 — Problem Statement
The recognition gap for writers in Indian cinema and globally. 
The Final Draft monopoly and its 8 structural failures 
(pricing, desktop-only, bugs, bloat, weak collab, import chaos, 
no AI, steep learning curve). The industry-wide gaps no tool 
solves: unified story workspace, intelligent character tracking, 
real Writers' Room support, transparent AI, git-style versioning, 
fair pricing, dialogue read-aloud, cross-platform consistency. 
The "thick PDF" problem — scripts sit unread. The discovery gap.

§2 — Market Positioning
Competitor table with columns: Tool, Price, AI Features, 
Real-Time Collab, Web-Based, Story Tools, Beginner-Friendly, 
Social Discovery, Offline Support.
Rows: Final Draft 13, Fade In Pro, WriterDuet, Arc Studio Pro, 
Celtx, Squibler/Melies, LIPILY.
LIPILY row must show dominance in every column.

§3 — User Personas
All 6 personas (Arjun, Priya, Marcus, Aisha, Dev, Rajesh) with: 
age, location, occupation, primary device, current tools used, 
top 3 pain points, what success looks like on LIPILY, 
which stories are built for them.

§4 — Epic Table
All epics with: ID, name, description, priority (P0/P1/P2/P3), 
phase (1/2/3/4), story count.
Epics: Core Editor, AI Scene Intelligence, AI Feature Suite, 
Collaboration & CRDT, Version Control & Branching, Story Tools, 
Offline & Sync, Export & Import, Social Platform, 
Discovery & Feed, Review Marketplace, Production Tools, 
System & Infrastructure.

§5 — All 55 User Stories
Every story in full GIVEN/WHEN/THEN format. Minimum 8 acceptance 
criteria per story. Stories grouped by epic.

§6 — Feature Gate Table
Every feature mapped to Free / Pro / Studio with exact limits 
(numbers, not "unlimited" where a specific limit exists).

§7 — Success Metrics
v1: DAU target (Month 3), D7 retention, D30 retention, 
Free→Pro conversion rate, NPS target, script completion rate.
v2: "Listen the Script" average plays per post, 
review turnaround SLA hit rate, producer contact requests per week.
v3: Writers' Room sessions per day, MRR growth rate, 
churn rate target.

──────────────────────────────────────────────────────────────
FILE 3: architecture.md
──────────────────────────────────────────────────────────────

§1 — System Architecture Overview
Full Mermaid diagram showing:
Flutter (Vercel) → Rust Axum API (Cloud Run) → Supabase DB
Flutter → PowerSync → Supabase (offline sync path)
Flutter ↔ Rust API WebSocket (Yjs CRDT)
Flutter ← Supabase anon (read-only RLS-protected reads)
Rust API → Supabase (service_role writes)
Rust API → Upstash Redis (cache, rate limits, presence)
Rust API → Supabase Storage / R2 (files)
TypeScript MCP (Cloud Run) → BullMQ (Upstash Redis)
TypeScript MCP → OpenRouter API (AI)
TypeScript MCP → Rust API (all writes via internal API)
Rust API → Resend (email)
Rust API → Stripe (payments)
Rust API → Puppeteer Cloud Run (PDF export)
Describe read path and write path separately in prose.

§2 — Complete PostgreSQL Schema
ALL tables with full SQL DDL (CREATE TABLE, column types, 
constraints, CHECK constraints, indexes, RLS policies). 
Every table has RLS enabled, created_at, updated_at.

Required tables (minimum — add any others needed):
profiles (id, user_id, username CITEXT UNIQUE, display_name, 
  bio, location, avatar_url, account_type ENUM, subscription_tier, 
  is_admin, deleted_at)
scripts (id, owner_id, title, format ENUM, fountain_content, 
  word_count, scene_count, page_count_estimate, language, 
  rtl BOOLEAN, status ENUM, deleted_at)
scenes (id, script_id, heading, position, color_tag, deleted_at)
blocks (id, scene_id, script_id, type ENUM, content TEXT, 
  position, metadata JSONB — stores alternate variants, 
  inline tag refs, content_source ContentSource ENUM, 
  deleted_at)
scene_intelligence (id, scene_id, script_id, environment JSONB, 
  props JSONB, characters JSONB, music JSONB, emotion JSONB, 
  last_extracted_at, extraction_model)
story_bible_entries (id, script_id, type ENUM, title, content, 
  metadata JSONB, position, content_source)
beat_notes (id, scene_id, script_id, summary, emotional_beat, 
  structural_tag, content_source)
collaborators (id, script_id, user_id, role ENUM, invited_at, 
  accepted_at)
invites (id, script_id, inviter_id, email, role, token, 
  expires_at, accepted_at, status ENUM)
comments (id, block_id, script_id, author_id, content, 
  parent_id, resolved_at, resolved_by)
drafts (id, script_id, created_by, name, revision_color ENUM, 
  fountain_snapshot, block_count, page_count_estimate, 
  is_autosave, created_at)
branches (id, script_id, name, created_by, base_draft_id, 
  status ENUM, fountain_snapshot, created_at)
merge_requests (id, branch_id, script_id, submitted_by, 
  reviewer_id, status ENUM, review_comments JSONB, 
  merged_at, rejected_at, created_at)
script_tags (id, block_id, scene_id, script_id, tag_type ENUM, 
  tagged_text, position_start, position_end, created_by)
characters (id, script_id, name, aliases TEXT[], role ENUM, 
  description, age_range, arc_summary, voice_notes, 
  first_scene_id, content_source)
locations (id, script_id, name, type ENUM, description, 
  content_source)
ai_usage (id, user_id, script_id, feature_type, model_used, 
  tokens_used, created_at)
agent_approvals (id, user_id, script_id, agent_type, 
  proposed_change JSONB, status ENUM, approved_at, rejected_at, 
  created_at)
subscriptions (id, user_id, stripe_customer_id, 
  stripe_subscription_id, tier ENUM, status ENUM, 
  current_period_end, cancel_at_period_end)
script_publications (id, script_id, user_id, genres TEXT[], 
  logline, tags TEXT[], visibility ENUM, published_at, 
  view_count, like_count, bookmark_count)
script_presentations (id, script_id, user_id, 
  voice_assignments JSONB, music_track_id, 
  scene_images JSONB, assembly_metadata JSONB, 
  generation_status ENUM, generated_at, playback_url)
reviews (id, script_id, reviewer_id, status ENUM, 
  scores JSONB, written_notes TEXT, submitted_at, 
  completed_at)
reviewer_profiles (id, user_id, tier ENUM, credentials TEXT, 
  verified_at, total_reviews, stripe_connect_id)
producer_profiles (id, user_id, company_name, verified_at, 
  genre_preferences TEXT[])
recently_deleted (id, user_id, script_id, scene_id, block_id, 
  deleted_type ENUM, content JSONB, original_position, 
  restore_expires_at, created_at)
writing_sessions (id, user_id, script_id, started_at, ended_at, 
  words_written, scenes_modified, session_type ENUM)
shots (id, scene_id, script_id, shot_type ENUM, 
  camera_movement ENUM, description, ai_image_url, 
  custom_image_url, position, content_source)
user_follows (follower_id, following_id, created_at — 
  composite PK, unique constraint)
script_likes (id, user_id, script_publication_id, created_at — 
  unique constraint on user+publication)
script_bookmarks (id, user_id, script_publication_id, created_at)
notifications (id, user_id, type ENUM, reference_id UUID, 
  reference_type TEXT, is_read, created_at)
reports (id, reporter_id, script_id, reason ENUM, 
  description TEXT, status ENUM, resolved_at, resolved_by)
research_notes (id, user_id, script_id, scene_id nullable, 
  title, content, reference_images JSONB, created_at)
formatting_preferences (id, script_id, user_id, page_size ENUM, 
  font_size INTEGER, margins JSONB, scene_numbers_display ENUM, 
  header_footer JSONB, created_at)

§3 — Drift Local Schema (Dart)
Write complete Dart Drift table definitions for:
offline_write_queue: id TEXT PK, operation_type TEXT, 
  table_name TEXT, record_id TEXT, payload TEXT (JSON), 
  retry_count INTEGER DEFAULT 0, last_error TEXT nullable, 
  created_at INTEGER (unix ms), status TEXT DEFAULT 'pending'
cached_blocks: id TEXT PK, script_id TEXT, scene_id TEXT, 
  content TEXT (JSON), synced_at INTEGER, is_dirty BOOLEAN
Write as proper @DataClassName and @UseRowClass Drift annotations.

§4 — Rust API Contract
Every endpoint with: HTTP method, path, auth requirement 
(public / bearer JWT / internal service key), 
request struct (Rust serde with field types and validators), 
response struct, error codes.
Grouped by domain:
/auth/* (magic link init, OAuth callback, session refresh, logout)
/scripts/* (CRUD, list, search)
/scenes/* (CRUD, reorder)
/blocks/* (CRUD, reorder, tag, alternate variants)
/collaboration/* (invite, accept, revoke, list collaborators)
/branches/* (create, list, submit merge request, review)
/versioning/* (save draft, list drafts, get diff, restore)
/ai/* (trigger scene intelligence, dialogue suggest, 
  continue scene, continuity check, pacing analysis, 
  pitch generate, voice guardian, approval queue CRUD)
/export/* (PDF, FDX, Fountain, breakdown PDF)
/import/* (FDX, Fountain, txt)
/social/* (publish, unpublish, feed, like, bookmark, follow, profile)
/listen/* (generate, status, get playback)
/review/* (submit, assign, complete, reviewer CRUD)
/producer/* (search scripts, request contact)
/subscription/* (checkout, portal, webhook)
/notifications/* (list, mark read)
/moderation/* (report, admin queue)
/search/* (global Cmd+K search)
/presence/* (WebSocket endpoint for Yjs + cursor)

§5 — Riverpod Provider Hierarchy
Full provider tree in Dart pseudocode using riverpod_generator 
syntax. Show all AsyncNotifier providers, dependencies, 
and what they expose. Must include:
authProvider, profileProvider, scriptListProvider, 
activeScriptProvider, sceneListProvider, blockListProvider, 
sceneIntelligenceProvider, storyBibleProvider, 
collaboratorsProvider, branchListProvider, 
mergeRequestProvider, draftListProvider, 
aiSuggestionProvider, approvalQueueProvider, 
publicationProvider, listenPresentationProvider, 
reviewProvider, subscriptionProvider, notificationsProvider, 
followProvider, storyboardProvider, formattingPreferencesProvider, 
writingSessionProvider, tagListProvider, searchProvider.

§6 — go_router Route Tree
All named routes with: path, name constant, parameters, 
auth guard (public / requires auth / requires Pro / requires Studio), 
redirect logic.
Include all routes: /splash, /onboarding, /auth/login, 
/auth/callback, /dashboard, /editor/:scriptId, 
/editor/:scriptId/scene/:sceneId, 
/beat-board/:scriptId, /story-bible/:scriptId, 
/storyboard/:sceneId, /breakdown/:scriptId, 
/branch/:branchId, /merge-request/:mergeRequestId,
/publish/:scriptId, /listen/:scriptId (public), 
/listen/:scriptId/edit (Pro), 
/discover, /discover/genre/:genre, /leaderboard, 
/@:username (public), 

/@:username/script/:scriptId (public), 
/review/:scriptId, /reviewer/dashboard, 
/producer/portal, /producer/script/:scriptId,
/settings, /settings/shortcuts, 
/settings/formatting, /settings/account, 
/admin (admin role only).

§7 — Editor Widget Tree
Full Flutter widget hierarchy for the EditorScreen. 
Write as indented pseudocode showing every widget:
EditorScreen (ConsumerStatefulWidget)
  └─ EditorScaffold
       ├─ EditorTopBar
       │    ├─ ScriptTitleField
       │    ├─ CollaboratorAvatarRow (live cursors list)
       │    ├─ FocusModeButton
       │    ├─ BranchStatusChip (shows current branch name)
       │    ├─ DraftSaveButton
       │    └─ ExportMenuButton
       ├─ EditorBody (Row)
       │    ├─ SceneNavigatorPanel (collapsible left panel)
       │    │    ├─ SceneListView
       │    │    │    └─ SceneCard (heading, page est, 
       │    │    │         color tag, 5-pillar completeness dots)
       │    │    └─ AddSceneButton
       │    ├─ BlockEditorView (center, expanded)
       │    │    ├─ BlockList (CustomScrollView)
       │    │    │    └─ BlockItem (per block)
       │    │    │         ├─ BlockTypeLabel (left margin)
       │    │    │         ├─ BlockTextField (main content)
       │    │    │         ├─ AlternateDialogueRow (if dialogue block)
       │    │    │         ├─ AIBadge (if content_source = aiGenerated)
       │    │    │         ├─ InlineTagChips (non-printing)
       │    │    │         └─ CommentThreadDot (if has comments)
       │    │    ├─ SmartTypeAutocompleteOverlay
       │    │    └─ CollaboratorCursorLayer (positioned overlay)
       │    └─ SceneIntelligenceSidebar (collapsible right panel)
       │         ├─ EnvironmentPillar
       │         │    └─ ExtractedItemChip (editable, 
       │         │         dismissible, confirmable)
       │         ├─ PropsPillar (same chip pattern)
       │         ├─ CharactersPillar (same)
       │         ├─ MusicPillar (same)
       │         └─ EmotionPillar (same)
       └─ EditorBottomBar (mobile only)
            ├─ ElementTypeSelector (swipeable)
            └─ AIToolsButton

§8 — Yjs CRDT + WebSocket Architecture
Describe the full real-time collaboration architecture:
- Yjs document structure: Y.Doc contains Y.Array<Y.Map> for blocks.
  Each Y.Map has keys: id, type, content, position, metadata, 
  content_source, deleted_at.
- WebSocket endpoint: GET /presence/:scriptId (Rust Axum upgrade)
- On connection: verify JWT, verify user has access to scriptId, 
  load Y.Doc from Supabase (serialized as binary in scripts table 
  or a separate yjs_snapshots table), send initial sync message.
- On update: receive Y.Update binary from client, apply to Y.Doc, 
  broadcast to all other connected clients on same scriptId.
- Persistence: on each update, persist Y.Doc snapshot to 
  Supabase (debounced 5s). On reconnect, send full Y.Doc 
  state to reconnecting client. Yjs handles merge automatically.
- Cursor presence: separate from Yjs. Stored in Upstash Redis 
  key: presence:{scriptId}. TTL 5s (refreshed every 3s by client). 
  Rust broadcasts cursor updates to all WebSocket connections 
  on same scriptId via Tokio broadcast channel.
- Cursor payload: {userId, username, displayName, avatarColor, 
  blockId, position, updatedAt}
- Flutter client: y_crdt package. Auto-reconnect with exponential 
  backoff. On reconnect, resend pending Y.Updates from 
  Drift offline queue.

§9 — "Listen the Script" Generation Pipeline
Step-by-step BullMQ background job in TypeScript MCP service.
Each step is a separate BullMQ job with retry logic:

Job: listen_generate:{scriptId}:{userId}

Step 1 (fetch_data):
  - Call Rust API: GET /listen/:scriptId/data
  - Receive: all blocks in order, scene_intelligence for each scene, 
    character list with voice_notes, script title, writer username
  - Store in Redis job context

Step 2 (assign_voices):
  - For each unique character in characters array:
    If voice already assigned (from script_presentations table): reuse
    Else: assign from voice pool based on character gender hint 
    from scene_intelligence characters JSONB
  - Store {characterName: voiceId} map in job context
  - Call Rust API: PATCH /listen/:scriptId/voices with assignments
  - APPROVAL GATE: writer sees voice assignments, 
    can change any voice before proceeding

Step 3 (generate_audio — parallel per scene):
  - For each dialogue block: call OpenRouter TTS with assigned voice
  - For each action block: call OpenRouter TTS with narrator voice
  - Store audio file URLs in Supabase Storage under 
    listen/{scriptId}/{sceneId}/{blockId}.mp3
  - All audio files are writer-replaceable via upload

Step 4 (generate_imagery — parallel per scene):
  - If scene has storyboard shots (shots table): 
    use shot.custom_image_url if set, else shot.ai_image_url
  - Else: build DALL-E prompt from scene_intelligence 
    environment + props + emotion + characters
  - Generate image via OpenRouter DALL-E 3
  - Store in Supabase Storage under 
    listen/{scriptId}/{sceneId}/image.jpg
  - All images writer-replaceable with custom prompt 
    or file upload

Step 5 (select_music):
  - Map scene_intelligence.emotion.scene_tone to music category:
    {tension: 'suspense', romantic: 'romance', action: 'action', 
    grief: 'emotional', comedy: 'lighthearted', default: 'neutral'}
  - Select track from curated Supabase Storage music library
  - Store track selection in job context
  - APPROVAL GATE: writer sees music selection, 
    can swap from picker before final assembly

Step 6 (assemble_timeline):
  - Build assembly_metadata JSONB: array of timed segments
    [{type: 'dialogue', blockId, audioUrl, characterName, 
    voiceId, subtitleText, startMs, durationMs},
    {type: 'action', blockId, audioUrl, subtitleText, 
    imageUrl, musicTrackId, startMs, durationMs}, ...]
  - Calculate total duration

Step 7 (finalize):
  - Call Rust API: PATCH /listen/:scriptId/presentation with 
    final assembly_metadata, generation_status: 'ready', 
    playback_url
  - Notify writer via notifications table

Step 8 (publish — only if writer triggers):
  - Writer reviews full presentation in preview player
  - Writer can: re-record any block audio (regenerate with 
    custom prompt), replace any image, change any voice, 
    change music — at any time before and after publishing
  - On publish: set script_presentations.published = true

Show full Mermaid sequence diagram for this pipeline.

═══════════════════════════════════════════════════════════════
PART 9 — AGENTS.md SPECIFICATION
═══════════════════════════════════════════════════════════════

AGENTS.md must contain all of the following sections, 
fully written out with no truncation:

SECTION 1: Agent Role & Philosophy
This agent is a senior full-stack engineer implementing LIPILY — 
a professional screenwriting tool + AI social platform. 
You implement one story at a time, following TDD. You read 
constitution.md before every task. You never add features 
not in the current story's acceptance criteria. You treat 
the Human Authorship Principle (§0) as the supreme law.

SECTION 2: ALWAYS Rules (30 rules minimum)
Include all of:
- Read constitution.md §0 (Human Authorship Principle) 
  before implementing any AI feature
- Read the complete story file before writing any code
- Read constitution.md §1 (tech stack) before importing 
  any new dependency
- Write failing tests first, then implementation (TDD)
- Use sealed Result<T, AppError> types for all Dart operations 
  that can fail
- Use Riverpod AsyncNotifier pattern for all state — 
  never setState, never ChangeNotifier
- Verify JWT on every authenticated Rust handler using 
  jsonwebtoken + Supabase JWKS — every handler, no exceptions
- Use sqlx compile-time checked queries — never string 
  concatenation in SQL
- Apply rate limiting check (Upstash Redis) on every 
  AI endpoint before processing
- Enforce ContentSource enum on every AI-generated item 
  created or updated
- Route all agent writes through approval queue 
  (agent_approvals table) — never write directly to 
  script content tables from TypeScript MCP service
- Handle all three UI states on every screen: 
  loading (skeleton), error (with retry), empty (with action)
- Handle offline state in every Flutter feature — 
  what happens with no internet must be explicitly coded
- Test at 375px mobile viewport for every Flutter Web screen
- Enforce subscription tier checks server-side in Rust — 
  never trust client-side tier claims
- Use PowerSync for all offline sync — never write manual 
  sync logic
- Use y_crdt for all collaborative block editing — 
  never use direct API calls for content sync on collaborative scripts
- Run cargo audit before every Rust PR merge
- Run npm audit before every TypeScript PR merge
- Write OpenTelemetry span for every Rust handler and 
  TypeScript agent job
- Update STATUS.md after completing every story
- Check that every new DB table has RLS enabled and 
  a deny-all default policy before creating it
- Use Upstash Redis (fred.rs in Rust, ioredis in TypeScript) 
  for all caching — never in-memory cache in Cloud Run 
  (stateless containers)
- All secrets must come from environment variables 
  injected by Secret Manager — never hardcode
- Follow the folder structure defined in §4 exactly

SECTION 3: NEVER Rules (30 rules minimum)
Include all of:
- Never expose service_role key in Flutter client
- Never write directly to Supabase from TypeScript MCP service
- Never skip JWT verification on any authenticated handler
- Never use raw string SQL in Rust — sqlx only
- Never use setState or ChangeNotifier in Flutter
- Never use dynamic or any type in TypeScript
- Never use dynamic type in Dart
- Never add a feature not in the current story's acceptance criteria
- Never truncate story files, architecture docs, or constitution
- Never commit secrets, API keys, or credentials to git
- Never use localStorage or sessionStorage in Flutter Web 
  (sandboxed iframes block it — use Drift or in-memory)
- Never generate AI content without explicit user trigger 
  (except debounced 5-pillar extraction which user can disable)
- Never commit AI-generated content directly to script blocks 
  without going through agent_approvals queue
- Never show AI-generated content without visual AI badge 
  (until human edits or confirms it)
- Never skip the ContentSource enum on AI-generated items
- Never allow AI to lock or permanently set anything — 
  every AI output is editable forever
- Never skip the offline behavior implementation in any story
- Never skip empty states — every list view needs 
  an empty state with action
- Never use colored side borders on cards (visual anti-pattern)
- Never skip mobile (375px) testing on any Flutter Web screen
- Never skip subscription tier enforcement on Pro/Studio features
- Never use direct DB connection string — Supavisor pooler only
- Never skip rate limit check on AI endpoints
- Never skip Stripe webhook signature verification
- Never mark a story as DONE without verifying all acceptance 
  criteria pass

SECTION 4: Project Folder Structure
Write the complete folder structure to file level for both 
frontend/ and backend/ and mcp-service/.

frontend/ (Flutter):
lib/
  main.dart
  app.dart (MaterialApp.router setup)
  core/
    constants/ (api_constants.dart, app_constants.dart, 
                route_constants.dart)
    errors/ (app_error.dart — sealed class, 
             failure.dart, result.dart)
    extensions/ (string_ext.dart, datetime_ext.dart, 
                 context_ext.dart)
    theme/ (app_theme.dart, app_colors.dart, 
            app_typography.dart, app_spacing.dart)
    utils/ (debouncer.dart, fountain_formatter.dart, 
            content_source.dart)
    widgets/ (loading_skeleton.dart, empty_state.dart, 
              error_state.dart, ai_badge.dart, 
              subscription_gate.dart)
  features/
    auth/ (data/, domain/, presentation/)
    dashboard/ (data/, domain/, presentation/)
    editor/ (data/, domain/, presentation/)
      presentation/
        widgets/ (block_item.dart, block_editor_view.dart,
                  scene_navigator_panel.dart,
                  scene_intelligence_sidebar.dart,
                  pillar_widget.dart,
                  alternate_dialogue_row.dart,
                  collaborator_cursor_layer.dart,
                  smart_type_overlay.dart,
                  focus_mode_overlay.dart)
    story_bible/ (data/, domain/, presentation/)
    beat_board/ (data/, domain/, presentation/)
    storyboard/ (data/, domain/, presentation/)
    collaboration/ (data/, domain/, presentation/)
    versioning/ (data/, domain/, presentation/)
    branching/ (data/, domain/, presentation/)
    ai/ (data/, domain/, presentation/)
      presentation/
        widgets/ (approval_queue_sheet.dart,
                  ai_suggestion_panel.dart,
                  writers_block_panel.dart,
                  continuity_flags_panel.dart,
                  pacing_dashboard.dart,
                  pitch_generator_panel.dart)
    export/ (data/, domain/, presentation/)
    import/ (data/, domain/, presentation/)
    social/ (data/, domain/, presentation/)
    listen/ (data/, domain/, presentation/)
    discovery/ (data/, domain/, presentation/)
    review/ (data/, domain/, presentation/)
    subscription/ (data/, domain/, presentation/)
    notifications/ (data/, domain/, presentation/)
    settings/ (data/, domain/, presentation/)
    admin/ (data/, domain/, presentation/)
  l10n/ (app_en.arb, app_hi.arb, etc.)
  
backend/ (Rust):
src/
  main.rs
  lib.rs
  config.rs (Config struct, from_env())
  db/ (pool.rs — Supavisor connection)
  auth/ (jwt.rs — verify_jwt(), claims.rs, middleware.rs)
  cache/ (redis.rs — fred.rs client, rate_limit.rs, presence.rs)
  routes/ (mod.rs, auth.rs, scripts.rs, scenes.rs, 
           blocks.rs, collaboration.rs, branches.rs,
           versioning.rs, ai.rs, export.rs, import.rs,
           social.rs, listen.rs, review.rs, producer.rs,
           subscription.rs, notifications.rs, 
           moderation.rs, search.rs, presence.rs,
           admin.rs)
  models/ (one file per domain entity, serde structs)
  services/ (script_service.rs, scene_service.rs, 
             block_service.rs, ai_service.rs, 
             export_service.rs, import_service.rs,
             subscription_service.rs, 
             notification_service.rs,
             fountain_parser.rs, fdx_parser.rs,
             presence_service.rs, yjs_service.rs)
  ws/ (handler.rs — WebSocket upgrade, 
       yjs_handler.rs — Yjs sync logic,
       cursor_handler.rs — presence broadcast)
  errors/ (app_error.rs — AppError enum, 
           error_handler.rs)
  middleware/ (auth_middleware.rs, 
               rate_limit_middleware.rs,
               cors_middleware.rs,
               security_headers_middleware.rs,
               logging_middleware.rs)
  telemetry/ (tracing_setup.rs)

mcp-service/ (TypeScript):
src/
  index.ts
  config.ts
  queue/ (worker.ts — BullMQ worker, 
          jobs/ (listen_generate.ts, 
                 continuity_check.ts,
                 pacing_analysis.ts,
                 pitch_generate.ts,
                 breakdown_generate.ts,
                 voice_assign.ts,
                 imagery_generate.ts))
  agents/ (scene_intelligence_agent.ts,
           dialogue_suggest_agent.ts,
           continue_scene_agent.ts,
           continuity_agent.ts,
           pacing_agent.ts,
           pitch_agent.ts,
           voice_guardian_agent.ts,
           translation_agent.ts)
  tools/ (mcp tool definitions for each agent)
  api/ (rust_client.ts — typed HTTP client for Rust API,
        openrouter_client.ts)
  schemas/ (zod schemas for all job payloads and API inputs)
  middleware/ (auth.ts — service-to-service key verification)
  utils/ (content_source.ts, approval_queue.ts)
  telemetry/ (otel_setup.ts)

SECTION 5: Dart Style Guide (20 rules)
Include patterns with complete code examples for:
- AsyncNotifier with riverpod_generator (full example: 
  ScriptListNotifier with fetchScripts, createScript, 
  deleteScript)
- Sealed Result class (full implementation: 
  Result<T, AppError>, Success<T>, Failure<AppError>)
- AppError sealed class (full implementation: 
  NetworkError, AuthError, ValidationError, ServerError, 
  OfflineError)
- Drift DAO pattern (full example: BlockDao with 
  watchBlocksForScene, insertBlock, updateBlock, 
  markAsDeleted)
- go_router guard pattern (full example: redirect function 
  checking auth state and subscription tier)
- ContentSource enum usage in provider (full example: 
  how to set source when accepting AI suggestion)
- Approval queue consumer pattern (full example: 
  polling approvalQueueProvider, showing diff, 
  calling approve/reject)

SECTION 6: Rust Style Guide (20 rules)
Include patterns with complete code examples for:
- Axum handler with JWT extraction (full example: 
  async fn get_script handler with Extension<Claims>, 
  Path<Uuid>, sqlx query, error mapping)
- AppError enum (full implementation with IntoResponse)
- JWT verification middleware (full implementation using 
  jsonwebtoken + cached JWKS)
- Rate limit middleware (full implementation using fred.rs 
  + Upstash Redis)
- sqlx query pattern (full example: query_as! macro with 
  typed struct)
- WebSocket handler (full example: upgrade handshake, 
  JWT check, Yjs document load, sync loop)
- Security headers middleware (full implementation setting 
  all 6 required headers)

SECTION 7: TypeScript Style Guide (15 rules)
Include patterns with complete code examples for:
- BullMQ job definition (full example: ListenGenerateJob 
  with payload type, processor function, retry config)
- Zod schema pattern (full example: CreateScriptSchema 
  with validation)
- OpenRouter call via Anthropic SDK (full example: 
  claude-sonnet-4-5 call with system prompt, 
  model fallback logic)
- Rust API client call with service auth header 
  (full example: typed fetch wrapper)
- Approval queue write pattern (full example: 
  writing proposed change to agent_approvals via Rust API)

SECTION 8: ContentSource Pattern
Full description + code in both Dart and TypeScript:
- ContentSource enum definition (both languages)
- Rules: only aiGenerated shows AI badge; 
  humanEdited and humanConfirmed are treated as humanWritten 
  in UI; the source field travels from DB → API → provider → UI
- Dart: how AIBadge widget reads source from block model
- Dart: how acceptance updates source via provider
- Dart: how "regenerate with instructions" resets source 
  to aiGenerated and shows badge again
- TypeScript: how agent sets source when creating approval record

SECTION 9: Error Handling Patterns
- Full Dart sealed AppError class
- Full Rust AppError enum with IntoResponse
- How errors bubble: Rust handler → HTTP response → 
  Dio error → Dart service → Result<T, AppError> → 
  Riverpod AsyncValue.error → UI error state widget
- Toast vs inline error decision matrix:
  Auth errors → redirect to login
  Validation errors → inline next to field
  Network errors → retry banner
  Permission errors → upgrade prompt (if subscription gate)
  Server errors (500) → inline error with support link
  AI errors → inline in AI panel with retry button

SECTION 10: Story Execution Protocol
Exact steps the agent MUST follow for every single story:
1. Read the complete story file — all acceptance criteria, 
   all error states, offline behavior, mobile behavior
2. Read constitution.md §0 (Human Authorship Principle) 
   if story touches AI
3. Read constitution.md §1 to confirm all dependencies 
   are in tech stack
4. Read architecture.md §2 for all DB tables this story touches
5. Read architecture.md §4 for all API endpoints this story uses
6. Write test file first — unit tests for services, 
   widget tests for screens, integration test for happy path
7. Implement DB schema changes first (if any)
8. Implement Rust API changes
9. Implement TypeScript MCP changes (if any)
10. Implement Flutter provider changes
11. Implement Flutter UI
12. Verify every acceptance criteria passes manually 
    before marking done
13. Verify at 375px mobile viewport
14. Verify offline behavior (disable network, confirm 
    Drift queue fills correctly)
15. Verify dark mode
16. Update STATUS.md: change story status from DRAFT → DONE
17. Write a one-line "Decision Log" entry in STATUS.md 
    for any architecture decision made during implementation

═══════════════════════════════════════════════════════════════
PART 10 — STATUS.md SPECIFICATION
═══════════════════════════════════════════════════════════════

STATUS.md must contain:

Header: LIPILY — Sprint Tracker
Current Sprint: Sprint 0
Sprint 0 Goal: Infrastructure + Auth + Dashboard + 
  Supabase schema + Rust API skeleton + Flutter project setup
Sprint 0 Dates: [today] → [today + 14 days]

Story Status Table (all 55 stories):
Columns: Story ID | Title | Epic | Phase | Priority 
         | Status | Sprint | Assigned | Blockers

All stories start with Status: DRAFT

Sprint Plan Table:
Sprint 0 (stories 001-003): Infrastructure, Auth, Dashboard
Sprint 1 (stories 004-008): Editor Engine, Navigator, 
  Focus Mode, Scene Intelligence, Story Bible
Sprint 2 (stories 009-015): Collaboration, Comments, 
  Versioning, Branching, Smart Deletion, Alternate Dialogue, 
  Character Manager
Sprint 3 (stories 016-024): Beat Board, Offline, Export, 
  Import, Sharing, Search, Inline Tagging, Title Page, 
  Find & Replace
Sprint 4 (stories 025-032): Formatting Prefs, Stats, 
  Revision Mode, Storyboard, Breakdown, Research Mode, 
  AI Dialogue Suggest, AI Continue Scene
Sprint 5 (stories 033-039): AI Continuity, Pacing Analyzer, 
  Pitch Generator, Voice Guardian, Multilingual, 
  Writer Profiles, Publishing
Sprint 6 (stories 040-047): Listen The Script, Feed, 
  Leaderboard, Reviews, Reviewer Program, Producer Portal, 
  Subscriptions, Notifications
Sprint 7 (stories 048-055): Content Moderation, Shortcuts, 
  Onboarding, Writers Room, Read-Aloud TTS, Admin Dashboard, 
  Full QA Pass, Performance Audit

Blockers Section: empty

Decisions Log Section: empty (agent writes here during build)

═══════════════════════════════════════════════════════════════
PART 11 — ALL 55 STORY FILE SPECIFICATIONS
═══════════════════════════════════════════════════════════════

Every story file MUST include these sections:
- Story ID, Title, Epic, Phase, Priority (P0/P1/P2/P3), 
  Status: DRAFT
- Persona (which persona this story primarily serves)
- Problem Statement (1 paragraph — why this matters)
- User Story: "As a [persona], I want [action] so that [outcome]"
- GIVEN/WHEN/THEN acceptance criteria (minimum 8)
- Performance criteria (specific ms/s targets from §3)
- Offline behavior (what works, what degrades, what queues)
- Mobile behavior (375px specific layout + gesture considerations)
- Error states (minimum 3 failure scenarios with exact behavior)
- Security considerations (what auth checks, what rate limits)
- Human Authorship considerations (if AI involved — what is 
  editable, what shows AI badge, what goes through approval queue)
- Dependencies (story IDs that must be DONE first)
- DB tables touched
- API endpoints used
- Test cases (minimum 5: happy path + 4 edge cases)
- Out of scope (explicit list)

Story descriptions for all 55 files:

story-001: Initialize Flutter project (all 5 platforms), 
Rust Axum backend with health endpoint, Docker setup, 
Cloud Run deployment config (minInstances=1, 
sessionAffinity=true, timeoutSeconds=3600), complete 
Supabase schema (all tables from architecture §2), 
all RLS policies with deny-all defaults and explicit 
allow policies, PowerSync configuration, GitHub Actions 
CI/CD pipeline (PR build → staging deploy → production 
deploy on main), Sentry setup (Rust + Flutter + TypeScript), 
PostHog setup (Flutter Web + mobile), all environment 
variables in Secret Manager, Vercel deployment, 
Upstash Redis setup, TypeScript MCP service skeleton 
with BullMQ connected.

story-002: Magic Link email auth via Supabase + Resend, 
Google OAuth, profile creation on first login with unique 
@username selection (validation: 3-30 chars, alphanumeric + 
underscore, CITEXT unique check, 1 change per 30 days), 
JWT storage in Flutter secure storage, JWT refresh logic 
via supabase_flutter, JWT verification middleware in Rust, 
auth guards on all protected go_router routes, 
logout from all devices, age verification checkbox 
(no under-13 policy), account_type selection 
(Writer / Producer / Student / Reviewer).

story-003: Script list on dashboard, create script 
(title required, format: Feature Film / TV Half-Hour / 
TV One-Hour / Limited Series / Short Film / Stage Play), 
soft delete with recently deleted recovery, script card 
(title, format chip, last edited, word count, scene count, 
collaborator avatars, branch badge if active branch), 
empty state (animated — "Your first script is waiting"), 
search/filter scripts, sort by: last edited / created / 
alphabetical, Free tier: hard 3-script limit enforced 
server-side with upgrade prompt.

story-004: Full Fountain-compliant block editor. All 7 
element types with correct formatting. Smart Flow Logic 
matrix (full 7×2 Enter/Tab matrix — write out all 14 
transitions). SmartType autocomplete (character after 
1 char in CHARACTER block, location after INT./EXT. in 
SCENE_HEADING). All blocks stored as typed Fountain 
elements via Rust API. Yjs integration via y_crdt for 
collaborative editing. Input latency < 16ms. Page 
estimate calculation (1 page = 55 lines rule). 
Word count, scene count, real-time. Autosave every 
30 seconds. Handles Unicode (Hindi, Tamil, Arabic, etc). 
RTL support via Flutter Directionality.

story-005: Left panel scene list. Scene cards with: heading, 
page estimate, color tag (8 colors), 5-pillar completeness 
dots (one per pillar, green if confirmed items exist). 
Drag-to-reorder using gap position strategy. Tap to scroll 
editor to scene. Add scene (inserts new SCENE_HEADING block). 
Delete scene (soft delete). Scene collapse toggle. 
On 375px: scene navigator becomes a bottom sheet, 
accessible via swipe-up gesture or button.

story-006: Focus Mode toggle (Cmd+Shift+F). On activate: 
hides SceneNavigatorPanel and SceneIntelligenceSidebar, 
fades all non-active block text to 20% opacity, centers 
active line (typewriter mode — viewport scrolls to keep 
cursor at 40% from top), disables all in-app notifications 
(queued and shown on exit), forces offline save mode 
(PowerSync sync disabled while active, all writes queue 
to Drift), Midnight Mode (background #000000, text #E8E8E8). 
Exit via Escape or floating exit FAB. 
Keyboard shortcut: Cmd+Shift+F toggle.

story-007: The core AI differentiator. On every block save 
(debounced 2s): Flutter → Rust API: POST /ai/extract/:sceneId 
→ Rust API queues BullMQ job → TypeScript agent extracts 
5 pillars using claude-haiku-3-5 (fast, cheap) with 
surrounding 10-block context. Result stored in 
scene_intelligence table. Flutter polls or receives 
WebSocket notification. SceneIntelligenceSidebar updates. 
Each extracted item carries content_source: aiGenerated, 
shows AI badge. Writer can: tap item to edit text 
(source → humanEdited), tap confirm checkmark 
(source → humanConfirmed), tap X to dismiss (stored 
as dismissed, not re-extracted), tap + to add item manually 
(source: humanWritten). All confirmed items remain editable. 
Rate limited: counted against AI usage quota. 
User can globally disable 5-pillar extraction in settings.

story-008: Persistent right-side Story Bible panel 
(separate route on mobile: /story-bible/:scriptId). 
5 tabs: Characters, Locations, Themes, Backstory, Timeline. 
Characters tab: list of character cards (auto-populated 
from scene_intelligence + story-016 character manager). 
Each card: name, role chip, arc summary (editable), 
voice notes (editable), scenes count, aliases. 
Locations tab: location cards with type, description, 
scene count. Themes/Backstory/Timeline tabs: freeform 
rich text editor. Hover over character name in 
BlockEditorView shows character hover card linking to 
Story Bible entry. All content editable at any time. 
All content synced offline via PowerSync.

story-009: Owner invites by email via POST /collaboration/invite. 
Rust API creates invite record, calls Resend to send email 
with JWT-signed accept link (7-day expiry). Accept link 
validates JWT, creates collaborator record, redirects to script. 
Roles: Owner, Lead Writer, Editor, Writer, Viewer. 
Permissions matrix: Owner (all), Lead Writer 
(edit + invite + merge), Editor (edit + comment), 
Writer (edit own sections + comment), Viewer (view + comment). 
Revoke collaborator. Transfer ownership (requires 
confirmation from both parties). Pending invites list. 
Email uses Resend with LIPILY-branded template.

story-010: Yjs real-time collaboration. Flutter y_crdt 
client connects via WebSocket to Rust /presence/:scriptId. 
JWT verified on upgrade. Y.Doc synced bidirectionally. 
All block edits go through Y.Doc — no direct API writes 
for content changes on collaborative scripts. Cursor presence 
via Upstash Redis (TTL 5s, refreshed every 3s). 
CollaboratorCursorLayer renders colored name tags at 
block positions. Max 10 visible cursors (Free/Pro). 
Auto-reconnect with exponential backoff. 
On reconnect: Yjs reconciles state, pending offline 
queue flushed. Graceful degradation: if WebSocket drops, 
show "Working offline — changes will sync on reconnect."

story-011: Inline comment threads on any block. Tap block 
→ "Add Comment" option. Comment shows: avatar, 
@username, timestamp, text (supports @mentions). 
Reply threads (2 levels max). Resolve thread button 
(owner or commenter). Resolved threads collapsible. 
CommentThreadDot on block right margin (red if unresolved). 
Unresolved count badge in top bar. Comments panel 
(right slide-over): all comments for script, filter: 
all/unresolved/mine. Tap comment → scrolls editor 
to block. Notification on new comment to script owner 
and mentioned users (@mention lookup).

story-012: Named draft saves (Cmd+S brings up save dialog 
if new draft, else saves to current). WGA revision color 
auto-assignment sequence. Visual diff viewer: select 
any two drafts → diff computed server-side (Rust, 
based on Fountain line diff) → returned as annotated 
Fountain. Diff rendered in read-only editor with 
color highlighting. Any draft can be restored 
(creates new draft from that content — never overwrites). 
Scene-specific history: list of all changes to a scene 
with author, timestamp, diff preview. Autosave: every 
60s creates is_autosave=true draft. Autosaves retained 
30 days then purged by scheduled Cloud Run job.

story-013: Create isolated branch from any draft or 
current main. Branch editor shows "BRANCH: [name]" 
banner in top bar. All edits on branch do not affect 
main script. Branch stored as fountain_snapshot in 
branches table (not live Yjs — branch editing is solo, 
not collaborative). Submit Merge Request when ready. 
Merge request screen: shows diff (same diff viewer as 
story-012). Owner sees MR queue badge on dashboard card. 
Owner can: Approve & Merge (applies fountain_snapshot 
to main, creates new named draft), Request Changes 
(inline comments on diff), Reject (with reason). 
Conflict detection: if main changed since branch was 
created, show conflict resolver (side-by-side, 
writer chooses which version of each conflicted section wins).

story-014: Shift+Backspace triggers smart deletion. 
Before deleting: content is saved to recently_deleted 
table with original position metadata. Auto-protection 
triggers: (1) entire scene deleted, (2) 5+ consecutive 
blocks selected-and-deleted, (3) rapid bulk deletion 
(>10 blocks in 5s). Recently Deleted panel: accessible 
from editor sidebar. Shows deleted items: preview, 
time deleted, script/scene name, "Restore" button. 
Restore re-inserts at original position (or end of 
scene if position is now occupied). Items purged after 
30 days. Permanent delete button with confirmation.

story-015: Any Dialogue block stores variants as JSONB 
array in blocks.metadata.variants. Each variant: 
{id, content, content_source, created_at}. 
AlternateDialogueRow widget appears on hover/focus: 
left arrow, right arrow, variant count (e.g., "2 of 3"), 
add variant (+), delete current variant (if > 1 variant), 
mark as current (exports this variant). Cycle keyboard 
shortcut: Cmd+Alt+← / →. All variants editable. 
AI suggestions (story-033) can be inserted as new variants 
with content_source: aiGenerated (shows AI badge). 
Variant 1 is always the "current" unless writer 
explicitly marks another. Only "current" variant exports.

story-016: Dedicated Characters screen and Locations screen 
(accessible from story bible). Character record: name, 
aliases, physical description, age range, role ENUM, 
arc summary, voice notes (for AI Voice Guardian and TTS), 
key traits, first appearance scene, all scenes they appear 
in (computed from scene_intelligence). Merge duplicates 
(AI suggests merges: "RAVI" and "Ravi" are likely the same). 
Location record: name, INT/EXT type, description, 
atmosphere notes, all scenes. Both auto-populated from 
5-pillar extraction (story-007) with content_source: aiGenerated. 
All fields editable. SmartType in editor uses this data.

story-017: Beat Board view (separate view toggle, not a 
separate route). Each scene renders as an index card: 
scene heading, beat summary (editable, max 200 chars), 
color tag, page estimate, emotional beat label (from 
scene_intelligence). Cards in horizontal scrollable 
timeline. Drag to reorder (updates scene positions — 
same gap strategy). Structural overlay toggle: 
3-Act Structure, Save the Cat (15 beats with labels), 
Sequence Method. Overlay renders beat name labels at 
their expected page positions as horizontal markers. 
Cards falling >15% outside expected range get a 
soft amber highlight. Beat notes stored in beat_notes 
table (content_source tracked).

story-018: PowerSync SDK configuration for Flutter. 
All tables that need offline access defined in 
PowerSync schema. Drift offline_write_queue: all writes 
when offline go to queue first. Queue drain on reconnect: 
exponential backoff 1s→2s→4s→8s→16s, max 5 retries, 
then dead letter. Conflict resolution: server wins for 
collaborative scripts (Yjs handles), client wins for 
solo. Sync status: per-script sync indicator 
("Synced ✓" / "Syncing..." / "Offline — X changes pending"). 
Focus Mode integration: sync pauses while Focus Mode active.

story-019: Two PDF export modes. 
(1) Full Script PDF: standard screenplay format, 
Courier Prime 12pt, 1" margins (left 1.5" for binding), 
page numbers top right, scene numbers if enabled in 
formatting prefs, WGA standard page breaks, title page 
from story-025. 
(2) Scene Production PDF: for each scene — scene heading, 
INT/EXT, day/night, page count, all characters present 
with emotional states (from scene_intelligence), props list, 
environment details, music/sound cues, emotion/tone, 
inline tags (PROP/VFX/STUNT etc). Batch mode: all scenes 
as ZIP. Cloud Run Puppeteer service generates. < 30s 
for 120-page script. Progress indicator. Download from 
browser/share sheet on mobile.

story-020: Export to .fountain (plain text, Fountain spec 
compliant) and .fdx (Final Draft 12 XML). Both must be 
round-trip compatible. FDX includes: title page, 
scene numbers, WGA revision color markers. 
Fountain is clean plain text. Export via Rust parser 
(fountain_parser.rs + fdx_parser.rs). Available from 
editor File menu and dashboard script card context menu.

story-021: Import .fountain, .fdx (Final Draft 11/12/13), 
.txt. Rust parser converts to LIPILY Fountain block format. 
Handles encoding normalization (UTF-8). Pre-import preview 
with ambiguous lines flagged. On import: auto-trigger 
5-pillar extraction (story-007) to populate story bible. 
< 10s for 120-page script. (Imported) label until renamed. 
Respects multilingual content (story-039 dependency optional — 
import works without AI for any language).

story-022: POST /scripts/:scriptId/share-links creates 
signed JWT link (configurable: 7 days / 30 days / never). 
Public /view/:token route in go_router (no auth required). 
Viewer sees: full formatted script in read-only editor, 
scene navigator, basic stats (word count, scene count, 
page estimate). No edit controls. No account required. 
Revoke: DELETE /scripts/:scriptId/share-links/:linkId. 
View count tracked (anonymous). Free tier: 3 active links. 
Pro: unlimited.

story-023: Cmd+K opens command palette overlay. 
SearchBar with autofocus. Real-time search (debounced 300ms) 
via GET /search?q=&scope=[script|all]&scriptId=. 
Rust uses PostgreSQL pg_trgm FTS on blocks, scene headings, 
character names, script titles. Results grouped by type 
with section headers. Keyboard navigation (↑↓ arrows, 
Enter to jump, Escape to close). Recent searches in memory. 
Scope toggle: current script / all scripts. 
< 200ms p95 response.

story-024: Writer highlights text in any block and 
right-click (or long-press on mobile) → Tag As submenu. 
Tag types: PROP, CAST, LOCATION, VFX, STUNT, WARDROBE, 
MAKEUP, VEHICLE. Tag stored in script_tags table 
(block_id, position_start, position_end, tag_type). 
Tags render as non-printing colored underlines in editor 
(visible only in editor, not in exports). Tag chips 
appear in corresponding scene_intelligence pillar 
(Props pillar, Characters pillar) with content_source: 
humanWritten — these are always treated as human-written, 
never overridden by AI extraction. Tag manager panel: 
all tags by category with scene references. 
Click tag → jump to scene.

story-025: Title page editor accessible from editor 
File menu. Fields: Title, Written By (auto-populated 
from profile displayName), Based On (optional), 
Contact Info (email, phone, address — optional), 
Draft Date (auto-populated, editable), WGA Registration 
Number (optional). Formatted exactly per WGA standard 
(centered title, upper case, proper spacing). 
Preview shows formatted title page. 
Exported as page 1 in all PDF exports.

story-026: Cmd+F opens find panel (bottom bar, non-modal). 
Search field with result count "X of Y". Previous/Next 
navigation (Cmd+G / Cmd+Shift+G). Current match 
highlighted in editor viewport. Replace field (toggle). 
Replace current / Replace All with undo support. 
Scope: entire script / current scene. 
Case-sensitive toggle. Whole word toggle. 
Character rename mode: search for character name across 
all block types (CHARACTER blocks, action mentions, 
dialogue attribution) — replace all instances with 
confirmation count shown before applying.

story-027: Per-script formatting preferences stored in 
formatting_preferences table. Settings: page size 
(US Letter / A4), font size (10pt / 11pt / 12pt Courier Prime), 
margins (standard / wide / narrow — precise mm values stored), 
scene number display (off / left / right / both), 
header (title / draft color / custom text), 
footer (page number position: none / center / right). 
Industry presets: WGA Feature (US Letter, 12pt, standard margins), 
BBC Drama (A4, 12pt, UK standard), Bollywood (A4, 12pt, 
Hindi-friendly), Stage Play (US Letter, 12pt, stage margins). 
Save as named template (stored in formatting_preferences 
with is_template: true). Apply template to new script.

story-028: Writing session auto-starts when user enters 
editor, ends on navigate away or 5min inactivity. 
Session data in writing_sessions table: words_written 
(delta from session start), scenes_modified count, 
duration. Daily goal: set from settings (words per day 
or scenes per day). Progress bar toward goal visible 
in editor top bar (subtle, collapsible). Stats dashboard 
(route: /settings/stats): total words, scenes, scripts, 
average session length, writing streak (consecutive days 
with activity), longest streak, words per genre breakdown. 
Weekly email digest via Resend (opt-in) — sent every 
Monday morning with previous week's stats.

story-029: Owner activates Revision Mode from 
versioning menu. Requires a named draft to be selected 
as the "locked production draft." In Revision Mode: 
new/changed lines get asterisk (*) in right margin 
(WGA standard). Pages are locked — new lines push 
to A/B pages (A1, A2 etc). WGA revision color assigned 
to current revision cycle (shown as colored chip in top bar). 
Scene Omit: marks scene with "OMITTED" label, 
preserves original scene number (scene card in navigator 
shows "OMITTED" state). Only Owner can toggle Revision Mode. 
Revision Mode indicator in top bar. 
Export in Revision Mode respects all page locks and 
asterisk markers.

story-030: Storyboard view accessible per-scene 
(button in scene card and in SceneIntelligenceSidebar). 
Each shot panel: shot type selector (Wide/Medium/Close-Up/
Extreme Close-Up/POV/Insert/Establishing), camera movement 
selector (Static/Pan/Tilt/Zoom/Dolly Track/Handheld/Crane/
Drone), description field (max 300 chars), image area. 
Image area: shows AI-generated image if ai_image_url set, 
else shows "Generate Image" button. AI image generation: 
builds prompt from scene_intelligence environment + props + 
character + shot type + camera movement. 
Writer can: regenerate with custom instructions 
(content_source: aiGenerated → shows AI badge on image), 
or upload own reference image 
(custom_image_url set, content_source: humanWritten — 
this is always used in "Listen the Script" over AI image). 
Shot panels draggable to reorder. Shot list 
(auto-generated production document from all shots) 
exportable as PDF.

story-031: Pro/Studio feature. Triggered from 
script dashboard card: "Generate Breakdown." 
BullMQ background job in TypeScript MCP. 
Sources all data from scene_intelligence + script_tags. 
Generates: (1) Scene Breakdown Sheet — one row per scene: 
scene number, INT/EXT, DAY/NIGHT, pages, location name, 
cast list, props list, VFX notes, stunt notes, 
special requirements. (2) Cast Report — character name, 
scenes count, total pages, all scene numbers. 
(3) Props Master List — prop name, scenes, 
source (scene_intelligence AI or human tag). 
(4) Locations Report — location name, INT/EXT, 
scenes count, total pages. All data writer-editable 
in a breakdown review screen before export. 
Export as formatted PDF or CSV. 
Content_source tracked: AI-extracted items vs 
human-tagged items visually distinguished.

story-032: Research Mode toggle in editor sidebar. 
Opens a slide-over panel. Features: 
(1) Web search — query field, 
results shown inline (title, snippet, URL). 
Results are read-only references, not imported to script. 
(2) Scene Notes — rich text notes attached to current scene 
(stored in research_notes, scene_id set). 
(3) Global Notes — notes not attached to a specific scene 
(scene_id null). (4) Character Mood Board — per character 
in story bible, pin reference images (URLs or uploads, 
stored in research_notes metadata). 
(5) Visual Reference — pin images to any scene as 
visual reference. All research content private, 
never exported, never included in backup exports.

story-033: Pro feature. Manual trigger only. 
AI Dialogue Suggest button appears as subtle icon 
on CHARACTER and DIALOGUE blocks on hover. 
Tap → POST /ai/dialogue-suggest/:blockId. 
Rust queues BullMQ job → TypeScript agent uses 
claude-haiku-3-5 with: character profile 
(from characters table), voice notes, previous 10 blocks 
context, current block content. Returns 3 suggestions. 
Each suggestion shown in AISuggestionPanel: 
text + "Insert as Variant" button + "Replace Current" button. 
Both actions: content_source: aiGenerated, shows AI badge. 
"Replace Current" replaces block content but keeps old 
content as alternate variant. Logged in ai_usage. 
Respects script language (story-039).

story-034: Pro feature. "Unstick Me" button in editor 
toolbar. Shows modal to choose mode FIRST 
(no AI generated until mode chosen): 
(1) Socratic Mode — returns 5 questions as a list, 
no generated script content, shown in panel for writer 
to reflect on. Questions not inserted into script. 
(2) Three Directions — returns 3 scene direction summaries 
(max 3 sentences each, no dialogue). Shown in panel. 
Writer can tap "Use this direction" to insert as a 
beat note (content_source: aiGenerated). 
(3) Rough Placeholder — returns 3-5 blocks of rough draft. 
Writer must explicitly confirm "Insert Placeholder" button. 
Inserted blocks all have content_source: aiGenerated 
and show AI banner. Writer expected to rewrite these. 
Approval queue: each insertion creates approval record, 
but auto-approved for this feature (writer already 
explicitly chose to insert). All inserted content 
fully editable. Logged in ai_usage.

story-035: Pro feature. Manual trigger from AI panel. 
POST /ai/continuity-check/:scriptId → BullMQ job. 
Non-blocking: returns job_id immediately, 
Flutter polls job status. TypeScript agent uses 
claude-sonnet-4-5 with full script content + 
scene_intelligence data. Checks: characters who 
die/leave but reappear, location contradictions 
(e.g., basement with windows), age/timeline 
inconsistencies, prop continuity (object 
appears/disappears), weather inconsistencies 
within continuous time. Each flag: flag_type, 
description, page_references array, block_ids array. 
Stored in continuity_flags table. 
Flutter shows flags as dismissible cards 
("Jump to Scene" link + "Not an Error" button). 
Dismissed flags stored with is_dismissed: true. 
Re-check only on explicit trigger.

story-036: Pro feature. Manual trigger. 
POST /ai/pacing-analysis/:scriptId → BullMQ job. 
Returns: dialogue_to_action_ratio per act 
(computed from block type counts), scene length 
distribution (bar chart data), b_story_gap_data 
(scenes since each non-protagonist character last appeared), 
emotional_intensity_curve (from scene_intelligence 
emotion data per scene), act_balance_percentages. 
Flutter renders as PacingDashboard screen with charts 
(fl_chart package). Flags: 3+ consecutive dialogue-heavy 
scenes, Act 2 >25 pages no major beat, any act >15% 
off structural norm. Each flag: dismissible card with 
page reference + one-line suggestion. 
Zero content rewriting. All output read-only analysis.

story-037: Pro feature. Manual trigger. 
Requires script ≥ 60 pages. 
POST /ai/pitch-generate/:scriptId → BullMQ job. 
TypeScript agent uses claude-sonnet-4-5 with full 
script + scene_intelligence + story bible data. 
Returns: logline (1 sentence), synopsis (1 paragraph), 
one_page_treatment (structured), comp_titles (3-5 films). 
Each output in separate tab in PitchGeneratorPanel. 
All tabs fully editable rich text 
(content_source: aiGenerated → humanEdited on first edit). 
Export all tabs as single formatted PDF pitch document 
(via Cloud Run Puppeteer). Logged in ai_usage.

story-038: Pro feature. Passive — runs per scene on 
every block save (alongside story-007 5-pillar extraction, 
same debounce). TypeScript agent checks: does this 
dialogue line match the character's established voice profile 
(from voice_notes in characters table)? If inconsistency 
detected above confidence threshold: creates a flag in 
scene_intelligence characters JSONB: 
{type: "voice_inconsistency", characterName, blockId, 
confidence, message: "Is this [CHARACTER]?"}. 
Flutter shows flag as a subtle underline on the block 
with a tooltip. Writer can: dismiss flag, 
edit dialogue (flag removed on edit), or view voice profile. 
NEVER rewrites. NEVER inserts content. Only flags.

story-039: Language selector per script in script settings 
(not per account). Language code stored in scripts.language. 
RTL flag in scripts.rtl. Flutter Directionality widget 
wraps BlockEditorView based on scripts.rtl value. All AI agents 
receive language code in system prompt and respond 
in the same language. Font fallback stack includes 
Noto Sans for non-Latin scripts (Hindi, Tamil, Arabic, 
Korean, etc.). SmartType works for non-Latin character 
names. Fountain parsing handles non-Latin content. 
Export (PDF, FDX, Fountain) preserves non-Latin content 
with proper encoding. RTL scripts: PDF export mirrors 
margin layout (binding margin moves to right side).

story-040: Unique @username system. Available on profile 
settings page and during onboarding (story-052). 
Username rules: 3-30 chars, alphanumeric + underscore only, 
CITEXT unique in profiles table, case-insensitive check, 
changeable once every 30 days (last_username_change_at 
timestamp check enforced server-side). Public profile 
route: /@:username (no auth required to view). 
Profile page: display name, bio (max 300 chars), location, 
avatar (Supabase Storage, max 5MB, MIME validated), 
published scripts grid, follower count, following count. 
Follow/unfollow button (auth required). 
Producer account badge. Reviewer badge. 
Profile SEO meta tags for social preview. 
404 for unknown usernames.

story-041: Writer publishes a script via 
"Publish Script" button on dashboard card or editor. 
Publish wizard: (1) Choose visibility: Public / Unlisted 
/ Invite-Only. (2) Add genres (multi-select, max 5 from 
fixed list). (3) Write logline (max 250 chars). 
(4) Add tags (max 10 freeform). (5) Preview card 
as it will appear in feed. (6) Confirm publish. 
Creates record in script_publications. 
Unpublish: one-tap from same menu, 
script_publications.published_at set to null. 
Unpublished scripts not visible to anyone except owner. 
Published scripts viewable at /@:username/script/:scriptId. 
Script format chips auto-populated from scripts.format. 
Update logline/genres/tags post-publish without unpublishing.

story-042: The flagship social format. Full spec 
already written in architecture §9 (Listen The Script 
Generation Pipeline). Additional Flutter implementation:
(1) Generation trigger: writer taps "Create Listen" 
from published script screen. 
(2) BullMQ job created, Flutter shows generation progress 
screen (polling GET /listen/:scriptId/status with 
job step progress: 
fetching → voice assignment → audio gen → 
image gen → music → assembling → ready). 
Each step shows step name and progress %.
(3) APPROVAL GATES displayed inline in progress screen: 
Step 2 (voice assignment): shows voice assignment table, 
writer can change any voice from voice picker, 
tap "Confirm Voices" to continue job.
Step 5 (music): shows selected music with preview, 
writer can change, tap "Confirm Music" to continue.
(4) Preview player screen: full playback experience 
before publish. Writer can: replace any image 
(per-scene regenerate or upload), change any voice 
(triggers re-generation of that character's audio only), 
change music. Each edit re-runs only the affected step.
(5) Published Listen post on social feed: 
auto-plays on scroll (mobile, like a Reel), 
shows subtitles, scene imagery, character name on dialogue. 
Like, bookmark, share. View count tracked.
(6) Playback without LIPILY account: 
public URL works in any browser, 
no login prompt during playback.

story-043: Social discovery feed at /discover. 
Tabs: For You (algorithmic — based on genres followed, 
scripts liked, similar writers), Trending 
(by plays + likes in last 24h), Following 
(scripts and Listen posts from followed writers), 
By Genre (filter chips: Drama, Thriller, Comedy, 
Romance, Horror, Sci-Fi, Action, Bollywood, 
Hollywood, Nollywood, K-Drama, Short Film, 
Stage Play + more). Each feed item is either: 
Listen post (preview thumbnail, play button, 
duration, first line of logline, writer @username, 
like count, play count) or Script Card 
(title, genre chips, logline preview, 
writer @username, review badge if reviewed, 
page count, like count). Infinite scroll 
(cursor-based pagination). Feed algorithm: 
Rust API with Upstash Redis cache (feed TTL: 5 min). 
Genre/format/language/region filter panel. 
Feed visible without auth (For You shows popular, 
not personalized). Like and bookmark require auth.

story-044: Leaderboard at /leaderboard. 
Data updated every 24 hours via scheduled Cloud Run job 
(TypeScript MCP BullMQ cron job). Computed from: 
script_publications (view_count, like_count), 
script_presentations (play count), 
user_follows (follower_count delta), 
reviews (average score). Leaderboard table columns: 
Rank, Writer (@username, avatar, display name), 
Scripts Published, Total Plays, Total Likes, 
Followers, Avg Review Score, Badges. 
Filters: Genre (dropdown), Format (dropdown), 
Region (dropdown — India / US / Africa / Korea / 
UK / Global), Time: Weekly / Monthly / All-time. 
Badges (awarded by scheduled job): 
Rising Star (top 10% follower growth this week), 
Top 10% (top 10% total plays this month), 
Genre Champion (top 3 in any genre filter), 
Reviewed Writer (at least 1 completed formal review). 
Badges stored in profiles.badges TEXT[]. 
Badge award creates notification.

story-045: Pro feature. Review Submission system. 
Writer submits a published script from script page 
("Submit for Review" button — disabled until 
script meets minimum pages: 70 for Feature Film, 
25 for TV format, 10 for Short Film). 
POST /review/:scriptId creates review record 
with status: submitted. Rust API auto-assigns 
to available verified reviewer (round-robin, 
weighted by reviewer queue depth). 
Status lifecycle: Submitted → Assigned 
(notification to reviewer) → In Review → Complete. 
14-day SLA enforced: if no activity from reviewer 
in 14 days → reassigned + admin alert. 
Writer sees status badge on script card 
and notification on each status change. 
On completion: Reviewed badge added to 
script_publications.

story-046: Reviewer Program. Two reviewer tiers: 
(1) Verified Critics: apply via /reviewer/apply, 
manual admin approval (constitution.md §8 scope: 
admin approval flow in v1), approval notification via 
Resend. (2) Student Reviewers: apply with .edu email 
or student ID upload (stored in Supabase Storage, 
MIME-validated), 48h auto-approval if email verified. 
Reviewer dashboard at /reviewer/dashboard: 
assigned scripts queue, review form, completed reviews, 
earnings. Review form: structured fields — 
logline_clarity (1-10), character_depth (1-10), 
structure (1-10), dialogue (1-10), originality (1-10), 
overall_score (1-10), written_notes (TEXT, 300 char min). 
Submit review → status: complete → trigger payout via 
Stripe Connect. Payout amounts: Verified Critics $15, 
Students $5. Stripe Connect onboarding required before 
first payout. Reviewer rating system: writers rate 
reviewer usefulness (1-5 stars) after receiving review.

story-047: Producer Account type. Sign up with 
company_email (verified — must not be personal Gmail/Yahoo). 
Producer profile: company name, verified badge. 
Producer portal at /producer/portal. 
Script browse: filters — genre, format, 
language, region, page range, review score range, 
review status (reviewed only toggle). 
Sort: newest / most reviewed / highest rated / most played. 
Results show script cards (same as feed). 
Click through to /@:username/script/:scriptId (public view). 
"Listen the Script" playback available without auth. 
"Request Contact" button: creates notification to writer. 
Writer receives: producer company name, genre interest, 
message (max 500 chars). Writer can accept or decline. 
Contact requests stored in producer_contact_requests table. 
No personal contact info shared unless writer accepts. 
Writer can block producer.

story-048: Stripe Checkout for Pro and Studio subscriptions. 
POST /subscription/checkout creates Stripe Checkout Session 
with correct price_id (from Secret Manager). 
Redirect to Stripe-hosted checkout page. 
Stripe webhook: /subscription/webhook. 
Verify Stripe-Signature header on every webhook event 
using Stripe webhook secret (from Secret Manager). 
Process events: checkout.session.completed 
(create subscription record, upgrade user tier), 
customer.subscription.updated 
(update tier, period_end), 
customer.subscription.deleted 
(downgrade to Free, enforce Free tier limits). 
Tier enforcement: every Rust API handler checks 
subscription_tier from subscriptions table, 
not from JWT claims. Customer Portal: 
POST /subscription/portal creates Stripe Portal Session 
for self-service plan management and cancellation. 
Annual discount: Pro $99/year (vs $144/year monthly). 
Prorate upgrades (Stripe handles). 
Studio: team seat management in Stripe. 
Downgrade enforcement: if user has >3 scripts 
on Free tier after downgrade, excess scripts archived 
(not deleted) with notification.

story-049: In-app notification center (bell icon in top bar, 
badge with unread count). Notification types:
(1) new_collaborator_joined — script: {scriptId}
(2) merge_request_submitted — branch: {branchId}
(3) merge_request_reviewed — MR: {mrId}, status
(4) comment_added — block: {blockId}, script: {scriptId}
(5) comment_reply — comment: {commentId}
(6) mention_in_comment — comment: {commentId}
(7) review_status_change — review: {reviewId}, status
(8) review_completed — review: {reviewId}
(9) follower_new — follower: {userId}
(10) script_liked — script: {scriptId}
(11) listen_generation_complete — presentation: {id}
(12) badge_earned — badge: {badgeName}
(13) producer_contact_request — request: {id}
(14) invite_accepted — script: {scriptId}, user
(15) subscription_renewed — tier, next_date
(16) subscription_payment_failed — retry_date
Notification panel: sorted newest first, 
grouped by today/yesterday/older, 
tap → navigates to relevant screen, 
mark all as read, delete notification. 
Unread count in tab bar badge (mobile). 
Email digest: weekly summary of unread notifications 
(opt-in, via Resend, Monday morning IST).

story-050: Content moderation and reporting system. 
Report button on every published script and Listen post. 
Report form: reason (ENUM: plagiarism, inappropriate_content, 
copyright_violation, spam, harassment, other), 
description (optional, max 500 chars). 
POST /moderation/report creates record in reports table. 
Auto-hide trigger: script/post receives 3+ reports 
within 24 hours → script_publications.auto_hidden = true 
→ not visible in feed or search (still accessible at 
direct URL with "Under Review" banner). 
Admin moderation queue at /admin (admin role only, 
story-055): lists all pending reports grouped by script. 
Admin actions: approve (restore visibility, 
mark reports as resolved), remove (unpublish script, 
notify writer with reason), permanent ban (deactivate account). 
Appeal system: writer can submit one appeal per report 
via support link. Appeal creates ticket record with 
48h SLA. AI-assisted moderation (Phase 4 scope — 
mark as out of scope for v1 in this story file).

story-051: Full keyboard shortcut system with custom 
remapping. All shortcuts stored in Flutter in-memory map 
(user overrides persisted in Drift locally, synced to 
Supabase via PowerSync). Default shortcut map: 
Cmd+S (save draft), Cmd+F (find), Cmd+G (find next), 
Cmd+Shift+G (find previous), Cmd+K (command palette), 
Cmd+Shift+F (focus mode), Cmd+Z (undo), 
Cmd+Shift+Z (redo), Cmd+/ (shortcut reference panel), 
Tab (next element type in Smart Flow), 
Enter (new block per Smart Flow matrix), 
Cmd+Alt+← (prev dialogue variant), 
Cmd+Alt+→ (next dialogue variant), 
Cmd+Shift+E (export menu), 
Cmd+B (bold — action/scene description emphasis), 
Cmd+I (italic), Cmd+U (underline), 
F1-F7 (force block type: F1=Scene Heading, F2=Action, 
F3=Character, F4=Dialogue, F5=Parenthetical, 
F6=Transition, F7=Shot), 
Escape (close any overlay/panel/focus mode). 
Shortcut reference panel: Cmd+/ opens overlay 
showing all shortcuts grouped by category. 
Remap screen at /settings/shortcuts: 
all shortcuts listed, click shortcut to remap, 
conflict detection (shows warning if conflict). 
WCAG 2.1 AA: all interactive elements 44px touch target. 
Full keyboard navigability: Tab through all interactive 
elements in logical order, focus rings visible at all times. 
Skip link: "Skip to editor" as first focusable element.

story-052: First-run onboarding flow for new users. 
Shown after first auth and username selection. 
Multi-step flow (go_router with onboarding guard, 
redirects to /dashboard on complete or skip). 
Steps:
(1) Welcome screen — LIPILY value prop, 
3 feature highlights with animations.
(2) Account type selection — Writer / Producer / Student / 
Reviewer (updates profiles.account_type).
(3) Interests — genre preferences selection 
(used for feed personalization, stored in 
profiles.genre_preferences TEXT[]).
(4) First script prompt — "Create your first script" 
(name field + format selector) or "Import existing script" 
(Fountain/FDX upload). Skip button available.
(5) Editor tutorial — interactive overlay tutorial 
on the editor with 5 steps: element types, Smart Flow, 
5-pillar sidebar, Focus Mode, Collaboration. 
Can be dismissed and re-triggered from Help menu.
(6) Mobile-specific: tutorial step for bottom element 
selector and swipe-up scene navigator. 
Track onboarding completion via 
profiles.onboarding_complete BOOLEAN. 
PostHog event: onboarding_step_completed, 
onboarding_completed.

story-053: Studio tier feature — Writers' Room Mode. 
Activated by Studio admin on a script from script settings. 
Enables: 20 simultaneous WebSocket connections 
(Yjs handles merge for all), Showrunner Controls panel 
(only account with Owner + Studio admin role): 
scene locking (lock any scene → other writers get 
"Scene locked by Showrunner" tooltip, cannot edit), 
scene assignment (assign scene to a specific writer — 
badge on scene card showing assigned writer's avatar), 
Writers' Room Chat (a separate real-time text channel 
per script via Upstash Redis pub/sub, not saved to DB — 
ephemeral, exists only while WebSocket session is active), 
show runner announcement banner (sticky notice at 
top of editor visible to all collaborators). 
Admin dashboard for Writers' Room: 
activity heatmap (which writer edited which scene, 
when), contribution stats per writer 
(blocks added/modified), session log.

story-054: Dialogue Read-Aloud TTS feature 
(separate from "Listen the Script"). 
Available to all Pro users, triggered in editor. 
Single-scene read-aloud: button in scene card context menu 
or keyboard shortcut Cmd+R. Calls TypeScript MCP 
POST /tts/scene/:sceneId (queued BullMQ job, fast — 
< 10s for a single scene). Uses voice assignments from 
script_presentations if "Listen" has been generated, 
else uses default voice assignments. Audio streamed 
back and played inline in editor — script_presentation 
not required. Playback controls: play/pause, speed 
(0.75x / 1x / 1.25x / 1.5x), current dialogue line 
highlighted in editor as it plays. 
Single block read-aloud: Cmd+Shift+R reads only 
the focused block. Voice settings shortcut: 
right-click CHARACTER block → "Change Voice for [NAME]" 
opens voice picker inline. All generated audio is 
temporary (not stored in Supabase Storage — 
streamed from memory in Cloud Run). 
Part of Phase 3 scope.

story-055: Admin Dashboard at /admin route 
(requires profiles.is_admin = true, enforced in Rust 
middleware). Sections:
(1) Overview: total users, MAU, scripts created, 
    Listen posts generated, active subscriptions, 
    MRR (from Stripe API), today's new signups, 
    today's new scripts, system health indicators 
    (Cloud Run health, Supabase latency, 
    Redis latency from recent pings).
(2) User Management: search by @username / email. 
    View profile. Manual tier override. 
    Suspend account. Delete account 
    (triggers 30-day PII purge job). 
    Grant admin role.
(3) Moderation Queue: pending reports table 
    (from story-050). Batch approve/remove actions.
(4) Reviewer Management: pending reviewer applications, 
    approve/reject, view reviewer profiles, 
    force-complete overdue reviews, 
    manage payout disputes.
(5) Feature Flags: toggle any feature on/off without 
    redeploy (flags stored in Upstash Redis, 
    Rust middleware checks on every request). 
    Emergency AI kill switch (disables all 
    POST /ai/* endpoints).
(6) Audit Log: all admin actions logged with 
    admin_id, action_type, target_id, 
    timestamp (admin_audit_log table, 
    write-only from UI — no delete).
Admin routes protected by both auth middleware 
AND admin role check middleware in Rust. 
All admin actions generate OpenTelemetry spans. 
Admin dashboard itself is Flutter — same codebase, 
gated by is_admin role from profile.

═══════════════════════════════════════════════════════════════
PART 12 — FINAL GENERATION INSTRUCTIONS
═══════════════════════════════════════════════════════════════

You now have EVERY piece of information needed to create 
all files. Execute in this exact order:

GENERATION ORDER:
1. constitution.md — The law. Write this first, 
   in full, all 12 sections, no truncation.
2. architecture.md — All 9 sections (§1-§9). 
   Full PostgreSQL DDL with all tables. 
   Full Mermaid diagrams. No truncation.
3. prd.md — All 7 sections. 
   All 55 stories in GIVEN/WHEN/THEN format. No truncation.
4. AGENTS.md — All 10 sections. 
   All code examples complete and runnable. No truncation.
5. STATUS.md — Sprint tracker with all 55 stories. 
   All tables complete.
6. story-001.md through story-055.md — 
   ALL 55 files, one at a time. 
   Each file with ALL required sections. 
   Every single story must be complete. 
   DO NOT say "remaining stories follow the same pattern." 
   Each story is written in full.

QUALITY CHECKS before you output any file:
□ Does every DB table have RLS enabled and explicit allow policies?
□ Does every API endpoint have JWT verification noted 
  (or explicitly marked public with reason)?
□ Does every AI feature note: manual trigger, AI badge, 
  content_source tracking, approval queue, 
  editable after acceptance?
□ Does every story have offline behavior documented?
□ Does every story have mobile (375px) behavior documented?
□ Does every story have minimum 8 acceptance criteria?
□ Does every story have minimum 3 error states?
□ Does every story have security considerations?
□ Does every story have subscription tier gating 
  where applicable?
□ Is the Human Authorship Principle applied to every 
  AI story (§0 reference)?
□ Are all 20 security rules from Part 3 referenced 
  where applicable throughout all files?
□ Does constitution.md reference all 20 security rules 
  in §7 verbatim?
□ Is the TypeScript MCP service never writing directly 
  to Supabase in any story?
□ Are all Rust handlers using sqlx compile-time queries?
□ Are all Flutter providers using Riverpod AsyncNotifier?
□ Does every story list its DB tables, API endpoints, 
  and story dependencies?
□ Are all 55 stories accounted for in STATUS.md?
□ Is architecture §9 ("Listen the Script" pipeline) 
  a complete Mermaid sequence diagram?
□ Does AGENTS.md §5 have complete runnable Dart examples 
  for all 7 patterns?
□ Does AGENTS.md §6 have complete runnable Rust examples 
  for all 7 patterns?
□ Does AGENTS.md §7 have complete runnable TypeScript 
  examples for all 5 patterns?

NO TRUNCATION RULE: 
If at any point you would write "... (continues)", 
"... (similar pattern for remaining items)", 
"// abbreviated for brevity", 
"[remaining fields follow same pattern]", 
"[see above for similar implementation]", 
or any equivalent shortcut — STOP and write it out in full.
Every list item. Every table row. Every code example.
Every acceptance criterion. Every error state.
No exceptions.

This is a complete, production-ready planning document 
for a real product being built. Every file will be read 
by a developer agent implementing real code. 
Incomplete documentation means incomplete software.

═══════════════════════════════════════════════════════════════
END OF PROMPT
═══════════════════════════════════════════════════════════════


