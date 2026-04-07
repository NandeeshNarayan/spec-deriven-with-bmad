# LIPILY — Complete API Contract

> **Purpose:** Single source of truth for every REST endpoint and WebSocket message.  
> **Agent rule:** When implementing a feature, map your work to an endpoint here. Never invent endpoint paths.  
> **Base URL (Rust):** `https://api.lipily.com` (Cloud Run)  
> **Base URL (MCP):** `https://mcp.lipily.com` (Cloud Run, internal only)  
> **Auth:** All endpoints require `Authorization: Bearer {supabase_jwt}` unless marked `[PUBLIC]` or `[INTERNAL]`.  
> **Internal endpoints:** require `X-Internal-Key: {SERVICE_API_KEY}` header (Rust ↔ MCP only).  
> **Last updated:** 2026-04-07

---

## Auth & Profiles

| Method | Path | Description | Auth |
|---|---|---|---|
| GET | `/profiles/me` | Current user's profile | JWT |
| GET | `/profiles/:username` | Public profile by username | [PUBLIC] |
| PATCH | `/profiles/:id` | Update display_name, bio, writer_type, etc. | JWT own |
| PATCH | `/profiles/:id/username` | Change username (2/year limit enforced) | JWT own |
| POST | `/profiles/:id/avatar` | Upload profile photo (multipart) | JWT own |
| GET | `/accessibility-preferences` | Get accessibility prefs | JWT |
| PATCH | `/accessibility-preferences` | Update accessibility prefs | JWT |
| GET | `/notification-preferences` | Get notification prefs | JWT |
| PATCH | `/notification-preferences` | Update notification prefs | JWT |

---

## Scripts

| Method | Path | Description | Auth |
|---|---|---|---|
| GET | `/scripts` | List user's scripts (with pagination) | JWT |
| POST | `/scripts` | Create new script | JWT |
| GET | `/scripts/:id` | Get script with blocks | JWT (owner/collaborator) |
| PATCH | `/scripts/:id` | Update script metadata | JWT (owner) |
| DELETE | `/scripts/:id` | Soft delete script | JWT (owner) |
| POST | `/scripts/:id/restore` | Restore soft-deleted script | JWT (owner) |
| GET | `/scripts/deleted` | List soft-deleted scripts | JWT |
| PATCH | `/scripts/:id/writers-room` | Enable/disable Writers' Room | JWT (owner, Studio) |

---

## Blocks

| Method | Path | Description | Auth |
|---|---|---|---|
| GET | `/scripts/:id/blocks` | Get all blocks in position order | JWT |
| POST | `/scripts/:id/blocks` | Create block | JWT (editor+) |
| PATCH | `/blocks/:id` | Update block content/type | JWT (editor+) |
| DELETE | `/blocks/:id` | Soft delete block | JWT (editor+) |
| PATCH | `/blocks/:id/lock` | Lock block for editing | JWT (editor+) |
| DELETE | `/blocks/:id/lock` | Unlock block | JWT (editor+) |

---

## Collaboration

| Method | Path | Description | Auth |
|---|---|---|---|
| GET | `/scripts/:id/collaborators` | List collaborators | JWT (owner/collaborator) |
| POST | `/scripts/:id/invites` | Send invite email | JWT (owner) |
| GET | `/invites/:token` | Accept invite (returns script details) | [PUBLIC] |
| POST | `/invites/:token/accept` | Join script via invite token | JWT |
| PATCH | `/collaborators/:id` | Change collaborator role | JWT (owner) |
| DELETE | `/collaborators/:id` | Remove collaborator | JWT (owner) |

---

## Comments

| Method | Path | Description | Auth |
|---|---|---|---|
| GET | `/scripts/:id/comments` | List comments (with threads) | JWT |
| POST | `/scripts/:id/comments` | Create comment | JWT (commenter+) |
| PATCH | `/comments/:id` | Edit comment content | JWT (own) |
| DELETE | `/comments/:id` | Delete comment | JWT (own or owner) |
| PATCH | `/comments/:id/resolve` | Resolve comment | JWT (owner/editor) |
| DELETE | `/comments/:id/resolve` | Reopen comment | JWT (owner/editor) |

---

