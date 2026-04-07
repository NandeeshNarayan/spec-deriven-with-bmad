# story-009 — Collaboration Invite & Roles

| Field | Value |
|---|---|
| **Story ID** | story-009 |
| **Title** | Collaboration Invite & Roles |
| **Epic** | E04 — Collaboration & CRDT |
| **Phase** | 2 |
| **Sprint** | Sprint 2 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Priya (the TV staff writer) |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-003 |

---

## Problem Statement

Professional screenwriting is collaborative. Staff writers on TV shows, writing partnerships, and showrunner/writer relationships all require fine-grained access control. LIPILY must support inviting collaborators by email with distinct roles (Owner, Editor, Commenter, Viewer), role change, and removal — with all access enforced server-side via RLS.

---

## User Story

As a script owner, I want to invite other writers to my script with specific roles — so that my collaborators have exactly the access they need, no more and no less.

---

## Acceptance Criteria

**GIVEN** the script owner is in the editor and opens "Share" (Cmd+Shift+S),  
**WHEN** the Share modal opens,  
**THEN** it shows: a list of current collaborators with their role badges, an "Invite People" input field (accepts email addresses), a role selector (Owner / Editor / Commenter / Viewer), and an "Invite" button.

**GIVEN** the owner types an email and selects role "Editor" and clicks "Invite",  
**WHEN** the invite is submitted via `POST /scripts/:id/invites`,  
**THEN** a row is inserted into `invites` table with `status = 'pending'`; a Resend email is dispatched with a personalised invite link containing a signed JWT invite token (24-hour expiry); the collaborators list in the modal shows the pending invite with a "Pending" badge.

**GIVEN** the invited user clicks the invite link in their email,  
**WHEN** the link opens LIPILY (deep link),  
**THEN** if the user is not authenticated, they are directed to `/auth` first with the invite token preserved in the URL; after authentication, the invite is accepted via `POST /invites/:token/accept`; a `collaborators` row is inserted with `role = 'editor'`; the user is navigated to `/editor/:script_id`.

**GIVEN** the invite link has expired (> 24 hours),  
**WHEN** the user clicks it,  
**THEN** the app shows an error screen: "This invite link has expired. Ask the script owner to send a new invite." with a "Go to Dashboard" button; no collaborator row is created.

**GIVEN** the owner wants to change a collaborator's role,  
**WHEN** they click the role badge next to a collaborator's name in the Share modal,  
**THEN** a dropdown shows all available roles (excluding Owner if downgrading); selecting a new role calls `PATCH /scripts/:id/collaborators/:user_id` with the new role; the change is reflected immediately in the modal.

**GIVEN** the owner removes a collaborator via the "Remove" action in the Share modal,  
**WHEN** they confirm the removal,  
**THEN** `DELETE /scripts/:id/collaborators/:user_id` is called; the collaborator's row is deleted; if the removed user is currently in the editor, their session is terminated with a message: "Your access to this script has been removed by the owner."

**GIVEN** a script has at least one accepted collaborator,  
**WHEN** the script card is rendered on the dashboard,  
**THEN** it shows avatar stack (up to 3 avatars + "+N" overflow) of collaborators below the script title.

**GIVEN** a Studio team has the "Writers' Room" mode enabled,  
**WHEN** a new collaborator is invited,  
**THEN** the invite modal shows an additional checkbox: "Add to Writers' Room" which assigns the `writers_room_member` flag in the `collaborators` table.

**GIVEN** a Commenter-role collaborator is in the editor,  
**WHEN** they try to type in a block,  
**THEN** the text input is disabled; a tooltip appears on hover: "You have comment-only access. Ask the script owner to upgrade your role to edit."; they can still add comments.

**GIVEN** a Viewer-role collaborator is in the editor,  
**WHEN** they open the editor,  
**THEN** all edit controls are disabled and hidden; the formatting toolbar is not shown; they see a read-only banner: "Viewing in read-only mode"; comments are also disabled.

---

## Role Permission Matrix

| Permission | Owner | Editor | Commenter | Viewer |
|---|---|---|---|---|
| Edit blocks | ✓ | ✓ | ✗ | ✗ |
| Add comments | ✓ | ✓ | ✓ | ✗ |
| View script | ✓ | ✓ | ✓ | ✓ |
| Invite collaborators | ✓ | ✗ | ✗ | ✗ |
| Change roles | ✓ | ✗ | ✗ | ✗ |
| Export script | ✓ | ✓ | ✗ | ✗ |
| Delete script | ✓ | ✗ | ✗ | ✗ |
| Create branch | ✓ | ✓ | ✗ | ✗ |
| Merge branch | ✓ | ✗ | ✗ | ✗ |
| Publish script | ✓ | ✗ | ✗ | ✗ |

