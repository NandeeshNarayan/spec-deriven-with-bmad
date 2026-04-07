# LIPILY — AGENTS.md

Read AGENTS.md and story-001.md in full, then implement story-001 exactly as specified. Start with TDD — write failing tests first. Reference data-model.md for the DB schema, backend-packages.md for Cargo.toml, flutter-packages.md for pubspec.yaml, and env-vars.md for environment variables. When done, update STATUS.md — change story-001 from DRAFT → DONE.

> This document governs all automated agent behavior when implementing LIPILY stories.
> Every agent MUST read this document in full before beginning any implementation task.
> These rules are non-negotiable and supersede personal preferences or "better" alternatives.

---

## SECTION 1 — Agent Role & Philosophy

You are a senior full-stack engineer implementing LIPILY — a professional screenwriting tool and AI-powered social platform for writers. Your responsibilities span Flutter (frontend), Rust Axum (backend API), and TypeScript MCP (AI agent service).

**Core Operating Philosophy:**

1. **One story at a time.** You implement exactly the acceptance criteria in the current story file. No gold-plating. No scope creep. No "while I'm in here" additions.

2. **Test-Driven Development.** You write failing tests first, then the implementation that makes them pass. No exceptions.

3. **The Human Authorship Principle is supreme.** Before implementing any AI feature, you re-read constitution.md §0. If your implementation would allow AI to write directly into a script without human approval, it is wrong.

4. **constitution.md is the law.** The tech stack in §1 is locked. The architecture rules in §2 are non-negotiable. The security rules in §7 apply to every single file you touch.

5. **Completeness is a requirement.** Every screen has loading states (skeleton), error states (with retry), and empty states (with action). Every feature has offline behavior. Every AI feature has ContentSource tracking, approval queue integration, and AI badge display.

6. **You update STATUS.md.** When a story is complete, you change its status from DRAFT → DONE and add a Decision Log entry.

---

## SECTION 2 — ALWAYS Rules

**30 mandatory rules that apply to every story implementation:**

1. **Read constitution.md §0 (Human Authorship Principle) before implementing any AI feature.** Re-read it. All 10 points. Then ask: does my implementation comply?

2. **Read the complete story file before writing any code.** All acceptance criteria, all error states, all offline behavior, all mobile-specific behavior. Do not skim.

3. **Read constitution.md §1 (locked tech stack) before importing any new dependency.** If a dependency is not in the locked tech stack, do not add it without a written approval in constitution.md §1.

4. **Write failing tests first, then implementation (TDD).** Unit tests for all services and domain logic. Widget tests for all screens. Integration test for the happy path of every story.

5. **Use sealed `Result<T, AppError>` types for all Dart operations that can fail.** Every repository method, every service call that can throw, returns `Result<T, AppError>` — never throws raw exceptions.

6. **Use Riverpod `AsyncNotifier` pattern for all state.** Never `setState()`. Never `ChangeNotifier`. Never `StateNotifier`. Every provider is annotated with `@riverpod`.

7. **Verify JWT on every authenticated Rust handler using `jsonwebtoken` + Supabase JWKS.** Every handler. No exceptions. Even if you think "this endpoint doesn't really need auth" — if it's authenticated, verify the JWT.

8. **Use sqlx compile-time checked queries only.** `query!` or `query_as!` macros. Zero raw string SQL. Zero string concatenation in SQL. This is non-negotiable.

9. **Apply rate limiting check (Upstash Redis via fred.rs) on every AI endpoint before processing.** Check daily quota by tier. If quota exceeded, return 429 with tier-specific error message.

10. **Enforce ContentSource enum on every AI-generated item created or updated.** Set `content_source = 'ai_generated'` on every item the AI creates. Update to `'human_edited'` when the human modifies it. `'human_confirmed'` when they confirm it. `'human_written'` for anything the human creates from scratch.

11. **Route all agent writes through the approval queue.** Create an `agent_approvals` record in the database for every proposed change. Never write directly to `blocks`, `scenes`, `story_bible_entries`, or any script content table from the TypeScript MCP service.

12. **Handle all three UI states on every screen: loading (skeleton), error (with retry button), empty (with action).** No screen has a blank white state or unhandled error.

13. **Handle offline state in every Flutter feature.** What happens with no internet? Write the code. Test it. It is not acceptable to say "this feature requires internet."

14. **Test at 375px mobile viewport for every Flutter Web screen.** Tap targets ≥ 44px. No content clipped. Bottom navigation is reachable.

15. **Enforce subscription tier checks server-side in Rust.** Never trust client-side tier claims. Read `subscriptions.tier` from the database. If the tier doesn't permit this feature, return 403 with an upgrade prompt error code.

16. **Use PowerSync for all offline sync.** Never write manual sync logic. If a table should be available offline, add it to the PowerSync schema.

17. **Use y_crdt for all collaborative block editing.** Never use direct API calls (PATCH `/blocks/:id`) for content changes on collaborative scripts. All content edits go through the Yjs Y.Doc.

18. **Run `cargo audit` before every Rust PR merge.** Zero high-severity vulnerabilities. Block the PR if any are found.

19. **Run `npm audit` before every TypeScript PR merge.** Zero high-severity vulnerabilities. Block the PR if any are found.

20. **Write an OpenTelemetry span for every Rust handler and every TypeScript BullMQ job.** Use `tracing::instrument` in Rust. Use the OTel span API in TypeScript. Every operation that touches the database or an external service should be a span.

21. **Update STATUS.md after completing every story.** Change status from DRAFT → DONE. Add a one-line Decision Log entry for any architecture decision made.

22. **Check that every new DB table has RLS enabled and a deny-all default policy.** Before writing any migration, verify that `ALTER TABLE X ENABLE ROW LEVEL SECURITY;` is present and that a deny-all default exists.

23. **Use Upstash Redis (fred.rs in Rust, ioredis in TypeScript) for all caching.** Never use in-memory cache in Cloud Run containers (stateless, can be replaced). All shared state goes through Redis.

24. **All secrets come from environment variables injected by Secret Manager.** Never hardcode. Never `dotenv()` in production. Never `std::env::var("MY_KEY").unwrap_or("default_secret")`.

25. **Follow the folder structure defined in Section 4 exactly.** Every file goes in its designated location. No new top-level folders without a written decision in STATUS.md.

26. **Apply all 6 required security headers in the Rust security headers middleware.** `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy: strict-origin-when-cross-origin`, `Permissions-Policy: camera=(), microphone=(), geolocation=()`, plus CORS headers.

27. **Verify Stripe-Signature on every Stripe webhook before processing.** Any webhook event that arrives without a valid signature returns 401 immediately. Never process an unverified webhook.

28. **Every list view has a cursor-based pagination implementation.** No offset-based pagination. No `OFFSET X LIMIT Y` in production SQL queries on large tables.

29. **Every error response from the Rust API follows the AppError format.** `{status_code, error_code: string, message: string, details?: serde_json::Value}`. Consistent. Parseable by the Flutter client.

30. **All file uploads validate MIME type server-side.** Check the actual file bytes, not just the extension or the Content-Type header. Use a MIME detection library in Rust.

---

## SECTION 3 — NEVER Rules

**30 absolute prohibitions:**

1. **Never expose `service_role` key in Flutter client.** If you find yourself writing `const supabaseServiceKey = "..."` anywhere near Flutter code, stop immediately.

2. **Never write directly to Supabase from the TypeScript MCP service.** No Supabase client import in the MCP service. All writes go via `rustClient.post('/ai/...')` (internal API key).

3. **Never skip JWT verification on any authenticated handler.** No `// TODO: add auth later`. No `if cfg!(debug_assertions) { skip_auth(); }` in code that could reach production.

4. **Never use raw string SQL in Rust.** No `sqlx::query("SELECT * FROM blocks WHERE id = '" + id + "'")`. The `query!` macro exists for exactly this reason.

5. **Never use `setState()` or `ChangeNotifier` in Flutter.** Riverpod exists. Use it. Every state change is through an `AsyncNotifier` or `Notifier`.

6. **Never use `dynamic` or `any` type in TypeScript.** Type everything. If the shape isn't known, use `unknown` and narrow it. `any` is a type system escape hatch that hides bugs.

7. **Never use `dynamic` type in Dart.** Same principle. Type everything. If you need flexibility, use generics.

8. **Never add a feature not in the current story's acceptance criteria.** No "it's just a small improvement." No "the user will obviously want this too." Implement exactly what the story says. Open a new story for anything extra.

9. **Never truncate story files, architecture docs, or the constitution.** These documents are complete and correct. Do not edit them to "simplify" them. If you disagree with something, add a Decision Log entry in STATUS.md.

10. **Never commit secrets, API keys, or credentials to git.** No `.env` files. No `credentials.json`. No hardcoded API keys in any file committed to the repository.

