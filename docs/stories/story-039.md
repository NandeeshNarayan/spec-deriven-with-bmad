# story-039 — AI Multilingual Support

| Field | Value |
|---|---|
| **Story ID** | story-039 |
| **Title** | AI Multilingual Support |
| **Epic** | E03 — AI Feature Suite |
| **Phase** | 2 |
| **Sprint** | Sprint 5 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Arjun (the aspiring feature-film screenwriter) |
| **Estimated Effort** | 4 days |
| **Dependencies** | story-007 |

---

## Problem Statement

LIPILY serves writers globally — Hindi film writers in Mumbai, Tamil cinema writers in Chennai, Arabic-language writers in the Middle East, and multilingual writers writing for global OTT platforms. The tool must render non-Latin scripts correctly (right-to-left for Arabic, complex scripts for Hindi/Tamil), provide formatting that is culturally appropriate, and allow AI features to function in these languages with the same quality as in English.

---

## User Story

As a screenwriter writing in Hindi, Tamil, or Arabic, I want LIPILY to correctly render my script's text, provide Smart Flow Logic in my language, and deliver AI features in my language — so that I have the same professional writing experience as English-language writers.

---

## Acceptance Criteria

**GIVEN** a writer creates a new script and sets the language to "Hindi" (or "Tamil", "Arabic"),  
**WHEN** the editor loads,  
**THEN** the font used is a Unicode-compatible monospace font with support for the selected script (Noto Sans Mono for Devanagari; Noto Sans Tamil Mono for Tamil; for Arabic, the editor renders right-to-left with Noto Naskh Arabic); the editor container sets `textDirection: TextDirection.rtl` for Arabic; `ltr` for Hindi and Tamil.

**GIVEN** a writer types in Devanagari (Hindi) in the editor,  
**WHEN** the text renders,  
**THEN** Devanagari conjunct consonants and vowel signs are rendered correctly; there is no ligature breakage; the text appears at the same 12pt equivalent size as Courier Prime in English; line breaks occur at word boundaries (not within conjuncts).

**GIVEN** a writer types in Arabic,  
**WHEN** the text renders,  
**THEN** the text flows right-to-left; line breaks are at Arabic word boundaries; character-level Arabic joining rules are respected (connected vs. isolated letterforms); Scene Heading block still appears with bold styling (applied via CSS/Flutter TextStyle).

**GIVEN** the editor is in Hindi/Tamil/Arabic mode and the writer presses Tab in an Action block,  
**WHEN** Smart Flow runs,  
**THEN** the block transitions to a Character block; the character name is expected in the same script (no Latin-script enforcement); ALL CAPS enforcement is disabled for non-Latin scripts (it is meaningless for Devanagari/Arabic).

**GIVEN** the AI Scene Intelligence runs on a Hindi-language scene (story-007),  
**WHEN** the MCP service generates the analysis,  
**THEN** the system prompt instructs Claude to "Analyse the following Hindi-language screenplay scene and provide your analysis IN HINDI"; the five-pillar analysis is returned in Hindi; the scene summary for the navigator is also in Hindi.

**GIVEN** the AI Dialogue Suggest runs on a Tamil-language dialogue block (story-033),  
**WHEN** the suggestions are generated,  
**THEN** Claude is instructed to generate suggestions in Tamil; the 3 suggestions are in Tamil; no code-switching to English occurs unless the character's dialogue naturally uses English words.

**GIVEN** the PDF export is run for an Arabic-language script,  
**WHEN** the PDF is generated,  
**THEN** the PDF uses right-to-left text flow; Noto Naskh Arabic is embedded as a subset; all text renders correctly; scene headings are right-aligned (adapting the standard WGA layout for RTL).

**GIVEN** the Global Search (story-023) is used in a Hindi-language script,  
**WHEN** the user types a Hindi search query,  
**THEN** the PostgreSQL `simple` dictionary (language-agnostic) handles the search; results are returned correctly for Devanagari text; no romanisation is needed.

