# LIPILY — Product Requirements Document (PRD)

> Version 1.0 — April 7, 2026  
> Status: Active  
> Every agent reads this document before sprint planning.

---

## §1 — Problem Statement

### The Recognition Gap

Writers in Indian cinema — and globally — have no discovery platform. A screenwriter in Mumbai can spend two years writing a masterwork. It sits as a 120-page PDF on their hard drive. Producers won't cold-read unsolicited scripts. Agents don't represent unknown writers. There is no YouTube for screenwriters. No way to share your voice, build an audience, or be discovered based on the merit of your writing alone.

LIPILY changes this with "Listen the Script" — the world's first audio-visual presentation format for screenplays, auto-generated from the script itself. A producer in Mumbai can experience a 10-page scene in 7 minutes on their phone without reading a single line.

### The Final Draft Monopoly and Its 8 Structural Failures

Final Draft 13 costs $249 USD. That's ₹20,000+ in India. For a student or emerging writer, that's a dealbreaker. And for that price, what do you get?

1. **Pricing:** $249 one-time, but each major version requires another purchase. Not competitive in 2024.
2. **Desktop-only:** No web version. No real mobile support. A writer on their phone or tablet is locked out.
3. **Bugs:** Final Draft has longstanding bugs (rendering, crash on large scripts) that have never been fixed.
4. **Bloat:** Legacy feature set with UI designed in the early 2000s. Learning curve for new writers.
5. **Weak collaboration:** No real-time collaboration. Email chains and Dropbox for "collaborative" work.
6. **Import chaos:** Importing from Google Docs, Word, or other tools results in formatting disasters.
7. **No AI:** Zero AI integration. In 2026, this is a competitive extinction event.
8. **Steep learning curve:** Fountain format, industry conventions, and Final Draft's own quirks create weeks of onboarding.

### The Industry-Wide Tooling Gap

No existing tool solves all of these simultaneously:
- Unified story workspace (editor + story bible + beat board + storyboard)
- Intelligent character tracking without manual data entry
- Real Writers' Room support with scene locking and showrunner controls
- Transparent AI that writes suggestions, never mandates
- Git-style versioning for creative content
- Fair pricing accessible to emerging writers globally
- Dialogue read-aloud for self-editing
- Cross-platform consistency across web, mobile, and desktop

### The Thick PDF Problem

Scripts sit unread. A producer cannot evaluate 20 unsolicited 120-page PDFs per week. Even a logline and first 10 pages is often too much friction. "Listen the Script" solves this — it converts any scene into a 3-8 minute immersive audio-visual experience, playable on mobile without even creating an account. A producer can evaluate a writer's voice in 8 minutes on their commute.

---

## §2 — Market Positioning