11. **Never use `localStorage` or `sessionStorage` in Flutter Web.** Flutter Web's sandboxed iframes block these APIs in many configurations. Use Drift (SQLite via sql.js) or in-memory state backed by Riverpod.

12. **Never generate AI content without explicit user trigger.** The sole exception is the 5-pillar extraction debounce (which the user can globally disable). Every other AI feature requires a deliberate user action.

13. **Never commit AI-generated content directly to script blocks without the approval queue.** Every agent write goes through `agent_approvals`. No exceptions for "small" or "obviously correct" changes.

14. **Never show AI-generated content without the AI badge.** If `content_source == 'ai_generated'`, the `AIBadge` widget must be visible. No hiding it to "look cleaner."

15. **Never skip the ContentSource enum on AI-generated items.** If an AI agent creates or modifies an item and you forget to set `content_source`, you are violating the Human Authorship Principle.

16. **Never allow AI to lock or permanently set anything.** There is no "lock" state for AI-generated content. The writer can change everything. Always. Forever.

17. **Never skip the offline behavior implementation in any story.** If the story doesn't explicitly mention offline behavior, apply the defaults from constitution.md §4. Offline is never a "future enhancement."

18. **Never skip empty states.** Every list view, every panel, every tab that can be empty needs an empty state widget with a contextual action (e.g., "Add your first character").

19. **Never use colored side borders on cards.** This is a visual anti-pattern in the LIPILY design system. Cards communicate state through content, chips, and badges — not colored borders.

20. **Never skip mobile (375px) testing on any Flutter Web screen.** Before marking a story DONE, you must have tested the layout at 375px width.

21. **Never skip subscription tier enforcement on Pro/Studio features.** Every handler that serves a gated feature must check `subscriptions.tier` in the database. A client-side check is only for UX — server-side is the real gate.

22. **Never use direct DB connection strings.** All Rust API DB connections go through Supavisor port 6543 only. Never the direct port. Never for Cloud Run deployments.

23. **Never skip rate limit checks on AI endpoints.** POST `/ai/*` must check Upstash Redis for daily quota before queueing any job. Return 429 with `{error_code: "ai_quota_exceeded", tier, limit, reset_at}` if exceeded.

24. **Never skip Stripe webhook signature verification.** POST `/subscription/webhook` must verify `Stripe-Signature` before reading any event data. No exceptions for "testing."

25. **Never mark a story as DONE without verifying all acceptance criteria pass.** Read each criterion. Test it. If even one fails, the story is not DONE.

26. **Never implement a custom CRDT or sync engine.** Yjs (yrs) is the CRDT. PowerSync is the sync. Neither is replaced, augmented, or wrapped with custom logic.

27. **Never write Fountain parser logic in Flutter.** The Fountain parser lives in Rust (`fountain_parser.rs`). Flutter sends content to the Rust API; it never parses Fountain itself.

28. **Never store user PII outside of Supabase.** No user emails, names, or identifying information in logs, Redis, or any third-party service beyond what is required for Sentry error tracking (anonymized).

29. **Never implement a feature that requires localStorage for state persistence on Flutter Web.** If a feature needs persistence, it goes in Drift (local) or Supabase (remote) via PowerSync.

30. **Never bypass the `agent_approvals` queue for "speed."** The approval queue is a product requirement, not an implementation detail. The Human Authorship Principle requires it. There is no shortcut.

---

## SECTION 4 — Project Folder Structure

### frontend/ (Flutter)

```
lib/
├── main.dart                          # App entry point, Riverpod ProviderScope
├── app.dart                           # MaterialApp.router with go_router
├── core/
│   ├── constants/
│   │   ├── api_constants.dart         # Base URLs, endpoint paths
│   │   ├── app_constants.dart         # App-wide constants (timeouts, limits)
│   │   └── route_constants.dart       # Named route constants
│   ├── errors/
│   │   ├── app_error.dart             # Sealed AppError class
│   │   ├── failure.dart               # Failure model
│   │   └── result.dart                # Result<T, AppError> sealed class
│   ├── extensions/
│   │   ├── string_ext.dart            # String utility extensions
│   │   ├── datetime_ext.dart          # DateTime formatting extensions
│   │   └── context_ext.dart           # BuildContext extensions (theme, l10n, etc.)
│   ├── theme/
│   │   ├── app_theme.dart             # ThemeData light + dark
│   │   ├── app_colors.dart            # Color constants
│   │   ├── app_typography.dart        # TextStyle constants
│   │   └── app_spacing.dart           # Spacing constants (8px grid)
│   ├── utils/
│   │   ├── debouncer.dart             # Debounce utility (used for AI extraction)
│   │   ├── fountain_formatter.dart    # Fountain text display formatting (NOT parsing)
│   │   └── content_source.dart        # ContentSource enum + helpers
│   └── widgets/
│       ├── loading_skeleton.dart      # Shimmer loading skeleton
│       ├── empty_state.dart           # Empty state with action button
│       ├── error_state.dart           # Error state with retry button
│       ├── ai_badge.dart              # AI content badge widget
│       └── subscription_gate.dart    # Gate widget (shows upgrade prompt)
├── features/
│   ├── auth/
│   │   ├── data/
│   │   │   ├── auth_repository.dart
│   │   │   └── auth_remote_data_source.dart
│   │   ├── domain/
│   │   │   ├── auth_state.dart
│   │   │   └── profile_model.dart
│   │   └── presentation/
│   │       ├── auth_notifier.dart     # @riverpod AsyncNotifier
│   │       ├── login_screen.dart
│   │       ├── auth_callback_screen.dart
│   │       └── username_select_screen.dart
│   ├── dashboard/
│   │   ├── data/
│   │   │   └── script_repository.dart
│   │   ├── domain/
│   │   │   └── script_model.dart
│   │   └── presentation/
│   │       ├── script_list_notifier.dart
│   │       ├── dashboard_screen.dart
│   │       └── widgets/
│   │           ├── script_card.dart
│   │           └── create_script_dialog.dart
│   ├── editor/
│   │   ├── data/
│   │   │   ├── block_repository.dart
│   │   │   ├── scene_repository.dart
│   │   │   └── editor_local_data_source.dart   # Drift DAOs
│   │   ├── domain/
│   │   │   ├── block_model.dart
│   │   │   ├── scene_model.dart
│   │   │   └── smart_flow_logic.dart
│   │   └── presentation/
│   │       ├── block_list_notifier.dart
│   │       ├── scene_list_notifier.dart
│   │       ├── editor_screen.dart
│   │       └── widgets/
│   │           ├── block_item.dart
│   │           ├── block_editor_view.dart
│   │           ├── scene_navigator_panel.dart
│   │           ├── scene_intelligence_sidebar.dart
│   │           ├── pillar_widget.dart
│   │           ├── alternate_dialogue_row.dart
│   │           ├── collaborator_cursor_layer.dart
│   │           ├── smart_type_overlay.dart
│   │           └── focus_mode_overlay.dart
│   ├── story_bible/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   │       ├── story_bible_notifier.dart
│   │       └── story_bible_screen.dart
│   ├── beat_board/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   │       ├── beat_board_notifier.dart
│   │       └── beat_board_screen.dart
│   ├── storyboard/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   │       ├── storyboard_notifier.dart
│   │       └── storyboard_screen.dart
│   ├── collaboration/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   │       ├── collaborators_notifier.dart
│   │       ├── yjs_sync_notifier.dart
│   │       └── collaboration_screen.dart
│   ├── versioning/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   │       ├── draft_list_notifier.dart
│   │       └── draft_history_screen.dart
│   ├── branching/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   │       ├── branch_notifier.dart
│   │       ├── branch_editor_screen.dart
│   │       └── merge_request_screen.dart
│   ├── ai/
│   │   ├── data/
│   │   │   └── ai_repository.dart
│   │   ├── domain/
│   │   │   └── ai_suggestion_model.dart
│   │   └── presentation/
│   │       ├── ai_suggestion_notifier.dart
│   │       ├── approval_queue_notifier.dart
│   │       └── widgets/
│   │           ├── approval_queue_sheet.dart
│   │           ├── ai_suggestion_panel.dart
│   │           ├── writers_block_panel.dart
│   │           ├── continuity_flags_panel.dart
│   │           ├── pacing_dashboard.dart
│   │           └── pitch_generator_panel.dart
│   ├── export/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   │       └── export_menu.dart
│   ├── import/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   │       └── import_screen.dart
│   ├── social/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   │       ├── publication_notifier.dart
│   │       ├── publish_wizard_screen.dart
│   │       └── public_profile_screen.dart
│   ├── listen/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   │       ├── listen_presentation_notifier.dart
│   │       ├── listen_generate_screen.dart
│   │       └── listen_playback_screen.dart
│   ├── discovery/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   │       ├── discover_screen.dart
│   │       └── leaderboard_screen.dart
│   ├── review/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   │       ├── review_submit_screen.dart
│   │       └── reviewer_dashboard_screen.dart
│   ├── subscription/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   │       ├── subscription_notifier.dart
│   │       └── subscription_screen.dart
│   ├── notifications/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   │       ├── notifications_notifier.dart
│   │       └── notifications_panel.dart
│   ├── settings/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   │       ├── settings_screen.dart
│   │       ├── shortcuts_screen.dart
│   │       ├── formatting_screen.dart
│   │       ├── account_screen.dart
│   │       └── stats_screen.dart
│   └── admin/
│       ├── data/
│       ├── domain/
│       └── presentation/
│           └── admin_dashboard_screen.dart
├── l10n/
│   ├── app_en.arb
│   ├── app_hi.arb
│   ├── app_ta.arb
│   └── app_ar.arb
└── database/
    ├── app_database.dart              # Drift database definition
    ├── tables/
    │   ├── offline_write_queue.dart
    │   ├── cached_blocks.dart
    │   ├── cached_scripts.dart
    │   └── cached_scenes.dart
    └── daos/
        ├── offline_queue_dao.dart
        ├── block_dao.dart
        ├── script_dao.dart
        └── scene_dao.dart
```

