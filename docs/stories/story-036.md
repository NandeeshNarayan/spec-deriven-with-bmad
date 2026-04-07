# story-036 — AI Pacing & Structure Analyzer

| Field | Value |
|---|---|
| **Story ID** | story-036 |
| **Title** | AI Pacing & Structure Analyzer |
| **Epic** | E03 — AI Feature Suite |
| **Phase** | 2 |
| **Sprint** | Sprint 5 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Arjun (the aspiring feature-film screenwriter) |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-007 |

---

## Problem Statement

A well-paced screenplay alternates between tension and release, action and reflection, fast scenes and slow scenes. Many first-draft scripts have pacing problems: too many consecutive dialogue-heavy scenes, an Act 2 that flatlines emotionally, an Act 3 that starts too late. The Pacing Analyzer provides a visual dashboard of the script's structural rhythm so writers can identify and fix pacing before going to draft.

---

## User Story

As a screenwriter, I want a Pacing & Structure Analyzer that shows me a visual dashboard of my script's emotional rhythm, scene length distribution, and structural milestones — so that I can identify and fix pacing problems.

---

## Acceptance Criteria

**GIVEN** a Pro or Studio user clicks "Analyse Pacing" (from the AI tools menu),  
**WHEN** the analysis is triggered via `POST /ai/pacing-analysis`,  
**THEN** the MCP service aggregates Scene Intelligence data (emotional arc scores, story structure scores) for all scenes in the script; no new model call is made if all scenes have recent Scene Intelligence data; the analysis runs synchronously if data is available.

**GIVEN** the analysis completes,  
**WHEN** the Pacing Dashboard opens,  
**THEN** it shows using fl_chart:
- Emotional Arc Chart: line chart, x-axis = scenes (1–N), y-axis = emotional intensity score (1–10) from Scene Intelligence
- Scene Length Bar Chart: bar chart, x-axis = scenes, y-axis = page count estimate per scene
- Structural Milestone Markers: vertical lines on the charts marking Act breaks (if Beat Board assignments exist from story-017)
- Tension/Release Pattern: alternating colour bands showing high-tension vs. low-tension scenes

**GIVEN** the charts are rendered,  
**WHEN** the user hovers over (desktop) or taps (mobile) any data point,  
**THEN** a tooltip shows: scene number, scene heading, emotional arc score, scene length; clicking navigates to that scene in the editor.

**GIVEN** the Pacing Analyzer detects a "flat section" (more than 5 consecutive scenes with emotional arc scores between 4–6 with < 1.0 variance),  
**WHEN** the issue is found,  
**THEN** a warning card appears below the charts: "Potential pacing issue: Scenes 23–28 have flat emotional variation. Consider adding a high-stakes moment or a tonal shift." The relevant range is highlighted on the charts.

**GIVEN** the Pacing Analyzer detects a structural milestone problem,  
**WHEN** found,  
**THEN** if the first major plot point is past page 40 in a 120-page feature script: show an advisory "Your first act break may be late (page [N] of 120). Industry standard: pages 25–30." This is advisory only.

**GIVEN** a Free user opens Pacing Analyzer,  
**WHEN** access is attempted,  
**THEN** paywall: "Pacing Analyzer is available on Pro."

**GIVEN** some scenes are missing Scene Intelligence data,  
**WHEN** the dashboard loads,  
**THEN** those scenes appear as grey bars/points with a tooltip "No scene intelligence data — run Scene Intelligence for this scene"; an aggregate warning shows "X scenes lack data. Run Scene Intelligence for a more complete analysis."

---

## Performance Criteria

- Dashboard open (if all SI data available): < 500ms
- fl_chart render (120 data points): < 300ms
- Data point hover tooltip: < 50ms

---

## Offline Behaviour

- Pacing dashboard reads from Drift-cached Scene Intelligence data
- If Scene Intelligence data is in Drift, the dashboard is fully viewable offline
- "Analyse Pacing" (fresh job) requires internet

---

## Mobile Behaviour (375px)

- Charts stack vertically on mobile
- Emotional Arc chart: 300px height; horizontal scroll for > 30 scenes
- Scene Length chart: below Emotional Arc, same dimensions

---

## Error States

**E1 — No Scene Intelligence Data**  
Zero scenes have SI data: show "Run Scene Intelligence on your scenes before using Pacing Analyzer."

**E2 — Chart Render Failure**  
fl_chart throws an error (corrupted score data): show "Chart could not be rendered. Try running Scene Intelligence again." Log to Sentry.

---

## Security Considerations

- Pacing analysis uses cached Scene Intelligence data — no new data leaves LIPILY
- Pacing advisory is purely derived from the user's own script data

---

## Human Authorship Considerations

Pacing analysis is advisory only. All suggestions are read-only. No content in the script is modified.

---

## DB Tables Touched

- `scene_intelligence` (SELECT emotional arc scores, story structure scores for all scenes)
- `scenes` (SELECT scene count and page estimates)

---

## API Endpoints Used

- `POST /ai/pacing-analysis` — trigger or load pacing analysis (uses cached SI data where possible)
- `GET /scripts/:id/pacing` — load pacing dashboard data

---

## Test Cases

**TC-001:** Script with full SI data → open Pacing Dashboard → assert charts render in < 500ms.  
**TC-002:** Hover data point → assert tooltip shows scene number, heading, emotional score.  
**TC-003:** Click data point → assert editor navigates to that scene.  
**TC-004:** 5 consecutive flat scenes → assert warning card shown.  
**TC-005:** Free user → assert Pro paywall shown.  
**TC-006:** Some scenes missing SI → assert grey bars for missing scenes + warning message.

---

## Out of Scope

- Pacing comparison across multiple scripts (Phase 2)
- Pacing benchmarks against successful films in the same genre (Phase 2)
