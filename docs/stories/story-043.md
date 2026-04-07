# story-043 — Social Feed & Discovery

| Field | Value |
|---|---|
| **Story ID** | story-043 |
| **Title** | Social Feed & Discovery |
| **Epic** | E11 — Community |
| **Phase** | 3 |
| **Sprint** | Sprint 6 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Vikram (the producer) / Asha (the film student) |
| **Estimated Effort** | 4 days |
| **Dependencies** | story-040, story-041, story-042 |

---

## Problem Statement

A published script needs to reach an audience. The social feed is the discovery surface where the community browses scripts, likes them, bookmarks them for later, and engages with writers. It must be fast, relevant, and well-organized with tabs for "For You" (personalised), "Trending" (score-based), and "Following" (social graph).

---

## User Story

As a writer and community member, I want to browse a feed of published scripts — filtered by what's trending, what my followed writers are publishing, and what's relevant to me — so that I can discover great work and build an audience for my own scripts.

---

## Acceptance Criteria

**GIVEN** a user opens the Discover tab in LIPILY,  
**WHEN** the feed loads,  
**THEN** three tabs are shown: "For You", "Trending", "Following"; the default tab is "For You" for logged-in users, "Trending" for anonymous visitors.

**GIVEN** the "Trending" tab is selected,  
**WHEN** the feed loads,  
**THEN** scripts are sorted by `leaderboard_score DESC` (computed by the daily scheduler from story-044); each script card shows: cover image, title, writer name, genre badges, format badge, page count, like count, listen play count, and a ▶ button if a Listen is available.

**GIVEN** the "Following" tab is selected,  
**WHEN** the feed loads,  
**THEN** only scripts from writers the current user follows are shown, sorted by `published_at DESC`; if the user follows no one, an empty state shows "Follow writers to see their scripts here."

**GIVEN** the "For You" tab is selected,  
**WHEN** the feed loads,  
**THEN** scripts are ranked by a personalisation signal: genres the user has previously liked or bookmarked are weighted × 1.5; language preference (from profiles.ui_language) is weighted × 1.2; writers the user follows are weighted × 2.0; the ranked list is computed as a pre-built feed stored in `personalised_feeds` and refreshed every 4 hours by a Cloud Scheduler job.

**GIVEN** a user scrolls to the bottom of the feed,  
**WHEN** the last card is within 200px of the viewport bottom,  
**THEN** the next page is fetched using cursor-based pagination (cursor = `published_at::TEXT || '_' || id`); 20 scripts per page; a loading skeleton shows during fetch.

**GIVEN** a user taps the ♥ (Like) button on a script card,  
**WHEN** the tap registers,  
**THEN** `script_likes` row is inserted; the like count on the card increments optimistically; the like button fills with colour; `script_publications.like_count` is incremented via a DB trigger; if already liked, tapping again removes the like.

**GIVEN** a user taps the 🔖 (Bookmark) button,  
**WHEN** the tap registers,  
**THEN** `script_bookmarks` row is inserted; the script appears in the user's "Saved" list (accessible from the profile page); `script_publications.bookmark_count` increments; second tap removes the bookmark.

**GIVEN** a user applies a filter (Genre or Format),  
**WHEN** the filter chip is selected from the filter tray (accessible via filter icon),  
**THEN** the feed is re-fetched with the genre/format filter applied; active filters are shown as dismissible chips above the feed; filters persist for the session.

**GIVEN** a script has `content_rating = 'mature'`,  
**WHEN** the script appears in the feed,  
**THEN** the cover image is blurred with a "Mature Content — Tap to reveal" label; the blur is removed after the user taps; users under 18 (inferred from DOB in profile, if set) never see mature content unblurred.

---

## Performance Criteria

- Feed initial load: < 1.5 seconds (first 20 cards)
- Pagination next-page fetch: < 500ms
- Like/bookmark tap → optimistic UI update: < 100ms
- Trending feed: cached in Redis with 5-minute TTL; database query runs in background
- For You feed: pre-computed every 4 hours; served from `personalised_feeds` cache table

---

## Offline Behaviour

- Feed is not available offline (it is a network-dependent feature)
- Previously loaded feed cards are cached in Drift for 30 minutes; shown with a "Cached" label when offline

---

## Mobile Behaviour (375px)

- Feed cards: full-width cards (not grid)
- Cover image: 16:9 thumbnail top of card
- Like/bookmark buttons: 44dp tap targets
- Filter tray: bottom sheet with scrollable genre chips
- Tab bar: sticky at top below header

---

## Error States

**E1 — Empty Feed (no published scripts in system)**  
Shows illustration + "The community is just getting started. Be the first to publish your script!" with a CTA button.

**E2 — Following Tab Empty**  
"You're not following anyone yet. Discover writers to follow." with link to Trending tab.

**E3 — Feed Fetch Failure**  
Toast: "Could not load feed. Showing cached scripts." Retry button in feed header.

**E4 — Like/Bookmark Failure**  
If the server request fails, the optimistic UI update is rolled back; toast: "Could not save. Please try again."

---

## Security Considerations

- Feed query enforces `is_published = true` and `visibility = 'public'` (server-side filter, not client)
- Anonymous users can view the feed but cannot like, bookmark, or comment
- Rate limit: like 60 times per minute per user (anti-spam)
- Report button on every script card (story-050 integration)

---

## Human Authorship Considerations

- No AI involvement in feed curation algorithm (it is a deterministic scoring function)
- AI badge (✦) count shown on script cards to indicate degree of AI assistance

---

## DB Tables Touched

- `script_likes` (INSERT/DELETE)
- `script_bookmarks` (INSERT/DELETE)
- `script_publications` (UPDATE like_count, bookmark_count via DB trigger)
- `personalised_feeds` (pre-computed cache, refreshed by scheduler)

```sql
CREATE TABLE script_likes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  publication_id UUID NOT NULL REFERENCES script_publications(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(user_id, publication_id)
);

CREATE TABLE script_bookmarks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  publication_id UUID NOT NULL REFERENCES script_publications(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(user_id, publication_id)
);
```

---

## API Endpoints Used

- `GET /feed?tab={for_you|trending|following}&cursor=&genre=&format=` — paginated feed
- `POST /likes` — like a publication
- `DELETE /likes/:publication_id` — unlike
- `POST /bookmarks` — bookmark
- `DELETE /bookmarks/:publication_id` — remove bookmark
- `GET /bookmarks` — list user's bookmarks

---

## Test Cases

**TC-001:** Trending tab loads 20 scripts sorted by leaderboard_score DESC.  
**TC-002:** Following tab shows scripts from followed writers only; empty state for unfollowed users.  
**TC-003:** Like script → `script_likes` row created; count increments; like again → row deleted; count decrements.  
**TC-004:** Bookmark script → appears in user's Saved list.  
**TC-005:** Scroll to bottom → next 20 scripts load with cursor pagination.  
**TC-006:** Filter by Genre "Thriller" → only Thriller scripts shown; filter chip visible.  
**TC-007:** Mature content script → cover blurred; tap reveals.  
**TC-008:** Anonymous user taps Like → redirected to login page.

---

## Out of Scope

- Algorithmic A/B testing of feed ranking (Phase 4)
- Script comments (as distinct from editor comments in story-011) — Phase 4
- Script sharing to external social media — Phase 4
