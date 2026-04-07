# story-044 — Leaderboard

| Field | Value |
|---|---|
| **Story ID** | story-044 |
| **Title** | Leaderboard |
| **Epic** | E11 — Community |
| **Phase** | 3 |
| **Sprint** | Sprint 6 |
| **Priority** | P2 |
| **Status** | DRAFT |
| **Persona** | Asha (the film student) |
| **Estimated Effort** | 2 days |
| **Dependencies** | story-041, story-042, story-043 |

---

## Problem Statement

Writers are motivated by recognition and visibility. A leaderboard drives engagement by surfacing the best-performing scripts and writers, creates aspiration for newer writers, and gives producers a quick signal of what the community values. The scoring formula must be transparent and fair.

---

## User Story

As a writer, I want to see how my published scripts rank on the community leaderboard — so that I have a measurable goal to work toward and visibility to producers and collaborators.

---

## Acceptance Criteria

**GIVEN** a user navigates to the Discover tab,  
**WHEN** they select the "Leaderboard" sub-tab (or dedicated Leaderboard screen),  
**THEN** they see two tabs: "This Week" and "All Time"; each shows a ranked list of the top 50 scripts.

**GIVEN** the leaderboard is displayed,  
**WHEN** a script entry is shown,  
**THEN** each entry shows: rank (#1, #2, …), cover thumbnail, title, writer name (linked to profile), genre badges, composite score, and individual metric breakdown (👁 views, ♥ likes, ▶ listen plays, ⭐ reviews).

**GIVEN** the scoring formula is applied,  
**WHEN** the Cloud Scheduler runs the leaderboard job at 02:00 UTC daily,  
**THEN** the composite score is computed as:  
  `score = (view_count × 1) + (like_count × 2) + (listen_play_count × 3) + (review_count × 5)`  
  For "This Week" tab: only events from the last 7 days are counted.  
  For "All Time" tab: all historical events are counted.  
  The top 50 results are stored in `leaderboard_snapshots` with a `snapshot_type` of `weekly` or `all_time`.

**GIVEN** a writer views the leaderboard and their own script is ranked,  
**WHEN** the page loads,  
**THEN** their own script entry is highlighted with a gold border; if they are not in the top 50, a "Your Best Script" section is shown below the top 50 with their highest-ranked script and its rank (#73, for example).

**GIVEN** a user taps a script entry on the leaderboard,  
**WHEN** the tap registers,  
**THEN** they are taken to the Script Detail Page (story-041); the leaderboard view is preserved as a navigation stack item (back navigation returns to the leaderboard at the same scroll position).

**GIVEN** the leaderboard is viewed on its first load of the day,  
**WHEN** the data is served,  
**THEN** the data is served from the latest `leaderboard_snapshots` record (max 24-hour staleness); a "Last updated: {time}" label is shown on the page.

**GIVEN** a new writer has no published scripts,  
**WHEN** they view the leaderboard,  
**THEN** an empty state banner encourages them: "Publish your script and start climbing the leaderboard!"

---

## Performance Criteria

- Leaderboard page load: < 500ms (served from pre-computed snapshot)
- Scheduler job: must complete leaderboard computation in < 60 seconds
- Snapshot query: the `leaderboard_snapshots` table is queried by a simple `SELECT WHERE snapshot_type = 'weekly' ORDER BY rank LIMIT 50` — no complex real-time computation at read time

---

## Offline Behaviour

- The leaderboard is cached in Drift for 1 hour; shown with "Cached" label if offline

---

## Mobile Behaviour (375px)

- Leaderboard entries: compact list view (cover thumbnail 48×64dp, rank badge, title/writer, score)
- Metric breakdown: shown in a collapsed row, tap to expand
- "Your Best Script" section: pinned to bottom of screen for easy access

---

## Error States

**E1 — Scheduler Failed (no snapshot for today)**  
Show the most recent available snapshot with a notice: "Leaderboard last updated {date}. Our scoring system is catching up."

**E2 — Empty Leaderboard (no published scripts yet)**  
Show illustration + "No scripts published yet. Be the first!"

---

## Security Considerations

- Leaderboard is public (no auth required)
- Scores are computed server-side and cannot be manipulated by clients
- Anti-gaming: the scoring job excludes self-views (views from the script owner's user_id are not counted in view_count)

---

## Human Authorship Considerations

- No AI involvement in leaderboard computation

---

## DB Tables Touched

- `leaderboard_snapshots` (INSERT daily by scheduler)
- `script_publications` (READ `like_count`, `listen_play_count`, `review_count`)
- `script_views` (INSERT when script detail page is viewed, used for view_count)

```sql
CREATE TABLE script_views (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  publication_id UUID NOT NULL REFERENCES script_publications(id) ON DELETE CASCADE,
  viewer_id UUID REFERENCES profiles(id),
  viewed_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_script_views_publication ON script_views(publication_id, viewed_at DESC);

CREATE TABLE leaderboard_snapshots (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  snapshot_type TEXT NOT NULL,
  rank INT NOT NULL,
  publication_id UUID NOT NULL REFERENCES script_publications(id),
  score INT NOT NULL,
  view_count INT NOT NULL,
  like_count INT NOT NULL,
  listen_play_count INT NOT NULL,
  review_count INT NOT NULL,
  computed_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_leaderboard_snapshots_type_rank ON leaderboard_snapshots(snapshot_type, rank, computed_at DESC);
```

---

## API Endpoints Used

- `GET /leaderboard?type={weekly|all_time}` — returns top 50 from latest snapshot
- `GET /leaderboard/my-rank` — returns the requesting writer's best-ranked script

---

## Test Cases

**TC-001:** 5 scripts published with different like/view/listen/review counts → leaderboard after scheduler run shows correct rank order.  
**TC-002:** Same script liked 100 times by the owner → self-view exclusion confirmed; score unchanged.  
**TC-003:** Writer's script is rank 73 → "Your Best Script" section shows rank 73.  
**TC-004:** Weekly leaderboard → only counts events from last 7 days.  
**TC-005:** Leaderboard page load < 500ms (served from snapshot).  
**TC-006:** Script with listen_play_count 10, like_count 50 → score = 10×3 + 50×2 = 130.

---

## Out of Scope

- Writer leaderboard (ranking writers by total score across all their scripts) — Phase 4
- Leaderboard badges / trophies on profile — Phase 4
- Real-time leaderboard (sub-minute refresh) — Phase 4
