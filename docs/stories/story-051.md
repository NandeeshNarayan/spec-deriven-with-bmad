# story-051 — Keyboard Shortcuts & Accessibility

| Field | Value |
|---|---|
| **Story ID** | story-051 |
| **Title** | Keyboard Shortcuts & Accessibility |
| **Epic** | E01 — Core Editor |
| **Phase** | 4 |
| **Sprint** | Sprint 7 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | All personas |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-004 |

---

## Problem Statement

Professional screenwriters use keyboard-centric workflows. An incomplete keyboard shortcut system slows them down. Additionally, LIPILY must be usable by writers with disabilities (screen reader users, keyboard-only users, low vision writers). This story defines all shortcuts and the WCAG 2.1 AA compliance baseline.

---

## User Story

As a screenwriter, I want a complete set of keyboard shortcuts and a fully accessible interface — so that I can write efficiently without touching the mouse and so that LIPILY is usable regardless of my abilities.

---

## Acceptance Criteria

**GIVEN** a user presses `?` anywhere in the editor (or Cmd+/),  
**WHEN** no modal is open,  
**THEN** a Keyboard Shortcuts cheat sheet overlay appears showing all shortcuts grouped by category; it can be dismissed with Escape.

**GIVEN** all keyboard shortcuts are implemented,  
**WHEN** tested against the complete shortcut table,  
**THEN** every shortcut in the table below functions correctly on both macOS (Cmd) and Windows/Linux (Ctrl).

**GIVEN** a screen reader (VoiceOver / NVDA / TalkBack) is active,  
**WHEN** the editor is used,  
**THEN** all interactive elements have semantic labels via Flutter `Semantics` widget; the current block type is announced when the cursor moves to a new block; AI suggestions are announced as "AI suggestion available"; modals are announced as dialogs with focus trapped inside.

**GIVEN** the user relies on keyboard navigation only (no mouse/touch),  
**WHEN** they navigate through the editor UI,  
**THEN** focus order is logical (left-to-right, top-to-bottom); all interactive elements are reachable via Tab; focus is never lost (i.e., focus never goes to `null`); the focused element has a visible 2dp blue outline.

**GIVEN** the UI is rendered at any font size,  
**WHEN** the system font size is increased to 200% (OS-level accessibility setting),  
**THEN** all UI text scales; no text is clipped or overlaps other elements; the editor text scales with system font but can be overridden by the writer's font size preference (story-027).

**GIVEN** the app is used in high contrast mode (OS-level),  
**WHEN** high contrast is active,  
**THEN** all text meets a minimum contrast ratio of 4.5:1 (WCAG AA for normal text) and 3:1 for large text; interactive elements have a contrast ratio of at least 3:1 against their background.

---

## Complete Keyboard Shortcut Table

### Formatting (Editor)
| Shortcut (Mac) | Shortcut (Win/Linux) | Action |
|---|---|---|
| Tab | Tab | Smart Flow: advance block type |
| Shift+Tab | Shift+Tab | Smart Flow: reverse block type |
| Cmd+B | Ctrl+B | Bold (for Action blocks in non-standard mode) |
| Cmd+I | Ctrl+I | Italic |
| Cmd+Z | Ctrl+Z | Undo |
| Cmd+Shift+Z | Ctrl+Y | Redo |
| Cmd+A | Ctrl+A | Select all text in current block |
| Cmd+X / Cmd+C / Cmd+V | Ctrl+X/C/V | Cut / Copy / Paste |