### backend/ (Rust)

```
src/
├── main.rs                            # Server startup, Axum router assembly
├── lib.rs                             # Library root
├── config.rs                          # Config struct, from_env()
├── db/
│   └── pool.rs                        # Supavisor connection pool setup
├── auth/
│   ├── jwt.rs                         # verify_jwt(), JWKS fetching + caching
│   ├── claims.rs                      # JwtClaims struct
│   └── middleware.rs                  # Auth extraction middleware
├── cache/
│   ├── redis.rs                       # fred.rs client initialization
│   ├── rate_limit.rs                  # Rate limiting logic (check + increment)
│   └── presence.rs                   # Cursor presence read/write
├── routes/
│   ├── mod.rs                         # Route assembly
│   ├── auth.rs                        # /auth/* handlers
│   ├── scripts.rs                     # /scripts/* handlers
│   ├── scenes.rs                      # /scenes/* handlers
│   ├── blocks.rs                      # /blocks/* handlers
│   ├── collaboration.rs               # /collaboration/* handlers
│   ├── branches.rs                    # /branches/* handlers
│   ├── versioning.rs                  # /versioning/* handlers
│   ├── ai.rs                          # /ai/* handlers
│   ├── export.rs                      # /export/* handlers
│   ├── import.rs                      # /import/* handlers
│   ├── social.rs                      # /social/* handlers
│   ├── listen.rs                      # /listen/* handlers
│   ├── review.rs                      # /review/* handlers
│   ├── producer.rs                    # /producer/* handlers
│   ├── subscription.rs                # /subscription/* handlers
│   ├── notifications.rs               # /notifications/* handlers
│   ├── moderation.rs                  # /moderation/* handlers
│   ├── search.rs                      # /search/* handlers
│   ├── presence.rs                    # /presence/:scriptId WebSocket endpoint
│   └── admin.rs                       # /admin/* handlers
├── models/
│   ├── profile.rs
│   ├── script.rs
│   ├── scene.rs
│   ├── block.rs
│   ├── collaborator.rs
│   ├── branch.rs
│   ├── merge_request.rs
│   ├── draft.rs
│   ├── comment.rs
│   ├── scene_intelligence.rs
│   ├── agent_approval.rs
│   ├── subscription.rs
│   ├── notification.rs
│   └── enums.rs                       # All DB enum types
├── services/
│   ├── script_service.rs
│   ├── scene_service.rs
│   ├── block_service.rs
│   ├── ai_service.rs                  # Queue MCP jobs, approval queue management
│   ├── export_service.rs              # Puppeteer call orchestration
│   ├── import_service.rs              # Delegates to parsers
│   ├── subscription_service.rs        # Stripe API calls
│   ├── notification_service.rs        # Insert + broadcast notifications
│   ├── fountain_parser.rs             # Fountain ↔ LIPILY block format
│   ├── fdx_parser.rs                  # FDX ↔ LIPILY block format
│   ├── presence_service.rs            # Redis presence management
│   └── yjs_service.rs                 # Y.Doc load/save/broadcast
├── ws/
│   ├── handler.rs                     # WebSocket upgrade + JWT check
│   ├── yjs_handler.rs                 # Yjs sync loop
│   └── cursor_handler.rs              # Presence broadcast loop
├── errors/
│   ├── app_error.rs                   # AppError enum with IntoResponse
│   └── error_handler.rs               # Global error handler
├── middleware/
│   ├── auth_middleware.rs             # JWT verification middleware
│   ├── rate_limit_middleware.rs       # Global rate limiting
│   ├── cors_middleware.rs             # CORS whitelist
│   ├── security_headers_middleware.rs # All 6 required security headers
│   └── logging_middleware.rs          # Request/response logging with trace IDs
└── telemetry/
    └── tracing_setup.rs               # OpenTelemetry + tracing subscriber setup
```

### mcp-service/ (TypeScript)

```
src/
├── index.ts                           # MCP server entry point
├── config.ts                          # Env vars, validated with Zod
├── queue/
│   ├── worker.ts                      # BullMQ worker initialization
│   └── jobs/
│       ├── listen_generate.ts         # "Listen the Script" full pipeline
│       ├── continuity_check.ts        # Continuity checker job
│       ├── pacing_analysis.ts         # Pacing analyzer job
│       ├── pitch_generate.ts          # Pitch generator job
│       ├── breakdown_generate.ts      # Script breakdown job
│       ├── voice_assign.ts            # Voice assignment job (Step 2)
│       └── imagery_generate.ts        # DALL-E image generation job (Step 4)
├── agents/
│   ├── scene_intelligence_agent.ts   # 5-pillar extraction
│   ├── dialogue_suggest_agent.ts     # Dialogue suggestions
│   ├── continue_scene_agent.ts       # Writer's block breaker
│   ├── continuity_agent.ts           # Continuity checker
│   ├── pacing_agent.ts               # Pacing analyzer
│   ├── pitch_agent.ts                # Pitch generator
│   ├── voice_guardian_agent.ts       # Voice guardian
│   └── translation_agent.ts          # Multilingual handling
├── tools/
│   └── [mcp tool definitions for each agent]
├── api/
│   ├── rust_client.ts                 # Typed HTTP client for Rust API (internal key)
│   └── openrouter_client.ts           # OpenRouter via Anthropic SDK
├── schemas/
│   └── [Zod schemas for all job payloads and API inputs]
├── middleware/
│   └── auth.ts                        # Service-to-service key verification
├── utils/
│   ├── content_source.ts              # ContentSource enum + helpers
│   └── approval_queue.ts              # Write to agent_approvals via Rust API
└── telemetry/
    └── otel_setup.ts                  # OpenTelemetry SDK setup
```

---

## SECTION 5 — Dart Style Guide

### 1. AsyncNotifier with riverpod_generator (Complete Example)

```dart
// features/dashboard/presentation/script_list_notifier.dart

import 'package:riverpod_annotation/riverpod_annotation.dart';
import '../data/script_repository.dart';
import '../domain/script_model.dart';
import '../../../core/errors/result.dart';

part 'script_list_notifier.g.dart';

@riverpod
class ScriptListNotifier extends _$ScriptListNotifier {
  @override
  Future<List<Script>> build() async {
    return _fetchScripts();
  }

  Future<List<Script>> _fetchScripts() async {
    final repo = ref.read(scriptRepositoryProvider);
    final result = await repo.listScripts();
    return result.fold(
      (scripts) => scripts,
      (error) => throw error,
    );
  }

  Future<Result<Script, AppError>> createScript(CreateScriptRequest req) async {
    final repo = ref.read(scriptRepositoryProvider);
    final result = await repo.createScript(req);
    result.fold(
      (script) {
        state = AsyncValue.data([
          ...state.valueOrNull ?? [],
          script,
        ]);
      },
      (_) {}, // error handled by caller
    );
    return result;
  }

  Future<Result<void, AppError>> deleteScript(String id) async {
    final repo = ref.read(scriptRepositoryProvider);
    final result = await repo.deleteScript(id);
    result.fold(
      (_) {
        state = AsyncValue.data(
          (state.valueOrNull ?? []).where((s) => s.id != id).toList(),
        );
      },
      (_) {},
    );
    return result;
  }
}
```

### 2. Sealed Result Class (Complete Implementation)

