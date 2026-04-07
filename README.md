# spec-deriven-with-bmad
# AGENT TASK — LIPILY Documentation Suite Generation

## ⚠️ READ THIS ENTIRE FILE BEFORE DOING ANYTHING

You are a **top 1% senior staff engineer and technical documentation specialist**.  
Your task is to generate a complete, production-grade planning documentation suite  
for a product called **LIPILY**, based entirely on the specification file at:

```
Prompt.md  (in this same workspace root)
```

---

## 🔴 CRITICAL RULES — NON-NEGOTIABLE

1. **DO NOT use the terminal.** Do not run any shell commands, CLI tools, scripts,  
   or package managers. Every single file must be created manually using the  
   `create_file` tool only.

2. **DO NOT truncate ANY file.** If you are about to write `...continues`, `// abbreviated`,  
   `[remaining items follow same pattern]`, `[see above]`, `TBD`, or any equivalent shortcut —  
   **STOP and write the full content.** No exceptions. Every list item. Every table row.  
   Every code example. Every acceptance criterion. Every error state.

3. **DO NOT reference any pre-existing files** other than `Prompt.md`.  
   Build everything from scratch, fresh, from `Prompt.md` alone.

4. **DO NOT hallucinate** technology choices, table names, field names, route paths,  
   component names, or any architectural details. Every decision must be  
   traceable to a specific section in `Prompt.md`. If something is not in `Prompt.md`,  
   do not invent it.

5. **DO NOT skip any file.** All 61 files listed in the target structure below must  
   be created. Every story from story-001 to story-055 must be its own complete file.

6. **DO NOT create any file twice** or skip any section within a file.

7. **Read `Prompt.md` in full before generating any file.** The prompt is 2,528 lines.  
   Every part matters. Understand the full scope before you begin.

---

## 📁 TARGET FILE STRUCTURE TO CREATE

Create ALL files in this exact folder structure (relative to workspace root):

```
docs/
├── constitution.md
├── prd.md
├── architecture.md
├── AGENTS.md
├── STATUS.md
└── stories/
    ├── story-001-project-setup-infrastructure.md
    ├── story-002-authentication.md
    ├── story-003-dashboard.md
    ├── story-004-core-editor-engine.md
    ├── story-005-scene-navigator.md
    ├── story-006-focus-mode.md
    ├── story-007-scene-intelligence-5-pillar.md
    ├── story-008-story-bible-panel.md
    ├── story-009-collaboration-invite-roles.md
    ├── story-010-realtime-collaboration-yjs.md
    ├── story-011-comment-threads.md
    ├── story-012-git-style-versioning-drafts.md
    ├── story-013-branch-editing-merge-requests.md
    ├── story-014-smart-deletion-recovery.md
    ├── story-015-alternate-dialogue-variants.md
    ├── story-016-character-location-manager.md
    ├── story-017-beat-board.md
    ├── story-018-offline-first-powersync.md
    ├── story-019-export-production-pdf.md
    ├── story-020-export-fountain-fdx.md
    ├── story-021-script-import.md
    ├── story-022-readonly-sharing-link.md
    ├── story-023-global-search.md
    ├── story-024-inline-scene-tagging.md
    ├── story-025-title-page-editor.md
    ├── story-026-find-replace.md
    ├── story-027-script-formatting-preferences.md
    ├── story-028-script-statistics-goals.md
    ├── story-029-revision-mode-production.md
    ├── story-030-storyboard-shot-visualization.md
    ├── story-031-script-breakdown-auto-generation.md
    ├── story-032-inspiration-research-mode.md
    ├── story-033-ai-dialogue-suggest.md
    ├── story-034-ai-continue-scene-writers-block.md
    ├── story-035-ai-continuity-checker.md
    ├── story-036-ai-pacing-structure-analyzer.md
    ├── story-037-ai-logline-pitch-generator.md
    ├── story-038-ai-character-voice-guardian.md
    ├── story-039-ai-multilingual-support.md
    ├── story-040-writer-profiles-usernames.md
    ├── story-041-script-publishing.md
    ├── story-042-listen-the-script.md
    ├── story-043-social-feed-discovery.md
    ├── story-044-leaderboard.md
    ├── story-045-review-submission.md
    ├── story-046-reviewer-program-payouts.md
    ├── story-047-producer-discovery-portal.md
    ├── story-048-subscription-stripe.md
    ├── story-049-notifications.md
    ├── story-050-content-moderation-reporting.md
    ├── story-051-keyboard-shortcuts-accessibility.md
    ├── story-052-onboarding-flow.md
    ├── story-053-writers-room-mode.md
    ├── story-054-dialogue-read-aloud-tts.md
    └── story-055-admin-dashboard.md
```

