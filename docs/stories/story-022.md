# story-022 — Read-Only Sharing Link

| Field | Value |
|---|---|
| **Story ID** | story-022 |
| **Title** | Read-Only Sharing Link |
| **Epic** | E09 — Editor Utilities |
| **Phase** | 2 |
| **Sprint** | Sprint 3 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Aisha (the independent short-film writer) |
| **Estimated Effort** | 2 days |
| **Dependencies** | story-004 |

---

## Problem Statement

Writers frequently need to share scripts with people who don't have LIPILY accounts — casting directors, directors, actors reading for roles, workshop participants. A read-only sharing link lets the writer send a URL that anyone can open in a browser and read the formatted script without sign-up. The writer controls whether the link is active.

---

## User Story

As a writer, I want to generate a shareable read-only link to my script — so that anyone can read it in a browser without needing a LIPILY account.

---

## Acceptance Criteria

**GIVEN** the writer is in the editor and opens "Share" (Cmd+Shift+S),  
**WHEN** the Share modal opens,  
**THEN** there is a "Read-Only Link" section showing a toggle (disabled by default), a text field (blank when disabled), and a "Copy Link" button.

**GIVEN** the writer enables the "Read-Only Link" toggle,  
**WHEN** `POST /scripts/:id/share-link` is called,  
**THEN** a row is inserted into `script_publications` (or a dedicated `share_links` table with `token`, `script_id`, `created_by`, `is_active = true`, `expires_at` (optional)); a unique URL is generated: `https://app.lipily.io/read/:token`; the URL is displayed in the text field and the "Copy Link" button copies it to the clipboard.

**GIVEN** a non-authenticated user opens the share link URL in a browser,  
**WHEN** the page loads at `/read/:token`,  
**THEN** the script renders in a read-only view with WGA-standard formatting (same CSS/layout as the PDF export but rendered in-browser); no account creation is required; no editor toolbars are shown; the page title is "[Script Title] — LIPILY"; there is a small "Write your own on LIPILY" CTA in the footer (not obtrusive).

**GIVEN** the read-only view is rendered,  
**WHEN** the viewer scrolls through it,  
**THEN** all block types are formatted correctly: Scene Headings are bold ALL CAPS, Character names are centred, Dialogue is indented — all in Courier Prime 12pt on a white page background.

**GIVEN** the writer disables the "Read-Only Link" toggle,  
**WHEN** `DELETE /scripts/:id/share-link` is called,  
**THEN** the share link is deactivated (`is_active = false`); any subsequent attempt to open the link shows "This script is no longer available. The link has been disabled by the author."

**GIVEN** the writer clicks "Reset Link",  
**WHEN** the reset action fires,  
**THEN** the old token is deactivated; a new token is generated; the new URL is shown in the text field; old links sharing the previous URL immediately stop working.

**GIVEN** a shared link has an optional expiry date set by the writer,  
**WHEN** the expiry date passes,  
**THEN** the link auto-deactivates; visitors see the "no longer available" message.

**GIVEN** the script has AI-generated content (blocks with `content_source = 'ai_generated'`),  
**WHEN** the read-only view renders,  
**THEN** AI-generated blocks are shown without the ✦ badge (the public view is the clean submission copy); no AI metadata is exposed to the public viewer.

**GIVEN** a Free-tier user tries to create a sharing link,  
**WHEN** the toggle is enabled,  
**THEN** the link is created successfully — sharing links are available on all plans.

**GIVEN** the share link page is crawled by a search engine bot,  
**WHEN** the bot requests the page,  
**THEN** the server returns `<meta name="robots" content="noindex, nofollow">` to prevent the script from being indexed; the canonical URL is the share link itself.

---

## Performance Criteria

- Share link generation: < 300ms
- Read-only page first render (no auth, cold): < 2 seconds
- Page renders correctly on all major browsers (Chrome, Safari, Firefox, Edge)
- Mobile read-only view: readable on 375px without horizontal scroll

---

## Offline Behaviour

- Generating share links requires internet
- Previously generated links continue to work (they are served from the LIPILY backend, not the client)

---

## Mobile Behaviour (375px)

- Share modal: "Read-Only Link" section at bottom of the Share bottom sheet
- Toggle + URL field + "Copy" button in a single row
- After copy, button changes to "Copied ✓" for 2 seconds

---

## Error States

**E1 — Token Collision**  
If the generated share token already exists (astronomically rare with UUID v4): generate a new one; log to Sentry.

**E2 — Invalid or Revoked Token**  
Link is invalid/revoked: show full-screen message "This script is no longer available." with LIPILY branding and a "Discover other scripts" CTA linking to the public feed.

**E3 — Script Deleted While Link Active**  
If the script is soft-deleted while the share link is active: the link immediately returns "Script no longer available"; no content is shown.

---

## Security Considerations

- Share tokens are cryptographically random UUID v4 (no sequential IDs)
- Share link pages do not expose the script UUID, user ID, or any internal identifiers
- Only the active variant of blocks is shown (inactive variants not accessible)
- The route `/read/:token` is served by the Rust API which validates the token and returns the read-only script data
- AI-generated content is shown as plain text in the read-only view — no internal `content_source` metadata is exposed in the response

---

## Human Authorship Considerations

Not applicable — the sharing link exposes the writer's completed, formatted script with no AI metadata visible.

---

## DB Tables Touched

- `share_links` (INSERT on enable, UPDATE `is_active = false` on disable)
- `blocks` (SELECT — for read-only rendering)
- `scenes` (SELECT)
- `scripts` (SELECT — for title and metadata)

---

## API Endpoints Used

- `POST /scripts/:id/share-link` — create/enable share link
- `DELETE /scripts/:id/share-link` — disable share link
- `POST /scripts/:id/share-link/reset` — regenerate token
- `GET /read/:token` — public read-only view (no auth required)

---

## Test Cases

**TC-001:** Enable share link → assert URL appears; copy to clipboard works.  
**TC-002:** Open share link in incognito browser → assert script renders without auth.  
**TC-003:** Disable share link → open old URL → assert "no longer available" message.  
**TC-004:** Reset link → assert old URL stops working; new URL works.  
**TC-005:** Share link with AI content → assert ✦ badges NOT shown in public view.  
**TC-006:** Share link page → assert `noindex, nofollow` meta tag present.  
**TC-007:** Script deleted while link active → assert link returns "no longer available".  
**TC-008:** Free user enables share link → assert link created without paywall.

---

## Out of Scope

- Password-protected share links (Phase 2)
- Share links with expiry date UI (dates are technically supported in schema; the UI picker is Phase 2)
- Download PDF from share link (Phase 2)
- Analytics on share link views (Phase 2)