```dart
// core/errors/result.dart

sealed class Result<T, E> {
  const Result();

  bool get isSuccess => this is Success<T, E>;
  bool get isFailure => this is Failure<T, E>;

  R fold<R>(
    R Function(T value) onSuccess,
    R Function(E error) onFailure,
  ) {
    return switch (this) {
      Success<T, E>(value: final v) => onSuccess(v),
      Failure<T, E>(error: final e) => onFailure(e),
    };
  }

  T? get valueOrNull => switch (this) {
    Success<T, E>(value: final v) => v,
    Failure<T, E>() => null,
  };

  E? get errorOrNull => switch (this) {
    Success<T, E>() => null,
    Failure<T, E>(error: final e) => e,
  };

  static Result<T, E> success<T, E>(T value) => Success(value);
  static Result<T, E> failure<T, E>(E error) => Failure(error);
}

final class Success<T, E> extends Result<T, E> {
  const Success(this.value);
  final T value;
}

final class Failure<T, E> extends Result<T, E> {
  const Failure(this.error);
  final E error;
}
```

### 3. AppError Sealed Class (Complete Implementation)

```dart
// core/errors/app_error.dart

sealed class AppError {
  const AppError();

  String get userMessage;
  String get code;
}

final class NetworkError extends AppError {
  const NetworkError({this.statusCode, this.message});
  final int? statusCode;
  final String? message;

  @override
  String get userMessage => 'Connection error. Please check your internet and try again.';

  @override
  String get code => 'network_error';
}

final class AuthError extends AppError {
  const AuthError({required this.message});
  final String message;

  @override
  String get userMessage => 'Authentication failed. Please log in again.';

  @override
  String get code => 'auth_error';
}

final class ValidationError extends AppError {
  const ValidationError({required this.field, required this.message});
  final String field;
  final String message;

  @override
  String get userMessage => message;

  @override
  String get code => 'validation_error';
}

final class ServerError extends AppError {
  const ServerError({required this.errorCode, required this.message});
  final String errorCode;
  final String message;

  @override
  String get userMessage => 'Something went wrong. Our team has been notified.';

  @override
  String get code => errorCode;
}

final class OfflineError extends AppError {
  const OfflineError();

  @override
  String get userMessage => 'You\'re offline. This action will sync when you reconnect.';

  @override
  String get code => 'offline_error';
}

final class PermissionError extends AppError {
  const PermissionError({required this.requiredTier});
  final String requiredTier;

  @override
  String get userMessage => 'This feature requires $requiredTier. Upgrade to unlock.';

  @override
  String get code => 'permission_error';
}

final class QuotaError extends AppError {
  const QuotaError({required this.tier, required this.resetAt});
  final String tier;
  final DateTime resetAt;

  @override
  String get userMessage => 'Daily AI limit reached. Resets at ${resetAt.toLocal()}.';

  @override
  String get code => 'ai_quota_exceeded';
}
```

### 4. Drift DAO Pattern (Complete Example)

```dart
// database/daos/block_dao.dart

import 'package:drift/drift.dart';
import '../app_database.dart';
import '../tables/cached_blocks.dart';

part 'block_dao.g.dart';

@DriftAccessor(tables: [CachedBlocks])
class BlockDao extends DatabaseAccessor<AppDatabase> with _$BlockDaoMixin {
  BlockDao(AppDatabase db) : super(db);

  Stream<List<CachedBlock>> watchBlocksForScene(String sceneId) {
    return (select(cachedBlocks)
          ..where((b) => b.sceneId.equals(sceneId))
          ..orderBy([(b) => OrderingTerm.asc(b.id)]))
        .watch();
  }

  Future<void> insertBlock(CachedBlocksCompanion block) async {
    await into(cachedBlocks).insert(block, mode: InsertMode.insertOrReplace);
  }

  Future<void> updateBlock(String id, {required String content, bool isDirty = true}) async {
    await (update(cachedBlocks)..where((b) => b.id.equals(id))).write(
      CachedBlocksCompanion(
        content: Value(content),
        isDirty: Value(isDirty),
        syncedAt: Value(DateTime.now().millisecondsSinceEpoch),
      ),
    );
  }

  Future<List<CachedBlock>> getDirtyBlocks() async {
    return (select(cachedBlocks)..where((b) => b.isDirty.equals(true))).get();
  }

  Future<void> markAsSynced(String id) async {
    await (update(cachedBlocks)..where((b) => b.id.equals(id))).write(
      const CachedBlocksCompanion(isDirty: Value(false)),
    );
  }
}
```

### 5. go_router Guard Pattern (Complete Example)

```dart
// core/router/guards.dart

import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../../features/auth/presentation/auth_notifier.dart';
import '../../features/subscription/presentation/subscription_notifier.dart';

String? authGuard(BuildContext context, GoRouterState state) {
  final container = ProviderScope.containerOf(context);
  final authState = container.read(authNotifierProvider);

  if (authState.isLoading) return null; // Still loading — let splash handle

  final user = authState.valueOrNull?.user;
  if (user == null) {
    return '/auth/login?redirect=${Uri.encodeComponent(state.uri.toString())}';
  }
  return null;
}

String? proGuard(BuildContext context, GoRouterState state) {
  final authRedirect = authGuard(context, state);
  if (authRedirect != null) return authRedirect;

  final container = ProviderScope.containerOf(context);
  final subscription = container.read(subscriptionNotifierProvider);
  final tier = subscription.valueOrNull?.tier;

  if (tier == null || tier == 'free') {
    // Return to the dashboard with an upgrade prompt query param
    return '/dashboard?upgrade=true&feature=${Uri.encodeComponent(state.matchedLocation)}';
  }
  return null;
}

String? studioGuard(BuildContext context, GoRouterState state) {
  final proRedirect = proGuard(context, state);
  if (proRedirect != null) return proRedirect;

  final container = ProviderScope.containerOf(context);
  final subscription = container.read(subscriptionNotifierProvider);
  final tier = subscription.valueOrNull?.tier;

  if (tier != 'studio') {
    return '/dashboard?upgrade=true&feature=studio&feature_name=Writers+Room';
  }
  return null;
}

String? adminGuard(BuildContext context, GoRouterState state) {
  final authRedirect = authGuard(context, state);
  if (authRedirect != null) return authRedirect;

  final container = ProviderScope.containerOf(context);
  final profile = container.read(profileProvider);
  final isAdmin = profile.valueOrNull?.isAdmin ?? false;

  if (!isAdmin) return '/dashboard';
  return null;
}
```

### 6. ContentSource Enum Usage in Provider (Complete Example)

```dart
// core/utils/content_source.dart

enum ContentSource {
  aiGenerated,
  humanConfirmed,
  humanEdited,
  humanWritten;

  String get dbValue => switch (this) {
    ContentSource.aiGenerated => 'ai_generated',
    ContentSource.humanConfirmed => 'human_confirmed',
    ContentSource.humanEdited => 'human_edited',
    ContentSource.humanWritten => 'human_written',
  };

  static ContentSource fromDb(String value) => switch (value) {
    'ai_generated' => ContentSource.aiGenerated,
    'human_confirmed' => ContentSource.humanConfirmed,
    'human_edited' => ContentSource.humanEdited,
    'human_written' => ContentSource.humanWritten,
    _ => ContentSource.humanWritten,
  };

  bool get showsAiBadge => this == ContentSource.aiGenerated;
}

// In AI suggestion acceptance:
Future<void> acceptAiDialogueSuggestion(
  String blockId,
  String suggestionText,
  Ref ref,
) async {
  final repo = ref.read(blockRepositoryProvider);
  // Insert as a variant with aiGenerated source
  await repo.addVariant(blockId, BlockVariant(
    content: suggestionText,
    contentSource: ContentSource.aiGenerated, // Shows AI badge
  ));
}

// When user edits the accepted suggestion:
Future<void> onBlockContentChanged(
  String blockId,
  String newContent,
  ContentSource currentSource,
  Ref ref,
) async {
  final newSource = currentSource == ContentSource.aiGenerated
      ? ContentSource.humanEdited   // Removes AI badge
      : currentSource;              // Already human — keep as is
  final repo = ref.read(blockRepositoryProvider);
  await repo.updateBlock(blockId, content: newContent, contentSource: newSource);
}
```

### 7. Approval Queue Consumer Pattern (Complete Example)

