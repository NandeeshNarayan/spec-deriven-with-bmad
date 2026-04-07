# story-018 — Offline-First with PowerSync

| Field | Value |
|---|---|
| **Story ID** | story-018 |
| **Title** | Offline-First with PowerSync |
| **Epic** | E07 — Offline & Sync |
| **Phase** | 1 |
| **Sprint** | Sprint 1 |
| **Priority** | P0 |
| **Status** | DRAFT |
| **Persona** | Aisha (the independent short-film writer on a flight) |
| **Estimated Effort** | 4 days |
| **Dependencies** | story-001 |

---

## Problem Statement

Writers work on planes, in locations with no WiFi, and in remote residencies. A cloud-only editor is a productivity killer. LIPILY must work identically whether online or offline — all edits persist locally first, sync in the background when connected, and never require internet to write. This is a P0 requirement from the platform's core promise: "Write anywhere, sync everywhere."

---

## User Story

As a writer on a flight with no internet, I want LIPILY to work exactly like it does online — so that I never lose a writing session because of connectivity.

---

## Acceptance Criteria

**GIVEN** a user opens the LIPILY app with no internet connection,  
**WHEN** the app starts,  
**THEN** the authentication session is restored from FlutterSecureStorage; the dashboard loads scripts from the Drift `CachedScripts` table; the editor opens from the Drift `CachedBlocks` table; all previously synced data is immediately available with no network request.

**GIVEN** a user is writing in the editor and their internet connection drops,  
**WHEN** the connection drops,  
**THEN** the editor continues to function identically; keystrokes are captured; the Y.Doc is updated in memory; each block save is written to the Drift `OfflineWriteQueue` table; a subtle amber connectivity indicator appears in the editor status bar (not a modal, not a banner — just an icon).

**GIVEN** there are pending writes in the `OfflineWriteQueue`,  
**WHEN** internet connectivity is restored,  
**THEN** PowerSync automatically detects the reconnection; the `OfflineWriteQueue` is processed FIFO; each queued operation is sent to the Rust API in order; the Y.Doc state is pushed to the server; the connectivity indicator disappears.

**GIVEN** the user is offline and creates a new block,  
**WHEN** the block is created,  
**THEN** a temporary `block_id` (UUID generated on-device) is assigned; the block is saved to Drift `CachedBlocks` with `is_pending_sync = true`; when synced, the server-assigned UUID replaces the temporary one in Drift; all in-memory references update silently.

**GIVEN** a collaborator edited the same script online while the user was offline,  
**WHEN** the user comes back online and sync runs,  
**THEN** the PowerSync client pulls down all server-side changes; the Y.Doc merges the remote and local changes using CRDT (same as story-010); no data from either party is lost; the editor shows the merged state.

**GIVEN** the PowerSync schema is defined,  
**WHEN** the app initialises,  
**THEN** the following tables are synced via PowerSync with the defined sync rules:
- `scripts` — filter: `user_id = auth.uid() OR user_id IN (SELECT script_id FROM collaborators WHERE user_id = auth.uid())`
- `scenes` — filter: `script_id IN (user's scripts)`
- `blocks` — filter: `scene_id IN (user's scenes)`
- `story_bible_entries` — filter: `script_id IN (user's scripts)`
- `scene_intelligence` — filter: `script_id IN (user's scripts)`

**GIVEN** the user has been offline for a very long time (> 24 hours) and many changes have accumulated,  
**WHEN** they reconnect,  
**THEN** PowerSync performs a full sync (not incremental); a sync progress indicator is shown in the app: "Syncing [N] changes…"; the editor is not blocked during sync — the user can continue writing.

**GIVEN** the user opens the app after reinstalling it (empty Drift database),  
**WHEN** the app starts with internet connection,  
**THEN** PowerSync downloads all the user's scripts, scenes, blocks, and story bible entries in a full sync; a "Downloading your scripts…" splash screen is shown; the dashboard is accessible once the sync completes.

**GIVEN** the user's device has limited storage (< 100MB available),  
**WHEN** the sync tries to write to Drift,  
**THEN** a storage warning is shown: "Low device storage. Some scripts may not be available offline. Free up storage to ensure all scripts sync." Scripts are synced in `updated_at DESC` order (most recent first) so the user's active projects are always available.

**GIVEN** the Drift database schema changes between app versions (migration),  
**WHEN** the user upgrades the app,  
**THEN** the Drift migration runs automatically on first launch; the user's data is preserved; no data loss occurs; if migration fails, the app shows an error and does not start (preventing data corruption).

---

## PowerSync Schema (Dart)