## Drafts & Versioning

| Method | Path | Description | Auth |
|---|---|---|---|
| GET | `/scripts/:id/drafts` | List drafts | JWT (owner) |
| POST | `/scripts/:id/drafts` | Save named draft | JWT (owner/editor) |
| GET | `/drafts/:id` | Get draft snapshot | JWT (owner) |
| POST | `/drafts/:id/restore` | Restore draft (auto-saves before restore) | JWT (owner) |
| DELETE | `/drafts/:id` | Delete draft | JWT (owner) |

---

## Branches & Merge Requests

| Method | Path | Description | Auth |
|---|---|---|---|
| GET | `/scripts/:id/branches` | List branches | JWT (owner, Studio) |
| POST | `/scripts/:id/branches` | Create branch | JWT (editor+, Studio) |
| GET | `/branches/:id` | Get branch content | JWT |
| POST | `/branches/:id/merge-request` | Open merge request | JWT (branch creator) |
| PATCH | `/branches/:id/merge-request/accept` | Accept MR | JWT (owner) |
| PATCH | `/branches/:id/merge-request/reject` | Reject MR | JWT (owner) |

---

## Scene Intelligence & Story Bible

| Method | Path | Description | Auth |
|---|---|---|---|
| GET | `/scripts/:id/scene-intelligence` | Get all scene intelligence | JWT |
| GET | `/blocks/:id/scene-intelligence` | Get intelligence for one block | JWT |
| PATCH | `/scene-intelligence/:id/approve` | Approve AI analysis | JWT (owner/editor) |
| GET | `/scripts/:id/story-bible` | Get story bible entries | JWT |
| POST | `/scripts/:id/story-bible` | Create entry | JWT (editor+) |
| PATCH | `/story-bible/:id` | Update entry | JWT (editor+) |
| DELETE | `/story-bible/:id` | Delete entry | JWT (owner/editor) |

---

## AI Features

| Method | Path | Description | Auth |
|---|---|---|---|
| POST | `/ai/scene-intelligence` | Trigger Scene Intelligence | JWT (Pro+) |
| POST | `/ai/dialogue-suggest` | AI Dialogue Suggest | JWT (Pro+) |
| POST | `/ai/continue-scene` | AI Continue Scene | JWT (Pro+) |
| POST | `/ai/continuity-check` | Run continuity check | JWT (Pro+) |
| POST | `/ai/pacing-analysis` | Run pacing analysis | JWT (Pro+) |
| POST | `/ai/logline-pitch` | Generate logline + pitch | JWT (Pro+) |
| POST | `/ai/character-voice-check` | Character voice guardian | JWT (Pro+) |
| POST | `/tts/read-aloud` | Dialogue Read-Aloud TTS | JWT (Pro+) |
| GET | `/ai/usage/today` | AI requests used today vs limit | JWT |

---

## Export & Import

| Method | Path | Description | Auth |
|---|---|---|---|
| POST | `/scripts/:id/export/pdf` | Generate production PDF | JWT (Pro+) |
| GET | `/scripts/:id/export/fountain` | Export as .fountain | JWT |
| POST | `/scripts/:id/export/fdx` | Export as .fdx | JWT (Pro+) |
| POST | `/scripts/:id/import` | Import .fountain / .fdx / .txt | JWT |

---

## Search

| Method | Path | Description | Auth |
|---|---|---|---|
| GET | `/search?q=&scope=&script_id=` | Global search | JWT |

---

## Publishing & Social

| Method | Path | Description | Auth |
|---|---|---|---|
| POST | `/scripts/:id/publish` | Publish script | JWT (owner) |
| DELETE | `/scripts/:id/publish` | Unpublish script | JWT (owner) |
| POST | `/scripts/:id/cover` | Upload cover image | JWT (owner) |
| GET | `/publications` | Feed (For You / Trending / Following) | JWT / [PUBLIC] |
| GET | `/publications/:id` | Script detail page | [PUBLIC] |
| GET | `/feed?tab=&genre=&format=&cursor=` | Paginated feed | JWT / [PUBLIC] |
| POST | `/likes` | Like a publication | JWT |
| DELETE | `/likes/:publication_id` | Unlike | JWT |
| POST | `/bookmarks` | Bookmark | JWT |
| DELETE | `/bookmarks/:publication_id` | Remove bookmark | JWT |
| GET | `/bookmarks` | List bookmarks | JWT |