---

## Performance Criteria

- Share modal open: < 200ms
- Invite dispatch (email sent): < 3 seconds
- Role change update reflected in UI: < 300ms (optimistic)
- Collaborator list load (up to 20 collaborators): < 200ms

---

## Offline Behaviour

- The Share modal is disabled when offline with a banner: "Managing collaborators requires an internet connection."
- The collaborator list (read-only) is displayed from cache even offline
- Role permission enforcement (viewer/commenter can't edit) is enforced both client-side AND server-side regardless of connectivity

---

## Mobile Behaviour (375px)

- Share modal opens as a full-screen bottom sheet
- Current collaborators: list of row tiles (avatar + name + role badge + "..." menu)
- Invite input: full-width email field with keyboard type `emailAddress`
- Role selector: segmented control (Owner / Editor / Commenter / Viewer)
- "Invite" button: full-width, primary colour, height 56px

---

## Error States

**E1 — Invite to Non-Existing Email**  
If the invited email is not registered in Supabase: the invite is still created and the email is sent; on clicking the link, the user is directed to `/auth` to create an account, then the invite is accepted.

**E2 — Self-Invite**  
If the owner invites their own email: show inline error "You can't invite yourself to your own script."

**E3 — Duplicate Invite**  
If a pending invite already exists for the email: show inline error "An invite has already been sent to [email]. Resend?" with a "Resend" button.

**E4 — Studio Plan Required for >5 Editors**  
Pro plan: maximum 2 editors per script. Attempting to invite a 3rd editor on Pro: show modal "Upgrade to Studio for unlimited collaborators per script."

**E5 — Invite Token Tampering**  
If the invite JWT is tampered with: the Rust API's `jsonwebtoken` verification fails; return 401; show "Invalid invite link. Please request a new invite from the script owner."

---

## Security Considerations

- Invite tokens are RS256-signed JWTs with 24-hour expiry; they cannot be forged without the private key
- `collaborators` RLS: only the script owner can INSERT, UPDATE, DELETE collaborator rows
- `PATCH /blocks/:id` checks the `collaborators` role server-side; Editor or Owner only; no client-side trust
- Removed collaborators immediately lose access; their active WebSocket connection is terminated via a server-side broadcast: `{"type": "access_revoked", "userId": "..."}`
- Invite emails are sent to the exact invited email only; no BCC or forwarding; Resend logs delivery

---

## Human Authorship Considerations

Not applicable — this story involves no AI-generated content.

---

## DB Tables Touched

- `invites` (INSERT pending invite, UPDATE status to accepted/expired)
- `collaborators` (INSERT on accept, UPDATE role, DELETE on remove)
- `scripts` (SELECT — verify ownership before invite)
- `profiles` (SELECT — for collaborator name/avatar in modal)

---

## API Endpoints Used

- `POST /scripts/:id/invites` — create and send invite
- `POST /invites/:token/accept` — accept invite (after auth)
- `GET /scripts/:id/collaborators` — list current collaborators
- `PATCH /scripts/:id/collaborators/:user_id` — change role
- `DELETE /scripts/:id/collaborators/:user_id` — remove collaborator

---

## Test Cases

**TC-001:** Send invite to new email → assert `invites` row created with `status = 'pending'` and Resend email sent.  
**TC-002:** Click valid invite link → assert `collaborators` row created with correct role.  
**TC-003:** Click expired invite link → assert error screen shown, no collaborator row created.  
**TC-004:** Change role from Editor to Commenter → assert DB updated; editor controls disabled for that user.  
**TC-005:** Remove collaborator → assert row deleted; if they're in the editor, their session is terminated.  
**TC-006:** Commenter-role user tries to type in editor → assert input disabled and tooltip shown.  
**TC-007:** Owner invites their own email → assert inline error "You can't invite yourself."  
**TC-008:** Tampered invite JWT → assert 401 returned from Rust API.  
**TC-009:** Pro user tries to add 3rd editor → assert upgrade modal shown.

---

## Out of Scope

- Real-time cursor presence (story-010)
- Comment threads (story-011)
- Writers' Room mode (story-053)
- Showrunner controls (story-053)
- Collaboration audit log (Phase 2)
