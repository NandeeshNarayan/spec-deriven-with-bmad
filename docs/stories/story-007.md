# story-007 — Scene Intelligence (5-Pillar)

| Field | Value |
|---|---|
| **Story ID** | story-007 |
| **Title** | Scene Intelligence (5-Pillar) |
| **Epic** | E02 — AI Scene Intelligence |
| **Phase** | 2 |
| **Sprint** | Sprint 1 |
| **Priority** | P0 |
| **Status** | DRAFT |
| **Persona** | Arjun (the aspiring feature-film screenwriter) |
| **Estimated Effort** | 6 days |
| **Dependencies** | story-004, story-001 |

---

## Problem Statement

Professional script development involves five distinct analytical lenses: dramatic structure, character psychology, visual storytelling, emotional arc, and thematic resonance. Currently these analyses require a human script consultant or separate expensive software. LIPILY's Scene Intelligence feature provides all five in a unified sidebar that updates every time the writer saves a scene, making expert-level script analysis continuously available at zero extra cost for Pro/Studio users.

---

## User Story

As a screenwriter, I want a Scene Intelligence sidebar that analyses each scene across five pillars — Story Structure, Character Psychology, Visual Grammar, Emotional Arc, and Thematic Resonance — so that I have expert-level developmental notes available as I write, without hiring a script consultant.

---

## Acceptance Criteria

**GIVEN** a writer finishes editing a scene (all blocks in the scene have been quiet for 2 seconds and the scene has at least 50 words),  
**WHEN** the auto-analysis trigger fires,  
**THEN** a `POST /ai/scene-intelligence` request is dispatched from the Rust API to the MCP service BullMQ job queue; the Scene Intelligence sidebar shows a "Analysing…" skeleton loader state; the job runs with model `claude-sonnet-4-5`.

**GIVEN** the Scene Intelligence job completes successfully,  
**WHEN** the result is pushed back to the Rust API via the internal service-to-service endpoint,  
**THEN** the five pillar analysis results are saved to the `scene_intelligence` table; the sidebar updates with the five sections: Story Structure, Character Psychology, Visual Grammar, Emotional Arc, Thematic Resonance; each section shows a 2–4 sentence analysis and a 1–10 score; the AI badge (✦) marks all content.

**GIVEN** the sidebar is showing Scene Intelligence results,  
**WHEN** the writer taps any pillar section header to expand it,  
**THEN** the expanded view shows: the pillar score as a coloured ring (0–3: red, 4–6: amber, 7–10: green), the analysis text in full, and 2–3 actionable improvement suggestions formatted as bullet points.

**GIVEN** the scene has a Scene Intelligence result with `content_source = 'ai_generated'`,  
**WHEN** the writer reads a suggestion and taps the ✓ "Confirm" button next to it,  
**THEN** the `content_source` for that suggestion changes to `human_confirmed`; the ✦ badge disappears from the confirmed item; the approval is logged to `agent_approvals`.

**GIVEN** the writer is on a Free plan and the Scene Intelligence sidebar is open,  
**WHEN** they attempt to trigger scene analysis (Scene Intelligence is a Pro feature),  
**THEN** the sidebar shows an upgrade prompt: "Scene Intelligence is available on Pro. Upgrade to unlock expert script analysis." with an "Upgrade" CTA; no AI request is made.

**GIVEN** a Pro user has used 500 AI requests today (daily limit reached),  
**WHEN** the auto-analysis trigger fires,  
**THEN** the analysis is queued but not dispatched until the next UTC midnight reset; the sidebar shows "Daily AI limit reached. Analysis queued for tomorrow." No error.

**GIVEN** a Studio user reaches their 500 AI request limit (they have 500/day),  
**WHEN** more AI requests are needed,  
**THEN** the system returns a `429` response from the rate-limit middleware; the sidebar shows the same queuing message as above; usage resets at UTC midnight.

