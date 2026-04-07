# story-011 — Comment Threads

| Field | Value |
|---|---|
| **Story ID** | story-011 |
| **Title** | Comment Threads |
| **Epic** | E04 — Collaboration & CRDT |
| **Phase** | 2 |
| **Sprint** | Sprint 2 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Priya (the TV staff writer) |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-010 |

---

## Problem Statement

Script development requires structured feedback — showrunners leave notes on specific lines, producers request rewrites on scenes, writing partners discuss character motivations inline. Inline comment threads allow all this feedback to be attached directly to the relevant text, keeping notes in context and ensuring no note is missed.

---

## User Story

As a collaborator, I want to add comment threads to specific blocks or text selections so that feedback is attached directly to the relevant part of the script, with @mentions to notify specific team members.

---

## Acceptance Criteria

**GIVEN** a user selects text within any block (or right-clicks a block),  
**WHEN** they click the "Comment" icon in the selection toolbar,  
**THEN** a comment input panel slides in from the right (desktop) or opens as a bottom sheet (mobile); the selected text is highlighted in amber; a new `comments` row is initialised (not yet saved).

**GIVEN** the user types a comment (minimum 1 character) and clicks "Submit",  
**WHEN** the comment is submitted via `POST /comments`,  
**THEN** the comment is saved with `parent_id = null` (thread root), `block_id` set to the target block, `text_anchor_start` and `text_anchor_end` set to the selection range; an amber underline appears under the selected text in the editor; the comment count indicator (+1 💬 icon) appears in the block's gutter.

**GIVEN** a user mentions another collaborator with `@username` in the comment text,  
**WHEN** they type `@` followed by at least 2 characters,  
**THEN** an autocomplete dropdown appears showing matching collaborator names; selecting one inserts `@username` as a mention token; on comment submit, a notification is dispatched to the mentioned user.

**GIVEN** a comment thread exists on a block,  
**WHEN** another user clicks the 💬 icon on that block,  
**THEN** the Comments panel opens (right sidebar on desktop, bottom sheet on mobile) and scrolls to the relevant thread; all replies in the thread are shown in chronological order.

**GIVEN** a user wants to reply to an existing comment,  
**WHEN** they click "Reply" under a comment,  
**THEN** a reply input field appears indented below the comment; their reply is submitted via `POST /comments` with `parent_id` set to the root comment's ID; the reply appears in the thread.

**GIVEN** the comment author or script owner selects "Resolve" on a thread,  
**WHEN** the resolve action fires,  
**THEN** `PATCH /comments/:id` sets `resolved = true` and `resolved_at = NOW()`; the amber underline and 💬 icon disappear from the editor; the thread moves to a "Resolved" section at the bottom of the comments panel (collapsed by default); the mentioned users receive a notification.

**GIVEN** a viewer-role collaborator is in the editor,  
**WHEN** they try to add a comment,  
**THEN** the "Comment" icon in the selection toolbar is disabled with tooltip "You have view-only access."; they can read existing comments.

**GIVEN** a commenter-role collaborator is in the editor,  
**WHEN** they try to edit another user's comment,  
**THEN** the edit button is not shown on comments they did not author; they can only edit their own comments.

**GIVEN** there are more than 50 unresolved comments on a script,  
**WHEN** the comments panel opens,  
**THEN** comments are virtualised (ListView.builder); the panel shows a count badge "50 unresolved"; there is a filter bar: All / Unresolved / Resolved / @Mentions.

**GIVEN** a comment's anchor text (the highlighted selection) is deleted from the editor,  
**WHEN** the deletion is committed,  
**THEN** the comment's `text_anchor_start` and `text_anchor_end` are set to `null`; the comment persists but is shown as "The referenced text was deleted" in the thread; the 💬 icon moves to the line where the deleted text was.

---

## Performance Criteria

- Comment panel open: < 200ms
- Comment submit (API + optimistic UI): < 500ms round-trip
- Comment thread load (up to 50 threads): < 300ms
- @mention autocomplete: < 100ms (collaborative profile list is cached)
- Amber underline rendering for 50 concurrent comments: no jank (60fps maintained)

---

## Offline Behaviour

- New comments are queued in `OfflineWriteQueue` when offline
- Existing comments are readable from Drift cache
- A subtle badge on the 💬 icon indicates "1 comment pending sync" for queued offline comments
- @mention notifications are dispatched when the comment syncs on reconnect

---

## Mobile Behaviour (375px)

- Comment is triggered by long-pressing any block → "Comment" appears in the pop-up menu
- Comment panel opens as a bottom sheet (80% height)
- Each comment thread: avatar + name + text + timestamp + "Reply" and "Resolve" buttons
- Reply input: fixed at bottom of sheet above keyboard

---

## Error States

**E1 — Comment Submit Failure**  
`POST /comments` returns 5xx: comment is queued in `OfflineWriteQueue`; the amber highlight persists locally; a "pending" spinner appears on the comment in the panel.

**E2 — Mentioned User Removed**  
If an `@mentioned` user's collaborator access is revoked before their notification is delivered: the notification is dropped silently; no error.

**E3 — Resolved Comment Reopened**  
Owner can click "Reopen" on a resolved comment thread: `PATCH /comments/:id` sets `resolved = false`; the comment moves back to unresolved; amber underline reappears.

---

## Security Considerations

- `comments` RLS: SELECT policy — user is owner or collaborator; INSERT policy — commenter/editor/owner role; UPDATE — own comment only; DELETE — own comment or owner
- Comment text is stored as plain text; markdown is rendered safely (no HTML injection)
- @mention notifications use `user_id` from `collaborators` table — only accepted collaborators can be mentioned

---

## Human Authorship Considerations

Not applicable — comments are always human-written.

---

## DB Tables Touched

- `comments` (SELECT, INSERT, UPDATE resolved/text, DELETE)
- `profiles` (SELECT for @mention autocomplete)
- `notifications` (INSERT — for @mentions and resolve events)

---

## API Endpoints Used

- `GET /scripts/:id/comments` — list all comment threads
- `POST /comments` — add comment or reply
- `PATCH /comments/:id` — resolve/reopen/edit
- `DELETE /comments/:id` — delete comment

---

## Test Cases

**TC-001:** Select text → click Comment → submit → assert amber underline + 💬 icon appear on block.  
**TC-002:** Reply to comment → assert reply appears indented under root comment.  
**TC-003:** Type `@P` in comment → assert autocomplete dropdown with matching collaborators appears.  
**TC-004:** Resolve thread → assert amber underline disappears; thread moves to resolved section.  
**TC-005:** Viewer tries to comment → assert Comment button disabled.  
**TC-006:** Delete anchored text → assert comment persists with "referenced text deleted" message.  
**TC-007:** Open panel offline → assert cached comments shown; new comment queued.  
**TC-008:** 50+ comments → assert list is virtualised and filter bar is shown.

---

## Out of Scope

- AI-generated comment suggestions (future feature)
- Comment export to PDF (future feature)
- Emoji reactions on comments (Phase 2)