| Feature | Final Draft 13 | Fade In Pro | WriterDuet | Arc Studio Pro | Celtx | Squibler/Melies | **LIPILY** |
|---|---|---|---|---|---|---|---|
| **Price** | $249 one-time | $79.99 one-time | $11.99/mo | $9.99/mo | $15/mo | $29/mo | **Free tier + $12/mo Pro** |
| **AI Features** | None | None | None | Basic suggestions | Basic | Basic | **Full AI Suite (5-pillar, dialogue, continuity, pacing, pitch, voice guardian)** |
| **Real-Time Collab** | No | No | Yes (basic) | Yes (basic) | Yes | No | **Yes (Yjs CRDT, cursor presence, Writers' Room)** |
| **Web-Based** | No | No | Yes | Yes | Yes | Yes | **Yes (CanvasKit, offline-first)** |
| **Story Tools** | Beat Board only | None | None | Basic | Basic | Basic | **Beat Board + Story Bible + Storyboard + Breakdown + Research Mode** |
| **Beginner-Friendly** | No | Moderate | Moderate | Yes | Yes | Yes | **Yes (Smart Flow, SmartType, onboarding, mobile-first)** |
| **Social Discovery** | None | None | None | None | None | None | **Full platform (Listen the Script, feed, leaderboard, reviews, producer portal)** |
| **Offline Support** | Desktop only | Desktop only | Limited | Limited | Limited | No | **Full offline-first (PowerSync + Drift)** |
| **Mobile** | No | No | iOS only | iOS + Android | iOS + Android | Web only | **Flutter: iOS, Android, Web, macOS, Windows** |
| **Git Versioning** | No | No | No | No | No | No | **Named drafts + WGA colors + diff viewer + branch editing + merge requests** |
| **India / Bollywood** | No | No | No | No | No | No | **First-class: Hindi, multilingual, Bollywood format preset, Indian creator focus** |

**LIPILY's positioning:** "Where screenwriters write, get discovered, and get heard — literally."

---

## §3 — User Personas

### Arjun (Primary — Emerging Writer)

| Attribute | Value |
|---|---|
| **Age** | 25 |
| **Location** | Mumbai, India |
| **Occupation** | Aspiring screenwriter, part-time content writer |
| **Primary Device** | Android phone (Redmi) + occasional laptop |
| **Current Tools** | Google Docs, WhatsApp (for sharing), occasional free Celtx trial |
| **Pain Points** | (1) Cannot afford Final Draft. (2) No one reads his scripts — no discovery platform. (3) Writes in Hindi/English but tools only support English. |
| **Success on LIPILY** | Writes his first feature on LIPILY free tier, publishes it, gets 500 plays on "Listen the Script," a producer sends a contact request. |
| **Key Stories** | story-002, story-003, story-004, story-007, story-039, story-040, story-041, story-042, story-043, story-052 |

### Priya (Primary — Professional TV Writer)

| Attribute | Value |
|---|---|
| **Age** | 32 |
| **Location** | Mumbai, India |
| **Occupation** | Staff writer on 3 streaming shows simultaneously |
| **Primary Device** | MacBook Pro + iPhone |
| **Current Tools** | Final Draft (company license), Google Docs for collab, WhatsApp for drafts, Notion for story notes |
| **Pain Points** | (1) Collaboration is a nightmare — email chains, version conflicts. (2) 4 different tools for one job. (3) Final Draft crashes on 150-page scripts. |
| **Success on LIPILY** | Her entire writers' room moves to LIPILY Studio. Real-time collaboration. Story bible auto-populated. Zero email threads about "latest version." |
| **Key Stories** | story-004, story-008, story-009, story-010, story-012, story-013, story-017, story-053 |

### Marcus (Secondary — Staff Writer, Hollywood)

| Attribute | Value |
|---|---|
| **Age** | 38 |
| **Location** | Los Angeles, USA |
| **Occupation** | Staff writer, TV drama |
| **Primary Device** | MacBook Pro |
| **Current Tools** | Final Draft, Google Docs, Slack, Notion |
| **Pain Points** | (1) Real-time multi-user editing doesn't exist in Final Draft. (2) Scene locking and showrunner approval are manual. (3) Branch editing requires Dropbox + naming conventions. |
| **Success on LIPILY** | A 6-person writers' room runs live on LIPILY Studio. Showrunner locks Act 3 while others write Act 1. Merge request system replaces "please review my draft" emails. |
| **Key Stories** | story-010, story-013, story-029, story-053 |

### Aisha (Secondary — Independent Filmmaker)

| Attribute | Value |
|---|---|
| **Age** | 28 |
| **Location** | Lagos, Nigeria |
| **Occupation** | Independent filmmaker |
| **Primary Device** | MacBook Air + iPhone |
| **Current Tools** | Scrivener (story bible), Final Draft (editor), Notion (research), Google Docs (collab) |
| **Pain Points** | (1) 4 tools for one job — context switching kills focus. (2) No integration between story tools and script. (3) No affordable discovery platform for West African cinema. |
| **Success on LIPILY** | All story tools unified in one place. Her Nollywood-genre script gets discovered through the feed. Producer contact request arrives from Lagos. |
| **Key Stories** | story-004, story-007, story-008, story-017, story-030, story-031, story-040, story-041, story-043 |

### Dev (Tertiary — Film Student / Reviewer)

| Attribute | Value |
|---|---|
| **Age** | 21 |
| **Location** | Pune, India (film school) |
| **Occupation** | Film studies student |
| **Primary Device** | Laptop + Android phone |
| **Current Tools** | None (reads scripts as PDFs) |
| **Pain Points** | (1) No entry point into the industry. (2) No way to build a portfolio as a script analyst. (3) No paid opportunity to develop reviewing skills. |
| **Success on LIPILY** | Verified as Student Reviewer in 48 hours. Completes 10 reviews. Earns $50. Gets hired as a reader at a Mumbai production house based on his LIPILY reviewer profile. |
| **Key Stories** | story-045, story-046, story-048 |

### Rajesh (Tertiary — Film Producer)

| Attribute | Value |
|---|---|
| **Age** | 50 |
| **Location** | Mumbai, India |
| **Occupation** | Independent film producer |
| **Primary Device** | iPhone + iPad |
| **Current Tools** | Email (receives PDFs), WhatsApp (scripts sent by agents/friends) |
| **Pain Points** | (1) Cannot cold-read 120-page PDFs. (2) No way to discover unagented writers. (3) No quality filter on submissions — no way to know if a script has been reviewed. |
| **Success on LIPILY** | Discovers 3 reviewed scripts on the producer portal. Watches 2 "Listen the Script" presentations on his commute. Sends contact request to one writer. |
| **Key Stories** | story-042, story-043, story-044, story-045, story-047 |

---

## §4 — Epic Table

| Epic ID | Epic Name | Description | Priority | Phase | Story Count |
|---|---|---|---|---|---|
| E01 | Core Editor | Fountain-compliant editor, Smart Flow, SmartType, autosave, RTL | P0 | 1 | 8 |
| E02 | AI Scene Intelligence | 5-pillar extraction, scene sidebar, ContentSource tracking | P0 | 2 | 2 |
| E03 | AI Feature Suite | Dialogue suggest, continue scene, continuity, pacing, pitch, voice guardian, multilingual | P1 | 2 | 7 |
| E04 | Collaboration & CRDT | Yjs real-time sync, cursor presence, roles, invites, comments | P0 | 1-2 | 4 |
| E05 | Version Control & Branching | Named drafts, WGA colors, diff viewer, branch editing, merge requests | P1 | 2 | 2 |
| E06 | Story Tools | Story bible, beat board, character manager, location manager, storyboard, research mode | P1 | 1-3 | 7 |
| E07 | Offline & Sync | PowerSync, Drift, offline queue, Focus Mode sync, reconnect | P0 | 1 | 2 |
| E08 | Export & Import | PDF, FDX, Fountain, breakdown, scene production PDF, import | P1 | 1-2 | 3 |
| E09 | Editor Utilities | Find/Replace, global search, inline tagging, title page, alt dialogue, smart deletion, revision mode, shortcuts | P1 | 2-3 | 9 |
| E10 | Social Platform | Profiles, usernames, publishing, Listen the Script | P1 | 3 | 5 |
| E11 | Discovery & Feed | Social feed, leaderboard, notifications | P2 | 3 | 3 |
| E12 | Review Marketplace | Review submission, reviewer program, producer portal | P2 | 3-4 | 3 |
| E13 | System & Infrastructure | Project setup, auth, dashboard, subscription, onboarding, admin, moderation | P0-P1 | 1-4 | 6 |

---

## §5 — All 55 User Stories (GIVEN/WHEN/THEN)

### Epic E13 — System & Infrastructure

---

**story-001: Project Setup & Infrastructure**

GIVEN the development team is starting from scratch  
WHEN the infrastructure story is executed  
THEN:
1. Flutter project initializes on all 5 platforms with CanvasKit renderer
2. Rust Axum backend has health endpoint at `/health` returning `{status: "ok", version: "x.x.x"}`
3. Docker container builds successfully
4. Cloud Run deployment config is set with `minInstances=1`, `sessionAffinity=true`, `timeoutSeconds=3600`
5. All Supabase tables are created with RLS and deny-all defaults
6. PowerSync is configured and syncing test records
7. GitHub Actions pipeline runs on every PR and deploys to staging on `develop` merge
8. Sentry is capturing errors from Rust + Flutter + TypeScript
9. PostHog is capturing events from Flutter Web + mobile
10. All secrets are in Google Secret Manager — none in code or git

---

**story-002: Authentication**

GIVEN a new user opens LIPILY for the first time  
WHEN they complete the auth flow  
THEN:
1. They can request a magic link by entering their email (validates email format, max 254 chars)
2. They receive the magic link email via Resend within 60 seconds
3. Clicking the magic link authenticates them and redirects to username selection
4. Google OAuth login completes successfully and redirects to username selection
5. Username selection validates 3-30 chars, alphanumeric + underscore, case-insensitive uniqueness
6. JWT is stored securely (not in localStorage) and auto-refreshed by supabase_flutter
7. All protected routes redirect to `/auth/login` if JWT is missing or expired
8. Logout revokes the session and redirects to `/auth/login`
9. Age verification checkbox is required — no account created without it
10. Account type selection (Writer/Producer/Student/Reviewer) is required before dashboard

---

**story-003: Dashboard**

GIVEN an authenticated user navigates to `/dashboard`  
WHEN the dashboard loads  
THEN:
1. All user scripts are listed with: title, format chip, last edited timestamp, word count, scene count, collaborator avatars
2. Free tier shows a hard 3-script limit — "Create Script" button is disabled with upgrade prompt after 3 scripts
3. Create script modal requires title (1-200 chars) and format selection (6 formats)
4. Soft-deleted scripts appear in "Recently Deleted" panel for 30 days
5. Restoring a deleted script restores it to the script list
6. Scripts can be sorted: last edited / created / alphabetical
7. Script search filters the list in real time
8. Empty state shows animated "Your first script is waiting" with Create button
9. Branch badge appears on script card if an active branch exists
10. Script card context menu: Open, Publish, Duplicate, Export, Delete

---

**story-048: Subscriptions (Stripe)**

GIVEN a free-tier user wants to upgrade  
WHEN they initiate checkout  
THEN:
1. POST `/subscription/checkout` creates a Stripe Checkout Session with correct `price_id` from Secret Manager
2. User is redirected to Stripe-hosted checkout page
3. After successful payment, `checkout.session.completed` webhook fires
4. Webhook handler verifies `Stripe-Signature` header before processing
5. Subscription record is created/updated in `subscriptions` table
6. User's `profiles.subscription_tier` is updated to `pro` or `studio`
7. Customer Portal session allows self-service plan management and cancellation
8. On subscription cancellation, tier downgrades to `free` at period end
9. If user has >3 scripts after downgrade, excess scripts are archived (not deleted) with notification
10. Annual pricing ($99/year for Pro) is available alongside monthly ($12/month)

---

**story-052: Onboarding Flow**

GIVEN a new user has just completed auth and username selection  
WHEN the onboarding flow begins  
THEN:
1. Welcome screen shows LIPILY value prop with 3 animated feature highlights
2. Account type selection screen updates `profiles.account_type`
3. Genre preferences screen saves to `profiles.genre_preferences`
4. First script creation or import is offered (name + format, or Fountain/FDX upload)
5. Editor tutorial overlay shows 5 steps: element types, Smart Flow, 5-pillar sidebar, Focus Mode, Collaboration
6. Tutorial can be dismissed and re-triggered from Help menu
7. Mobile users see additional tutorial for bottom element selector and swipe-up scene navigator
8. PostHog events: `onboarding_step_completed`, `onboarding_completed` are tracked
9. Onboarding can be skipped entirely with a "Skip" button
10. `profiles.onboarding_complete` is set to `true` on completion or skip

---

**story-055: Admin Dashboard**

GIVEN an admin user navigates to `/admin`  
WHEN the dashboard loads  
THEN:
1. Route is blocked with 403 for non-admin users (enforced in Rust middleware)
2. Overview section shows: total users, MAU, scripts created, Listen posts, active subscriptions, MRR, today's signups
3. User management allows search by username/email, manual tier override, account suspension
4. Moderation queue shows pending reports with batch approve/remove actions
5. Reviewer management shows pending applications with approve/reject controls
6. Feature flags panel toggles features without redeploy (stored in Upstash Redis)
7. Emergency AI kill switch disables all POST `/ai/*` endpoints
8. Audit log records all admin actions in `admin_audit_log` table (write-only from UI)
9. All admin actions generate OpenTelemetry spans
10. Admin-only routes are protected by both auth middleware AND admin role check in Rust

---

### Epic E01 — Core Editor

---

**story-004: Core Editor Engine**

GIVEN an authenticated user opens a script in the editor  
WHEN they type content  
THEN:
1. All 7 Fountain element types are supported with correct formatting and visual distinction
2. Smart Flow Logic matrix (all 14 Enter/Tab transitions) works correctly for all block types
3. SmartType autocomplete appears after 1 char in CHARACTER block with character names from the script
4. SmartType autocomplete appears after INT./EXT. in SCENE_HEADING with location names
5. Input latency is < 16ms (single keypress to screen render)
6. Autosave fires every 30 seconds while editor is active
7. Page estimate is calculated (1 page = 55 lines) and shown in real time
8. Unicode content (Hindi, Tamil, Arabic, Korean) renders correctly
9. RTL scripts use Flutter `Directionality` widget for correct text layout
10. Word count and scene count update in real time

---

**story-005: Scene Navigator**

GIVEN a user is in the editor  
WHEN the scene navigator is visible  
THEN:
1. All scenes are listed with: heading, page estimate, color tag, 5-pillar completeness dots
2. Tapping a scene card scrolls the editor to that scene
3. Scenes can be drag-reordered (gap position strategy — 1000/2000/3000)
4. Add scene button inserts a new SCENE_HEADING block
5. Delete scene soft-deletes the scene (30-day recovery)
6. Color tags can be set from 8 color options on each scene card
7. Scene collapse toggle hides all blocks in that scene
8. On 375px mobile: scene navigator becomes a bottom sheet, accessible via swipe-up or button
9. Navigator panel is collapsible on desktop (click arrow or drag)
10. 5-pillar completeness dots: green if confirmed items exist in that pillar

---

**story-006: Focus Mode**

GIVEN a user is writing  
WHEN they activate Focus Mode (Cmd+Shift+F)  
THEN:
1. SceneNavigatorPanel and SceneIntelligenceSidebar are hidden
2. All non-active block text fades to 20% opacity
3. Active line is centered at 40% from viewport top (typewriter mode)
4. All in-app notifications are queued and shown on exit
5. PowerSync sync is disabled — all writes queue to Drift offline_write_queue
6. Background is `#000000`, text is `#E8E8E8` (Midnight Mode)
7. Exit via Escape key or floating FAB button
8. Cmd+Shift+F toggles Focus Mode on and off
9. On exit, queued notifications are shown and Drift queue drains
10. Focus Mode state persists if the user navigates away and returns within the same session

---

**story-014: Smart Deletion & Recovery**

GIVEN a user deletes content  
WHEN smart deletion criteria are met  
THEN:
1. Shift+Backspace saves content to `recently_deleted` before deleting
2. Auto-protection fires when: entire scene is deleted, 5+ consecutive blocks deleted, >10 blocks deleted in 5s
3. Recently Deleted panel shows: item preview, time deleted, script/scene name, Restore button
4. Restore re-inserts content at original position (or end of scene if position is occupied)
5. Items in recently_deleted are purged after 30 days
6. Permanent delete button requires confirmation dialog
7. Recently Deleted panel is accessible from editor sidebar
8. Free tier: Recently Deleted is available (same 30-day retention as Pro)
9. Deleted scenes show in Recently Deleted with all their blocks as a unit
10. Multiple items can be restored in sequence without conflicts

---

**story-015: Alternate Dialogue Variants**

GIVEN a user is editing a DIALOGUE block  
WHEN the AlternateDialogueRow is visible  
THEN:
1. Variants are stored as JSONB array in `blocks.metadata.variants`
2. Left/right arrow buttons cycle through variants (Cmd+Alt+← / →)
3. Variant count label shows "2 of 3" style indicator
4. Add variant (+) button adds a new empty variant
5. Delete current variant button removes it (disabled if only 1 variant)
6. "Mark as current" sets the active export variant
7. Only the "current" variant is included in any export (PDF, FDX, Fountain)
8. AI dialogue suggestions (story-033) insert as new variants with `content_source: aiGenerated`
9. Each variant tracks its own `content_source` enum value
10. All variants are fully editable at any time

---

**story-025: Title Page Editor**

GIVEN a user accesses the title page editor  
WHEN they fill in the fields  
THEN:
1. Fields: Title (required), Written By (auto-populated from profile), Based On (optional), Contact Info (optional), Draft Date (auto-populated, editable), WGA Registration Number (optional)
2. Preview renders WGA-standard layout: centered title, uppercase, proper spacing
3. Title page is exported as page 1 in all PDF exports
4. WGA-standard formatting is enforced in the PDF output
5. Draft Date updates automatically when a new draft is saved (editable override)
6. Contact Info fields: email, phone, address — each optional
7. Title page is accessible from editor File menu
8. Changes are saved per-script (not globally)
9. Import (story-021) populates title page fields from imported file if available
10. Empty state shows placeholder text for each field

---

**story-026: Find & Replace**

GIVEN a user presses Cmd+F  
WHEN the find panel opens  
THEN:
1. Find panel opens as a non-modal bottom bar (does not block editor content)
2. Result count "X of Y" updates as user types (debounced 300ms)
3. Cmd+G / Cmd+Shift+G navigate to next/previous match
4. Current match is highlighted and scrolled into view
5. Replace field toggles on demand
6. Replace Current replaces the active match (undoable)
7. Replace All replaces all matches with confirmation count shown first
8. Scope toggle: entire script / current scene
9. Case-sensitive toggle works correctly
10. Character rename mode: search CHARACTER block names and replaces across all block types (with confirmation count before applying)

---

**story-027: Script Formatting Preferences**

GIVEN a user accesses formatting preferences  
WHEN they change settings  
THEN:
1. Page size options: US Letter / A4 (stored in `formatting_preferences`)
2. Font size options: 10pt / 11pt / 12pt Courier Prime
3. Margin presets: standard / wide / narrow (precise mm values stored in JSONB)
4. Scene number display: off / left / right / both
5. Header and footer text customization (title, draft color, custom text, page numbers)
6. Industry presets: WGA Feature, BBC Drama, Bollywood, Stage Play — each applies correct values
7. Custom preferences can be saved as a named template
8. Named templates can be applied to new scripts from the create script flow
9. Changes apply immediately to PDF export preview
10. Per-script preferences (not global) — each script can have different formatting

---

**story-028: Script Statistics & Writing Goals**

GIVEN a user enters the editor  
WHEN a writing session begins  
THEN:
1. Session auto-starts on editor entry and ends on navigation away or 5 min inactivity
2. `writing_sessions` table records: words_written delta, scenes_modified, duration
3. Daily goal (words per day or scenes per day) can be set in settings
4. Progress bar toward goal is shown in editor top bar (subtle, collapsible)
5. Stats dashboard at `/settings/stats`: total words, scenes, scripts, avg session length, streak, longest streak
6. Writing streak counts consecutive days with any editor activity
7. Weekly email digest (opt-in, via Resend) sent every Monday morning with previous week's stats
8. Word count delta tracks accurately even across multiple sessions per day
9. Streak is not broken if user was offline and syncs later
10. PostHog events: `session_started`, `session_ended`, `goal_reached` are tracked

---

**story-051: Keyboard Shortcuts & Accessibility**

GIVEN a user is in the editor  
WHEN they use keyboard shortcuts  
THEN:
1. All default shortcuts work: Cmd+S, Cmd+F, Cmd+G, Cmd+Shift+G, Cmd+K, Cmd+Shift+F, Cmd+Z, Cmd+Shift+Z, Cmd+/, Tab, Enter, Cmd+Alt+←, Cmd+Alt+→, Cmd+Shift+E, F1-F7, Escape
2. Custom remapping is available at `/settings/shortcuts`
3. Remap screen detects and warns on shortcut conflicts
4. Shortcut reference panel opens with Cmd+/ showing all shortcuts grouped by category
5. All interactive elements have ≥ 44px tap target
6. Color contrast ≥ 4.5:1 for all body text
7. No information conveyed by color alone (secondary indicators present)
8. All images have meaningful alt text
9. Skip link "Skip to editor" is the first focusable element on editor screen
10. `prefers-reduced-motion` disables all non-essential animations

---

### Epic E07 — Offline & Sync

---

**story-018: Offline-First with PowerSync**

GIVEN a user loses their internet connection  
WHEN they continue writing  
THEN:
1. PowerSync SDK is configured for all 14 required tables
2. All writes while offline go to Drift `offline_write_queue` first
3. Queue drain on reconnect uses exponential backoff: 1s→2s→4s→8s→16s
4. After 5 failures, operation moves to `dead_letter` status
5. Conflict resolution: server wins for collaborative scripts (Yjs handles), client wins for solo scripts
6. Sync status indicator shows: "Synced ✓" / "Syncing..." / "Offline — X changes pending"
7. Focus Mode integration: sync pauses during Focus Mode
8. Offline indicator banner: subtle, non-intrusive, at screen bottom
9. Dead letter queue notification on next app foreground
10. Offline queue flushes within 5 seconds of reconnect under normal conditions

---

### Epic E04 — Collaboration & CRDT

---

**story-009: Collaboration Invite & Roles**

GIVEN a script owner wants to add a collaborator  
WHEN they send an invite  
THEN:
1. Owner invites by email via POST `/collaboration/:scriptId/invite`
2. Rust API creates invite record and calls Resend with LIPILY-branded email containing JWT-signed accept link (7-day expiry)
3. Accept link validates JWT, creates collaborator record, redirects to script
4. Roles: Owner, Lead Writer, Editor, Writer, Viewer with correct permission matrix
5. Owner can revoke any collaborator
6. Owner can transfer ownership (requires confirmation from both parties)
7. Pending invites list shows in collaboration settings
8. Role change takes effect immediately without re-invite
9. Maximum collaborators per tier: Free (3), Pro (10), Studio (20)
10. Revoked collaborator loses access immediately on next request

---

**story-010: Real-Time Collaboration (Yjs)**

GIVEN multiple users have a script open simultaneously  
WHEN they type concurrently  
THEN:
1. y_crdt Flutter client connects to Rust `/presence/:scriptId` WebSocket
2. JWT is verified on WebSocket upgrade handshake
3. Y.Doc syncs bidirectionally — all block edits go through Y.Doc
4. Cursor presence shows colored name tags at each collaborator's block position
5. Cursor sync latency < 200ms
6. Max 10 visible cursors simultaneously (Free/Pro)
7. Auto-reconnect with exponential backoff on WebSocket drop
8. On reconnect: Yjs reconciles state, Drift offline queue is flushed
9. "Working offline — changes will sync on reconnect" banner shows on WebSocket drop
10. Y.Doc snapshot is persisted to Supabase (debounced 5s) on every update

---

**story-011: Comment Threads**

GIVEN a collaborator wants to leave feedback  
WHEN they add a comment on a block  
THEN:
1. Any block can receive an inline comment via tap → "Add Comment"
2. Comments show: avatar, @username, timestamp, text (supports @mentions)
3. Reply threads support up to 2 levels (no deeper nesting)
4. Thread owner or script owner can resolve/unresolve a thread
5. CommentThreadDot (red) appears on block right margin when unresolved comments exist
6. Unresolved count badge shows in editor top bar
7. Comments panel (right slide-over) shows all comments for script, filterable: all/unresolved/mine
8. Tapping a comment in the panel scrolls the editor to the relevant block
9. @mentions trigger notifications to mentioned users
10. New comment triggers notification to script owner and all collaborators

---

### Epic E05 — Version Control & Branching

---

**story-012: Git-Style Versioning & Drafts**

GIVEN a writer wants to save a version of their script  
WHEN they save a named draft  
THEN:
1. Named draft saves via Cmd+S (dialog on first save, updates current draft on subsequent saves)
2. WGA revision colors auto-assigned in sequence: White→Blue→Pink→Yellow→Green→Goldenrod→Buff→Salmon
3. Visual diff viewer: select any two drafts and see additions (green), deletions (red strikethrough), modifications (yellow)
4. Any draft can be restored (creates new draft — never overwrites)
5. Scene-specific history shows all changes to a scene with author, timestamp, diff preview
6. Autosave creates `is_autosave: true` draft every 60 seconds
7. Autosave drafts retained 30 days then purged by scheduled Cloud Run job
8. Diff is computed server-side (Rust, Fountain line diff algorithm)
9. Draft list shows: name, revision color chip, date, page count, autosave indicator
10. Free tier: 3 named drafts. Pro/Studio: unlimited.

---

**story-013: Branch Editing & Merge Requests**

GIVEN a writer wants to experiment without affecting the main script  
WHEN they create a branch  
THEN:
1. Branch is created as an isolated copy of the script at that moment (stored as `fountain_snapshot` in `branches` table)
2. Branch editor shows "BRANCH: [name]" banner in top bar
3. Edits on the branch do not affect the main script
4. Branch editing is solo (not collaborative — Yjs not used for branches)
5. Submit Merge Request shows diff between branch and main (same diff viewer as story-012)
6. Script owner sees MR queue badge on dashboard card
7. Owner can: Approve & Merge, Request Changes (with inline comments), or Reject
8. Approve & Merge applies branch snapshot to main and creates a new named draft
9. Conflict detection: if main changed since branch creation, side-by-side conflict resolver shown
10. Conflict resolver lets writer choose which version of each conflicted section wins

---

### Epic E06 — Story Tools

---

**story-007: Scene Intelligence (5-Pillar)**

GIVEN a writer saves a block  
WHEN the 2-second debounce fires  
THEN:
1. Flutter POSTs to `/ai/extract/:sceneId` with surrounding 10-block context
2. Rust API validates auth, checks rate limits, queues BullMQ job
3. TypeScript agent extracts 5 pillars using `claude-haiku-3-5`
4. Result stored in `scene_intelligence` table
5. Flutter receives update (via WebSocket notification or polling) and sidebar updates
6. Each extracted item has `content_source: aiGenerated` and shows AI badge
7. Writer can: tap to edit (→ `humanEdited`), confirm (→ `humanConfirmed`), dismiss (never re-extracted), add manually (→ `humanWritten`)
8. Confirmed items remain editable forever
9. Rate limited against daily AI quota (Free: 50/day, Pro: 500/day)
10. User can globally disable 5-pillar extraction in settings

---

**story-008: Story Bible Panel**

GIVEN a writer opens the Story Bible  
WHEN they navigate the 5 tabs  
THEN:
1. Characters tab shows character cards auto-populated from scene_intelligence + character manager
2. Each character card shows: name, role chip, arc summary, voice notes, scene count, aliases
3. Locations tab shows location cards with type, description, scene count
4. Themes, Backstory, and Timeline tabs provide freeform rich text editing
5. Hovering over a character name in the block editor shows a character hover card linking to Story Bible
6. All Story Bible content is editable at any time
7. All content syncs offline via PowerSync
8. Free tier: 3 Story Bible entries max (enforced server-side)
9. Story Bible entries track `content_source` for AI-populated items
10. Story Bible accessible as full-screen route on mobile: `/story-bible/:scriptId`

---

**story-016: Character & Location Manager**

GIVEN a writer opens the character or location manager  
WHEN they view and edit records  
THEN:
1. Character records contain: name, aliases, description, age range, role, arc summary, voice notes, key traits, first appearance scene, all scenes they appear in
2. Location records contain: name, type (INT/EXT/BOTH), description, atmosphere notes, all scenes
3. Both are auto-populated from 5-pillar extraction with `content_source: aiGenerated`
4. AI suggests duplicate merges: "RAVI" and "Ravi" → prompt to merge with confirmation
5. SmartType in editor uses character name data from this table (real-time)
6. All fields editable at any time
7. Delete character/location shows confirmation and removes from SmartType
8. Character list sorted by: first appearance / alphabetical / scene count
9. Import (story-021) auto-populates characters/locations from script content
10. Character `voice_notes` field feeds AI Voice Guardian (story-038) and TTS voice selection (story-042)

---

**story-017: Beat Board**

GIVEN a writer switches to Beat Board view  
WHEN the beat board is displayed  
THEN:
1. Each scene shows as an index card: heading, beat summary (editable, max 200 chars), color tag, page estimate, emotional beat label
2. Cards are in a horizontal scrollable timeline
3. Drag-to-reorder cards updates scene positions (gap strategy) and reorders the script
4. Structural overlay toggle works: 3-Act Structure, Save the Cat (15 beats), Sequence Method
5. Overlay renders beat name labels at expected page positions as horizontal markers
6. Cards >15% outside expected page range get a soft amber highlight
7. Beat notes stored in `beat_notes` table with `content_source` tracked
8. Beat board is accessible from editor view toggle (not a separate route on desktop)
9. On mobile: full-screen route `/beat-board/:scriptId`
10. Free tier: beat board is view-only (no editing). Pro/Studio: full editing.

---

**story-030: Storyboard & Shot Visualization**

GIVEN a writer opens storyboard for a scene  
WHEN they add and configure shots  
THEN:
1. Each shot panel shows: shot type selector (7 types), camera movement selector (8 types), description (max 300 chars), image area
2. "Generate Image" button triggers AI generation via DALL-E 3 using scene_intelligence + shot type data
3. Generated image has `content_source: aiGenerated` and shows AI badge
4. "Regenerate with instructions" allows custom prompt (→ badge resets to `aiGenerated`)
5. Writer can upload own reference image (→ `custom_image_url` set, `content_source: humanWritten`)
6. Writer-uploaded images always take precedence over AI images in "Listen the Script"
7. Shot panels are reorderable
8. Shot list export: all shots as a formatted production PDF
9. Storyboard images feed directly into "Listen the Script" generation (story-042)
10. Accessible per-scene from scene card context menu and SceneIntelligenceSidebar

---

**story-032: Inspiration & Research Mode**

GIVEN a writer needs reference material while writing  
WHEN they open Research Mode  
THEN:
1. Research Mode opens as a slide-over panel from editor sidebar
2. Web search: query field, results shown inline (title, snippet, URL — read-only references)
3. Scene Notes: rich text notes attached to current scene (stored in `research_notes`)
4. Global Notes: notes not attached to a specific scene (`scene_id: null`)
5. Character Mood Board: per-character image pins (URLs or uploads, stored in `research_notes.reference_images`)
6. Visual Reference: images pinned to any scene as visual reference
7. All research content is private — never exported, never visible to collaborators
8. Research notes sync offline via PowerSync
9. Web search results are never automatically inserted into the script
10. Research Mode panel is dismissible via Escape or toggle button

---

### Epic E08 — Export & Import

---

**story-019: Export — Production PDF**

GIVEN a writer triggers PDF export  
WHEN the export job completes  
THEN:
1. Full Script PDF uses: Courier Prime 12pt, 1" margins (left 1.5" for binding), page numbers top right, WGA standard page breaks
2. Title page (story-025) is included as page 1
3. Scene numbers display according to formatting preferences
4. Scene Production PDF (per-scene) includes: heading, INT/EXT, day/night, pages, characters with emotional states, props list, environment, music/sound cues, emotion/tone, inline tags
5. Batch scene production PDFs export as a ZIP file
6. Cloud Run Puppeteer service generates PDFs (< 30s for 120-page script)
7. Progress indicator is shown during generation
8. PDF download triggers browser download dialog / share sheet on mobile
9. WGA revision marks (asterisks, colored revision indicators) are included when Revision Mode is active
10. RTL scripts mirror margin layout in PDF (binding margin on right side)