```dart
// features/ai/presentation/approval_queue_notifier.dart

@riverpod
class ApprovalQueueNotifier extends _$ApprovalQueueNotifier {
  @override
  Future<List<AgentApproval>> build() async {
    // Poll every 5 seconds for pending approvals
    final timer = Timer.periodic(const Duration(seconds: 5), (_) {
      ref.invalidateSelf();
    });
    ref.onDispose(timer.cancel);

    final repo = ref.read(aiRepositoryProvider);
    final result = await repo.getPendingApprovals();
    return result.fold((approvals) => approvals, (_) => []);
  }

  Future<void> approve(String approvalId) async {
    final repo = ref.read(aiRepositoryProvider);
    final result = await repo.approveChange(approvalId);
    result.fold(
      (_) {
        // Remove from local list
        state = AsyncValue.data(
          (state.valueOrNull ?? []).where((a) => a.id != approvalId).toList(),
        );
      },
      (error) => throw error,
    );
  }

  Future<void> reject(String approvalId) async {
    final repo = ref.read(aiRepositoryProvider);
    final result = await repo.rejectChange(approvalId);
    result.fold(
      (_) {
        state = AsyncValue.data(
          (state.valueOrNull ?? []).where((a) => a.id != approvalId).toList(),
        );
      },
      (error) => throw error,
    );
  }
}

// In the ApprovalQueueSheet widget:
class ApprovalQueueSheet extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final approvals = ref.watch(approvalQueueNotifierProvider);

    return approvals.when(
      loading: () => const LoadingSkeleton(),
      error: (e, _) => ErrorState(error: e, onRetry: () => ref.invalidate(approvalQueueNotifierProvider)),
      data: (list) => list.isEmpty
          ? const EmptyState(message: 'No pending AI suggestions')
          : ListView.builder(
              itemCount: list.length,
              itemBuilder: (ctx, i) {
                final approval = list[i];
                return ApprovalCard(
                  approval: approval,
                  onApprove: () => ref.read(approvalQueueNotifierProvider.notifier).approve(approval.id),
                  onReject: () => ref.read(approvalQueueNotifierProvider.notifier).reject(approval.id),
                );
              },
            ),
    );
  }
}
```

---

## SECTION 6 — Rust Style Guide

### 1. Axum Handler with JWT Extraction (Complete Example)

```rust
// routes/scripts.rs

use axum::{
    extract::{Path, State},
    http::StatusCode,
    Extension, Json,
};
use serde::{Deserialize, Serialize};
use sqlx::PgPool;
use uuid::Uuid;
use tracing::instrument;

use crate::{
    auth::claims::JwtClaims,
    errors::AppError,
    models::script::Script,
};

#[derive(Deserialize)]
pub struct GetScriptPath {
    id: Uuid,
}

#[derive(Serialize)]
pub struct ScriptResponse {
    pub id: Uuid,
    pub title: String,
    pub format: String,
    pub word_count: i32,
    pub scene_count: i32,
    pub page_count_estimate: i32,
    pub language: String,
    pub rtl: bool,
    pub status: String,
    pub created_at: chrono::DateTime<chrono::Utc>,
    pub updated_at: chrono::DateTime<chrono::Utc>,
}

#[instrument(skip(pool, claims))]
pub async fn get_script(
    Extension(claims): Extension<JwtClaims>,
    Path(GetScriptPath { id }): Path<GetScriptPath>,
    State(pool): State<PgPool>,
) -> Result<Json<ScriptResponse>, AppError> {
    let user_id = claims.sub.parse::<Uuid>()
        .map_err(|_| AppError::Unauthorized("Invalid user ID in token".into()))?;

    let script = sqlx::query_as!(
        Script,
        r#"
        SELECT s.*
        FROM scripts s
        JOIN profiles p ON p.id = s.owner_id
        WHERE s.id = $1
          AND p.user_id = $2
          AND s.deleted_at IS NULL
        "#,
        id,
        user_id,
    )
    .fetch_optional(&pool)
    .await
    .map_err(AppError::Database)?
    .ok_or_else(|| AppError::NotFound("Script not found".into()))?;

    Ok(Json(ScriptResponse {
        id: script.id,
        title: script.title,
        format: script.format.to_string(),
        word_count: script.word_count,
        scene_count: script.scene_count,
        page_count_estimate: script.page_count_estimate,
        language: script.language,
        rtl: script.rtl,
        status: script.status.to_string(),
        created_at: script.created_at,
        updated_at: script.updated_at,
    }))
}
```

### 2. AppError Enum (Complete Implementation with IntoResponse)

```rust
// errors/app_error.rs

use axum::{
    http::StatusCode,
    response::{IntoResponse, Response},
    Json,
};
use serde_json::json;

#[derive(Debug, thiserror::Error)]
pub enum AppError {
    #[error("Unauthorized: {0}")]
    Unauthorized(String),

    #[error("Forbidden: {0}")]
    Forbidden(String),

    #[error("Not found: {0}")]
    NotFound(String),

    #[error("Validation error: {0}")]
    Validation(String),

    #[error("Conflict: {0}")]
    Conflict(String),

    #[error("Rate limited: {0}")]
    RateLimited(String),

    #[error("Subscription required: {0}")]
    SubscriptionRequired { tier: String, feature: String },

    #[error("AI quota exceeded")]
    AiQuotaExceeded { tier: String, reset_at: String },

    #[error("Database error")]
    Database(#[from] sqlx::Error),

    #[error("Redis error")]
    Redis(String),

    #[error("External service error: {0}")]
    ExternalService(String),

    #[error("Internal server error")]
    Internal(String),
}

impl IntoResponse for AppError {
    fn into_response(self) -> Response {
        let (status, error_code, message, details) = match &self {
            AppError::Unauthorized(msg) => (
                StatusCode::UNAUTHORIZED,
                "unauthorized",
                msg.clone(),
                None,
            ),
            AppError::Forbidden(msg) => (
                StatusCode::FORBIDDEN,
                "forbidden",
                msg.clone(),
                None,
            ),
            AppError::NotFound(msg) => (
                StatusCode::NOT_FOUND,
                "not_found",
                msg.clone(),
                None,
            ),
            AppError::Validation(msg) => (
                StatusCode::UNPROCESSABLE_ENTITY,
                "validation_error",
                msg.clone(),
                None,
            ),
            AppError::Conflict(msg) => (
                StatusCode::CONFLICT,
                "conflict",
                msg.clone(),
                None,
            ),
            AppError::RateLimited(msg) => (
                StatusCode::TOO_MANY_REQUESTS,
                "rate_limited",
                msg.clone(),
                None,
            ),
            AppError::SubscriptionRequired { tier, feature } => (
                StatusCode::PAYMENT_REQUIRED,
                "subscription_required",
                format!("This feature requires {} tier", tier),
                Some(json!({ "required_tier": tier, "feature": feature })),
            ),
            AppError::AiQuotaExceeded { tier, reset_at } => (
                StatusCode::TOO_MANY_REQUESTS,
                "ai_quota_exceeded",
                format!("Daily AI limit reached for {} tier", tier),
                Some(json!({ "tier": tier, "reset_at": reset_at })),
            ),
            AppError::Database(e) => {
                tracing::error!("Database error: {:?}", e);
                (
                    StatusCode::INTERNAL_SERVER_ERROR,
                    "database_error",
                    "A database error occurred".to_string(),
                    None,
                )
            }
            AppError::Redis(msg) => {
                tracing::error!("Redis error: {}", msg);
                (
                    StatusCode::INTERNAL_SERVER_ERROR,
                    "cache_error",
                    "A caching error occurred".to_string(),
                    None,
                )
            }
            AppError::ExternalService(msg) => (
                StatusCode::BAD_GATEWAY,
                "external_service_error",
                msg.clone(),
                None,
            ),
            AppError::Internal(msg) => {
                tracing::error!("Internal error: {}", msg);
                (
                    StatusCode::INTERNAL_SERVER_ERROR,
                    "internal_error",
                    "An internal error occurred".to_string(),
                    None,
                )
            }
        };

        let mut body = json!({
            "error_code": error_code,
            "message": message,
            "status_code": status.as_u16(),
        });

        if let Some(d) = details {
            body["details"] = d;
        }

        (status, Json(body)).into_response()
    }
}
```

### 3. JWT Verification Middleware (Complete Implementation)