---

## Leaderboard

| Method | Path | Description | Auth |
|---|---|---|---|
| GET | `/leaderboard?type=weekly\|all_time` | Top 50 scripts | [PUBLIC] |
| GET | `/leaderboard/my-rank` | Current user's best rank | JWT |

---

## Listen the Script

| Method | Path | Description | Auth |
|---|---|---|---|
| POST | `/listen-the-script` | Enqueue Listen pipeline | JWT (owner) |
| GET | `/listen/:id` | Get production status + manifest | JWT / [PUBLIC] |
| DELETE | `/listen/:id` | Delete Listen production | JWT (owner) |
| POST | `/internal/listen-pipeline-step` | MCP callback (step complete) | [INTERNAL] |

---

## Follows

| Method | Path | Description | Auth |
|---|---|---|---|
| POST | `/follows` | Follow a user | JWT |
| DELETE | `/follows/:following_id` | Unfollow | JWT |
| GET | `/follows/followers` | List my followers | JWT |
| GET | `/follows/following` | List who I follow | JWT |

---

## Sharing

| Method | Path | Description | Auth |
|---|---|---|---|
| POST | `/scripts/:id/share-link` | Create/get share link | JWT (owner) |
| DELETE | `/scripts/:id/share-link` | Disable/reset share link | JWT (owner) |
| GET | `/shared/:token` | Read-only shared script | [PUBLIC] |

---

## Subscriptions (Stripe)

| Method | Path | Description | Auth |
|---|---|---|---|
| POST | `/subscriptions/checkout` | Create Stripe Checkout Session | JWT |
| POST | `/subscriptions/webhook` | Stripe webhook handler | [PUBLIC, Stripe-Signature] |
| GET | `/subscriptions/me` | Get current subscription | JWT |
| POST | `/subscriptions/portal` | Create Stripe Customer Portal session | JWT |

---

## Notifications

| Method | Path | Description | Auth |
|---|---|---|---|
| GET | `/notifications?limit=50&cursor=` | List notifications | JWT |
| PATCH | `/notifications/read-all` | Mark all as read | JWT |
| PATCH | `/notifications/:id/read` | Mark single as read | JWT |
| POST | `/internal/notify` | Internal: create notification | [INTERNAL] |

---

## Review Marketplace

| Method | Path | Description | Auth |
|---|---|---|---|
| GET | `/reviewers` | List marketplace reviewers | JWT |
| POST | `/reviewer-applications` | Apply to reviewer program | JWT |
| PATCH | `/reviewer-profiles/:id/pricing` | Update review price | JWT (own reviewer) |
| GET | `/reviewer-profiles/me` | Reviewer dashboard data | JWT |
| POST | `/reviews` | Request a review (triggers Stripe) | JWT |
| PATCH | `/reviews/:id/accept` | Reviewer accepts | JWT (reviewer) |
| PATCH | `/reviews/:id/decline` | Reviewer declines (triggers refund) | JWT (reviewer) |
| PATCH | `/reviews/:id/submit` | Reviewer submits review | JWT (reviewer) |
| GET | `/reviews?script_id=` | Writer's reviews | JWT |

---

## Producer Portal

| Method | Path | Description | Auth |
|---|---|---|---|
| POST | `/producer-applications` | Apply for producer access | JWT |
| GET | `/producer/scripts?genre=&format=&min_score=&has_listen=` | Discovery | JWT (approved producer) |
| POST | `/producer/saved-scripts` | Save to reading list | JWT (producer) |
| PATCH | `/producer/saved-scripts/:id` | Update notes | JWT (producer) |
| POST | `/producer/contact-requests` | Contact a writer | JWT (producer) |
| GET | `/producer/contact-requests/sent` | Sent requests | JWT (producer) |
| GET | `/producer/contact-requests/received` | Received requests | JWT (writer) |

---

## Content Moderation

| Method | Path | Description | Auth |
|---|---|---|---|
| POST | `/reports` | Submit a content report | JWT |

---

## Writers' Room