---

**story-020: Export — Fountain & FDX**

GIVEN a writer exports to Fountain or FDX  
WHEN the export completes  
THEN:
1. Fountain export is plain text, Fountain spec compliant, includes all blocks in correct format
2. FDX export is Final Draft 12 XML format with title page, scene numbers, and WGA revision color markers
3. Both formats are round-trip compatible (import back → same content)
4. Export is available from editor File menu and dashboard script card context menu
5. Fountain export handles Unicode content (Hindi, Tamil, Arabic, etc.)
6. FDX export handles Unicode content with proper XML encoding
7. Export completes in < 5 seconds for any script size
8. Export does not include research notes (private content)
9. Inline tags (story-024) are omitted from exports (they are non-printing)
10. Alternate dialogue variants: only the "current" variant is exported

---

**story-021: Script Import**

GIVEN a writer imports an existing script  
WHEN the import completes  
THEN:
1. Supported formats: .fountain, .fdx (Final Draft 11/12/13), .txt
2. Rust parser converts all formats to LIPILY typed Fountain block format
3. Encoding normalization handles UTF-8, UTF-16, Windows-1252 inputs
4. Pre-import preview shows ambiguous/unrecognized lines flagged for review
5. After import, 5-pillar extraction is auto-triggered (story-007) to populate story bible
6. Import completes in < 10s for a 120-page script
7. Imported script is labeled "(Imported)" in the title until renamed
8. Non-Latin content (Hindi, Tamil, Arabic) is preserved correctly through import
9. Import respects multilingual content without requiring story-039 (AI multilingual)
10. Import errors are shown with specific line references and suggestions