---

## 📋 GENERATION ORDER (MANDATORY — DO NOT CHANGE)

Generate files in this exact sequence:

1. `docs/constitution.md`
2. `docs/architecture.md`
3. `docs/prd.md`
4. `docs/AGENTS.md`
5. `docs/STATUS.md`
6. `docs/stories/story-001-project-setup-infrastructure.md`
7. `docs/stories/story-002-authentication.md`
8. ... continue through ...
9. `docs/stories/story-055-admin-dashboard.md`

---

## 📖 SOURCE OF TRUTH — HOW TO USE `Prompt.md`

`Prompt.md` is structured in 12 parts. Here is what each part maps to:

| Prompt.md Part | Use For |
|---|---|
| Part 1 — Product Vision (Layers 1, 2, 3) | All story problem statements, user stories, acceptance criteria |
| Part 2 — Tech Stack (LOCKED) | constitution.md §1, architecture.md §1, AGENTS.md §1 |
| Part 3 — Security Architecture | constitution.md §7, every story's Security Considerations section |
| Part 4 — Pricing Tiers | constitution.md §8, prd.md §6 Feature Gate Table, every story's subscription gating |
| Part 5 — User Personas | prd.md §3, every story's Persona field |
| Part 6 — Build Phases | STATUS.md sprint plan, prd.md §4 Epic Table |
| Part 7 — Files To Create | Exact file list (above) |
| Part 8 — Core Planning File Specifications | Detailed spec for constitution.md, prd.md, architecture.md |
| Part 9 — AGENTS.md Specification | Detailed spec for AGENTS.md (all 10 sections) |
| Part 10 — STATUS.md Specification | Detailed spec for STATUS.md |
| Part 11 — All 55 Story File Specifications | Detailed spec for every story file |
| Part 12 — Final Generation Instructions | Quality checklist — run this checklist on EVERY file before outputting it |

---

## 📐 REQUIRED SECTIONS IN EVERY FILE

### `constitution.md` must contain all 12 sections:
- §0 Human Authorship Principle (Supreme Law)
- §1 Locked Tech Stack (every technology, version, purpose, which service, marked LOCKED)
- §2 Hard Architecture Rules (25+ rules, full prose)
- §3 Performance Non-Negotiables (15+ targets with exact ms/s numbers)
- §4 Offline-First Rules (PowerSync config, Drift schema, retry strategy, conflict resolution)
- §5 Editor Rules (Fountain canonical format, all 7 block types, full 7×2 Smart Flow matrix, SmartType rules, autosave, Yjs structure)
- §6 AI Rules (all triggers, rate limits, ContentSource enum, approval queue, model selection, fallback chain)
- §7 Security Rules (all 20 rules from Prompt.md Part 3, written in full)
- §8 Scope (v1/v2/v3, what is explicitly out of scope)
- §9 Data Model Constraints (UUIDs, timestamps, position ordering, soft deletes, ContentSource enum)
- §10 Deployment Rules (Cloud Run config, GitHub Actions, zero-downtime, rollback, secret rotation)
- §11 Multilingual Rules (UTF-8, RTL, font fallback, AI language detection)
- §12 Accessibility Rules (WCAG 2.1 AA, tap targets, contrast ratios, keyboard navigation)