**GIVEN** the writer publishes a script to the social feed (story-041) with language set to "Tamil",  
**WHEN** the script appears in the discovery feed,  
**THEN** the language is shown as a badge on the script card ("Tamil"); the feed can be filtered by language.

**GIVEN** the writer has set the app UI language to "Hindi" (from Profile → Settings → Language),  
**WHEN** the app loads,  
**THEN** all UI strings (buttons, labels, tooltips, error messages) are shown in Hindi using the `flutter_localizations` package with `intl` ARB files; the fallback language is English.

---

## Supported Languages for v1

| Language | Script | Direction | AI Model Support |
|---|---|---|---|
| English | Latin | LTR | ✓ Full |
| Hindi | Devanagari | LTR | ✓ Full |
| Tamil | Tamil script | LTR | ✓ Full |
| Arabic | Arabic | RTL | ✓ Full |
| Urdu | Nastaliq/Naskh | RTL | ✓ Full (same as Arabic) |

---

## Performance Criteria

- RTL editor render: same < 16ms keystroke latency as LTR
- Font loading (Noto fonts bundled in app, not fetched): first render < 100ms
- AI analysis in Hindi/Tamil: same < 8 second SLA as English
- PDF RTL generation: within the < 30s PDF SLA

---

## Offline Behaviour

- All language-specific font rendering works offline (fonts are bundled)
- AI features in non-English languages require internet (same as English)

---

## Mobile Behaviour (375px)

- RTL mode: all UI layout mirrors (navigation is on the right side, back button is on the left for RTL users)
- Keyboard input: uses device's native keyboard for each language; no in-app keyboard
- Character names in non-Latin scripts: centred on the page (same as Latin)

---

## Error States

**E1 — Unsupported Language Selected**  
Writer selects a language not in the supported list (e.g., Japanese): show advisory "Japanese is coming soon. Defaulting to English. You can still write in Japanese, but some formatting features may not be optimised."

**E2 — Font Rendering Failure**  
If the Noto font for a script fails to load: fall back to the system's default Unicode font; log to Sentry; show advisory: "Custom font could not be loaded. Using system font."

**E3 — AI Model Returns English Despite Non-English Instruction**  
Claude code-switches to English in a Hindi analysis (known LLM behaviour): the MCP service detects language of response using a simple heuristic (% non-ASCII characters < 30%); re-prompts once with stricter instruction; if still English, serves the English analysis with a disclaimer "Analysis in English — model did not maintain Hindi."

---

## Security Considerations

- No additional security concerns beyond the standard AI security rules
- RTL text input is handled by the Flutter TextEditingController natively; no special injection risk

---

## Human Authorship Considerations

- Same Human Authorship First Principle applies in all languages
- AI suggestions in non-English languages are all `ai_generated` and require writer confirmation

---

## DB Tables Touched

- `scripts` (UPDATE `language` field)
- `blocks` (content stored as UTF-8; no schema changes needed)
- `profiles` (UPDATE `ui_language` preference)

---

## API Endpoints Used

- `PATCH /scripts/:id` — update script language
- `PATCH /profiles/:id` — update UI language preference
- All AI endpoints accept a `language` parameter that is forwarded to the MCP service

---

## Test Cases

**TC-001:** Create Hindi script → open editor → assert Devanagari renders without ligature breakage.  
**TC-002:** Create Arabic script → open editor → assert text flows RTL; UI mirrors.  
**TC-003:** Tab in Action block (Hindi) → assert block type changes; no ALL CAPS applied to Devanagari.  
**TC-004:** Run Scene Intelligence on Hindi scene → assert analysis returned in Hindi.  
**TC-005:** AI Dialogue Suggest on Tamil block → assert suggestions in Tamil.  
**TC-006:** Export Arabic script to PDF → assert RTL flow in PDF; Noto font embedded.  
**TC-007:** Search in Hindi → assert results returned for Devanagari query.  
**TC-008:** Set UI language to Hindi → assert all UI strings in Hindi.

---

## Out of Scope

- Japanese, Chinese, Korean support (Phase 2)
- Script translation (script in English → translate to Hindi) — Phase 2
- Romanisation / transliteration — Phase 2