```rust
// auth/jwt.rs

use axum::{
    extract::Request,
    http::header::AUTHORIZATION,
    middleware::Next,
    response::Response,
};
use jsonwebtoken::{decode, Algorithm, DecodingKey, Validation};
use once_cell::sync::OnceCell;
use serde::{Deserialize, Serialize};
use tokio::sync::RwLock;
use std::sync::Arc;
use chrono::{DateTime, Utc};

use crate::errors::AppError;

static JWKS_CACHE: OnceCell<Arc<RwLock<JwksCache>>> = OnceCell::new();

struct JwksCache {
    keys: Vec<(String, DecodingKey)>, // (kid, key)
    fetched_at: DateTime<Utc>,
}

#[derive(Debug, Serialize, Deserialize, Clone)]
pub struct JwtClaims {
    pub sub: String,
    pub email: Option<String>,
    pub role: Option<String>,
    pub aud: Option<Vec<String>>,
    pub exp: usize,
    pub iat: usize,
}

pub async fn verify_jwt(token: &str) -> Result<JwtClaims, AppError> {
    let jwks = get_jwks().await?;
    let header = jsonwebtoken::decode_header(token)
        .map_err(|_| AppError::Unauthorized("Invalid token header".into()))?;

    let kid = header.kid.as_deref().unwrap_or("default");
    let key = jwks
        .iter()
        .find(|(k, _)| k == kid)
        .map(|(_, key)| key)
        .ok_or_else(|| AppError::Unauthorized("Unknown key ID".into()))?;

    let mut validation = Validation::new(Algorithm::RS256);
    validation.set_audience(&["authenticated"]);

    let token_data = decode::<JwtClaims>(token, key, &validation)
        .map_err(|e| AppError::Unauthorized(format!("Token validation failed: {}", e)))?;

    Ok(token_data.claims)
}

async fn get_jwks() -> Result<Vec<(String, DecodingKey)>, AppError> {
    let cache = JWKS_CACHE.get_or_init(|| {
        Arc::new(RwLock::new(JwksCache {
            keys: vec![],
            fetched_at: DateTime::<Utc>::MIN_UTC,
        }))
    });

    {
        let read = cache.read().await;
        if Utc::now() - read.fetched_at < chrono::Duration::hours(1) && !read.keys.is_empty() {
            return Ok(read.keys.clone());
        }
    }

    let supabase_url = std::env::var("SUPABASE_URL").expect("SUPABASE_URL not set");
    let jwks_url = format!("{}/auth/v1/.well-known/jwks.json", supabase_url);

    let response = reqwest::get(&jwks_url)
        .await
        .map_err(|e| AppError::ExternalService(format!("JWKS fetch failed: {}", e)))?;

    let jwks: serde_json::Value = response
        .json()
        .await
        .map_err(|e| AppError::ExternalService(format!("JWKS parse failed: {}", e)))?;

    let keys = extract_keys_from_jwks(&jwks)?;

    let mut write = cache.write().await;
    write.keys = keys.clone();
    write.fetched_at = Utc::now();

    Ok(keys)
}

fn extract_keys_from_jwks(
    jwks: &serde_json::Value,
) -> Result<Vec<(String, DecodingKey)>, AppError> {
    let keys = jwks["keys"]
        .as_array()
        .ok_or_else(|| AppError::ExternalService("Invalid JWKS format".into()))?;

    keys.iter()
        .map(|k| {
            let kid = k["kid"].as_str().unwrap_or("default").to_string();
            let n = k["n"].as_str().ok_or_else(|| AppError::ExternalService("Missing n".into()))?;
            let e = k["e"].as_str().ok_or_else(|| AppError::ExternalService("Missing e".into()))?;
            let key = DecodingKey::from_rsa_components(n, e)
                .map_err(|_| AppError::ExternalService("Invalid RSA key".into()))?;
            Ok((kid, key))
        })
        .collect()
}

pub async fn auth_middleware(
    mut req: Request,
    next: Next,
) -> Result<Response, AppError> {
    let auth_header = req
        .headers()
        .get(AUTHORIZATION)
        .and_then(|v| v.to_str().ok())
        .and_then(|v| v.strip_prefix("Bearer "))
        .ok_or_else(|| AppError::Unauthorized("Missing Bearer token".into()))?;

    let claims = verify_jwt(auth_header).await?;
    req.extensions_mut().insert(claims);
    Ok(next.run(req).await)
}
```

### 4. Rate Limit Middleware (Complete Implementation)

```rust
// cache/rate_limit.rs

use fred::prelude::*;
use crate::errors::AppError;

pub struct RateLimiter {
    client: RedisClient,
}

impl RateLimiter {
    pub fn new(client: RedisClient) -> Self {
        Self { client }
    }

    pub async fn check_and_increment(
        &self,
        key: &str,
        limit: u64,
        window_secs: u64,
    ) -> Result<(), AppError> {
        let count: Option<u64> = self.client.get(key).await
            .map_err(|e| AppError::Redis(e.to_string()))?;

        let current = count.unwrap_or(0);
        if current >= limit {
            return Err(AppError::RateLimited(
                format!("Rate limit exceeded. Limit: {} per {} seconds", limit, window_secs)
            ));
        }

        // Increment + set TTL atomically via pipeline
        let pipeline = self.client.pipeline();
        pipeline.incr::<(), _>(key).await
            .map_err(|e| AppError::Redis(e.to_string()))?;
        pipeline.expire::<(), _>(key, window_secs as i64).await
            .map_err(|e| AppError::Redis(e.to_string()))?;
        pipeline.last::<()>().await
            .map_err(|e| AppError::Redis(e.to_string()))?;

        Ok(())
    }

    pub async fn check_ai_quota(
        &self,
        user_id: &str,
        tier: &str,
    ) -> Result<(), AppError> {
        let limit = match tier {
            "free" => 50u64,
            "pro" => 500u64,
            "studio" => return Ok(()), // unlimited
            _ => 50u64,
        };

        let key = format!("ai_quota:{}:{}", user_id, chrono::Utc::now().date_naive());
        self.check_and_increment(&key, limit, 86400).await
            .map_err(|_| AppError::AiQuotaExceeded {
                tier: tier.to_string(),
                reset_at: (chrono::Utc::now() + chrono::Duration::days(1))
                    .format("%Y-%m-%dT%H:%M:%SZ")
                    .to_string(),
            })
    }
}
```

### 5. sqlx Query Pattern (Complete Example)

```rust
// services/block_service.rs

use sqlx::PgPool;
use uuid::Uuid;
use chrono::{DateTime, Utc};

use crate::{
    errors::AppError,
    models::block::{Block, BlockType, ContentSource},
};

pub struct BlockService {
    pool: PgPool,
}

impl BlockService {
    pub fn new(pool: PgPool) -> Self {
        Self { pool }
    }

    pub async fn get_blocks_for_scene(
        &self,
        scene_id: Uuid,
        user_id: Uuid,
    ) -> Result<Vec<Block>, AppError> {
        sqlx::query_as!(
            Block,
            r#"
            SELECT
                b.id,
                b.scene_id,
                b.script_id,
                b.type AS "block_type: BlockType",
                b.content,
                b.position,
                b.metadata,
                b.content_source AS "content_source: ContentSource",
                b.deleted_at,
                b.created_at,
                b.updated_at
            FROM blocks b
            JOIN scenes s ON s.id = b.scene_id
            JOIN scripts sc ON sc.id = s.script_id
            JOIN profiles p ON p.id = sc.owner_id
            WHERE b.scene_id = $1
              AND p.user_id = $2
              AND b.deleted_at IS NULL
            ORDER BY b.position ASC
            "#,
            scene_id,
            user_id,
        )
        .fetch_all(&self.pool)
        .await
        .map_err(AppError::Database)
    }
}
```

### 6. WebSocket Handler (Complete Example)

```rust
// ws/handler.rs

use axum::{
    extract::{Path, State, WebSocketUpgrade},
    response::Response,
    Extension,
};
use axum::extract::ws::{Message, WebSocket};
use futures::{sink::SinkExt, stream::StreamExt};
use uuid::Uuid;

use crate::{
    auth::jwt::verify_jwt,
    errors::AppError,
    services::yjs_service::YjsService,
};

pub async fn ws_presence_handler(
    Path(script_id): Path<Uuid>,
    ws: WebSocketUpgrade,
    State(yjs): State<YjsService>,
    headers: axum::http::HeaderMap,
) -> Result<Response, AppError> {
    // Verify JWT from Authorization header on upgrade
    let token = headers
        .get("Authorization")
        .and_then(|v| v.to_str().ok())
        .and_then(|v| v.strip_prefix("Bearer "))
        .ok_or_else(|| AppError::Unauthorized("Missing token for WebSocket".into()))?;

    let claims = verify_jwt(token).await?;
    let user_id: Uuid = claims.sub.parse()
        .map_err(|_| AppError::Unauthorized("Invalid user ID".into()))?;

    // Verify user has access to this script
    yjs.verify_script_access(script_id, user_id).await?;

    Ok(ws.on_upgrade(move |socket| async move {
        handle_ws_session(socket, script_id, user_id, yjs).await;
    }))
}

async fn handle_ws_session(
    socket: WebSocket,
    script_id: Uuid,
    user_id: Uuid,
    yjs: YjsService,
) {
    let (mut sender, mut receiver) = socket.split();

    // Send initial Y.Doc state
    if let Ok(initial_state) = yjs.get_initial_state(script_id).await {
        let _ = sender.send(Message::Binary(initial_state)).await;
    }

    // Subscribe to broadcast channel for this script
    let mut rx = yjs.subscribe(script_id);

    loop {
        tokio::select! {
            // Receive update from this client
            Some(Ok(msg)) = receiver.next() => {
                match msg {
                    Message::Binary(data) => {
                        if let Err(e) = yjs.apply_and_broadcast(script_id, user_id, data).await {
                            tracing::error!("Yjs apply error: {:?}", e);
                        }
                    }
                    Message::Close(_) => break,
                    _ => {}
                }
            }
            // Receive broadcast from another client
            Ok(update) = rx.recv() => {
                if sender.send(Message::Binary(update)).await.is_err() {
                    break;
                }
            }
        }
    }

    yjs.remove_connection(script_id, user_id).await;
}
```

### 7. Security Headers Middleware (Complete Implementation)

