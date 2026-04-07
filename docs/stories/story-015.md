# story-015 — Alternate Dialogue Variants

| Field | Value |
|---|---|
| **Story ID** | story-015 |
| **Title** | Alternate Dialogue Variants |
| **Epic** | E01 — Core Editor |
| **Phase** | 2 |
| **Sprint** | Sprint 2 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Marcus (the Hollywood rewrite specialist) |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-004 |

---

## Problem Statement

Experienced writers often write 3–5 versions of a single dialogue line before choosing the best one. Currently they paste alternatives into notes or use comments — messy and disconnected from the actual line. Alternate Dialogue Variants allow multiple versions of any block to coexist in the script, with one marked as "active" and the others hidden but instantly accessible via keyboard shortcut.

---

## User Story

As a screenwriter, I want to store multiple alternative versions of any dialogue block — and switch between them with a keyboard shortcut — so that I can quickly compare options without losing any version.

---

## Acceptance Criteria

**GIVEN** a user is in any block in the editor and presses Cmd+Alt+N (create variant),  
**WHEN** the variant is created,  
**THEN** a new row is inserted into `blocks` with the same `scene_id`, `parent_block_id` pointing to the original block, `variant_index = 2` (or next available), and `is_active = false`; the variant block appears as a visually distinct "ghost" block directly below the active block with a "[Variant 2]" label in the gutter.

**GIVEN** there are 3 variants of a block (variant indices 1, 2, 3),  
**WHEN** the user presses Cmd+Alt+→ (cycle variants forward),  
**THEN** the current active variant is deactivated (`is_active = false`); the next variant in the cycle is activated (`is_active = true`); the editor display updates within 1 frame; the deactivated variant fades to 30% opacity.

**GIVEN** a block has variants,  
**WHEN** the user presses Cmd+Alt+← (cycle variants backward),  
**THEN** the active variant cycles to the previous one; the same visual swap occurs.

**GIVEN** a block has variants and the user is not in the variant they want to delete,  
**WHEN** they right-click the variant ghost block and select "Delete Variant",  
**THEN** `DELETE /blocks/:id` (soft delete) is called for that specific variant; the remaining variants renumber (variant indices reindex continuously); the active variant does not change.

**GIVEN** a block has 2 variants and the user wants to promote a non-active variant to be the single canonical version,  
**WHEN** they right-click the variant and select "Keep Only This Version",  
**THEN** all other variants are soft-deleted; the selected variant becomes the sole block (no parent_block_id, no variant_index); the gutter variant indicator disappears.

**GIVEN** the script is exported as PDF (story-019) or Fountain/FDX (story-020),  
**WHEN** the export runs,  
**THEN** only the `is_active = true` variant is included in the export; all inactive variants are excluded.

**GIVEN** a collaborator's cursor is inside a variant block,  
**WHEN** the owner cycles to a different variant,  
**THEN** the collaborator's cursor is moved to the same offset in the newly active variant (closest character match); a subtle toast appears on the collaborator's screen: "[Name] switched to Variant 2."

**GIVEN** the scene navigator (story-005) is open,  
**WHEN** the writer views a scene card that has at least one block with variants,  
**THEN** a small "V" badge appears on the scene card indicating "this scene has dialogue variants"; it shows the total count of variants across all blocks in the scene.

**GIVEN** a user is on the Free plan and already has 2 variants on a block and tries to add a 3rd,  
**WHEN** Cmd+Alt+N is pressed,  
**THEN** an upgrade prompt appears: "Free plan allows 2 variants per block. Upgrade to Pro for unlimited variants."

**GIVEN** a user has the Variants panel open (toggle via Cmd+Alt+P),  
**WHEN** the panel is visible,  
**THEN** all blocks with variants are listed; each entry shows the block's active text and a count of variants; clicking an entry in the panel jumps to that block in the editor and shows all variants inline.

---

## Performance Criteria

- Variant creation: < 200ms (optimistic — new ghost block appears immediately)
- Variant cycle (keyboard shortcut → visual swap): < 16ms (same frame)
- Variants panel load (50 blocks with variants): < 300ms
- Export correctly excludes inactive variants: verified in < 100ms of export start

---

## Offline Behaviour

- Variant creation and cycling work fully offline (all changes stored in Drift and Y.Doc)
- Variants queue in `OfflineWriteQueue` for sync on reconnect
- The Y.Doc tracks variant state (active/inactive flags) as part of each block's `Y.Map`

---

## Mobile Behaviour (375px)

- "Create Variant" accessible via long-press on a block → context menu → "Add Variant"
- Variant cycling via swipe-left on the ghost block (SwipeAction widget)
- Variant ghost blocks are shown at 50% opacity below the active block on mobile
- Variants panel not available on mobile (deferred); accessible via desktop only

---

## Error States

**E1 — Maximum Variants Reached (Pro)**  
Pro users: maximum 10 variants per block; on 11th attempt: "You've reached the maximum of 10 variants per block."

**E2 — Delete Active Variant**  
If the user tries to delete the only active variant: show warning "You must keep at least one active version. Activate another variant first."

**E3 — Cycle on Single Block**  
If a block has only 1 variant and the cycle shortcut is pressed: a subtle "No other variants" toast appears for 1 second; no state change.

---

## Security Considerations

- Variants are child rows in the `blocks` table; they inherit all block RLS policies
- Only editor/owner roles can create, edit, or delete variants
- Export endpoint server-side filters `is_active = true` — the client cannot override this

---

## Human Authorship Considerations

- Variants typed by the user have `content_source = 'human_written'`
- Variants generated by AI Dialogue Suggest (story-033) have `content_source = 'ai_generated'`
- AI variants are marked with ✦ gutter icon; confirming them changes `content_source` to `human_confirmed`

---

## DB Tables Touched

- `blocks` (INSERT variant with `parent_block_id` + `variant_index`, UPDATE `is_active`, soft DELETE variant)

---

## API Endpoints Used

- `POST /blocks` — create variant block (with `parent_block_id` and `variant_index`)
- `PATCH /blocks/:id` — update variant content or activate/deactivate
- `DELETE /blocks/:id` — soft-delete a variant
- `POST /blocks/:id/consolidate` — "Keep only this version" action

---

## Test Cases

**TC-001:** Cmd+Alt+N on dialogue block → assert ghost variant appears with "[Variant 2]" label.  
**TC-002:** Cmd+Alt+→ → assert active variant switches; previous fades to 30% opacity.  
**TC-003:** Delete non-active variant → assert it disappears; remaining variants renumber correctly.  
**TC-004:** "Keep Only This Version" → assert all other variants deleted; selected becomes sole block.  
**TC-005:** Export to PDF with 2 variants → assert only active variant appears in PDF.  
**TC-006:** Collaborator in variant 1; owner cycles to variant 2 → assert collaborator sees toast.  
**TC-007:** Free user tries to add 3rd variant → assert upgrade prompt shown.  
**TC-008:** Cycle on single block → assert "No other variants" toast for 1 second.

---

## Out of Scope

- AI-generated variants (story-033 wires into this variant system but is a separate story)
- Variant comparison side-by-side view (Phase 2)
- Variants in non-dialogue blocks (Phase 2 — v1 supports any block type, but the UI affordance is most visible on Dialogue blocks)
