# story-041 — Script Publishing

| Field | Value |
|---|---|
| **Story ID** | story-041 |
| **Title** | Script Publishing |
| **Epic** | E10 — Social Platform |
| **Phase** | 3 |
| **Sprint** | Sprint 5 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Priya (the professional TV writer) |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-019, story-040 |

---

## Problem Statement

Writers need a clear, intentional workflow to publish their scripts to LIPILY's public community. Publishing opens the script to likes, reviews, producer discovery, and the leaderboard. The publish flow must be deliberate (not accidental), require a minimum level of script completeness, and give the writer control over metadata (genre, logline, content rating, visibility).

---

## User Story

As a screenwriter, I want to publish my completed script to the LIPILY community with genre tags, a logline, and a cover image — so that other writers, producers, and reviewers can discover and engage with my work.

---

## Acceptance Criteria

**GIVEN** a writer opens a script that has at least 15 pages and a title page,  
**WHEN** they click "Publish Script" (from the Dashboard script menu or the Editor header),  
**THEN** a multi-step Publish Wizard opens with 4 steps: (1) Script Details, (2) Visibility & Audience, (3) Cover Image, (4) Review & Confirm.

**GIVEN** the writer is on Step 1 of the Publish Wizard,  
**WHEN** they fill in the fields,  
**THEN** the required fields are: Logline (max 200 chars), Genre (multi-select, up to 3, from a fixed list: Drama / Thriller / Comedy / Horror / Sci-Fi / Action / Romance / Animation / Documentary / Other), Format (Feature / Pilot / Short / TV Episode / Web Series); optional: Page Count (auto-filled from script), Language (auto-filled from script language setting).

**GIVEN** the writer moves to Step 2,  
**WHEN** they set visibility,  
**THEN** the options are: Public (discoverable in feed and leaderboard), Unlisted (accessible via direct link only, not in feed), or Private (reverts to non-published); they also set a Content Rating (General / Teen / Mature).

**GIVEN** the writer moves to Step 3,  
**WHEN** they upload a cover image,  
**THEN** accepted formats are JPG/PNG/WebP, max 5MB; LIPILY generates a default cover if skipped (a styled card with script title and the writer's name on a gradient background); custom cover is stored at `s3://lipily-covers/{script_id}.webp` resized to 600×800px.

**GIVEN** the writer confirms on Step 4,  
**WHEN** they click "Publish",  
**THEN** `scripts.is_published = true`, `scripts.published_at = NOW()`, and `script_publications` row is inserted; the script now appears on the writer's public profile (story-040) and in the social feed (story-043).

**GIVEN** a published script is shown in the social feed,  
**WHEN** a reader clicks it,  
**THEN** a public Script Detail Page shows: cover image, title, writer's name (linked to profile), genre tags, format, page count, logline, like count, bookmark count, listen plays, and a "Read Sample" button (shows first 10 pages as read-only, using the same read-only view from story-022).

**GIVEN** a writer wants to unpublish a script,  
**WHEN** they click "Unpublish" from the Dashboard,  
**THEN** `scripts.is_published = false`; the script is removed from the feed within 5 seconds; the script detail page returns 404 for non-owners; any outstanding reviews in progress are notified via email that the script was unpublished.

**GIVEN** a writer updates the script content after publishing,  
**WHEN** they save a significant edit (page count changes by ≥ 5 or a title page change),  
**THEN** the existing `script_publications` record is updated with `updated_at = NOW()`; the feed card shows "Updated" badge for 48 hours; the writer is not required to re-publish.

**GIVEN** a Free tier writer attempts to publish more than 1 script,  
**WHEN** they click "Publish Script" on a second script,  
**THEN** a paywall is shown: "Free writers can publish 1 script. Upgrade to Pro to publish unlimited scripts."

---

## Performance Criteria

- Publish Wizard steps: each step navigates < 300ms
- Cover image upload and processing: < 5 seconds
- Feed appearance after publish: < 5 seconds (Supabase Realtime event triggers feed refresh)
- Script Detail Page (CDN-cached): < 1.5 seconds cold start

---

## Offline Behaviour

- The Publish Wizard is not available offline (it requires network)
- Previously published script metadata is cached in Drift for display in dashboard cards

---

## Mobile Behaviour (375px)

- Publish Wizard: one step per screen, swipe to go back is disabled (must use Back button to prevent accidental back)
- Cover image: shows a phone-optimised upload button with camera/gallery options
- Script Detail Page: single column; cover image is full-width at 2:3 ratio

---

## Error States

**E1 — Script Too Short to Publish**  
Publish button is disabled with tooltip: "Scripts must be at least 15 pages to publish. Your script has {n} pages."

**E2 — Missing Logline**  
Step 1 validation: "A logline is required to publish."

**E3 — Cover Upload Failure**  
Toast: "Cover image upload failed. Try a JPEG under 5MB." Option to skip and use generated cover.

**E4 — Publish Network Failure**  
Toast: "Publish failed. Your script has not been published. Please try again." Script remains private.

---

## Security Considerations

- Writers can only publish their own scripts (RLS on `scripts` and `script_publications`)
- Cover images are stored with script_id-based paths; no user-controlled paths
- Content Rating field is stored and enforced — Mature content is age-gated in the feed (story-043)
- AI badges (✦) on published scripts are visible to the public to maintain transparency

---

## Human Authorship Considerations

- No AI involvement in the publish workflow itself
- The published script's content_source fields are preserved; the public view displays AI badge markers on AI-generated content

---

## DB Tables Touched

- `scripts` (UPDATE `is_published`, `published_at`)
- `script_publications` (INSERT)

```sql
CREATE TABLE script_publications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  script_id UUID NOT NULL REFERENCES scripts(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES profiles(id),
  logline TEXT NOT NULL,
  genre TEXT[] NOT NULL,
  format TEXT NOT NULL,
  content_rating TEXT NOT NULL DEFAULT 'general',
  visibility TEXT NOT NULL DEFAULT 'public',
  cover_url TEXT,
  language TEXT NOT NULL DEFAULT 'en',
  like_count INT NOT NULL DEFAULT 0,
  bookmark_count INT NOT NULL DEFAULT 0,
  listen_play_count INT NOT NULL DEFAULT 0,
  review_count INT NOT NULL DEFAULT 0,
  published_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_script_publications_user ON script_publications(user_id);
CREATE INDEX idx_script_publications_published ON script_publications(published_at DESC);
```

---

## API Endpoints Used

- `POST /scripts/:id/publish` — trigger publish; body: `{ logline, genre, format, content_rating, visibility, language }`
- `DELETE /scripts/:id/publish` — unpublish
- `POST /scripts/:id/cover` — upload cover image (multipart)
- `GET /publications/:id` — public script detail
- `GET /publications` — feed/discovery (story-043)

---

## Test Cases

**TC-001:** Script with 14 pages → Publish button disabled; tooltip shown.  
**TC-002:** Script with 20 pages + title page → Publish Wizard opens.  
**TC-003:** Skip cover image → generated cover shown on script card.  
**TC-004:** Publish script → appears in writer's public profile within 5s.  
**TC-005:** Unpublish → script card disappears from feed within 5s; Script Detail returns 404 for non-owner.  
**TC-006:** Free writer publishing second script → paywall shown.  
**TC-007:** Update script content after publish → feed card shows "Updated" badge.  
**TC-008:** Set visibility to Unlisted → script NOT in discovery feed; direct link still works.

---

## Out of Scope

- Script monetisation / paid scripts (Phase 4)
- Collaborative publishing (co-authors listed) — Phase 3
- DMCA takedown from external parties (handled by story-050)