| Method | Path | Description | Auth |
|---|---|---|---|
| GET | `/scripts/:id/writers-room/dashboard` | Dashboard live data | JWT (showrunner) |
| POST | `/scripts/:id/writers-room/lock` | Lock script | JWT (showrunner) |
| DELETE | `/scripts/:id/writers-room/lock` | Unlock script | JWT (showrunner) |
| POST | `/scripts/:id/writers-room/mute/:user_id` | Mute writer | JWT (showrunner) |
| DELETE | `/scripts/:id/writers-room/mute/:user_id` | Unmute writer | JWT (showrunner) |
| POST | `/scripts/:id/writers-room/kick/:user_id` | Kick from session | JWT (showrunner) |

---

## Admin

All admin endpoints require JWT with `is_admin = true` claim.

| Method | Path | Description |
|---|---|---|
| GET | `/admin/overview` | Platform metrics |
| GET | `/admin/users?q=` | Search users |
| PATCH | `/admin/users/:id` | Update user (suspend, ban, tier) |
| POST | `/admin/users/:id/impersonate` | Start impersonation session |
| GET | `/admin/reports?status=pending` | Report queue |
| PATCH | `/admin/reports/:id/action` | Action a report |
| GET | `/admin/applications?type=&status=` | Applications queue |
| PATCH | `/admin/applications/:id/approve` | Approve application |
| PATCH | `/admin/applications/:id/reject` | Reject application |
| GET | `/admin/financials` | MRR + payout errors |
| GET | `/admin/audit-log?admin_id=&action=&from=&to=&limit=&cursor=` | Audit log |

---

## WebSocket (Yjs Collaboration)

**Endpoint:** `wss://api.lipily.com/ws/scripts/:script_id`  
**Auth:** `?token={supabase_jwt}` query param

### Client → Server messages

```typescript
// Join awareness
{ type: 'presence_join', user_id: string, display_name: string, colour: string }

// Update cursor position
{ type: 'cursor_update', block_id: string | null, block_type: string | null }

// Yjs update (binary encoded)
{ type: 'yjs_update', update: Uint8Array }

// Heartbeat
{ type: 'ping' }
```

### Server → Client messages

```typescript
// Full Yjs state sync on join
{ type: 'yjs_state', state: Uint8Array }

// Broadcast Yjs update from another client
{ type: 'yjs_update', from: string, update: Uint8Array }

// Presence list
{ type: 'presence_list', users: Array<{ user_id, display_name, colour, cursor_block_id }> }

// User joined/left
{ type: 'presence_join', user: { user_id, display_name, colour } }
{ type: 'presence_leave', user_id: string }

// Writers' Room: script locked/unlocked
{ type: 'script_locked', locked_by: string }
{ type: 'script_unlocked' }

// Writers' Room: muted
{ type: 'user_muted', user_id: string }
{ type: 'user_unmuted', user_id: string }

// Heartbeat response
{ type: 'pong' }
```

---

## Standard Error Response Shape

```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "Script with id abc123 not found.",
    "request_id": "req_abc123"
  }
}
```

### Error Codes

| Code | HTTP Status | Meaning |
|---|---|---|
| `UNAUTHORIZED` | 401 | Missing or invalid JWT |
| `FORBIDDEN` | 403 | Valid JWT but insufficient permissions |
| `RESOURCE_NOT_FOUND` | 404 | Entity does not exist or access denied |
| `VALIDATION_ERROR` | 422 | Request body failed validation |
| `RATE_LIMITED` | 429 | Too many requests |
| `TIER_REQUIRED` | 403 | Feature requires Pro/Studio tier |
| `AI_LIMIT_EXCEEDED` | 429 | Daily AI request limit reached |
| `INTERNAL_ERROR` | 500 | Unexpected server error |

---

## Rate Limits (Rust middleware)

| Route pattern | Limit |
|---|---|
| `/ai/**` | 60 req/min per user |
| `/search` | 60 req/min per user |
| `/profiles/:id/avatar` | 10 req/min per user |
| `/likes` | 60 req/min per user |
| `/producer/contact-requests` | 10 per day per producer |
| `/reports` | 10 req/hour per user |
| All others | 600 req/min per user |