---

### Epic E09 — Editor Utilities

---

**story-022: Read-Only Sharing Link**

GIVEN a writer wants to share their script with someone outside LIPILY  
WHEN they create a sharing link  
THEN:
1. POST `/scripts/:scriptId/share-links` creates a JWT-signed link with configurable expiry (7 days / 30 days / never)
2. Public `/view/:token` route is accessible without LIPILY account
3. Viewer sees: full formatted script in read-only editor, scene navigator, basic stats
4. No edit controls, no account prompt during viewing
5. View count is tracked (anonymous)
6. Link can be revoked via DELETE endpoint
7. Revoked links return 404 immediately
8. Free tier: 3 active sharing links. Pro: unlimited.
9. Sharing links do not expose any private research notes
10. Link expiry is enforced server-side (JWT `exp` claim validation)

---

**story-023: Global Search (Cmd+K)**

GIVEN a user presses Cmd+K  
WHEN the command palette opens  
THEN:
1. SearchBar with autofocus opens as a centered overlay
2. Results appear in real-time debounced 300ms via GET `/search?q=&scope=&scriptId=`
3. Rust uses PostgreSQL `pg_trgm` FTS on: blocks, scene headings, character names, script titles
4. Results grouped by type with section headers: Scenes, Characters, Scripts, Blocks
5. Keyboard navigation: ↑↓ arrows, Enter to jump to result, Escape to close
6. Recent searches stored in memory (not persisted)
7. Scope toggle: current script / all scripts
8. Response p95 < 200ms
9. Empty state with helpful "Try searching for a character name or scene heading" message
10. Search is accessible to all tiers (no rate limit beyond authenticated user check)