**GIVEN** the Scene Intelligence job fails (model API error or timeout),  
**WHEN** the failure is received,  
**THEN** the MCP service retries up to 3 times with exponential backoff (2s, 4s, 8s); if all retries fail, the sidebar shows "Analysis temporarily unavailable. We'll retry soon." and the failed job is dead-lettered to BullMQ dead-letter queue with full error context.

**GIVEN** the writer has multiple scenes open in the navigator,  
**WHEN** they click a different scene in the navigator,  
**THEN** the Scene Intelligence sidebar loads the cached analysis for the newly selected scene from `scene_intelligence` table (if available) instantly (< 100ms); it does not re-trigger analysis unless the scene content has changed since last analysis (`blocks_hash` comparison).

**GIVEN** a scene has never been analysed (new scene, < 50 words, or analysis has never run),  
**WHEN** the sidebar is shown for that scene,  
**THEN** the sidebar shows an empty state: "Write at least 50 words in this scene to unlock Scene Intelligence." with a progress indicator showing word count toward 50.

---

## 5-Pillar Definitions (MCP Prompt Requirements)

### Pillar 1 — Story Structure
- Identify the scene's dramatic function (setup / confrontation / resolution / transition)
- Score how well the scene advances the plot (1–10)
- Suggestions: missing story beats, redundant elements

### Pillar 2 — Character Psychology
- Identify character motivations, wants vs needs, hidden subtext
- Score psychological complexity (1–10)
- Suggestions: missed character moments, inconsistencies with story bible

### Pillar 3 — Visual Grammar
- Identify cinematic potential: camera-describable action, visual metaphors, spatial storytelling
- Score visual specificity (1–10)
- Suggestions: action lines that could be more visual

### Pillar 4 — Emotional Arc
- Map the emotional journey within the scene (start emotion → end emotion)
- Score emotional impact (1–10)
- Suggestions: flattened emotional arcs, missed tonal opportunities

### Pillar 5 — Thematic Resonance
- Identify how the scene reflects or subverts the script's stated themes
- Score thematic alignment (1–10)
- Suggestions: missed thematic symbols, opportunities for deeper resonance

---

## MCP Service Job Contract

```typescript
// Job name: 'scene-intelligence'
interface SceneIntelligenceJobData {
  scriptId: string;
  sceneId: string;
  userId: string;
  sceneText: string;        // Plain text of all blocks in scene
  characterList: string[];  // From story bible
  storyThemes: string[];    // From story bible themes field
  callbackEndpoint: string; // Internal Rust endpoint to POST results to
  requestId: string;        // For idempotency
}

interface SceneIntelligencePillar {
  score: number;            // 1–10
  analysis: string;         // 2–4 sentences
  suggestions: string[];    // 2–3 bullet points
}

interface SceneIntelligenceResult {
  sceneId: string;
  storyStructure: SceneIntelligencePillar;
  characterPsychology: SceneIntelligencePillar;
  visualGrammar: SceneIntelligencePillar;
  emotionalArc: SceneIntelligencePillar;
  thematicResonance: SceneIntelligencePillar;
  sceneSummary: string;     // One-line summary for scene navigator
  blocksHash: string;       // SHA-256 of scene text at time of analysis
  model: string;            // 'claude-sonnet-4-5'
  processingTimeMs: number;
}
```

---

## Performance Criteria

- Auto-analysis trigger: 2 seconds after last edit (debounced)
- MCP job queue pickup: < 500ms from enqueue
- Claude API response time: < 8 seconds for a typical 3-minute scene (≈ 400 words)
- Sidebar update after job completion: < 500ms (via WebSocket push or polling at 2s interval)
- Switching scenes in navigator (showing cached result): < 100ms
- Pro user hitting the AI rate limit: `429` response in < 50ms (Redis counter check)

---

## Offline Behaviour

- Auto-analysis does not trigger offline (no AI calls offline)
- Cached analysis results from `scene_intelligence` table are stored in Drift `CachedSceneIntelligence` and displayed offline
- A subtle offline indicator appears next to the scene intelligence header: "Showing cached analysis"
- Any pending analysis jobs are queued in `OfflineWriteQueue` and dispatched when connectivity returns

---

## Mobile Behaviour (375px)