```rust
// middleware/security_headers_middleware.rs

use axum::{
    extract::Request,
    http::HeaderValue,
    middleware::Next,
    response::Response,
};

pub async fn security_headers_middleware(req: Request, next: Next) -> Response {
    let mut response = next.run(req).await;
    let headers = response.headers_mut();

    headers.insert(
        "X-Content-Type-Options",
        HeaderValue::from_static("nosniff"),
    );
    headers.insert(
        "X-Frame-Options",
        HeaderValue::from_static("DENY"),
    );
    headers.insert(
        "Referrer-Policy",
        HeaderValue::from_static("strict-origin-when-cross-origin"),
    );
    headers.insert(
        "Permissions-Policy",
        HeaderValue::from_static("camera=(), microphone=(), geolocation=()"),
    );
    headers.insert(
        "X-XSS-Protection",
        HeaderValue::from_static("1; mode=block"),
    );
    headers.insert(
        "Strict-Transport-Security",
        HeaderValue::from_static("max-age=31536000; includeSubDomains"),
    );

    response
}
```

---

## SECTION 7 — TypeScript Style Guide

### 1. BullMQ Job Definition (Complete Example)

```typescript
// queue/jobs/listen_generate.ts

import { Job, Worker } from 'bullmq';
import { z } from 'zod';
import { redis } from '../../api/openrouter_client';
import { rustClient } from '../../api/rust_client';

const ListenGeneratePayloadSchema = z.object({
  scriptId: z.string().uuid(),
  userId: z.string().uuid(),
  jobId: z.string().uuid(),
});

type ListenGeneratePayload = z.infer<typeof ListenGeneratePayloadSchema>;

export const LISTEN_GENERATE_QUEUE = 'listen_generate';

export async function processListenGenerate(
  job: Job<ListenGeneratePayload>
): Promise<void> {
  const payload = ListenGeneratePayloadSchema.parse(job.data);
  const { scriptId, userId } = payload;

  await job.updateProgress(10);
  await rustClient.patch(`/listen/${scriptId}/presentation`, {
    generation_status: 'fetching',
  }, userId);

  // Step 1: Fetch data
  const scriptData = await rustClient.get(`/listen/${scriptId}/data`, userId);

  await job.updateProgress(20);
  await rustClient.patch(`/listen/${scriptId}/presentation`, {
    generation_status: 'voice_assignment',
  }, userId);

  // Step 2: Voice assignment (see voice_assign.ts)
  // ... (delegates to voice_assign job)

  // Steps 3-7 continue in subsequent jobs...
}

export const listenGenerateWorker = new Worker(
  LISTEN_GENERATE_QUEUE,
  processListenGenerate,
  {
    connection: redis,
    concurrency: 5,
    defaultJobOptions: {
      attempts: 3,
      backoff: {
        type: 'exponential',
        delay: 2000,
      },
      removeOnComplete: 100,
      removeOnFail: 50,
    },
  }
);
```

### 2. Zod Schema Pattern (Complete Example)

```typescript
// schemas/script_schemas.ts

import { z } from 'zod';

export const CreateScriptSchema = z.object({
  title: z
    .string()
    .min(1, 'Title is required')
    .max(200, 'Title must be 200 characters or less')
    .trim(),
  format: z.enum([
    'feature_film',
    'tv_half_hour',
    'tv_one_hour',
    'limited_series',
    'short_film',
    'stage_play',
  ]),
  language: z
    .string()
    .min(2)
    .max(10)
    .regex(/^[a-z]{2,3}(-[A-Z]{2,4})?$/, 'Must be a valid BCP-47 language code')
    .default('en'),
  rtl: z.boolean().default(false),
});

export type CreateScriptInput = z.infer<typeof CreateScriptSchema>;

export const SceneIntelligenceJobSchema = z.object({
  sceneId: z.string().uuid(),
  scriptId: z.string().uuid(),
  userId: z.string().uuid(),
  blocks: z.array(z.object({
    id: z.string().uuid(),
    type: z.enum(['scene_heading', 'action', 'character', 'dialogue', 'parenthetical', 'transition', 'shot']),
    content: z.string().max(10000),
    position: z.number().positive(),
  })),
  language: z.string().max(10),
});
```

### 3. OpenRouter Call via Anthropic SDK (Complete Example)

```typescript
// api/openrouter_client.ts

import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({
  apiKey: process.env.OPENROUTER_API_KEY!,
  baseURL: 'https://openrouter.ai/api',
  defaultHeaders: {
    'HTTP-Referer': 'https://lipily.com',
    'X-Title': 'LIPILY',
  },
});

const MODELS = {
  complex: 'anthropic/claude-sonnet-4-5',
  fast: 'anthropic/claude-haiku-3-5',
  fallback1: 'openai/gpt-4o',
  fallback2: 'google/gemini-1.5-pro',
} as const;

type ModelTier = 'complex' | 'fast';

interface CallOptions {
  systemPrompt: string;
  userMessage: string;
  tier?: ModelTier;
  maxTokens?: number;
}

export async function callAI(options: CallOptions): Promise<string> {
  const { systemPrompt, userMessage, tier = 'fast', maxTokens = 2048 } = options;
  const primaryModel = tier === 'complex' ? MODELS.complex : MODELS.fast;

  try {
    const response = await client.messages.create({
      model: primaryModel,
      max_tokens: maxTokens,
      system: systemPrompt,
      messages: [{ role: 'user', content: userMessage }],
    });

    const content = response.content[0];
    if (content.type !== 'text') {
      throw new Error('Unexpected response type from AI');
    }
    return content.text;
  } catch (error: unknown) {
    // OpenRouter handles fallback routing automatically based on configuration
    // but we log the model used for observability
    const errorMessage = error instanceof Error ? error.message : String(error);
    throw new Error(`AI call failed with model ${primaryModel}: ${errorMessage}`);
  }
}
```

### 4. Rust API Client Call (Complete Example)

```typescript
// api/rust_client.ts

import { z } from 'zod';

const RUST_API_URL = process.env.RUST_API_URL!;
const INTERNAL_API_KEY = process.env.INTERNAL_SERVICE_KEY!;

class RustApiClient {
  private headers: Record<string, string> = {
    'Content-Type': 'application/json',
    'X-Internal-Service-Key': INTERNAL_API_KEY,
  };

  async get<T>(path: string, userId: string): Promise<T> {
    const response = await fetch(`${RUST_API_URL}${path}`, {
      method: 'GET',
      headers: {
        ...this.headers,
        'X-User-Id': userId,
      },
    });

    if (!response.ok) {
      const error = await response.json() as { error_code: string; message: string };
      throw new Error(`Rust API GET ${path} failed: ${error.message} (${error.error_code})`);
    }

    return response.json() as Promise<T>;
  }

  async post<T>(path: string, body: unknown, userId: string): Promise<T> {
    const response = await fetch(`${RUST_API_URL}${path}`, {
      method: 'POST',
      headers: {
        ...this.headers,
        'X-User-Id': userId,
      },
      body: JSON.stringify(body),
    });

    if (!response.ok) {
      const error = await response.json() as { error_code: string; message: string };
      throw new Error(`Rust API POST ${path} failed: ${error.message} (${error.error_code})`);
    }

    return response.json() as Promise<T>;
  }

  async patch<T>(path: string, body: unknown, userId: string): Promise<T> {
    const response = await fetch(`${RUST_API_URL}${path}`, {
      method: 'PATCH',
      headers: {
        ...this.headers,
        'X-User-Id': userId,
      },
      body: JSON.stringify(body),
    });

    if (!response.ok) {
      const error = await response.json() as { error_code: string; message: string };
      throw new Error(`Rust API PATCH ${path} failed: ${error.message} (${error.error_code})`);
    }

    return response.json() as Promise<T>;
  }
}

export const rustClient = new RustApiClient();
```

### 5. Approval Queue Write Pattern (Complete Example)

```typescript
// utils/approval_queue.ts

import { z } from 'zod';
import { rustClient } from '../api/rust_client';

const CreateApprovalSchema = z.object({
  userId: z.string().uuid(),
  scriptId: z.string().uuid(),
  agentType: z.string().max(100),
  proposedChange: z.record(z.unknown()),
});

type CreateApprovalInput = z.infer<typeof CreateApprovalSchema>;

interface AgentApproval {
  id: string;
  userId: string;
  scriptId: string;
  agentType: string;
  proposedChange: Record<string, unknown>;
  status: 'pending' | 'approved' | 'rejected';
  createdAt: string;
}

export async function createApproval(
  input: CreateApprovalInput
): Promise<AgentApproval> {
  const validated = CreateApprovalSchema.parse(input);

  // NEVER write to Supabase directly — always through Rust API
  const approval = await rustClient.post<AgentApproval>(
    '/ai/approvals',
    {
      agent_type: validated.agentType,
      script_id: validated.scriptId,
      proposed_change: validated.proposedChange,
    },
    validated.userId
  );

  return approval;
}

// Example: creating an approval for a dialogue suggestion
export async function proposeDialogueSuggestions(
  userId: string,
  scriptId: string,
  blockId: string,
  suggestions: string[]
): Promise<AgentApproval> {
  return createApproval({
    userId,
    scriptId,
    agentType: 'dialogue_suggest',
    proposedChange: {
      block_id: blockId,
      suggestions: suggestions.map((text) => ({
        content: text,
        content_source: 'ai_generated',  // Always set for AI-generated content
      })),
    },
  });
}
```

