# LIPILY — Test Strategy

> **Purpose:** Define the testing pyramid, coverage requirements, test patterns, and CI gates for all three services.  
> **Agent rule:** Every story's acceptance criteria must have at least one automated test. No story is done without tests passing.  
> **Last updated:** 2026-04-07

---

## Testing Pyramid

```
         ╔══════════════════╗
         ║   E2E Tests (5%) ║  ← Full user flows (Playwright)
         ╠══════════════════╣
         ║ Integration (25%)║  ← API + DB, WebSocket, MCP pipelines
         ╠══════════════════╣
         ║   Unit Tests     ║  ← Business logic, formatters, AI prompts
         ║     (70%)        ║
         ╚══════════════════╝
```

---

## Coverage Requirements (CI Gate)

| Service | Minimum line coverage |
|---|---|
| `api/` (Rust) | 70% |
| `mcp/` (TypeScript) | 70% |
| `app/` (Flutter) | 60% |

CI fails if coverage drops below these thresholds.

---

## Rust API Tests (`api/tests/`)

### Unit tests (inline `#[cfg(test)]`)

Test location: Inside each handler/service file.  
Framework: `tokio::test`, `sqlx::test`.

```rust
// Example: api/src/handlers/scripts.rs
#[cfg(test)]
mod tests {
    use super::*;
    use sqlx::PgPool;

    #[sqlx::test(migrations = "../supabase/migrations")]
    async fn test_create_script_returns_201(pool: PgPool) {
        // arrange
        // act
        // assert
    }

    #[sqlx::test(migrations = "../supabase/migrations")]
    async fn test_create_script_enforces_free_tier_limit(pool: PgPool) {
        // arrange: 3 scripts already exist for user
        // act: create a 4th
        // assert: 403 TIER_REQUIRED
    }
}
```

### Integration tests (`api/tests/integration/`)

Use a real test database spun up by `sqlx::test`. Each test gets an isolated transaction that is rolled back on completion.

```rust
// api/tests/integration/test_collaboration.rs
#[sqlx::test(migrations = "../supabase/migrations")]
async fn test_invite_accept_creates_collaborator(pool: PgPool) { ... }

#[sqlx::test(migrations = "../supabase/migrations")]
async fn test_owner_can_remove_collaborator(pool: PgPool) { ... }
```

### WebSocket tests

Use `tokio-tungstenite` to test WebSocket frame exchange:
- Join a session → receive `yjs_state` frame
- Send `yjs_update` → all other clients receive broadcast
- Script lock → all non-showrunner clients receive `script_locked` event

---

## TypeScript MCP Tests (`mcp/tests/`)

Framework: `vitest`  
Mock: `vi.mock('@anthropic-ai/sdk')` for all Claude calls in unit tests.

### Unit tests — AI prompts

```typescript
// mcp/tests/prompts/scene-intelligence.test.ts
import { buildSceneIntelligencePrompt } from '../src/prompts/scene-intelligence.prompt'

describe('buildSceneIntelligencePrompt', () => {
  it('includes all 5 pillars in the system message', () => {
    const prompt = buildSceneIntelligencePrompt({ sceneText: 'INT. OFFICE - DAY', language: 'en' })
    expect(prompt.system).toContain('Story Structure')
    expect(prompt.system).toContain('Character Psychology')
    expect(prompt.system).toContain('Visual Grammar')
    expect(prompt.system).toContain('Emotional Arc')
    expect(prompt.system).toContain('Thematic Resonance')
  })

  it('instructs Claude to respond in Hindi when language is "hi"', () => {
    const prompt = buildSceneIntelligencePrompt({ sceneText: 'text', language: 'hi' })
    expect(prompt.system).toContain('respond IN HINDI')
  })
})
```

### Integration tests — BullMQ workers

```typescript
// mcp/tests/workers/listen-pipeline.test.ts
// Use a real Redis test instance (testcontainers or docker compose up redis in CI)
describe('ListenPipelineWorker', () => {
  it('transitions status from queued → processing → ready', async () => { ... })
  it('retries up to 3 times on pipeline failure', async () => { ... })
  it('calls internal Rust API with correct payload on completion', async () => { ... })
})
```

### Contract tests — Internal API calls

Verify that every `POST /internal/*` call from MCP to Rust sends the correct shape:

```typescript
it('notifies Rust with correct payload after listen pipeline completes', () => {
  const spy = vi.spyOn(internalApi, 'notify')
  // run pipeline
  expect(spy).toHaveBeenCalledWith({
    user_id: expect.any(String),
    type: 'listen_ready',
    entity_id: expect.any(String),
  })
})
```

---

## Flutter Tests (`app/test/`)

Framework: `flutter_test`, `mocktail`  
Test types: Widget tests (primary), Provider unit tests, Integration tests.

### Widget tests

```dart
// app/test/features/editor/block_widget_test.dart
testWidgets('Tab on Action block transitions to Character block', (tester) async {
  await tester.pumpWidget(const ProviderScope(child: BlockWidget(blockType: 'action')));
  await tester.sendKeyEvent(LogicalKeyboardKey.tab);
  expect(find.byKey(const Key('block-type-character')), findsOneWidget);
});

testWidgets('Editor renders 7 block types correctly', (tester) async { ... });

testWidgets('AI badge ✦ appears on ai_generated blocks', (tester) async { ... });
```