---

**story-024: Inline Scene Tagging**

GIVEN a writer wants to tag a prop, cast member, or VFX element in their script  
WHEN they tag a text selection  
THEN:
1. Writer highlights text → right-click (or long-press on mobile) → "Tag As" submenu
2. Tag types: PROP, CAST, LOCATION, VFX, STUNT, WARDROBE, MAKEUP, VEHICLE
3. Tag stored in `script_tags` table with block_id, position_start, position_end, tag_type
4. Tags render as non-printing colored underlines in editor (8 distinct colors per tag type)
5. Tags are not visible in any export (PDF, FDX, Fountain)
6. Tag chips appear in corresponding scene_intelligence pillar with `content_source: humanWritten`
7. Human-written tags are never overridden by AI extraction
8. Tag manager panel lists all tags by category with scene references and jump-to links
9. Clicking a tag in tag manager scrolls to its scene and highlights the block
10. Tags sync offline via PowerSync

---

**story-029: Revision Mode (Production Standard)**

GIVEN the script owner activates Revision Mode  
WHEN writers make changes  
THEN:
1. Revision Mode activation requires selecting a "locked production draft"
2. Changed lines show asterisk (*) in right margin (WGA standard)
3. Pages are locked — new lines push to A/B pages (A1, A2, etc.)
4. WGA revision color chip shows current revision cycle in top bar
5. Scene Omit marks scene with "OMITTED" label, preserves original scene number in navigator
6. Only Owner can toggle Revision Mode on/off
7. Revision Mode indicator is visible in editor top bar for all collaborators
8. PDF export in Revision Mode includes all page locks and asterisk markers
9. Revision Mode state persists across sessions (stored in `scripts.is_revision_mode`)
10. Revision color sequence: White→Blue→Pink→Yellow→Green→Goldenrod→Buff→Salmon (auto-advances on next activation)

