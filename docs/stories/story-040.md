# story-040 — Writer Profiles & Usernames

| Field | Value |
|---|---|
| **Story ID** | story-040 |
| **Title** | Writer Profiles & Usernames |
| **Epic** | E10 — Social Platform |
| **Phase** | 3 |
| **Sprint** | Sprint 5 |
| **Priority** | P1 |
| **Status** | DRAFT |
| **Persona** | Priya (the professional TV writer) |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-002, story-003 |

---

## Problem Statement

Writers need a public identity on LIPILY — a profile page that showcases their published work, bio, and credentials. This is the social foundation that enables the leaderboard, feed, review marketplace, and producer discovery portal to function. Without profiles, there is no social graph.

---

## User Story

As a screenwriter, I want a public profile page at `lipily.com/@myusername` that showcases my published scripts and bio — so that producers, collaborators, and other writers can discover my work.

---

## Acceptance Criteria

**GIVEN** a writer first logs in and has no username set,  
**WHEN** they navigate to the dashboard,  
**THEN** a banner prompts them to claim their username: "Claim your @username to publish your scripts and connect with the community."

**GIVEN** a writer enters a username in the claim modal,  
**WHEN** they submit,  
**THEN** the username is validated: 3–30 characters, alphanumeric + underscores only, no consecutive underscores, no reserved words (admin, lipily, support, api); if valid and not taken, `profiles.username` is saved and the modal closes; if taken, error "That username is taken" appears.

**GIVEN** a writer has a username,  
**WHEN** they visit their profile page at `/@username`,  
**THEN** the page shows: avatar (or initials), display name, bio (max 160 characters), writer type badge (WGA Member / Guild Member / Independent / Student), follower count, following count, total published scripts, and a grid of published script cards.

**GIVEN** a writer uploads a profile photo,  
**WHEN** the upload completes,  
**THEN** the image is stored in S3 at `s3://lipily-avatars/{user_id}.webp`; resized to 256×256 WebP; old photo is deleted from S3; the CDN URL is saved to `profiles.avatar_url`.

**GIVEN** any authenticated user views another writer's profile,  
**WHEN** the page loads,  
**THEN** they see a "Follow" button; if already following, it shows "Following"; clicking "Follow" inserts a row into `user_follows`; the follower count updates optimistically.

**GIVEN** a visitor (not logged in) views a writer's profile,  
**WHEN** the page loads,  
**THEN** the page is fully visible (no auth required); only public published scripts are shown; the "Follow" button shows, but clicking it redirects to the login page.

**GIVEN** a writer navigates to Profile → Edit,  
**WHEN** they update their bio, display name, or writer type,  
**THEN** changes are saved to `profiles` via `PATCH /profiles/:id`; the public page updates within 5 seconds (CDN cache bust).

**GIVEN** a writer changes their username,  
**WHEN** they enter a new username and confirm,  
**THEN** the old username is freed after 30 days (so it cannot be taken by others during that window); the writer gets only 2 username changes per year; the profile URL at the old username shows a redirect to the new URL for 30 days.

**GIVEN** the writer's profile page is loaded,  
**WHEN** a published script is clicked,  
**THEN** the user is taken to the script's public detail page (story-041); if the script has a Listen recording, a play button is shown on the card.

---

## Performance Criteria

- Profile page load (SSR/CDN cached): < 1.5 seconds
- Avatar upload: processed within 5 seconds; writer sees new avatar immediately
- Follow/unfollow: optimistic UI update < 100ms; server sync in background

---

## Offline Behaviour

- Profile page is not available offline (it is a social/public page)
- Writer's own profile data (username, bio) is cached in Drift for display in Settings

---

## Mobile Behaviour (375px)

- Avatar: 72×72 dp
- Script grid: 1 column
- Bio is truncated at 3 lines with "Show more"
- Follow button: full-width on mobile
- Stats row (followers/following/scripts): shown as a compact row

---

## Error States

**E1 — Username Already Taken**  
Error inline below the input field: "That username is taken. Try @{username}_writer or @{username}2."

**E2 — Avatar Upload Failure**  
Toast: "Photo upload failed. Try a JPEG or PNG under 5MB." Retry button available.

**E3 — Profile Not Found**  
If `/@username` returns 404: show "This writer hasn't joined LIPILY yet" page with a CTA to sign up.

**E4 — Username Change Limit Reached**  
Error: "You've changed your username twice this year. Next change available on {date}."

---

## Security Considerations

- Username can only be set/changed by the authenticated user who owns the profile (RLS enforced)
- Avatars are stored with a content-hash path; no user-controlled file paths
- Rate limit on profile updates: 10 per minute per user (Rust middleware)
- Reserved username list is enforced server-side (not only client-side)
- Profile photos are scanned by a moderation pipeline before going live (story-050 pipeline)

---

## Human Authorship Considerations

- No AI involvement in profile creation

---

## DB Tables Touched

- `profiles` (UPDATE `username`, `display_name`, `bio`, `writer_type`, `avatar_url`, `follower_count`, `following_count`)
- `user_follows` (INSERT/DELETE)
- `username_change_log` (INSERT on each change; enforce 2/year limit)

```sql
CREATE TABLE user_follows (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  follower_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  following_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE (follower_id, following_id)
);
CREATE INDEX idx_user_follows_follower ON user_follows(follower_id);
CREATE INDEX idx_user_follows_following ON user_follows(following_id);

CREATE TABLE username_change_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES profiles(id),
  old_username TEXT NOT NULL,
  new_username TEXT NOT NULL,
  changed_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## API Endpoints Used

- `PATCH /profiles/:id` — update display name, bio, writer type
- `PATCH /profiles/:id/username` — change username (enforces limit)
- `POST /profiles/:id/avatar` — upload avatar (multipart)
- `POST /follows` — follow a user
- `DELETE /follows/:following_id` — unfollow
- `GET /profiles/:username` — public profile data

---

## Test Cases

**TC-001:** Claim username "arjun_writes" → assert saved to `profiles.username`; profile page accessible at `/@arjun_writes`.  
**TC-002:** Claim already-taken username → assert error message shown; no DB change.  
**TC-003:** Upload valid PNG avatar → assert resized to 256×256 WebP; saved at correct S3 path.  
**TC-004:** Follow writer → assert `user_follows` row created; follower count increments.  
**TC-005:** Unfollow writer → assert `user_follows` row deleted; follower count decrements.  
**TC-006:** Unauthenticated user views profile → assert page loads; Follow button redirects to login.  
**TC-007:** Writer changes username twice → third attempt shows year limit error.  
**TC-008:** Profile page with 0 published scripts → assert empty state "No published scripts yet."

---

## Out of Scope

- Direct messaging (Phase 3)
- Verified checkmarks for industry professionals (Phase 3)
- Portfolio PDF export (Phase 3)
