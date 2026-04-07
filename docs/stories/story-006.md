# story-006 — Focus Mode

| Field | Value |
|---|---|
| **Story ID** | story-006 |
| **Title** | Focus Mode |
| **Epic** | E01 — Core Editor |
| **Phase** | 1 |
| **Sprint** | Sprint 1 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Arjun (the aspiring feature-film screenwriter) |
| **Estimated Effort** | 2 days |
| **Dependencies** | story-004, story-018 |

---

## Problem Statement

Distraction is the enemy of deep writing work. Writers on tight deadlines need a mode that removes every non-essential UI element and lets them write in an immersive, full-screen environment — just the words and the page. Focus Mode also powers the session-level writing goal tracking (words per session, scene targets), which is critical for professional writers with daily quotas.

---

## User Story

As a screenwriter, I want a distraction-free full-screen Focus Mode so that I can write deeply for extended sessions without UI clutter pulling my attention away from the story.

---

## Acceptance Criteria

**GIVEN** the user is in the editor and presses Cmd+Shift+F (desktop) or taps the Focus Mode icon in the editor toolbar,  
**WHEN** Focus Mode activates,  
**THEN** the scene navigator panel, AI sidebar, comment panel, collaboration presence bar, toolbar, and all chrome are hidden; the editor canvas expands to 100% of the screen; a subtle "Press Esc to exit Focus Mode" hint appears for 2 seconds then fades.

**GIVEN** Focus Mode is active,  
**WHEN** the user moves the mouse (desktop) or taps the top edge of the screen (mobile),  
**THEN** a minimal transparent toolbar fades in showing only: Exit Focus Mode (Esc), word count, and timer (optional session timer); it auto-hides after 3 seconds of inactivity.

**GIVEN** the user has set a session writing goal (e.g., "write 500 words"),  
**WHEN** Focus Mode is active and the word count reaches the goal,  
**THEN** a gentle celebration animation (✦ sparkle burst) appears for 1.5 seconds; a subtle "Goal reached! 500 words" toast shows; writing continues uninterrupted.

**GIVEN** Focus Mode is active,  
**WHEN** the user presses Escape or taps the Exit button,  
**THEN** all panels animate back to their last-open state within 300ms; the editor is restored to exactly the same scroll position.

**GIVEN** an incoming collaboration event occurs (another writer edits a block) while Focus Mode is active,  
**WHEN** the event is received via WebSocket,  
**THEN** the block content updates silently with no toast, no notification badge, and no panel showing; collaboration sync continues in background.

**GIVEN** the user activates Focus Mode on a 375px mobile device,  
**WHEN** Focus Mode is active,  
**THEN** the bottom navigation bar and top app bar are hidden; the formatting toolbar above the keyboard remains visible (it is essential for mobile writing); only the text canvas and keyboard are shown.

**GIVEN** the user has enabled "Typewriter Scroll" in settings,  
**WHEN** Focus Mode is active and they type,  
**THEN** the current line the cursor is on stays at 40% from the top of the screen (typewriter mode); the editor scrolls upward as new content is written so the active line never moves.

**GIVEN** a user in Focus Mode receives a system notification,  
**WHEN** the notification arrives,  
**THEN** LIPILY does not intercept or suppress OS-level notifications; the user's OS handles them; LIPILY's own in-app notification badges are suppressed during Focus Mode.

**GIVEN** the user is in Focus Mode and the app loses focus (they switch apps),  
**WHEN** they return to LIPILY,  
**THEN** Focus Mode is still active; the session timer (if running) pauses while the app is in background and resumes on return.

**GIVEN** the user's network disconnects while in Focus Mode,  
**WHEN** the connection drops,  
**THEN** the offline save mechanism continues silently (all writes go to Drift cache); a single subtle amber dot appears on the word count area; no modal or banner interrupts the writing session.

---

## Performance Criteria

- Focus Mode activation animation: < 200ms
- Focus Mode exit animation: < 300ms
- Typewriter scroll latency: < 16ms per keystroke
- Word count update: < 50ms per keystroke (debounced every 500ms)
- Session timer tick: exactly every 1 second, no drift over a 2-hour session

---

## Offline Behaviour

Focus Mode is designed to work fully offline. The offline write queue handles all persistence. The only indicator of offline state is the amber dot on the word count — no modal, banner, or interruption of any kind.

---

## Mobile Behaviour (375px)

- Focus Mode is triggered by a single tap on the "↗" full-screen icon in the editor toolbar
- Top app bar and bottom navigation are hidden
- The formatting block-type toolbar (horizontal scroll strip above keyboard) remains visible
- Word count shown in a small floating pill (top-right corner, 8px from safe area edge)
- Tap the word count pill to see: words written this session, time elapsed, goal progress (if set)

---

## Error States

**E1 — Focus Mode on Unsaved Script**  
If the script has pending unsaved changes (offline queue not empty) when Focus Mode is activated: activate Focus Mode normally; the offline queue continues flushing in background; no error state.

**E2 — Goal Not Set**  
If the user activates Focus Mode without a writing goal set: Focus Mode activates normally; no goal bar is shown; the word count is still tracked for the session.

**E3 — Session Timer Conflict**  
If the user has two editor tabs open (desktop) and activates Focus Mode in one, activating it in the other should dismiss Focus Mode in the first tab automatically (one active Focus Mode instance per user).

---

## Security Considerations

- No special security considerations beyond the base editor (story-004)
- Focus Mode does not transmit any additional data; it is purely a UI state

---

## Human Authorship Considerations

- All text written in Focus Mode has `content_source = 'human_written'`
- The AI suggestion system is paused in Focus Mode (no proactive AI suggestions pop up); the writer can still invoke AI manually with Cmd+K

---

## DB Tables Touched

- `writing_sessions` (INSERT session record when Focus Mode starts; UPDATE `end_time`, `words_written` when it ends)
- `blocks` (same as story-004 — writes continue normally)

---

## API Endpoints Used

- `POST /writing-sessions` — log start of writing session
- `PATCH /writing-sessions/:id` — update session on end/pause

---

## Test Cases

**TC-001:** Press Cmd+Shift+F → assert navigator, AI sidebar, toolbar hidden within 200ms.  
**TC-002:** Focus Mode active → move mouse → assert minimal toolbar fades in; stops moving → assert toolbar fades out after 3s.  
**TC-003:** Set goal 300 words → write 300 words in Focus Mode → assert celebration animation fires.  
**TC-004:** Press Escape in Focus Mode → assert all panels restore to previous state within 300ms.  
**TC-005:** Collaborate event in Focus Mode → assert block updates silently with no notification.  
**TC-006:** Enable typewriter scroll → type in Focus Mode → assert active line stays at 40% from top.  
**TC-007:** Disconnect network in Focus Mode → assert only amber dot appears; no interruption.

---

## Out of Scope

- Ambient sound / white noise player (future feature — out of v1 scope)
- Pomodoro timer integration (out of scope)
- Full writing statistics dashboard (story-028)
- Daily streak tracking (story-028)