### `architecture.md` must contain all 9 sections:
- §1 System Architecture Overview (full Mermaid diagram, read path + write path prose)
- §2 Complete PostgreSQL Schema (full SQL DDL for ALL 40+ tables with CREATE TABLE, columns, types, constraints, CHECK constraints, indexes, RLS policies)
- §3 Drift Local Schema (complete Dart Drift table definitions)
- §4 Rust API Contract (every endpoint: method, path, auth, request struct, response struct, error codes — grouped by domain)
- §5 Riverpod Provider Hierarchy (full provider tree in Dart pseudocode, all 25+ providers)
- §6 go_router Route Tree (all named routes with path, name constant, parameters, auth guard, redirect logic)
- §7 Editor Widget Tree (full Flutter widget hierarchy as indented pseudocode)
- §8 Yjs CRDT + WebSocket Architecture (Y.Doc structure, WebSocket endpoint, connection lifecycle, cursor presence, Flutter client reconnection)
- §9 "Listen the Script" Generation Pipeline (all 8 BullMQ job steps + full Mermaid sequence diagram)

### `prd.md` must contain all 7 sections:
- §1 Problem Statement
- §2 Market Positioning (full competitor table with all 8 rows × 9 columns)
- §3 User Personas (all 6 personas: Arjun, Priya, Marcus, Aisha, Dev, Rajesh — each with full profile)
- §4 Epic Table (all epics with ID, name, description, priority, phase, story count)
- §5 All 55 User Stories in GIVEN/WHEN/THEN format with ≥8 acceptance criteria per story
- §6 Feature Gate Table (every feature mapped to Free/Pro/Studio with exact limits)
- §7 Success Metrics (v1, v2, v3 targets with specific numbers)

### `AGENTS.md` must contain all 10 sections (from Prompt.md Part 9):
- Section 1: Agent Role & Philosophy
- Section 2: ALWAYS Rules (30+ rules)
- Section 3: NEVER Rules (30+ rules)
- Section 4: Project Folder Structure (to file level for frontend/, backend/, mcp-service/)
- Section 5: Dart Style Guide (20 rules + complete runnable code examples for all 7 patterns)
- Section 6: Rust Style Guide (20 rules + complete runnable code examples for all 7 patterns)
- Section 7: TypeScript Style Guide (15 rules + complete runnable code examples for all 5 patterns)
- Section 8: ContentSource Pattern (full description + code in both Dart and TypeScript)
- Section 9: Error Handling Patterns (full AppError in Dart + Rust, error bubbling chain, toast vs inline matrix)
- Section 10: Story Execution Protocol (all 17 steps)

### `STATUS.md` must contain:
- Header: LIPILY — Sprint Tracker, Current Sprint, Sprint Goal, Sprint Dates
- Story Status Table (all 55 stories, columns: Story ID | Title | Epic | Phase | Priority | Status | Sprint | Assigned | Blockers)
- All 55 stories with Status: DRAFT
- Sprint Plan Table (Sprint 0 through Sprint 7)
- Blockers Section (empty)
- Decisions Log Section (empty)

### Every story file (`story-NNN-*.md`) must contain ALL of these sections:
1. **Story ID, Title, Epic, Phase, Priority, Status: DRAFT**
2. **Persona** — which persona this story primarily serves
3. **Problem Statement** — 1 paragraph explaining why this matters
4. **User Story** — "As a [persona], I want [action] so that [outcome]"
5. **GIVEN/WHEN/THEN Acceptance Criteria** — minimum 8 distinct criteria
6. **Performance Criteria** — specific ms/s targets (reference constitution.md §3)
7. **Offline Behavior** — what works, what degrades, what queues to Drift
8. **Mobile Behavior** — 375px specific layout and gesture considerations
9. **Error States** — minimum 3 failure scenarios with exact UI behavior
10. **Security Considerations** — auth checks, rate limits, tier enforcement
11. **Human Authorship Considerations** — (for all AI stories) what is editable, what shows AI badge, what goes through approval queue
12. **Dependencies** — list of story IDs that must be DONE first
13. **DB Tables Touched** — list every table this story reads/writes
14. **API Endpoints Used** — list every Rust API endpoint this story calls
15. **Test Cases** — minimum 5: happy path + 4 edge cases, written as proper test descriptions
16. **Out of Scope** — explicit list of what this story does NOT include

---

## ✅ QUALITY CHECKLIST — RUN ON EVERY FILE BEFORE OUTPUTTING

Before writing any file, verify every item:

