# story-033 — AI Dialogue Suggest

| Field | Value |
|---|---|
| **Story ID** | story-033 |
| **Title** | AI Dialogue Suggest |
| **Epic** | E03 — AI Feature Suite |
| **Phase** | 2 |
| **Sprint** | Sprint 4 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Arjun (the aspiring feature-film screenwriter) |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-007, story-016 |

---

## Problem Statement

Writer's block in the middle of a dialogue exchange is common — the writer knows where the scene is going but can't find the right words for a specific line. AI Dialogue Suggest provides 3 contextually aware alternative dialogue options, generated with the character's established voice from the Story Bible, that the writer can accept, reject, or modify — while maintaining the Human Authorship First Principle.

---

## User Story

As a screenwriter, I want AI to suggest 3 alternative dialogue lines when I'm stuck — generated from my character's established voice — so that I can break writer's block without relinquishing authorship.

---

## Acceptance Criteria

**GIVEN** the cursor is in a Dialogue block,  
**WHEN** the writer presses Cmd+Shift+D (or taps the ✦ AI icon in the editor toolbar),  
**THEN** a `POST /ai/dialogue-suggest` request is made; the panel shows a "Generating suggestions…" skeleton; the MCP service generates 3 dialogue alternatives using `claude-haiku-3-5` (fast model) with context: character profile from Story Bible, previous 3 exchanges in the scene, current scene summary from Scene Intelligence.

**GIVEN** the 3 suggestions are returned,  
**WHEN** the AI Suggestion panel opens,  
**THEN** three option cards are shown, each displaying: the suggested dialogue text, a word count, and two buttons: "Use This" and "→" (insert as variant); each card has the ✦ AI badge; the cards are labelled Option 1, Option 2, Option 3.

**GIVEN** the writer taps "Use This" on a suggestion,  
**WHEN** the acceptance fires,  
**THEN** the current dialogue block content is replaced with the suggestion text; `content_source = 'ai_generated'`; a ✦ badge appears on the block in the editor; the suggestion panel closes; an entry is logged to `agent_approvals`.

**GIVEN** the writer taps "→" (insert as variant) on a suggestion,  
**WHEN** the action fires,  
**THEN** the suggestion is added as a new variant block (story-015 variant system) via `POST /blocks` with `parent_block_id` and `variant_index`; the original dialogue remains as the active variant; the suggestion appears as an inactive ghost variant below; `content_source = 'ai_generated'` on the new variant.

**GIVEN** a suggestion is inserted as a variant,  
**WHEN** the writer reads it and modifies the text in the variant block,  
**THEN** `content_source` changes from `ai_generated` to `human_edited`; the ✦ badge changes to a pencil icon indicating "AI-originated, human-edited."

**GIVEN** the writer dismisses the suggestion panel without selecting any option,  
**WHEN** the panel closes,  
**THEN** no changes are made to the script; no `content_source` changes; the dismissal is logged to `agent_approvals` with `action = 'dismissed'`.

**GIVEN** a Free-tier user triggers AI Dialogue Suggest,  
**WHEN** the request is made,  
**THEN** it counts against their 50/day AI request limit; if the limit is reached, show "Daily AI limit reached (50/50). Resets at midnight UTC."

**GIVEN** the character has no Story Bible profile yet,  
**WHEN** the suggestion is generated,  
**THEN** the MCP service generates suggestions based on the character's dialogue patterns in the last 10 scenes (extracted from block content directly); a note is shown in the suggestion panel: "No character profile found. Suggestions based on dialogue patterns."

**GIVEN** the writer is not in a Dialogue or Character block,  
**WHEN** they trigger AI Dialogue Suggest,  
**THEN** a toast appears: "Place your cursor in a Dialogue block to use AI Dialogue Suggest."

**GIVEN** the suggestions are displayed,  
**WHEN** the writer clicks the "Regenerate" button in the panel,  
**THEN** 3 new suggestions are generated (new API call); the previous 3 are discarded; the previous request counts as 1 AI usage; the new request counts as another usage.

---

## MCP Job Context Package (Dart → Rust → MCP)

