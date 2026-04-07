# story-052 — Onboarding Flow

| Field | Value |
|---|---|
| **Story ID** | story-052 |
| **Title** | Onboarding Flow |
| **Epic** | E13 — Infrastructure |
| **Phase** | 4 |
| **Sprint** | Sprint 7 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Asha (the film student) |
| **Estimated Effort** | 2 days |
| **Dependencies** | story-002, story-003, story-004 |

---

## Problem Statement

New users arrive at LIPILY and may not know what to do first. A well-designed onboarding flow increases activation rates by helping writers set up their profile, understand the editor, and create their first script. Time-to-first-value must be under 5 minutes.

---

## User Story

As a new screenwriter joining LIPILY, I want a quick, interactive onboarding that teaches me the key features — so that I can start writing my first script confidently within 5 minutes of signing up.

---

## Acceptance Criteria

**GIVEN** a user authenticates for the first time (first login, no existing scripts),  
**WHEN** the post-auth redirect fires,  
**THEN** they are routed to the Onboarding Flow at `/onboarding` instead of the Dashboard.

**GIVEN** the user is on Step 1 of Onboarding ("Tell us about yourself"),  
**WHEN** they fill in the form,  
**THEN** the required fields are: Display Name (pre-filled from auth if available), Writer Type (radio: Student / Emerging Writer / Working Professional / Hobbyist); optional: preferred format (Feature / Pilot / Short / Other), writing goal (dropdown: "Write my first feature" / "Improve my craft" / "Get professional feedback" / "Collaborate with others" / "Other").

**GIVEN** the user completes Step 1 and moves to Step 2 ("Choose your first script"),  
**WHEN** they see the options,  
**THEN** they are shown: "Start from scratch" (blank script with title they name), or "Use a template" (three templates: Thriller Opening, Romantic Comedy Scene, Action Set Piece — each 3–5 pages long); selecting a template creates a pre-populated script in Drift and the DB.

**GIVEN** the user selects "Start from scratch" or a template and moves to Step 3 ("Learn the editor"),  
**WHEN** Step 3 loads,  
**THEN** the editor opens with a 5-step interactive tooltip tour: (1) "This is a Scene Heading — press Tab to continue", (2) "This is an Action block — describe what we see", (3) "This is a Character Name — the name of who's speaking", (4) "This is Dialogue — what they say", (5) "Press Cmd+Shift+I to get AI analysis of your scene"; each tooltip has a "Got it" button to advance; the tour can be dismissed at any step.

**GIVEN** the user completes the editor tour (or skips it),  
**WHEN** the tour ends,  
**THEN** they land on the Dashboard with their first script card visible; a success confetti animation plays for 2 seconds; a persistent "Explore Features" quick guide appears in the sidebar (can be dismissed).

**GIVEN** a returning user navigates to Onboarding (by visiting `/onboarding` directly),  
**WHEN** they already have scripts,  
**THEN** they are redirected to the Dashboard; the Onboarding flow is one-time only (checked via `profiles.onboarding_completed = true`).

**GIVEN** the user skips onboarding from any step,  
**WHEN** they click "Skip for now",  
**THEN** `profiles.onboarding_completed = true` is set; they are routed to the Dashboard; they can restart the tour from Settings → Restart Onboarding Tour.

---

## Performance Criteria

- Onboarding flow: each step transition < 300ms
- Total time from Step 1 to first keystroke in editor: < 60 seconds (for "Start from scratch")
- Template loading: < 1 second (pre-built blocks inserted into Drift and DB)
- Editor tour tooltips: appear within 1 frame (16ms) of the target element rendering

---

## Offline Behaviour

- Step 1 (profile setup) requires network (saves to DB)
- Step 2–3 (template load + editor tour) work offline (template blocks written to Drift; syncs when back online)

---

## Mobile Behaviour (375px)

- Onboarding is designed mobile-first: single step per full screen
- Template cards: horizontal scroll on mobile
- Editor tour tooltips: position-aware (appear below the target element on mobile, above on desktop)
- "Skip" button: always visible in the top-right corner

---

## Error States

**E1 — Profile Save Failure (Step 1)**  
Toast: "Could not save your profile. Please try again." Step 1 does not advance until saved.

**E2 — Template Load Failure**  
If a template script fails to load: "Template unavailable. Start from scratch instead." Pre-filled script fallback is a blank script with the title "My First Script."

---

## Security Considerations

- Onboarding profile data is saved via the same `PATCH /profiles/:id` endpoint with standard auth
- Template scripts are pre-built and stored server-side; they are cloned for the user (not shared)

---

## Human Authorship Considerations

- Template scripts are written by human screenwriters; they are labelled `content_source = 'human_written'`
- No AI content is pre-populated during onboarding

---

## DB Tables Touched

- `profiles` (UPDATE `display_name`, `writer_type`, `writing_goal`, `preferred_format`, `onboarding_completed`)
- `scripts` (INSERT for new script from template or scratch)
- `blocks` (INSERT for template blocks)

---

## API Endpoints Used

- `PATCH /profiles/:id` — save Step 1 profile data
- `POST /scripts` — create first script (with template flag)
- `GET /onboarding-templates` — fetch available templates

---

## Test Cases

**TC-001:** First-time login → routed to `/onboarding`; NOT the dashboard.  
**TC-002:** Complete Step 1 → `profiles.writer_type` saved.  
**TC-003:** Select "Thriller Opening" template → script created with template blocks; script appears on Dashboard.  
**TC-004:** Editor tour: follow all 5 steps → tour ends; confetti animation plays.  
**TC-005:** Skip tour at Step 3 → `profiles.onboarding_completed = true`; Dashboard shown.  
**TC-006:** Returning user visits `/onboarding` → redirected to Dashboard.  
**TC-007:** Settings → "Restart Onboarding Tour" → `profiles.onboarding_completed = false`; flow restarts at Step 3 (editor tour only; profile step is skipped).

---

## Out of Scope

- Interactive video tutorial (Phase 4)
- In-app interactive checklist ("Your first week goals") — Phase 4
- Personalised AI-generated onboarding script (explicitly prohibited by Human Authorship Principle)