- [ ] Does every DB table have RLS enabled and explicit allow policies?
- [ ] Does every API endpoint have JWT verification noted (or explicitly marked public with reason)?
- [ ] Does every AI feature note: manual trigger, AI badge, content_source tracking, approval queue, editable after acceptance?
- [ ] Does every story have offline behavior documented?
- [ ] Does every story have mobile (375px) behavior documented?
- [ ] Does every story have minimum 8 acceptance criteria?
- [ ] Does every story have minimum 3 error states?
- [ ] Does every story have security considerations?
- [ ] Does every story have subscription tier gating where applicable?
- [ ] Is the Human Authorship Principle applied to every AI story (§0 reference)?
- [ ] Are all 20 security rules from Prompt.md Part 3 referenced where applicable?
- [ ] Is the TypeScript MCP service never writing directly to Supabase in any story?
- [ ] Are all Rust handlers using sqlx compile-time queries?
- [ ] Are all Flutter providers using Riverpod AsyncNotifier?
- [ ] Does every story list its DB tables, API endpoints, and story dependencies?
- [ ] Are all 55 stories accounted for in STATUS.md?
- [ ] Is architecture §9 ("Listen the Script" pipeline) a complete Mermaid sequence diagram?
- [ ] Does AGENTS.md §5 have complete runnable Dart examples for all 7 patterns?
- [ ] Does AGENTS.md §6 have complete runnable Rust examples for all 7 patterns?
- [ ] Does AGENTS.md §7 have complete runnable TypeScript examples for all 5 patterns?
- [ ] Is the NO TRUNCATION RULE followed in every file?

---

## ⚠️ NO TRUNCATION RULE (REPEAT)

If at any point you would write any of the following — **STOP and write it out in full instead**:

- `... (continues)`
- `... (similar pattern for remaining items)`
- `// abbreviated for brevity`
- `[remaining fields follow same pattern]`
- `[see above for similar implementation]`
- `TBD`
- `TODO`
- Any `...` shortcut

Every list item. Every table row. Every code example.  
Every acceptance criterion. Every error state.  
**No exceptions.**

---

## 🏁 AFTER ALL FILES ARE CREATED — FINAL REVIEW

Once all 61 files have been created, perform a **self-review** and output a structured report:

```
## LIPILY Documentation Suite — Creation Review

### Files Created
| # | File Path | Sections | Word Count (est.) | Status |
|---|-----------|----------|-------------------|--------|
| 1 | docs/constitution.md | §0–§12 | ~XXXX words | ✅ Complete |
| 2 | docs/architecture.md | §1–§9 | ~XXXX words | ✅ Complete |
...
| 61 | docs/stories/story-055-admin-dashboard.md | All 16 sections | ~XXXX words | ✅ Complete |

### Quality Checklist Results
[Run through every item in the quality checklist above. Mark ✅ or ❌ with specific file reference for any failure.]

### Coverage Verification
- Total story files created: XX / 55
- Stories with ≥ 8 acceptance criteria: XX / 55
- Stories with offline behavior documented: XX / 55
- Stories with mobile behavior documented: XX / 55
- Stories with ≥ 3 error states: XX / 55
- AI stories with Human Authorship section: XX / XX AI stories
- AI stories with approval queue documented: XX / XX AI stories

### Architecture Completeness
- DB tables defined in architecture.md §2: XX tables
- API endpoints defined in architecture.md §4: XX endpoints
- Riverpod providers defined in architecture.md §5: XX providers
- Routes defined in architecture.md §6: XX routes

### Potential Issues or Gaps Found
[List any ambiguities resolved during generation, any gaps found in the spec, any decisions made — these should also be noted as entries in STATUS.md Decisions Log]

### Recommendation for Next Step
[State which story (story-001) is the correct starting point for the developer agent and confirm the constitution.md + architecture.md are sufficient context for an agent to begin building without confusion]
```

---

## 🚀 BEGIN NOW

1. Read `Prompt.md` in its entirety first.  
2. Then begin generating files in the exact order specified above.  
3. Use `create_file` for every file — no terminal, no shortcuts.  
4. Do not stop until all 61 files are created and the final review is output.