### Navigation
| Shortcut (Mac) | Shortcut (Win/Linux) | Action |
|---|---|---|
| Cmd+K | Ctrl+K | Open Global Search |
| Cmd+[ | Ctrl+[ | Open Scene Navigator |
| Cmd+Shift+F | Ctrl+Shift+F | Toggle Focus Mode |
| Cmd+\ | Ctrl+\ | Toggle sidebar (AI panel / Story Bible) |
| Cmd+1…9 | Ctrl+1…9 | Jump to scene 1–9 in navigator |
| Opt+↑ / Opt+↓ | Alt+↑ / Alt+↓ | Move to previous/next scene |
| Cmd+Home / Cmd+End | Ctrl+Home / End | Go to start / end of script |

### AI Features
| Shortcut (Mac) | Shortcut (Win/Linux) | Action |
|---|---|---|
| Cmd+Shift+D | Ctrl+Shift+D | AI Dialogue Suggest |
| Cmd+Shift+C | Ctrl+Shift+C | AI Continue Scene |
| Cmd+Shift+I | Ctrl+Shift+I | Trigger Scene Intelligence (manual) |
| Cmd+Shift+L | Ctrl+Shift+L | AI Logline & Pitch Generator |

### Variants & Drafts
| Shortcut (Mac) | Shortcut (Win/Linux) | Action |
|---|---|---|
| Cmd+Alt+N | Ctrl+Alt+N | Create new dialogue variant |
| Cmd+Alt+→ | Ctrl+Alt+→ | Next dialogue variant |
| Cmd+Alt+← | Ctrl+Alt+← | Previous dialogue variant |
| Cmd+S | Ctrl+S | Force save (manual) |
| Cmd+Shift+S | Ctrl+Shift+S | Save as named draft |

### Find & Replace
| Shortcut (Mac) | Shortcut (Win/Linux) | Action |
|---|---|---|
| Cmd+F | Ctrl+F | Open Find |
| Cmd+H | Ctrl+H | Open Find & Replace |
| Cmd+G | Ctrl+G / F3 | Find next |
| Cmd+Shift+G | Ctrl+Shift+G | Find previous |

### Misc
| Shortcut (Mac) | Shortcut (Win/Linux) | Action |
|---|---|---|
| ? or Cmd+/ | ? or Ctrl+/ | Show keyboard shortcuts overlay |
| Escape | Escape | Close any modal/panel/overlay |

---

## Performance Criteria

- Shortcut response: < 16ms (same as keystroke latency requirement)
- Keyboard Shortcuts overlay: < 100ms to render
- Screen reader announcement: issued within 1 frame (16ms) of state change

---

## Offline Behaviour

- All keyboard shortcuts work offline (they are local editor operations)
- Accessibility features work offline

---

## Mobile Behaviour (375px)

- All touch targets are minimum 44×44dp (WCAG 2.1 SC 2.5.5)
- Mobile users can access a long-press context menu for block-type operations (replaces Tab shortcut on touch)
- TalkBack (Android) and VoiceOver (iOS) are the target screen readers for mobile

---

## Error States

**E1 — Shortcut Conflict**  
If the user's OS has claimed a shortcut that LIPILY uses (e.g., Cmd+Shift+I for Spotlight on some macOS versions): LIPILY logs the conflict in the console; the feature is still accessible via the toolbar button.

---

## Security Considerations

- No security implications for keyboard shortcuts

---

## Human Authorship Considerations

- No AI involvement

---

## DB Tables Touched

- `accessibility_preferences` (optional table for user's contrast mode, font size overrides)

```sql
CREATE TABLE accessibility_preferences (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL UNIQUE REFERENCES profiles(id),
  high_contrast_mode BOOLEAN NOT NULL DEFAULT FALSE,
  reduced_motion BOOLEAN NOT NULL DEFAULT FALSE,
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## API Endpoints Used

- `PATCH /accessibility-preferences` — save user's accessibility preferences

---

## Test Cases

**TC-001:** Every shortcut in the table tested via integration test; assert the correct action fires.  
**TC-002:** Screen reader (NVDA) test: navigate to a Character block → assert "Character Name block" is announced.  
**TC-003:** Tab through all interactive elements in the editor header → assert logical focus order; no focus loss.  
**TC-004:** System font size 200% → assert no text clipped; all labels visible.  
**TC-005:** High contrast OS mode → assert all text meets 4.5:1 contrast ratio (automated axe-core check on Flutter Web).  
**TC-006:** Keyboard Shortcuts overlay → open with `?`; dismiss with Escape.

---

## Out of Scope

- Custom user-defined keyboard shortcuts (Phase 4)
- WCAG 2.2 compliance (Phase 4)
- Full audit by a third-party accessibility firm (Phase 4 milestone)
