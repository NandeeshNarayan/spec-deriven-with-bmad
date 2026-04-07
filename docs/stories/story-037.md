# story-037 — AI Logline & Pitch Generator

| Field | Value |
|---|---|
| **Story ID** | story-037 |
| **Title** | AI Logline & Pitch Generator |
| **Epic** | E03 — AI Feature Suite |
| **Phase** | 2 |
| **Sprint** | Sprint 5 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Arjun (the aspiring feature-film screenwriter) |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-007, story-008 |

---

## Problem Statement

Pitching a screenplay is as much a skill as writing it. Writers who struggle to distil 120 pages into a single compelling logline often fail to get their work read. The AI Pitch Generator synthesises the script's Story Bible themes, character arcs, and structural analysis into a logline, a one-page synopsis, and a pitch deck outline — giving writers a professional pitch package without a separate pitch consultant.

---

## User Story

As a screenwriter preparing to submit my script, I want an AI-generated pitch package — logline, synopsis, and pitch deck outline — based entirely on my script content, so that I can approach agents and producers with a professional submission.

---

## Acceptance Criteria

**GIVEN** a Pro or Studio user's script has at least 60 pages (≈ page_count >= 60) and has Story Bible data,  
**WHEN** they click "Generate Pitch Package" (from the AI tools menu),  
**THEN** a `POST /ai/pitch-generator` job is dispatched; model: `claude-sonnet-4-5`; the job uses: script title, logline from Story Bible Premise tab (if present), all Scene Intelligence summaries, character profiles, themes from Story Bible; job runs in background.

**GIVEN** the pitch package is generated,  
**WHEN** the user opens the Pitch Package panel,  
**THEN** it shows three sections: Logline (1–2 sentences), One-Page Synopsis (4–6 paragraphs covering setup, conflict, midpoint, dark night, resolution, theme), and Pitch Deck Outline (10 bullet points: genre, logline, theme, protagonist, antagonist, key scenes, tone, comparable films, writer bio placeholder, "why this story, why now").

**GIVEN** the pitch package is displayed,  
**WHEN** the writer reviews any section,  
**THEN** each section has a ✦ AI badge, an "Edit" button (opens the section in an inline text editor), and a "Regenerate This Section" button (regenerates only that section without regenerating the others).

**GIVEN** the writer edits the logline,  
**WHEN** they save the edit,  
**THEN** the edited logline is saved to `scripts.pitch_package` JSON field with `content_source = 'human_edited'` for the logline section; the edit does not affect the Story Bible Premise tab (they are separate).

**GIVEN** the writer clicks "Export Pitch Package",  
**WHEN** the export dialog opens,  
**THEN** they can export as: PDF (styled with LIPILY branding, cover page with script title, all three sections formatted professionally); or copy each section to clipboard individually.

**GIVEN** the script has fewer than 60 pages,  
**WHEN** the writer tries to generate the pitch package,  
**THEN** an advisory: "Pitch generation works best with a complete or near-complete script (60+ pages). You can still generate, but the pitch may be incomplete." with "Generate Anyway" and "Cancel" options.

**GIVEN** a Free user tries to generate a pitch package,  
**WHEN** access is attempted,  
**THEN** paywall: "Pitch Generator is available on Pro."

**GIVEN** the writer regenerates only the synopsis section,  
**WHEN** the regeneration job completes,  
**THEN** only the synopsis is updated; the logline and pitch deck outline retain their current content (including any human edits).

**GIVEN** the pitch package export PDF is generated,  
**WHEN** the PDF is downloaded,  
**THEN** it contains: page 1 (cover: title, "Written by [author]", script format, contact info from title page), page 2 (logline + synopsis), page 3 (pitch deck outline + "Comparable Films" which the writer must fill in manually — marked [YOUR COMP TITLES HERE]).

---

## Performance Criteria

- Full pitch package generation (claude-sonnet-4-5): < 15 seconds
- Pitch Package panel open: < 200ms
- Single section regeneration: < 8 seconds
- PDF export: < 10 seconds

---

## Offline Behaviour

- Pitch Package panel viewable offline from cached `scripts.pitch_package` JSON
- Generation and regeneration require internet

---

## Mobile Behaviour (375px)

- Pitch Package panel as full-screen bottom sheet
- Sections as accordion cards (Logline / Synopsis / Pitch Deck)
- Edit opens a full-screen text editor
- "Export" in the panel header

---

## Error States

**E1 — Insufficient Script Data**  
Story Bible is empty and Scene Intelligence is missing for > 50% of scenes: show advisory "Limited script data. Pitch may be generic." with option to "Generate Anyway."

**E2 — Generation Failure**  
MCP job fails after 3 retries: "Pitch generation failed. Please try again."

---

## Security Considerations

- Pitch package is stored in `scripts.pitch_package` JSON — same RLS as the script
- Pitch export PDF follows same S3/pre-signed URL pattern as story-019

---

## Human Authorship Considerations

- All generated pitch content is `ai_generated` until edited by the writer
- Edited sections become `human_edited`
- The pitch package does NOT claim AI authorship in the exported PDF — it is the writer's submission document

---

## DB Tables Touched

- `scripts` (UPDATE `pitch_package` JSON)
- `story_bible_entries` (SELECT character profiles and themes)
- `scene_intelligence` (SELECT summaries)

---

## API Endpoints Used

- `POST /ai/pitch-generator` — trigger pitch package generation
- `PATCH /scripts/:id/pitch-package` — save edits to pitch sections
- `POST /scripts/:id/pitch-package/export` — export PDF

---

## Test Cases

**TC-001:** 120-page script with full SI and Story Bible → generate pitch → assert logline, synopsis, deck outline all generated in < 15s.  
**TC-002:** Edit logline → assert `human_edited` saved; other sections unchanged.  
**TC-003:** Regenerate synopsis → assert only synopsis updated; logline preserved.  
**TC-004:** Export PDF → assert cover page + logline + synopsis + pitch deck all present.  
**TC-005:** Script < 60 pages → assert advisory shown with "Generate Anyway" option.  
**TC-006:** Free user → assert Pro paywall.

---

## Out of Scope

- Pitch deck visual design (slides) — Phase 2
- Agent submission management and tracking — Phase 3
- Query letter generator — Phase 2
