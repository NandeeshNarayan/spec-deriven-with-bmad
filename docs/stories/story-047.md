# story-047 — Producer Discovery Portal

| Field | Value |
|---|---|
| **Story ID** | story-047 |
| **Title** | Producer Discovery Portal |
| **Epic** | E12 — Review Marketplace |
| **Phase** | 3 |
| **Sprint** | Sprint 6 |
| **Priority** | P2 |
| **Status** | DRAFT |
| **Persona** | Vikram (the producer) |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-040, story-041, story-044 |

---

## Problem Statement

Producers need a professional, private portal to discover scripts — separate from the public social feed. They need to filter by genre, format, page count, and leaderboard ranking; save scripts to a private reading list; and contact writers privately. Writers need to opt in to producer discovery to maintain privacy control.

---

## User Story

As a producer, I want a private portal where I can discover curated, high-quality scripts based on my specific criteria and contact writers who have opted in — so that I can efficiently find material worth developing.

---

## Acceptance Criteria

**GIVEN** a user applies for Producer access at Settings → Producer Portal,  
**WHEN** they submit their application with company name, IMDB page/portfolio URL, and a brief statement of intent,  
**THEN** `producer_profiles.status = 'pending'`; the admin queue (story-055) shows the application; the applicant is emailed: "Your producer application is under review."

**GIVEN** an admin approves the producer application,  
**WHEN** approval is saved,  
**THEN** `producer_profiles.status = 'approved'`; the producer can now access the Producer Portal at `/producer`; a badge appears on their profile: "Verified Producer."

**GIVEN** a producer is in the Producer Portal,  
**WHEN** the portal loads,  
**THEN** they see a dedicated discovery interface with advanced filters: Genre (multi-select), Format (multi-select), Page Count range (slider, 15–200), Minimum Leaderboard Score (slider), Language, and "Has Listen" toggle; scripts are shown in a professional table/card layout optimised for browsing.

**GIVEN** a producer applies filters and browses scripts,  
**WHEN** results appear,  
**THEN** only scripts where the writer has enabled "Producer Discovery" in their Publishing settings are shown; scripts with `visibility = 'public'` and `producer_discoverable = true` are included; the first 10 pages are always readable (preview).

**GIVEN** a producer finds a script they want to track,  
**WHEN** they click "Save to Reading List",  
**THEN** the script is added to `producer_saved_scripts`; the producer can view all saved scripts in a "My Reading List" section; they can add private notes per script (max 500 chars); these notes are private and never visible to writers.

**GIVEN** a producer wants to contact a writer about a script,  
**WHEN** they click "Contact Writer",  
**THEN** a contact request form opens with fields: message (max 500 chars), company name (pre-filled from producer profile), and purpose (Option / Development / General Interest); on submit, a `producer_contact_requests` row is inserted; the writer receives an in-app notification and email: "A verified producer is interested in your script '[Title]'."

**GIVEN** a writer receives a producer contact request,  
**WHEN** they view it in their notifications,  
**THEN** they see the producer's name, company, and message; they can Reply (opens a message thread) or Decline; the producer's email is NOT revealed to the writer until the writer replies (privacy protection for producer); the writer's email is NOT revealed to the producer until the writer replies.

**GIVEN** a writer wants to opt out of producer discovery,  
**WHEN** they toggle "Producer Discovery" off in their Publishing settings,  
**THEN** all their scripts are immediately removed from the Producer Portal; any pending contact requests are voided; they are notified that disabling this also removes pending requests.

---

## Performance Criteria

- Producer Portal load: < 1 second (filtered query with index on score + filters)
- Contact request delivery: writer notification within 5 seconds
- Saved scripts list: < 500ms

---

## Offline Behaviour

- Producer Portal is network-dependent
- Saved scripts list is not cached offline (contains private notes)

---

## Mobile Behaviour (375px)

- Producer Portal: card layout (one column); advanced filters in a bottom sheet
- Saved scripts: simple list with notes accordion
- Contact form: full-screen modal

---

## Error States

**E1 — Producer Application Rejected**  
Email: "Thank you for applying to the LIPILY Producer Portal. We could not verify your producer credentials. You may reapply in 90 days with additional documentation."

**E2 — Writer Has Disabled Producer Discovery**  
If a producer tries to contact a writer whose `producer_discoverable` has since been set to false: "This writer is no longer available for producer contact."

**E3 — Contact Request Rate Limit**  
Producer can send max 10 contact requests per day; beyond that: "You've reached the daily contact limit. Try again tomorrow."

---

## Security Considerations

- Producer Portal is only accessible to approved producers (`producer_profiles.status = 'approved'`)
- Contact requests are RLS-restricted; only the recipient writer and the sending producer can see them
- Producer notes on saved scripts are RLS-restricted to the producer only
- Producer verification is admin-approved; no self-designation

---

## Human Authorship Considerations

- No AI involvement

---

## DB Tables Touched

- `producer_profiles` (INSERT/UPDATE)
- `producer_saved_scripts` (INSERT/DELETE)
- `producer_contact_requests` (INSERT)

```sql
CREATE TABLE producer_profiles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL UNIQUE REFERENCES profiles(id),
  company_name TEXT NOT NULL,
  portfolio_url TEXT,
  statement TEXT NOT NULL,
  status TEXT NOT NULL DEFAULT 'pending',
  applied_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  approved_at TIMESTAMPTZ
);

CREATE TABLE producer_saved_scripts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  producer_id UUID NOT NULL REFERENCES producer_profiles(id),
  publication_id UUID NOT NULL REFERENCES script_publications(id),
  notes TEXT,
  saved_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(producer_id, publication_id)
);

CREATE TABLE producer_contact_requests (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  producer_id UUID NOT NULL REFERENCES producer_profiles(id),
  writer_id UUID NOT NULL REFERENCES profiles(id),
  publication_id UUID NOT NULL REFERENCES script_publications(id),
  message TEXT NOT NULL,
  purpose TEXT NOT NULL,
  status TEXT NOT NULL DEFAULT 'pending',
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## API Endpoints Used

- `POST /producer-applications` — submit producer application
- `GET /producer/scripts?genre=&format=&min_score=&has_listen=` — discovery query
- `POST /producer/saved-scripts` — save script to reading list
- `PATCH /producer/saved-scripts/:id` — update notes
- `POST /producer/contact-requests` — send contact request to writer
- `GET /producer/contact-requests/sent` — producer views their sent requests
- `GET /producer/contact-requests/received` — writer views received requests

---

## Test Cases

**TC-001:** Submit producer application → `producer_profiles` row with status 'pending'.  
**TC-002:** Admin approves → status 'approved'; portal accessible.  
**TC-003:** Producer filters by Genre=Drama, Format=Feature → only Drama Feature scripts with `producer_discoverable = true` shown.  
**TC-004:** Save script to reading list → appears in Reading List with notes field.  
**TC-005:** Contact writer → writer receives in-app + email notification.  
**TC-006:** Writer disables Producer Discovery → scripts removed from portal.  
**TC-007:** Producer sends 11th contact request in 24 hours → rate limit error.

---

## Out of Scope

- Direct messaging thread after initial contact (Phase 4 — full messaging system)
- Producer pitching to writers (reverse direction) — Phase 4
- Script option contracts or deal flow within LIPILY (Phase 4)