---

**story-031: Script Breakdown Auto-Generation**

GIVEN a Pro/Studio writer triggers breakdown generation  
WHEN the BullMQ job completes  
THEN:
1. Scene Breakdown Sheet: one row per scene with scene number, INT/EXT, DAY/NIGHT, pages, location, cast, props, VFX notes, stunt notes, special requirements
2. Cast Report: character name, scene count, total pages, all scene numbers
3. Props Master List: prop name, scenes, source (AI-extracted vs human-tagged) visually distinguished
4. Locations Report: location name, INT/EXT, scene count, total pages
5. All data is editable in a breakdown review screen before export
6. Export as formatted PDF or CSV for all 4 report types
7. AI-extracted items vs human-tagged items are visually distinguished in the review screen
8. Generation is a background BullMQ job — user is not blocked
9. Sources: `scene_intelligence` table data + `script_tags` table data
10. `content_source` is tracked for all breakdown items

---

### Epic E02 — AI Scene Intelligence

_(See story-007 above in E06)_

---

### Epic E03 — AI Feature Suite

---

**story-033: AI Dialogue Suggest**

GIVEN a Pro user focuses on a CHARACTER or DIALOGUE block  
WHEN they trigger AI Dialogue Suggest  
THEN:
1. Subtle icon appears on hover/focus on CHARACTER and DIALOGUE blocks (Pro only)
2. POST `/ai/dialogue-suggest/:blockId` queues BullMQ job
3. Agent uses `claude-haiku-3-5` with: character profile, voice notes, previous 10 blocks context
4. 3 dialogue suggestions returned and shown in AISuggestionPanel
5. "Insert as Variant" adds suggestion as alternate variant (`content_source: aiGenerated`, AI badge)
6. "Replace Current" replaces block content but saves old as alternate variant (`content_source: aiGenerated`)
7. Each suggestion is individually dismissible
8. "Regenerate with instructions" custom prompt field is available
9. Usage logged in `ai_usage` table
10. Respects script language (story-039) — suggestions are in the same language as the script

---