```typescript
interface DialogueSuggestJobData {
  blockId: string;           // The dialogue block requesting suggestions
  sceneId: string;
  scriptId: string;
  userId: string;
  characterName: string;     // From the character block above
  characterProfile?: {       // From story_bible_entries
    archetype: string;
    description: string;
    motivation: string;
    coreFlaw: string;
  };
  precedingExchanges: Array<{ // Last 6 blocks in scene (3 exchanges)
    blockType: string;
    character?: string;
    content: string;
  }>;
  sceneContext: string;       // From scene_intelligence.sceneSummary
  currentContent: string;    // Existing dialogue (may be empty)
  callbackEndpoint: string;
  requestId: string;
}
```

---

## Performance Criteria

- Suggestion panel appears (skeleton): < 300ms from trigger
- 3 suggestions generated (claude-haiku-3-5): < 3 seconds
- "Use This" application: < 100ms (optimistic)
- "Insert as Variant" creation: < 300ms
- Regenerate: same 3-second SLA as initial generation

---

## Offline Behaviour

- AI Dialogue Suggest is not available offline
- Trigger while offline shows: "AI suggestions require an internet connection."

---

## Mobile Behaviour (375px)

- Triggered by long-pressing in a Dialogue block → "AI Suggest" in context menu
- OR the ✦ button in the editor formatting toolbar
- Suggestion panel opens as a bottom sheet (70% height)
- Each option card: 1–2 lines of dialogue text visible; "Use This" button on right; "As Variant" button below

---

## Error States

**E1 — AI Service Unavailable**  
OpenRouter returns 503: retry once with claude-haiku-3-5; if it fails, fall back to gpt-4o via OpenRouter; if that also fails, show "AI suggestions temporarily unavailable."

**E2 — Daily Limit Reached**  
`ai_usage.count >= tier_limit`: return 429 from Rust API before dispatching job; show limit message in panel.

**E3 — Toxic Content in Context**  
If the scene context contains content that triggers the model's safety filter: return the 3 suggestions without context ("safety_filtered": true); log to Sentry for review.

---

## Security Considerations

- Dialogue context sent to MCP service is stripped of all user PII (user_id replaced with session token for this request)
- The MCP service does not store the script content after the job completes
- `agent_approvals` logs every AI interaction for audit

---

## Human Authorship Considerations

- Suggestions are ALWAYS shown to the writer before any content is changed
- "Use This" changes `content_source` to `ai_generated`; the ✦ badge is clearly visible
- Editing an accepted suggestion changes to `human_edited`
- Dismissing leaves the script unchanged
- The writer must confirm each AI suggestion individually — no bulk acceptance

---

## DB Tables Touched

- `blocks` (UPDATE content + content_source on acceptance; INSERT new variant on "As Variant")
- `agent_approvals` (INSERT for every accept/reject/dismiss event)
- `ai_usage` (SELECT + UPDATE daily counter)
- `story_bible_entries` (SELECT character profile)

---

## API Endpoints Used

- `POST /ai/dialogue-suggest` — trigger suggestion generation
- `POST /internal/ai/dialogue-suggest/complete` — MCP service callback

---

## Test Cases

**TC-001:** Cursor in Dialogue → Cmd+Shift+D → assert 3 suggestions appear in < 3s.  
**TC-002:** "Use This" on option 2 → assert block content replaced; `content_source = 'ai_generated'`; ✦ shown.  
**TC-003:** "Insert as Variant" → assert new variant block created; original stays active.  
**TC-004:** Edit accepted suggestion → assert `content_source` changes to `human_edited`.  
**TC-005:** Dismiss without selecting → assert script unchanged.  
**TC-006:** Free user at 50/day limit → trigger → assert 429 shown; no job dispatched.  
**TC-007:** Character with no Story Bible profile → suggestions generated from dialogue patterns.  
**TC-008:** Cursor in Action block → trigger → assert toast "Place cursor in Dialogue block."
**TC-009:** Regenerate → assert 3 new options generated; old discarded; 2nd AI usage counted.

---

## Out of Scope

- AI Continue Scene (story-034 — different trigger, generates full scene continuation)
- AI Voice Guardian (story-038 — passive checker, not a suggestion panel)
- AI Multilingual (story-039 — translation of existing content)
