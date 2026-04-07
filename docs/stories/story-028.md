# story-028 — Script Statistics & Writing Goals

| Field | Value |
|---|---|
| **Story ID** | story-028 |
| **Title** | Script Statistics & Writing Goals |
| **Epic** | E01 — Core Editor |
| **Phase** | 2 |
| **Sprint** | Sprint 4 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Arjun (the aspiring feature-film screenwriter) |
| **Estimated Effort** | 2 days |
| **Dependencies** | story-004 |

---

## Problem Statement

Professional writers work to daily word count goals and track their progress toward completing a draft. Without statistics, writers have no feedback loop — they don't know if they're on pace to finish, how consistent their sessions are, or when they're most productive. Statistics and goals provide accountability and motivation.

---

## User Story

As a screenwriter, I want to track my writing statistics — daily word counts, session lengths, page progress, and writing streaks — so that I stay accountable to my writing goals and can see my productivity patterns.

---

## Acceptance Criteria

**GIVEN** a user opens the Statistics panel (from the editor sidebar or Cmd+Shift+I),  
**WHEN** the panel loads,  
**THEN** it shows the following metrics for the current script:
- Total Word Count (all blocks, all scenes)
- Total Page Count estimate (words ÷ 250)
- Scene Count
- Character Count (unique character names from character blocks)
- Average Scene Length (pages)
- Words Written Today (in this script)
- Words Written This Week
- Current Writing Streak (consecutive days with at least 100 words written across all scripts)

**GIVEN** the user is writing in Focus Mode (story-006) or the regular editor,  
**WHEN** they write,  
**THEN** the `writing_sessions` table is updated: `words_written` increments every 60 seconds; `active_duration` tracks time with keyboard activity (pauses when inactive > 5 minutes).

**GIVEN** the user has set a daily writing goal (accessed from Settings → Writing Goals),  
**WHEN** they open the editor for the day,  
**THEN** a progress bar at the bottom of the Statistics panel shows: "Today: [N] / [goal] words" with a percentage fill; when the goal is reached, the bar turns green and a ✦ celebration appears.

**GIVEN** the user wants to set or change their daily writing goal,  
**WHEN** they open Settings → Writing Goals,  
**THEN** they can enter: Daily Word Count Goal (default: 500 words), Weekly Script Page Goal (default: 10 pages), Preferred Writing Time (Morning / Afternoon / Evening — used for focus insights only).

**GIVEN** the user has written for 7 consecutive days,  
**WHEN** the Statistics panel is open,  
**THEN** a 🔥 streak badge shows "7-day streak"; the streak increments if the user writes at least 100 words in any script on a given UTC day; the streak resets to 0 if a day is missed.

**GIVEN** the Statistics panel is open,  
**WHEN** the user views the "Sessions" section,  
**THEN** a list of the last 10 writing sessions is shown with: date, duration, words written, and which script was worked on.

**GIVEN** the user views the "Word Count History" chart,  
**WHEN** the chart renders,  
**THEN** a 30-day bar chart (rendered with fl_chart) shows daily word counts across all scripts; bars are colour-coded by script; hovering shows the exact word count for that day.

**GIVEN** a user is on the Free plan,  
**WHEN** they open Statistics,  
**THEN** basic stats (word count, page count, scene count) are available; session history and the word count chart are Pro features (shown with a lock icon and "Upgrade to Pro" prompt).

**GIVEN** the user reaches their daily goal,  
**WHEN** the celebration fires,  
**THEN** a push notification is dispatched (if the user has enabled writing goal notifications): "🎉 Daily goal reached! You wrote [N] words today."

---

## Performance Criteria

- Statistics panel open: < 200ms
- Word count re-calculation: < 50ms (computed from Drift cache, no API call needed)
- Session list load: < 200ms
- fl_chart render (30-day history): < 300ms

---

## Offline Behaviour

- Word count and page count are computed from Drift `CachedBlocks` — available offline
- Writing session tracking continues offline (sessions stored in Drift)
- Session sync and streak calculation run when back online

---

## Mobile Behaviour (375px)

- Statistics accessed from the "Stats" icon in the editor toolbar or the dashboard profile menu
- Opens as a full-screen bottom sheet with scrollable sections
- Word count chart: horizontal scroll for 30 days on mobile (single column visible at a time)

---

## Error States

**E1 — Session Sync Conflict**  
Two devices write concurrently (desktop + mobile); both sessions are recorded separately; total words for the day is the sum of both sessions; no conflict.

**E2 — Streak Timezone Edge Case**  
UTC midnight rollover: the streak is calculated based on UTC date, not local time. A note in the UI: "Streaks reset at midnight UTC."

---

## Security Considerations

- `writing_sessions` RLS: user can only read and write their own sessions
- Word count statistics are only visible to the script owner (not shared with collaborators)

---

## Human Authorship Considerations

Not applicable.

---

## DB Tables Touched

- `writing_sessions` (INSERT new session, UPDATE words_written + active_duration)
- `blocks` (SELECT COUNT + word count computation)
- `scenes` (SELECT COUNT)
- `profiles` (UPDATE `writing_streak` + `last_streak_date`)

---

## API Endpoints Used

- `GET /scripts/:id/stats` — load script statistics
- `GET /writing-sessions` — load session history
- `POST /writing-sessions` — start session
- `PATCH /writing-sessions/:id` — update session (words written, duration)

---

## Test Cases

**TC-001:** Write 500 words → open Statistics → assert word count shows 500.  
**TC-002:** Set daily goal 300 words → write 300 → assert progress bar turns green; celebration fires.  
**TC-003:** Write on 7 consecutive days → assert 7-day streak badge and 🔥 icon shown.  
**TC-004:** Miss a day → assert streak resets to 0.  
**TC-005:** Free user opens Statistics → assert chart and session history locked with Pro prompt.  
**TC-006:** fl_chart renders 30-day history → hover a bar → assert tooltip with word count.  
**TC-007:** Write offline → reconnect → assert session synced; word count updated in server.

---

## Out of Scope

- AI writing pattern insights ("You write best on Tuesdays") — Phase 2
- Team statistics for Studio plan (Phase 2)
- Exporting statistics as CSV (Phase 2)
