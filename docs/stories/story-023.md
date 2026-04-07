# story-023 — Global Search (Cmd+K)

| Field | Value |
|---|---|
| **Story ID** | story-023 |
| **Title** | Global Search (Cmd+K) |
| **Epic** | E09 — Editor Utilities |
| **Phase** | 2 |
| **Sprint** | Sprint 3 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Marcus (the Hollywood rewrite specialist) |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-004 |

---

## Problem Statement

Finding a specific line of dialogue spoken by a specific character in a 120-page script requires scrolling manually or using Find & Replace — which is keyword-matching only. Writers need a global search that searches across all scripts, all blocks, all scene headings, and all story bible entries, with results appearing in < 200ms. Think Cmd+K in Linear or Spotlight on macOS.

---

## User Story

As a screenwriter, I want a global search palette (Cmd+K) that searches across all my scripts, scenes, blocks, and Story Bible entries — so that I can find anything in my work instantly.

---

## Acceptance Criteria

**GIVEN** the user presses Cmd+K (or Ctrl+K on Windows) from anywhere in the app,  
**WHEN** the search palette opens,  
**THEN** a centred modal overlay appears with a search input field, a list of "Recent" actions (last 5 scripts opened, last 3 searches), and placeholder text "Search scripts, scenes, characters…"; focus is immediately on the input field.

**GIVEN** the user types a search query of at least 2 characters,  
**WHEN** the query is sent to `GET /search?q=:query`,  
**THEN** results appear in < 200ms; results are grouped into sections: Scripts (title matches), Scenes (scene heading matches), Blocks (content matches — showing 60-character context window around the match), Story Bible (character/location name matches); each section shows a maximum of 5 results.

**GIVEN** search results are displayed,  
**WHEN** the user clicks or presses Enter on a result,  
**THEN** the search palette closes; if the result is in the currently open script, the editor scrolls to and highlights the matching block; if the result is in a different script, the user is navigated to that script's editor and then scrolled to the matching block; the matching text is highlighted briefly in amber.

**GIVEN** the user presses the arrow keys in the search palette,  
**WHEN** the arrow keys are pressed,  
**THEN** the selection moves between results in the list; pressing Enter on the selected result navigates to it.

**GIVEN** the user types a query that matches no results,  
**WHEN** the search runs,  
**THEN** the palette shows "No results for '[query]'" with the LIPILY illustration and a suggestion: "Try searching for a character name, location, or scene."

**GIVEN** the user has > 1,000 blocks across all scripts,  
**WHEN** a search is performed,  
**THEN** the search is executed by the Rust API using PostgreSQL `tsvector` full-text search with `plainto_tsquery`; results are ranked by `ts_rank`; the response is paginated to 20 results per category maximum.

**GIVEN** the user wants to filter results to the current script only,  
**WHEN** they type `/script` before their query or click the "Current Script" filter tab,  
**THEN** the search is scoped to the current `script_id`; the `GET /search?q=:query&script_id=:id` endpoint is used.

**GIVEN** the user types a character name in all caps (e.g., "JAMES"),  
**WHEN** the search runs,  
**THEN** results include both uppercase and mixed-case occurrences; search is case-insensitive.

**GIVEN** the user performs a search while offline,  
**WHEN** the query is typed,  
**THEN** the search runs against the Drift SQLite `CachedBlocks` table using SQLite FTS5; results appear from local data only; an "Offline — showing local results" badge appears at the top of the results.

**GIVEN** the user types a query in a non-Latin script (Hindi, Arabic, Tamil),  
**WHEN** the search runs,  
**THEN** the PostgreSQL full-text search uses `simple` dictionary (language-agnostic); results are returned correctly for Unicode script content.

---

## Performance Criteria

- Search palette open: < 50ms (no network — just local state)
- Search results appear: < 200ms from keystroke (debounced 100ms)
- Offline search (Drift FTS5): < 100ms for up to 10,000 blocks
- Navigation to result (same script): < 100ms to scroll and highlight
- Navigation to result (different script): < 1.5 seconds to load script and scroll

---

## Offline Behaviour

- SQLite FTS5 virtual table created on `CachedBlocks.content` column in Drift migration
- Offline search searches local Drift only
- "Offline" badge shown in results list

---

## Mobile Behaviour (375px)

- Cmd+K not available on mobile; search triggered by tapping the 🔍 icon in the editor toolbar or the dashboard app bar
- Opens as a full-screen search overlay (same as iOS Spotlight behaviour)
- Keyboard opens automatically; results show below the search bar
- Result tap navigates to the block; the overlay closes

---

## Error States

**E1 — Search API Timeout**  
`GET /search` takes > 2 seconds: show "Search is taking longer than expected…" and fall back to Drift local search results; append "(local results only)" to the results header.

**E2 — Query Too Long**  
User types > 200 characters: truncate silently to 200 characters; no error shown.

**E3 — Special Characters in Query**  
Query contains PostgreSQL tsquery special characters (`&`, `|`, `!`, `:`): sanitise with `plainto_tsquery` (which handles raw user input safely); no injection possible.

---

## Security Considerations

- `GET /search` enforces RLS via the JWT: only the authenticated user's own scripts (and scripts they collaborate on) are returned
- The search API is rate-limited to 60 requests per minute per user
- No cross-user search results are possible

---

## Human Authorship Considerations

Not applicable — search is a read-only discovery tool.

---

## DB Tables Touched

- `blocks` (full-text search on `content` — PostgreSQL `tsvector` index)
- `scenes` (full-text search on `heading`)
- `scripts` (full-text search on `title`)
- `story_bible_entries` (full-text search on `entry_name` and `description`)

---

## API Endpoints Used

- `GET /search?q=:query` — global search
- `GET /search?q=:query&script_id=:id` — script-scoped search

---

## Test Cases

**TC-001:** Press Cmd+K → assert palette opens with focus in 50ms.  
**TC-002:** Type "JAMES" → assert results appear in < 200ms with correct grouping.  
**TC-003:** Click a Block result → assert editor scrolls to that block with amber highlight.  
**TC-004:** Click a result from a different script → assert navigation to that script and block.  
**TC-005:** Type query with no results → assert empty state with suggestion shown.  
**TC-006:** Type query offline → assert Drift FTS5 results shown with offline badge.  
**TC-007:** Type in Hindi script → assert results returned correctly.  
**TC-008:** Press arrow keys → assert keyboard navigation works through results.  
**TC-009:** Rate limit: 61 requests in 1 minute → assert 429 returned for the 61st.

---

## Out of Scope

- Find & Replace within the current script (story-026)
- AI-powered semantic search ("find scenes about redemption") — Phase 2
- Search history saved to server (only local recent searches in current session)
