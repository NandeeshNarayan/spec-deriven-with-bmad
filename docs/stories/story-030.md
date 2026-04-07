# story-030 — Storyboard & Shot Visualisation

| Field | Value |
|---|---|
| **Story ID** | story-030 |
| **Title** | Storyboard & Shot Visualisation |
| **Epic** | E06 — Story Tools |
| **Phase** | 3 |
| **Sprint** | Sprint 4 |
| **Priority** | P2 |
| **Status** | DRAFT |
| **Persona** | Dev (the director-writer) |
| **Estimated Effort** | 4 days |
| **Dependencies** | story-007 |

---

## Problem Statement

Writer-directors need to visualise scenes as they write them. A storyboard feature that generates AI reference images for each scene's key visual moments gives writer-directors a directorial preview during the writing phase — not just a script. This reduces the gap between what's on the page and what will appear on screen.

---

## User Story

As a writer-director, I want to generate AI storyboard images for key shots in my scenes — so that I can visualise the film as I write and catch visual storytelling weaknesses early.

---

## Acceptance Criteria

**GIVEN** a writer-director opens the Storyboard panel for a scene (accessible from the Scene Navigator scene card menu → "Storyboard"),  
**WHEN** the panel opens,  
**THEN** it shows: a "Generate Storyboard" button, a description of what will be generated ("AI will generate [N] reference shots based on your action blocks"), and an empty storyboard grid.

**GIVEN** the user clicks "Generate Storyboard" for a scene,  
**WHEN** the generation is triggered via `POST /ai/storyboard`,  
**THEN** the MCP service receives a job with the scene's action blocks; it calls DALL-E 3 via OpenRouter with a cinematic still-frame prompt for each key action line (up to 8 images per scene); images are stored in S3; `shots` table rows are created with `image_url` and `shot_description`.

**GIVEN** the storyboard images are generated,  
**WHEN** the panel updates,  
**THEN** a grid of 16:9 storyboard frames is shown, each with: the AI-generated image, the corresponding action line text below it, and an edit button.

**GIVEN** the writer wants to add a manual shot,  
**WHEN** they click "Add Shot",  
**THEN** a form opens with: Shot Type dropdown (WIDE / MEDIUM / CLOSE-UP / EXTREME CLOSE-UP / POV / OVER THE SHOULDER / AERIAL), Shot Description text field, and an optional "Generate Image" toggle; on save, a new `shots` row is inserted; if "Generate Image" is enabled, one DALL-E image is generated for this shot.

**GIVEN** a shot in the storyboard,  
**WHEN** the writer drags it to a new position in the grid,  
**THEN** the `shots.position` is updated via `PATCH /shots/:id`; the grid order updates immediately.

**GIVEN** a storyboard image is displayed,  
**WHEN** the user taps the image,  
**THEN** it opens in a full-screen lightbox with: the image at full size, the shot description, the shot type label, and navigation arrows to the previous/next shot.

**GIVEN** the user is on the Free or Pro plan,  
**WHEN** they try to generate storyboard images,  
**THEN** a paywall modal appears: "Storyboard generation is available on Studio plan. Upgrade to unlock AI visual storytelling."

**GIVEN** a storyboard image was generated with `content_source = 'ai_generated'`,  
**WHEN** the writer is satisfied and wants to mark it as approved,  
**THEN** they click the ✓ confirm button on the image; `content_source` changes to `human_confirmed`; the ✦ badge is removed.

**GIVEN** the writer exports the script to PDF (story-019),  
**WHEN** the export runs with "Include Storyboard" toggled ON,  
**THEN** after each scene heading in the PDF, a grid of the scene's storyboard images is inserted (if any exist); images are B&W (desaturated) in the PDF to save ink and maintain professional appearance.

**GIVEN** the AI generates too many shots (> 8 per scene),  
**WHEN** the limit is reached,  
**THEN** the MCP service stops generating at 8 shots; a note in the panel: "8 shots generated for this scene. Add manual shots for additional coverage."

---

## Performance Criteria

- Storyboard panel open: < 200ms
- Image generation per shot (DALL-E 3): 5–15 seconds per image (model SLA)
- Full scene storyboard (8 images): < 2 minutes
- Storyboard grid render (8 images): < 500ms after images are loaded (from S3)
- Image S3 URL load time: < 2 seconds per image (via S3 CDN)

---

## Offline Behaviour

- Previously generated storyboard images (cached URLs) can be viewed offline (browser/OS image cache)
- Image generation not available offline
- Storyboard panel shows cached images with offline indicator

---

## Mobile Behaviour (375px)

- Storyboard panel opens as a full-screen view from the scene navigator
- Images displayed in a single-column scroll (not a grid) on mobile
- Tap-to-expand full-screen lightbox
- Drag-to-reorder uses long-press gesture

---

## Error States

**E1 — DALL-E API Error**  
Image generation fails: retry once; on second failure, insert a placeholder grey frame with "Image generation failed. Tap to retry." No full storyboard failure — other shots still generate.

**E2 — Content Policy Violation**  
DALL-E refuses a prompt (violence, adult content): insert placeholder with "Shot description contains content not supported by image generation."; log to Sentry; do not block the rest of the storyboard.

**E3 — S3 Upload Failure**  
Image generated but S3 upload fails: retry 3 times; on failure, show "Image couldn't be saved. Try regenerating this shot."

---

## Security Considerations

- Storyboard images are stored in S3 at `s3://lipily-storyboards/{script_id}/{shot_id}.webp`
- Images are served via CloudFront CDN with signed URLs (same 1-hour expiry as export)
- The DALL-E prompt is sanitised by the MCP service to remove any PII before sending to OpenRouter

---

## Human Authorship Considerations

- All AI-generated images are `content_source = 'ai_generated'`; ✦ badge displayed on each frame
- Confirming an image changes to `human_confirmed`; it remains AI-generated in origin but the writer has approved it
- The storyboard is a visual development tool — it is NOT exported by default and has no impact on the submission PDF unless explicitly included

---

## DB Tables Touched

- `shots` (INSERT, SELECT, UPDATE position, DELETE)
- `scenes` (SELECT — for context when generating)

---

## API Endpoints Used

- `POST /ai/storyboard` — trigger storyboard generation for a scene
- `GET /scenes/:id/shots` — list all shots for a scene
- `POST /shots` — add manual shot
- `PATCH /shots/:id` — update shot (position, description)
- `DELETE /shots/:id` — remove shot

---

## Test Cases

**TC-001:** Click "Generate Storyboard" → assert MCP job dispatched; 8 images generated and displayed.  
**TC-002:** Add manual shot with type "CLOSE-UP" → assert new shot row created in grid.  
**TC-003:** Drag shot to new position → assert positions updated in DB.  
**TC-004:** Tap image → assert full-screen lightbox opens with navigation.  
**TC-005:** Free user tries to generate → assert Studio paywall shown.  
**TC-006:** Confirm image → assert ✦ removed; content_source = 'human_confirmed'.  
**TC-007:** DALL-E refuses prompt → assert placeholder shown; other shots still generated.  
**TC-008:** Export PDF with storyboard ON → assert B&W images inserted after scene headings.

---

## Out of Scope

- Video storyboard (animatic) generation (Phase 3)
- 3D shot blocking visualisation (Phase 3)
- Director's treatment generator (Phase 3)