**story-034: AI Continue Scene (Writer's Block Breaker)**

GIVEN a Pro user is stuck on a scene  
WHEN they click "Unstick Me"  
THEN:
1. Modal shows 3 mode choices FIRST — no AI content generated until mode is explicitly chosen
2. Socratic Mode returns 5 questions as a list — no generated script content
3. Questions are shown in a panel for reflection but not inserted into the script
4. Three Directions returns 3 scene direction summaries (max 3 sentences each, no dialogue)
5. Writer can tap "Use this direction" to insert as a beat note (`content_source: aiGenerated`)
6. Rough Placeholder returns 3-5 blocks of rough draft for writer to rewrite
7. "Insert Placeholder" is an explicit confirm button — not auto-inserted
8. All inserted blocks have `content_source: aiGenerated` and show AI banner
9. Approval queue: each insertion creates an approval record (auto-approved on explicit confirm)
10. All inserted content is fully editable and deletable at any time

---

**story-035: AI Continuity & Logic Checker**

GIVEN a Pro user triggers the continuity checker  
WHEN the background job completes  
THEN:
1. POST `/ai/continuity-check/:scriptId` returns `job_id` immediately (non-blocking)
2. Flutter polls GET `/ai/continuity-check/:scriptId/status` until complete
3. Agent uses `claude-sonnet-4-5` with full script content + scene_intelligence data
4. Checks: dead characters reappearing, location contradictions, age/timeline inconsistencies, prop continuity, weather inconsistencies
5. Each flag has: flag_type, description, page_references array, block_ids array
6. Flags stored in `continuity_flags` table
7. Flutter shows flags as dismissible cards with "Jump to Scene" link and "Not an Error" button
8. Dismissed flags stored with `is_dismissed: true` and not re-shown
9. Re-check only runs on explicit user trigger (not automatic)
10. Flag display never rewrites or modifies any script content

---

**story-036: AI Pacing & Structure Analyzer**

GIVEN a Pro user triggers the pacing analyzer  
WHEN the background job completes  
THEN:
1. POST `/ai/pacing-analysis/:scriptId` returns `job_id` immediately (non-blocking)
2. Returns: dialogue_to_action_ratio per act, scene length distribution, b_story_gap_data, emotional_intensity_curve, act_balance_percentages
3. Flutter renders PacingDashboard with charts (fl_chart package)
4. Flags: 3+ consecutive dialogue-heavy scenes, Act 2 >25 pages no major beat, any act >15% off structural norm
5. Each flag is a dismissible card with page reference and one-line suggestion
6. Zero content rewriting — all output is read-only analysis
7. Emotional intensity curve uses `scene_intelligence.emotion` data per scene
8. B-story gap detection shows scenes since each supporting character last appeared
9. Act balance percentages are calculated from scene count and page count per act
10. Pacing dashboard is accessible as a dedicated screen route

---

**story-037: AI Logline & Pitch Generator**

GIVEN a Pro user triggers pitch generation on a script ≥ 60 pages  
WHEN the background job completes  
THEN:
1. "Generate Pitch" button is disabled (with tooltip) for scripts < 60 pages
2. POST `/ai/pitch-generate/:scriptId` returns `job_id` immediately
3. Agent uses `claude-sonnet-4-5` with full script + scene_intelligence + story bible data
4. Returns: logline (1 sentence), synopsis (1 paragraph), one_page_treatment (structured), comp_titles (3-5 films)
5. Each output appears in its own tab in PitchGeneratorPanel
6. All tabs are fully editable rich text (`content_source: aiGenerated` → `humanEdited` on first edit)
7. "Regenerate with instructions" field available per tab
8. Export all tabs as a single formatted pitch PDF (via Cloud Run Puppeteer)
9. Usage logged in `ai_usage` table
10. All AI-generated pitch content shows AI badge until writer edits it

---

**story-038: AI Character Voice Guardian**

GIVEN a Pro writer's dialogue is being analyzed  
WHEN the voice guardian detects an inconsistency  
THEN:
1. Voice guardian runs per scene on every block save (alongside 5-pillar extraction, same debounce)
2. Agent uses `claude-haiku-3-5` to compare dialogue against character's `voice_notes` in `characters` table
3. Inconsistency above confidence threshold creates flag in `scene_intelligence.characters` JSONB: `{type: "voice_inconsistency", characterName, blockId, confidence, message: "Is this [CHARACTER]?"}`
4. Flutter shows flag as a subtle underline on the block with a tooltip
5. Writer can: dismiss flag, edit dialogue (flag auto-removed on edit), or view voice profile
6. Agent NEVER rewrites any dialogue
7. Agent NEVER inserts any content into the script
8. Voice guardian flags only appear — no changes are made
9. Flags are per-save, not stored permanently (re-evaluated on next save)
10. Voice guardian can be globally disabled in settings

---

**story-039: AI Multilingual Support**

GIVEN a writer has set their script language to a non-English language  
WHEN they use any AI feature  
THEN:
1. Language selector is per-script (in script settings) — not per-account
2. Language code stored in `scripts.language`, RTL flag in `scripts.rtl`
3. Flutter `Directionality` widget wraps `BlockEditorView` based on `scripts.rtl`
4. All AI agents receive language code in system prompt and respond in the same language
5. Font fallback stack includes Noto Sans for Hindi, Tamil, Arabic, Korean, Japanese, Chinese, Thai
6. SmartType works correctly for non-Latin character names (no `[a-zA-Z]` assumptions)
7. Fountain parsing handles non-Latin content without corruption
8. PDF export preserves non-Latin content with proper encoding
9. RTL scripts: PDF export mirrors margin layout (binding margin moves to right side)
10. No hardcoded language strings — all user-facing text is in Flutter l10n system

---

### Epic E10 — Social Platform

---

**story-040: Writer Profiles & Usernames**

GIVEN a writer visits their public profile  
WHEN the profile is viewed  
THEN:
1. Public profile at `lipily.com/@username` (no auth required)
2. Profile shows: display name, bio (max 300 chars), location, avatar, published scripts grid, follower count, following count
3. Username change allowed once every 30 days (`last_username_change_at` enforced server-side)
4. Avatar upload: Supabase Storage, max 5MB, MIME validated server-side
5. Follow/unfollow button is available (requires auth)
6. Producer account badge visible on producer profiles
7. Reviewer badge visible on verified reviewer profiles
8. Profile page has SEO meta tags for social preview (Open Graph)
9. 404 is returned for unknown usernames
10. Profile is NOT visible for deleted accounts

---

**story-041: Script Publishing**

GIVEN a writer clicks "Publish Script"  
WHEN the publish wizard completes  
THEN:
1. Publish wizard: (1) Choose visibility (Public/Unlisted/Invite-Only), (2) Add genres (max 5), (3) Write logline (max 250 chars), (4) Add tags (max 10), (5) Preview card, (6) Confirm
2. Creates record in `script_publications` table
3. Unpublish via one-tap sets `published_at` to null — not visible to anyone except owner
4. Published script viewable at `/@username/script/:scriptId` (no auth required)
5. Script format chips auto-populated from `scripts.format`
6. Logline, genres, and tags can be updated post-publish without unpublishing
7. Free tier: 1 published script max. Pro/Studio: unlimited.
8. Unlisted scripts are accessible via direct URL but not shown in feeds or search
9. Invite-Only scripts require a sharing link (story-022) to access
10. Published scripts are indexed by the discovery feed and leaderboard systems

---

**story-042: Listen the Script**

GIVEN a writer triggers "Create Listen" on a published script  
WHEN the generation pipeline completes (< 60s for 10-page scene)  
THEN:
1. BullMQ job is created; Flutter shows generation progress screen with step-by-step progress
2. Approval Gate 1 (voice assignment): writer reviews voice table, can change any voice, confirms to continue
3. Approval Gate 2 (music selection): writer previews selected music, can change, confirms to continue
4. Preview player screen shows full playback experience before publish
5. Writer can: replace any image (per-scene regenerate or upload), change any voice (triggers re-gen of that character's audio only), change music
6. Published Listen post auto-plays on scroll (mobile, like a Reel)
7. Shows animated subtitles, scene imagery, character name overlay on dialogue lines
8. Like, bookmark, share actions available
9. View count tracked
10. Playback works in any browser without LIPILY account — no login prompt

---

### Epic E11 — Discovery & Feed

---

**story-043: Social Feed & Discovery**

GIVEN a user opens the discovery feed at `/discover`  
WHEN they browse content  
THEN:
1. Feed tabs: For You (algorithmic), Trending (by plays+likes in 24h), Following, By Genre
2. Feed items: Listen posts (preview thumbnail, play button, duration, logline first line) and Script Cards (title, genres, logline, review badge, page count, like count)
3. Infinite scroll with cursor-based pagination
4. Genre/format/language/region filter panel is available
5. For You feed is personalized for logged-in users; shows popular content for guests
6. Like and bookmark actions require auth
7. Feed visible without auth
8. Feed algorithm runs in Rust API with Upstash Redis cache (TTL: 5 min)
9. Filter: Bollywood/Hollywood/Nollywood/K-Drama/and more regional options
10. Feed items link to the public script/Listen page (no auth required to view)

---

**story-044: Leaderboard**

GIVEN a user visits `/leaderboard`  
WHEN the leaderboard loads  
THEN:
1. Data updated every 24 hours via scheduled BullMQ cron job
2. Table columns: Rank, Writer (avatar, @username, display name), Scripts Published, Total Plays, Total Likes, Followers, Avg Review Score, Badges
3. Filters: Genre (dropdown), Format (dropdown), Region (dropdown — India/US/Africa/Korea/UK/Global), Time: Weekly/Monthly/All-time
4. Badges: Rising Star (top 10% follower growth this week), Top 10% (top 10% total plays this month), Genre Champion (top 3 in any genre filter), Reviewed Writer (≥1 completed review)
5. Badges stored in `profiles.badges` TEXT[] and awarded by scheduled job
6. Badge award creates a notification to the writer
7. Leaderboard is publicly accessible (no auth required)
8. Clicking a writer row navigates to their public profile `/@username`
9. My Rank indicator shows logged-in user's own rank even if outside top list
10. Leaderboard shows "Updated X hours ago" timestamp

---

**story-049: Notifications**

GIVEN a user has received activity on their content  
WHEN they open the notification center  
THEN:
1. All 16 notification types are implemented and trigger correctly (see story-049 spec)
2. Notification panel sorted newest first, grouped by: today / yesterday / older
3. Tapping a notification navigates to the relevant screen
4. "Mark all as read" clears unread count badge
5. Individual notification delete is available
6. Unread count badge shows in top bar bell icon and mobile tab bar
7. Bell icon badge count updates in real time (WebSocket or polling)
8. Weekly email digest (opt-in) via Resend every Monday morning IST with unread summary
9. Notification preference settings allow toggling notification types on/off
10. Notifications are cleaned up after 90 days (scheduled purge job)

---

### Epic E12 — Review Marketplace

---

**story-045: Review Submission**

GIVEN a Pro writer has a script meeting minimum pages  
WHEN they submit for review  
THEN:
1. "Submit for Review" button is disabled until minimum pages: 70 (Feature Film), 25 (TV), 10 (Short Film)
2. POST `/review/:scriptId/submit` creates review record with `status: submitted`
3. Rust API auto-assigns to available verified reviewer (round-robin, weighted by queue depth)
4. Status lifecycle: Submitted → Assigned → In Review → Complete
5. 14-day SLA: no activity from reviewer in 14 days → reassigned + admin alert
6. Writer receives notification on each status change
7. "Reviewed" badge added to `script_publications` on completion
8. Writer can see current review status on script card and script page
9. Only one active review per script at a time
10. Review submission requires Pro tier (enforced server-side)

---

**story-046: Reviewer Program & Payouts**

GIVEN a user applies to become a reviewer  
WHEN their application is processed  
THEN:
1. Verified Critics apply via `/reviewer/apply` — manual admin approval, notification via Resend
2. Student Reviewers: .edu email or student ID upload (Supabase Storage, MIME-validated), 48h auto-approval if email verified
3. Reviewer dashboard at `/reviewer/dashboard`: assigned scripts queue, review form, completed reviews, earnings
4. Review form structured fields: logline_clarity (1-10), character_depth (1-10), structure (1-10), dialogue (1-10), originality (1-10), overall_score (1-10), written_notes (300 char minimum)
5. Submit review → status: complete → trigger payout via Stripe Connect
6. Payout amounts: Verified Critics $15, Students $5 per completed review
7. Stripe Connect onboarding required before first payout
8. Writer rates reviewer usefulness 1-5 stars after receiving review
9. Reviewer ratings affect future assignment priority
10. Reviewers can be suspended by admin for low quality reviews

---

**story-047: Producer Discovery Portal**

GIVEN a verified producer user opens the producer portal  
WHEN they browse scripts  
THEN:
1. Producer Account type requires company email (not Gmail/Yahoo personal) — verified on signup
2. Script browse filters: genre, format, language, region, page range, review score range, reviewed-only toggle
3. Sort: newest / most reviewed / highest rated / most played
4. Script cards show same info as discovery feed
5. Click-through to `/@username/script/:scriptId` public view
6. "Listen the Script" playback available without auth
7. "Request Contact" button creates notification to writer
8. Writer receives: producer company name, genre interest, message (max 500 chars)
9. Writer can accept or decline contact request — contact info not shared until acceptance
10. Writer can block a producer (prevents further contact requests)

---

### Epic E09 (continued)

---

**story-050: Content Moderation & Reporting**

GIVEN a user encounters inappropriate or infringing content  
WHEN they submit a report  
THEN:
1. Report button is available on every published script and Listen post
2. Report form: reason (ENUM: plagiarism, inappropriate_content, copyright_violation, spam, harassment, other), optional description (max 500 chars)
3. POST `/moderation/report` creates record in `reports` table
4. Auto-hide trigger: 3+ reports within 24 hours → `script_publications.auto_hidden = true` → not shown in feed/search (still accessible at direct URL with "Under Review" banner)
5. Admin moderation queue at `/admin` shows pending reports grouped by script
6. Admin actions: approve (restore visibility), remove (unpublish + notify writer with reason), permanent ban
7. Appeal system: writer submits one appeal per report via support link (48h SLA)
8. Appeal creates ticket in a separate tracking system
9. Report status tracked through: pending → reviewed → approved/removed
10. AI-assisted moderation is Phase 4 scope (marked explicitly out of scope in v1)

---

**story-053: Writers' Room Mode**

GIVEN a Studio admin activates Writers' Room Mode  
WHEN the writers' room session is active  
THEN:
1. Activated by Studio admin from script settings (requires Owner + Studio admin role)
2. Supports up to 20 simultaneous WebSocket connections (Yjs handles all merges)
3. Showrunner Controls panel: scene locking (other writers get "Scene locked by Showrunner" tooltip), scene assignment (badge on scene card showing assigned writer's avatar)
4. Writers' Room Chat: real-time text channel via Upstash Redis pub/sub — ephemeral (not saved to DB, only exists during active WebSocket sessions)
5. Showrunner announcement banner: sticky notice at top of editor visible to all collaborators
6. Admin dashboard for Writers' Room: activity heatmap (who edited which scene, when)
7. Contribution stats per writer: blocks added/modified, session log
8. Studio tier enforcement: Writers' Room blocked for Free and Pro tier users (enforced server-side)
9. Scene locks are released when showrunner unlocks them or session ends
10. Writers' Room mode indicator badge visible on script card in dashboard

---

**story-054: Dialogue Read-Aloud TTS**

GIVEN a Pro writer wants to hear their dialogue read aloud  
WHEN they trigger single-scene read-aloud (Cmd+R)  
THEN:
1. Read-aloud button in scene card context menu and keyboard shortcut Cmd+R
2. POST `/tts/scene/:sceneId` queues BullMQ job (< 10s for a single scene)
3. Uses voice assignments from `script_presentations` if Listen has been generated, else uses default voices
4. Audio streams back and plays inline in editor — `script_presentation` not required
5. Playback controls: play/pause, speed (0.75x / 1x / 1.25x / 1.5x)
6. Current dialogue line is highlighted in editor as it plays
7. Cmd+Shift+R reads only the focused block (single block read-aloud)
8. Right-click CHARACTER block → "Change Voice for [NAME]" opens inline voice picker
9. All generated audio is temporary — not stored in Supabase Storage (streamed from Cloud Run memory)
10. Read-aloud is Pro tier feature (enforced server-side)

---

## §6 — Feature Gate Table

| Feature | Free | Pro ($12/mo) | Studio ($39/mo/team) |
|---|---|---|---|
| Scripts | 3 max | Unlimited | Unlimited |
| Editor (all 7 element types) | ✅ | ✅ | ✅ |
| Smart Flow Logic | ✅ | ✅ | ✅ |
| SmartType autocomplete | ✅ | ✅ | ✅ |
| Scene Navigator | ✅ | ✅ | ✅ |
| Focus Mode | ✅ | ✅ | ✅ |
| Story Bible entries | 3 max | Unlimited | Unlimited |
| Beat Board | View only | Full edit | Full edit |
| Storyboard | ❌ | ✅ | ✅ |
| Script Statistics | ✅ | ✅ | ✅ |
| Named Drafts | 3 | Unlimited | Unlimited |
| Version History | ❌ | 30 days | 90 days |
| Branch Editing | ❌ | ✅ | ✅ |
| Merge Requests | ❌ | ✅ | ✅ |
| Collaborators per script | 3 | 10 | 20 |
| Real-time collaboration | ✅ | ✅ | ✅ |
| Comment threads | ✅ | ✅ | ✅ |
| PDF export | ✅ | ✅ | ✅ |
| FDX + Fountain export | ✅ | ✅ | ✅ |
| Script Breakdown | ❌ | ✅ | ✅ |
| AI requests per day | 50 | 500 | Unlimited |
| 5-Pillar Scene Intelligence | ✅ | ✅ | ✅ |
| AI Dialogue Suggest | ❌ | ✅ | ✅ |
| AI Continue Scene | ❌ | ✅ | ✅ |
| AI Continuity Checker | ❌ | ✅ | ✅ |
| AI Pacing Analyzer | ❌ | ✅ | ✅ |
| AI Pitch Generator | ❌ | ✅ | ✅ |
| AI Voice Guardian | ❌ | ✅ | ✅ |
| Listen the Script generations | 1/month | Unlimited | Unlimited |
| Sharing links | 3 active | Unlimited | Unlimited |
| Public profile | ✅ | ✅ | ✅ |
| Published scripts | 1 | Unlimited | Unlimited |
| Review submission | ❌ | ✅ | ✅ |
| Dialogue Read-Aloud TTS | ❌ | ✅ | ✅ |
| Writers' Room Mode | ❌ | ❌ | ✅ (20 users) |
| Showrunner Controls | ❌ | ❌ | ✅ |
| Scene Locking | ❌ | ❌ | ✅ |
| Admin Dashboard | ❌ | ❌ | ✅ |
| SSO | ❌ | ❌ | ✅ |
| Dedicated Support | ❌ | ❌ | ✅ |
| Student seat discounts | ❌ | ❌ | ✅ (verified .edu) |

---

## §7 — Success Metrics

### v1 Targets (Phase 1–2, end of Month 3)

| Metric | Target |
|---|---|
| DAU (Month 3) | 2,000 daily active writers |
| D7 Retention | 35% |
| D30 Retention | 18% |
| Free → Pro Conversion | 8% within 30 days of signup |
| Net Promoter Score (NPS) | ≥ 40 |
| Script Completion Rate | 25% of started scripts reach 30+ pages |
| Export Usage | 40% of Pro users export at least once per month |
| Avg Session Length | ≥ 45 minutes |

### v2 Targets (Phase 3, end of Month 6)

| Metric | Target |
|---|---|
| "Listen the Script" avg plays per post | ≥ 250 plays within 7 days of publish |
| Review turnaround SLA hit rate | ≥ 85% (completed within 14 days) |
| Producer contact requests per week | ≥ 50 |
| Published scripts | 500+ on the platform |
| Feed DAU | 5,000 (including non-writing visitors) |

### v3 Targets (Phase 4, end of Month 9)

| Metric | Target |
|---|---|
| Writers' Room sessions per day | 30+ |
| MRR growth rate | 15% month-over-month |
| Churn rate | < 5% monthly for Pro tier |
| Studio team accounts | 20+ |
| Review completions per month | 200+ |