- Scene Intelligence sidebar is accessed via the ✦ AI icon in the mobile editor toolbar
- Opens as a bottom sheet (80% screen height)
- Each pillar is a collapsed accordion row (height 56px) showing pillar name and score ring
- Tapping expands to show full analysis and suggestions
- "Analysing…" state shows a pulsing skeleton card, height 200px

---

## Error States

**E1 — Model API Error (503)**  
MCP service catches Anthropic/OpenRouter 503: retry 3 times; if all fail, set job status to `failed`; notify Rust API via callback with `success: false`; sidebar shows "Analysis unavailable."

**E2 — Malformed AI Response**  
If the AI returns a response that fails Zod schema validation: log to Sentry; retry once with an explicit JSON format reminder in the prompt; if it fails again, dead-letter the job.

**E3 — Scene Too Long**  
If the scene exceeds 8,000 tokens: the MCP service truncates from the middle (keeps first 4,000 and last 4,000 tokens) and adds a note to the response: "Analysis based on condensed scene (scene is very long)."

**E4 — Daily Limit Exceeded**  
`ai_usage` count for today >= tier limit: return `429` from Rust API immediately; do not enqueue job; sidebar shows limit message.

---

## Security Considerations

- Scene text sent to MCP service is stripped of all user PII before sending (user name is replaced with "the writer")
- The MCP service NEVER writes to Supabase directly — all writes go through the Rust API internal callback endpoint authenticated with service-to-service API key stored in Secret Manager
- `scene_intelligence` rows are protected by RLS: only the script owner and accepted collaborators can read them
- The service-to-service API key is rotated every 90 days; rotation is automated via Cloud Scheduler

---

## Human Authorship Considerations

- ALL content in the Scene Intelligence sidebar is `content_source = 'ai_generated'` by default
- Confirming a suggestion changes it to `human_confirmed`; this is logged to `agent_approvals`
- The writer is always shown the AI badge to maintain transparency about what is human vs AI work
- Suggestions are advisory only — they are never automatically applied to the script text

---

## DB Tables Touched

- `scene_intelligence` (SELECT cached results; INSERT/UPDATE on job completion)
- `agent_approvals` (INSERT on confirmation action)
- `ai_usage` (SELECT + UPDATE daily counter for rate limiting)
- `scenes` (SELECT for `blocks_hash` comparison)

---

## API Endpoints Used

- `POST /ai/scene-intelligence` — enqueue scene intelligence job (from editor)
- `POST /internal/ai/scene-intelligence/complete` — internal callback from MCP service (not user-facing)
- `GET /scripts/:id/scene-intelligence/:scene_id` — fetch cached results for a scene

---

## Test Cases

**TC-001:** Write 50+ words in a scene → wait 2s → assert `POST /ai/scene-intelligence` called.  
**TC-002:** MCP job completes → assert all 5 pillars appear in sidebar with scores and analysis text.  
**TC-003:** Tap pillar header → assert expanded view shows suggestions.  
**TC-004:** Free user opens Scene Intelligence → assert upgrade prompt shown, no API call made.  
**TC-005:** Pro user at 500 AI requests/day → assert `429` returned, sidebar shows queue message.  
**TC-006:** Navigate to different scene with cached result → assert sidebar updates in < 100ms.  
**TC-007:** Scene has 30 words → assert "Write at least 50 words" empty state shown.  
**TC-008:** MCP service receives malformed AI response → assert Zod validation error logged to Sentry; retry attempted.  
**TC-009:** Confirm a suggestion → assert `content_source` changes to `human_confirmed`; ✦ badge disappears.  
**TC-010:** `blocks_hash` unchanged since last analysis → assert re-analysis NOT triggered on navigator scene switch.

---

## Out of Scope

- AI Dialogue Suggest (story-033)
- AI Continue Scene (story-034)
- AI Continuity Checker (story-035)
- AI Pacing Analyzer (story-036)
- AI Pitch Generator (story-037)
- AI Voice Guardian (story-038)
- Story Bible auto-population (story-008)