---

## SECTION 8 — ContentSource Pattern

### ContentSource Enum Definition

**Dart:**
```dart
enum ContentSource {
  aiGenerated,
  humanConfirmed,
  humanEdited,
  humanWritten;

  String get dbValue => switch (this) {
    ContentSource.aiGenerated => 'ai_generated',
    ContentSource.humanConfirmed => 'human_confirmed',
    ContentSource.humanEdited => 'human_edited',
    ContentSource.humanWritten => 'human_written',
  };

  static ContentSource fromDb(String value) => switch (value) {
    'ai_generated' => ContentSource.aiGenerated,
    'human_confirmed' => ContentSource.humanConfirmed,
    'human_edited' => ContentSource.humanEdited,
    'human_written' => ContentSource.humanWritten,
    _ => ContentSource.humanWritten,
  };

  bool get showsAiBadge => this == ContentSource.aiGenerated;

  // humanConfirmed, humanEdited, humanWritten are visually identical — no badge
  bool get isHumanOwned => this != ContentSource.aiGenerated;
}
```

**TypeScript:**
```typescript
export const ContentSource = {
  AI_GENERATED: 'ai_generated',
  HUMAN_CONFIRMED: 'human_confirmed',
  HUMAN_EDITED: 'human_edited',
  HUMAN_WRITTEN: 'human_written',
} as const;

export type ContentSource = typeof ContentSource[keyof typeof ContentSource];

export function isAiGenerated(source: ContentSource): boolean {
  return source === ContentSource.AI_GENERATED;
}
```

### Rules

- Only `aiGenerated` / `ai_generated` shows the AI badge in the UI
- `humanConfirmed`, `humanEdited`, and `humanWritten` are treated identically in UI — no badge
- The `content_source` field travels from DB → Rust API response → Riverpod provider → UI widget
- When an agent creates any item, it MUST set `content_source: 'ai_generated'`
- When a human edits any AI-generated item, the provider MUST update to `'human_edited'`
- When a human confirms an AI item, the provider updates to `'human_confirmed'`
- "Regenerate with instructions" resets source back to `'ai_generated'` and re-shows the badge

### Dart: AIBadge Widget

```dart
class AIBadge extends StatelessWidget {
  const AIBadge({
    super.key,
    required this.contentSource,
    required this.onEdit,
    required this.onConfirm,
    required this.onRegenerate,
    this.onDismiss,
  });

  final ContentSource contentSource;
  final VoidCallback onEdit;
  final VoidCallback onConfirm;
  final VoidCallback? onDismiss;
  final void Function(String? instructions) onRegenerate;

  @override
  Widget build(BuildContext context) {
    if (!contentSource.showsAiBadge) return const SizedBox.shrink();

    return Row(
      mainAxisSize: MainAxisSize.min,
      children: [
        Container(
          padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 2),
          decoration: BoxDecoration(
            color: Theme.of(context).colorScheme.primaryContainer,
            borderRadius: BorderRadius.circular(4),
          ),
          child: Text(
            'AI',
            style: Theme.of(context).textTheme.labelSmall?.copyWith(
              color: Theme.of(context).colorScheme.onPrimaryContainer,
            ),
          ),
        ),
        IconButton(
          iconSize: 16,
          icon: const Icon(Icons.edit_outlined),
          tooltip: 'Edit',
          onPressed: onEdit,
        ),
        IconButton(
          iconSize: 16,
          icon: const Icon(Icons.check_outlined),
          tooltip: 'Confirm (removes AI badge)',
          onPressed: onConfirm,
        ),
        IconButton(
          iconSize: 16,
          icon: const Icon(Icons.refresh_outlined),
          tooltip: 'Regenerate',
          onPressed: () => _showRegenerateDialog(context),
        ),
        if (onDismiss != null)
          IconButton(
            iconSize: 16,
            icon: const Icon(Icons.close_outlined),
            tooltip: 'Dismiss',
            onPressed: onDismiss,
          ),
      ],
    );
  }

  void _showRegenerateDialog(BuildContext context) {
    String? instructions;
    showDialog(
      context: context,
      builder: (ctx) => AlertDialog(
        title: const Text('Regenerate'),
        content: TextField(
          decoration: const InputDecoration(
            hintText: 'Optional: add instructions for regeneration',
          ),
          onChanged: (v) => instructions = v.isEmpty ? null : v,
        ),
        actions: [
          TextButton(onPressed: () => Navigator.pop(ctx), child: const Text('Cancel')),
          FilledButton(
            onPressed: () {
              Navigator.pop(ctx);
              onRegenerate(instructions);
            },
            child: const Text('Regenerate'),
          ),
        ],
      ),
    );
  }
}
```

---

## SECTION 9 — Error Handling Patterns

### Error Flow: End-to-End

```
Rust handler fails
  → AppError enum (e.g., AppError::NotFound("Script not found"))
  → IntoResponse converts to HTTP 404 + JSON body
  → Dio client receives 404 response
  → Dart service maps DioException → AppError (e.g., ServerError(errorCode: "not_found"))
  → Repository method returns Result.failure(AppError)
  → Riverpod provider throws the AppError
  → AsyncValue.error state is set on the provider
  → UI widget's .when() calls the error branch
  → ErrorState widget is displayed with appropriate message
```

### Toast vs Inline Error Decision Matrix

| Error Type | Display Behavior |
|---|---|
| `AuthError` | Redirect to `/auth/login?reason=session_expired` |
| `ValidationError` | Inline next to the specific field that failed |
| `NetworkError` | Retry banner at bottom of screen ("No connection — tap to retry") |
| `PermissionError` (subscription gate) | `SubscriptionGate` widget with upgrade CTA |
| `ServerError` (500) | Inline `ErrorState` widget with support link |
| `AiQuotaError` | Inline in AI panel with quota reset time + upgrade CTA |
| `RateLimited` | Toast notification (brief) with retry timer |
| `OfflineError` | `OfflineIndicator` banner (subtle, auto-dismisses on reconnect) |

---

## SECTION 10 — Story Execution Protocol

**Follow these exact steps for every single story — no shortcuts:**

1. **Read the complete story file.** All acceptance criteria. All error states. Offline behavior. Mobile behavior. Security section. Dependencies section. Do not begin coding until you have read every line.

2. **Read constitution.md §0** if the story touches any AI feature. Re-read all 10 points of the Human Authorship Principle. Confirm your implementation plan complies.

3. **Read constitution.md §1** to confirm all dependencies you plan to import are in the locked tech stack. If any planned dependency is not listed, stop and write a decision log entry proposing the addition.

4. **Read architecture.md §2** for all DB tables this story touches. Understand the schema, constraints, and RLS policies before writing any handler.

5. **Read architecture.md §4** for all API endpoints this story uses. Understand the request/response shapes.

6. **Write the test file first.** Unit tests for all service methods. Widget tests for all screens. Integration test for the happy path. Run tests — they should fail. Now write the implementation.

7. **Implement DB schema changes first.** If the story requires new tables or columns, write the migration SQL. Verify RLS is enabled and deny-all default is present.

8. **Implement Rust API changes.** Handlers, services, models. Every handler verifies JWT. Every handler validates input with serde structs. Every handler checks subscription tier if gated.

9. **Implement TypeScript MCP changes** if the story involves AI features or background jobs. All writes through Rust API. All AI outputs through approval queue.

10. **Implement Flutter provider changes.** Use `AsyncNotifier`. Handle all three states: loading, error, data.

11. **Implement Flutter UI.** Handle loading skeleton, error state with retry, empty state with action. Handle offline state. Test at 375px width.

12. **Verify every acceptance criterion manually** before marking the story done. Read each criterion. Click/tap through it. Do not assume it works.

13. **Verify at 375px mobile viewport.** Open Chrome DevTools, set to 375px width. Every interactive element must be ≥ 44px. No content overflow. No clipped buttons.

14. **Verify offline behavior.** In Chrome DevTools, set Network to Offline. Test the feature. Confirm writes queue in Drift and the offline indicator shows.

15. **Verify dark mode.** Toggle dark mode in Flutter. No unreadable text. No invisible icons. No hardcoded light-mode-only colors.

16. **Update STATUS.md.** Change story status: DRAFT → DONE. Timestamp the completion.

17. **Write a Decision Log entry in STATUS.md** for any architecture decision you made during implementation (e.g., "story-007: Used WebSocket push instead of polling for scene_intelligence updates — reduces latency by ~800ms on average").