```dart
// PowerSync sync rules — defined in backend/supabase/sync-rules.yaml
// Tables: scripts, scenes, blocks, story_bible_entries, scene_intelligence
// Each table uses userId-based filtering

// Drift local schema additions
class OfflineWriteQueue extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get operationType => text()(); // 'create' | 'update' | 'delete'
  TextColumn get tableName => text()();
  TextColumn get recordId => text()();
  TextColumn get payload => text()(); // JSON
  DateTimeColumn get createdAt => dateTime()();
  IntColumn get retryCount => integer().withDefault(const Constant(0))();
  BoolColumn get isPending => boolean().withDefault(const Constant(true))();
}

class CachedScripts extends Table {
  TextColumn get id => text().primaryKey()();
  TextColumn get title => text()();
  TextColumn get format => text()();
  TextColumn get ydocState => text().nullable()(); // Base64 encoded
  DateTimeColumn get updatedAt => dateTime()();
  BoolColumn get isSynced => boolean().withDefault(const Constant(true))();
}

class CachedBlocks extends Table {
  TextColumn get id => text().primaryKey()();
  TextColumn get sceneId => text()();
  TextColumn get blockType => text()();
  TextColumn get content => text()();
  IntColumn get position => integer()();
  TextColumn get contentSource => text()();
  BoolColumn get isPendingSync => boolean().withDefault(const Constant(false))();
}
```

---

## Performance Criteria

- App cold start (offline, data in Drift): dashboard visible in < 1.5 seconds
- Editor load from Drift cache (120-page script): < 2 seconds
- Offline write queue flush on reconnect: < 5 seconds for up to 1,000 queued operations
- PowerSync initial full sync (50 scripts, 10,000 blocks): < 30 seconds
- Sync progress UI updates: every 500ms during active sync

---

## Offline Behaviour

This entire story IS the offline behaviour definition. Key invariants:
1. The app must NEVER crash or show an error screen due to lack of network
2. All writes are local-first; remote sync is best-effort
3. The only features that explicitly require internet and show a "needs internet" state are: Authentication, Magic Link, Script Creation (new), Collaboration Invite, AI Features, Subscription

---

## Mobile Behaviour (375px)

- Connectivity indicator: small dot in the top-right status area of the editor (green = online, amber = syncing, grey = offline)
- "Downloading your scripts…" splash on fresh install: full-screen with progress bar and LIPILY wordmark
- Low storage warning: shows as a dismissible banner below the app bar on the dashboard

---

## Error States

**E1 — Sync Conflict (Data Loss Risk)**  
If two clients create a block with the same temporary UUID (extremely rare, UUID collision): PowerSync conflict resolution keeps the server-assigned ID and discards the duplicate; Sentry alert fired.

**E2 — Drift Migration Failure**  
Drift migration fails on app upgrade: app shows error screen "Database upgrade failed. Please contact support." with error code; no data is deleted; support can manually trigger migration via an admin tool.

**E3 — Offline Queue Stuck**  
An `OfflineWriteQueue` entry fails to sync 5 consecutive times (exponential backoff exhausted): it is moved to a "failed_operations" table; a subtle warning badge appears in Settings; the user can view failed operations and manually retry or discard.

---

## Security Considerations

- PowerSync uses the user's Supabase JWT for all sync operations — same RLS as the web client
- The Drift local database is stored in the app's private data directory (not accessible to other apps on Android/iOS)
- On Sign Out, all Drift tables are wiped (see story-002)
- The `OfflineWriteQueue` is also wiped on sign out to prevent data leakage

---

## Human Authorship Considerations

Not applicable — offline sync is purely an infrastructure concern.

---

## DB Tables Touched (Drift only, local)

- `OfflineWriteQueue`
- `CachedScripts`
- `CachedBlocks`
- PowerSync-managed copies of: `scripts`, `scenes`, `blocks`, `story_bible_entries`, `scene_intelligence`

---

## API Endpoints Used

- PowerSync SDK handles all sync operations automatically via WebSocket connection to Supabase realtime
- The `OfflineWriteQueue` consumer calls the same Rust API endpoints as the online path (e.g., `PATCH /blocks/:id`) when flushing

---

## Test Cases

**TC-001:** Put device in airplane mode → open editor → type 10 blocks → assert all 10 appear in Drift `CachedBlocks`.  
**TC-002:** Restore connectivity → assert offline queue flushes; all 10 blocks appear in Supabase.  
**TC-003:** Collaborator edits online while user is offline → reconnect → assert CRDT merge produces correct combined state.  
**TC-004:** App cold start with no network → assert dashboard loads from Drift in < 1.5s with no error screen.  
**TC-005:** Fresh install + connect → assert full sync runs and scripts appear in < 30s.  
**TC-006:** Simulate 1,000 queued operations → reconnect → assert all flush in < 5s.  
**TC-007:** Create block offline with temp UUID → sync → assert temp UUID replaced by server UUID in Drift.  
**TC-008:** Sign out → assert all Drift tables cleared.

---

## Out of Scope

- Selective offline (choosing which scripts to download) — all scripts sync by default
- Offline AI features (AI requires internet by design)
- Conflict resolution UI for humans (CRDT handles this automatically)