### Provider tests (Riverpod)

```dart
// app/test/shared/providers/subscription_provider_test.dart
test('subscriptionProvider returns Studio when plan is studio', () async {
  final container = ProviderContainer(overrides: [
    subscriptionRepositoryProvider.overrideWithValue(MockSubscriptionRepository())
  ]);
  // arrange: mock returns Studio subscription
  final subscription = await container.read(subscriptionProvider.future);
  expect(subscription.plan, equals('studio'));
});
```

### Integration tests (`app/integration_test/`)

Use `integration_test` package for full end-to-end Flutter tests against a local Supabase.

```dart
// app/integration_test/auth_flow_test.dart
testWidgets('Magic link auth → Dashboard shows empty state', (tester) async {
  await tester.pumpWidget(const LipilyApp());
  // Simulate magic link auth callback
  // Expect Dashboard to render
  expect(find.text('Your Scripts'), findsOneWidget);
  expect(find.text("You don't have any scripts yet"), findsOneWidget);
});
```

---

## E2E Tests (Playwright)

Location: `e2e/`  
Framework: Playwright  
Target: Flutter Web build served at `http://localhost:8080` (app)

### Critical user flows to cover

| Flow | Priority |
|---|---|
| Sign up → onboarding → create first script → write 3 blocks | P0 |
| Sign in → open existing script → Tab through block types → AI Dialogue Suggest | P0 |
| Export to PDF | P1 |
| Publish script → appears in social feed | P1 |
| Subscribe to Pro → feature unlocks immediately | P1 |
| WebSocket collab: User A edits → User B sees update in < 200ms | P1 |

---

## AI Feature Testing Strategy

AI features use real Claude API calls in integration tests (against OpenRouter).  
They must NOT use real API calls in unit tests — mock at the OpenRouter SDK boundary.

### ContentSource assertion pattern

Every test for an AI feature must assert the `content_source` value:

```dart
// Flutter widget test
test('AI Dialogue Suggest marks inserted content as ai_generated', () {
  // after inserting suggestion
  expect(block.contentSource, equals(ContentSource.aiGenerated));
});
```

```rust
// Rust integration test
assert_eq!(block.content_source, ContentSource::AiGenerated);
```

### AI daily limit enforcement test

```rust
#[sqlx::test]
async fn test_ai_limit_enforced_after_50_requests_for_free_tier(pool: PgPool) {
    // arrange: insert 50 agent_approvals for today
    // act: call POST /ai/dialogue-suggest
    // assert: 429 AI_LIMIT_EXCEEDED
}
```

---

## Offline / Drift Tests

```dart
// app/test/features/offline/offline_write_queue_test.dart
test('Write queue flushes in FIFO order on reconnect', () async {
  // arrange: 3 writes queued while offline
  // act: trigger reconnect
  // assert: writes flushed in order; Drift local state matches
});

test('App cold start < 1.5s when offline', () async {
  // Uses Flutter Driver to measure startup time
});
```

---

## Performance Tests

### API load tests (k6)

Location: `api/tests/load/`

```javascript
// api/tests/load/feed.k6.js
import http from 'k6/http';
export const options = { vus: 100, duration: '30s' };
export default function () {
  const res = http.get('https://api.lipily.com/feed?tab=trending');
  check(res, { 'status 200': (r) => r.status === 200 });
  check(res, { 'response < 1.5s': (r) => r.timings.duration < 1500 });
}
```

### WebSocket latency test

```javascript
// api/tests/load/collab.k6.js
// Measures end-to-end Yjs update latency across 20 concurrent connections
// Assert: 95th percentile < 200ms
```

---

## CI Gates (GitHub Actions)

All of these must pass before merging to `main`:

- [ ] `cargo fmt --check` (Rust formatting)
- [ ] `cargo clippy -- -D warnings` (Rust linting — zero warnings)
- [ ] `cargo test` with coverage ≥ 70%
- [ ] `npm run lint` (MCP TypeScript)
- [ ] `npm test` with coverage ≥ 70%
- [ ] `flutter analyze` (zero errors)
- [ ] `flutter test` with coverage ≥ 60%
- [ ] All E2E smoke tests pass against staging

---

## Test Data & Fixtures

Fixtures live in `api/tests/fixtures/` and `app/test/fixtures/`.

### Standard test users

| User | Role | Plan | Purpose |
|---|---|---|---|
| `test-free@lipily.com` | User | Free | Testing tier restrictions |
| `test-pro@lipily.com` | User | Pro | Testing Pro features |
| `test-studio@lipily.com` | User | Studio | Testing Studio features |
| `test-admin@lipily.com` | Admin | Studio | Testing admin actions |
| `test-reviewer@lipily.com` | Reviewer | Pro | Testing review marketplace |
| `test-producer@lipily.com` | Producer | Pro | Testing producer portal |

Fixtures are seeded by `supabase/seed.sql` on `supabase db reset`.

---

## Test Naming Conventions

```
test_{unit_under_test}_{scenario}_{expected_outcome}

Examples:
test_create_script_free_tier_fourth_script_returns_403
test_yjs_update_broadcast_reaches_all_connected_clients
test_ai_dialogue_suggest_marks_content_as_ai_generated
test_stripe_webhook_payment_failed_sets_past_due_status
```
