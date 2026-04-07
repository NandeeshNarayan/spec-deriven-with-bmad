# story-054 — Dialogue Read-Aloud TTS

| Field | Value |
|---|---|
| **Story ID** | story-054 |
| **Title** | Dialogue Read-Aloud TTS |
| **Epic** | E03 — AI Feature Suite |
| **Phase** | 4 |
| **Sprint** | Sprint 7 |
| **Priority** | P2 |
| **Status** | DRAFT |
| **Persona** | Priya (the professional TV writer) |
| **Estimated Effort** | 2 days |
| **Dependencies** | story-004, story-033, story-048 |

---

## Problem Statement

Hearing dialogue read aloud is a fundamental technique professional writers use to test whether their dialogue sounds natural. "Dialogue Read-Aloud TTS" is the in-editor feature that reads selected dialogue blocks aloud using TTS voices — without needing to generate a full Listen production. It is the writer's quick sanity-check tool.

---

## User Story

As a screenwriter editing dialogue, I want to hear my dialogue read aloud with distinct character voices — so that I can immediately tell if the words sound natural and adjust them before the script is complete.

---

## Acceptance Criteria

**GIVEN** a writer selects one or more Dialogue blocks in the editor (using click + Shift+click or Cmd+click),  
**WHEN** they press the Read-Aloud button in the editor toolbar (or right-click context menu → "Read Aloud"),  
**THEN** a Read-Aloud TTS job is initiated for only the selected Dialogue blocks.

**GIVEN** the Read-Aloud TTS job is initiated,  
**WHEN** the MCP service processes the request,  
**THEN** Claude (`claude-haiku-3-5`) assigns a voice profile from the catalogue to each character name found in the selected blocks (consistent with any prior voice assignments for this script stored in `tts_voice_cache`); voices assigned are cached so the same character always gets the same voice within a script session.

**GIVEN** voice assignment is complete,  
**WHEN** TTS audio is generated,  
**THEN** each Dialogue block is converted using the OpenRouter TTS endpoint with the assigned voice; audio is generated sequentially in script order; audio files are stored as temporary S3 objects with a 1-hour TTL at `s3://lipily-tts-temp/{script_id}/{request_id}/{block_id}.mp3`.

**GIVEN** audio generation is complete,  
**WHEN** the audio is ready (typically < 5 seconds for 10 lines of dialogue),  
**THEN** the editor shows a floating audio player below the selected blocks: character name, waveform visualisation (simplified bars), play/pause, current position, and a "Close" button to dismiss.

**GIVEN** the audio player is playing,  
**WHEN** the playback position reaches a new Dialogue block,  
**THEN** the corresponding block in the editor is highlighted with a soft amber background to show which line is currently being read aloud.

**GIVEN** the writer makes edits to the dialogue after triggering Read-Aloud,  
**WHEN** they click "Re-read",  
**THEN** a new TTS job is triggered for the modified blocks; the same voice cache is used (characters keep their voices).

**GIVEN** a Free tier user tries to use Dialogue Read-Aloud,  
**WHEN** the feature is triggered,  
**THEN** a paywall is shown: "Dialogue Read-Aloud is a Pro feature. Upgrade to hear your scripts come to life." (Free tier access is NOT granted for this feature — it is Pro/Studio only.)

**GIVEN** a Pro user triggers Read-Aloud,  
**WHEN** it counts against AI usage,  
**THEN** each TTS job counts as 1 AI request regardless of the number of lines (not per-line to avoid excessive consumption).

---

## Performance Criteria

- Voice assignment (Claude): < 1 second for up to 10 characters
- TTS generation: < 5 seconds for up to 20 lines of dialogue
- Audio player appears: < 100ms after audio is ready (S3 pre-signed URL opened immediately)
- Block highlight sync with playback: < 50ms offset

---

## Offline Behaviour

- Read-Aloud TTS requires internet (TTS API calls)
- Previously generated audio for the same blocks is cached in the S3 temp path for 1 hour; if the same blocks are triggered again within 1 hour, the cached audio is served without a new TTS call

---

## Mobile Behaviour (375px)

- Read-Aloud button: shown in the block-level action bar that appears above the keyboard on mobile (same bar as the block type switcher)
- Floating audio player: full-width bottom overlay (not a floating card) on mobile; 72dp height
- Block highlight: same amber background
- Waveform visualisation: simplified 3-bar animation (not full waveform on mobile)

---

## Error States

**E1 — TTS API Unavailable**  
Toast: "Read-Aloud is temporarily unavailable. Please try again in a moment." No AI request counted for failed attempts.

**E2 — No Dialogue Blocks Selected**  
If the writer triggers Read-Aloud with no Dialogue blocks selected: toast "Select one or more Dialogue blocks to use Read-Aloud."

**E3 — Audio Playback Error**  
If the pre-signed URL expires before playback (> 1 hour): regenerate the audio automatically with a "Re-read" trigger.

---

## Security Considerations

- Temporary S3 audio files are served via pre-signed URLs (1-hour expiry); no permanent public URLs
- TTS is only triggered for the authenticated user's own scripts (RLS)
- Voice assignments are session-scoped; no voice profile data is persisted beyond the `tts_voice_cache` (which is per-script, not global PII)

---

## Human Authorship Considerations

- TTS reads the writer's exact words — zero AI rewriting
- The feature is a listening aid only; it does not modify the script text
- Voice assignment by AI (character → voice profile) is the only AI involvement; it is non-creative assignment of a technical resource

---

## DB Tables Touched

- `tts_voice_cache` (INSERT/UPDATE — per-script character-to-voice mapping)

```sql
CREATE TABLE tts_voice_cache (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  script_id UUID NOT NULL REFERENCES scripts(id) ON DELETE CASCADE,
  character_name TEXT NOT NULL,
  voice_id TEXT NOT NULL,
  pitch FLOAT NOT NULL DEFAULT 1.0,
  pace FLOAT NOT NULL DEFAULT 1.0,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(script_id, character_name)
);
```

---

## API Endpoints Used

- `POST /tts/read-aloud` — body: `{ script_id, block_ids: UUID[] }`; returns `{ audio_manifest: { block_id, presigned_url }[] }`

---

## Test Cases

**TC-001:** Select 5 Dialogue blocks → trigger Read-Aloud → audio ready within 5s; player appears.  
**TC-002:** Same character in multiple blocks → same voice_id used consistently.  
**TC-003:** Playback reaches block 3 → block 3 highlighted in editor.  
**TC-004:** Edit dialogue after Read-Aloud → Re-read → new TTS generated; voice cache unchanged.  
**TC-005:** Free tier user → paywall shown; no TTS call made.  
**TC-006:** No Dialogue blocks selected → toast error; no TTS call made.  
**TC-007:** TTS API unavailable → toast error; no AI request counted.

---

## Out of Scope

- Reading Action blocks aloud (narrator voice for action) — Phase 4
- Full script read-aloud without generating a Listen production — Phase 4
- Custom voice selection by writer — Phase 4
