# story-002 — Authentication

| Field | Value |
|---|---|
| **Story ID** | story-002 |
| **Title** | Authentication |
| **Epic** | E13 — System & Infrastructure |
| **Phase** | 1 |
| **Sprint** | Sprint 0 |
| **Priority** | P0 |
| **Status** | DRAFT |
| **Persona** | Arjun (the aspiring feature-film screenwriter) |
| **Estimated Effort** | 3 days |
| **Dependencies** | story-001 |

---

## Problem Statement

LIPILY stores private creative work. Writers must be able to sign in and sign up securely without passwords. The platform must support Magic Link email sign-in (delivered via Resend) and Google OAuth. On first sign-in, a profile row must be created automatically. The auth state must persist across cold starts and be observable from any widget in the Flutter tree via Riverpod.

---

## User Story

As a writer, I want to sign in with my email (magic link) or Google account so that I can access my scripts securely on any device without managing a password.

---

## Acceptance Criteria

**GIVEN** a new user lands on the `/auth` screen,  
**WHEN** they enter their email address and tap "Send Magic Link",  
**THEN** a magic link email is sent via Resend within 5 seconds; the screen shows "Check your inbox" and the button is disabled to prevent duplicate sends.

**GIVEN** a user clicks the magic link in their email,  
**WHEN** the deep link opens the Flutter app (or browser),  
**THEN** the Supabase `supabase_flutter` client exchanges the token for an access + refresh token pair; the `AuthNotifier` emits an `authenticated` state; the router redirects to `/dashboard`.

**GIVEN** a user taps "Continue with Google",  
**WHEN** the Google OAuth flow completes,  
**THEN** the Supabase OAuth callback is handled; the user is authenticated; the router redirects to `/dashboard` on first login or `/` on return login.

**GIVEN** a user signs in for the first time (no profile row exists),  
**WHEN** authentication completes,  
**THEN** the Supabase `on_auth_user_created` PostgreSQL trigger fires and inserts a row into `profiles` with `id = auth.uid()`, `created_at = NOW()`, `tier = 'free'`, `ai_requests_today = 0`, `script_count = 0`.

**GIVEN** an authenticated user closes and reopens the app,  
**WHEN** the app starts,  
**THEN** the Supabase client restores the session from secure storage (FlutterSecureStorage); `AuthNotifier` emits `authenticated` without requiring the user to sign in again.

**GIVEN** a user's access token is expired,  
**WHEN** the Supabase client makes an authenticated request,  
**THEN** it automatically refreshes the token using the 7-day refresh token; the user is never shown a sign-in screen unless the refresh token is also expired.

**GIVEN** a user taps "Sign Out",  
**WHEN** the confirmation modal is confirmed,  
**THEN** `supabase.auth.signOut()` is called; the Supabase client clears the local session; the Drift database is wiped; the PowerSync client disconnects; the router redirects to `/auth`.

**GIVEN** the auth screen is loaded on a 375px wide mobile screen,  
**WHEN** the keyboard is open,  
**THEN** the email input field is not obscured by the keyboard; the layout scrolls to keep the input visible; the CTA button remains accessible.

**GIVEN** a user enters an invalid email format and taps "Send Magic Link",  
**WHEN** validation runs,  
**THEN** an inline red error message appears below the email field: "Please enter a valid email address"; no network request is made.

**GIVEN** the Supabase auth endpoint is unreachable (network error),  
**WHEN** the user attempts sign-in,  
**THEN** a toast appears: "Could not connect. Check your internet connection and try again."; no loading state persists; the user can retry immediately.

---

## Performance Criteria

- Magic link email delivered within 5 seconds of request (Resend SLA)
- Token exchange (magic link → session) completes in < 2 seconds
- Session restore from secure storage on app cold start: < 500ms
- Auth screen first paint: < 1.5 seconds on mid-range Android
- Google OAuth round-trip: < 3 seconds (including OAuth popup and callback)

---

## Offline Behaviour

- The auth screen is not usable offline — a clear offline banner must appear: "No internet connection. Authentication requires an internet connection."
- If the user is already authenticated and the app opens offline, the session is restored from FlutterSecureStorage and the user is taken to `/dashboard` with an offline indicator. No re-authentication is required.
- Sign-out is blocked when offline with a toast: "Sign out requires an internet connection."

---

## Mobile Behaviour (375px)

- The LIPILY wordmark is 40px height, centred, top 72px from safe area
- Email input field is full width with 16px horizontal padding; height 56px; border-radius 12px
- "Send Magic Link" button is full width; height 56px; primary colour `#6C63FF`; border-radius 12px; "Or" divider with horizontal lines
- "Continue with Google" button matches dimensions; uses official Google branding asset; white background; 1px border `#E0E0E0`
- Tapping the screen outside the keyboard dismisses the keyboard
- All tap targets minimum 44×44dp per WCAG 2.1

---

## Error States

**E1 — Network Timeout**  
Magic link request times out (> 10s): show toast "Request timed out. Please try again." Disable send button for 3 seconds before re-enabling.

**E2 — Email Rate Limit**  
Supabase returns a rate-limit error (user has requested too many magic links): show toast "Too many requests. Please wait a minute before trying again."

**E3 — Expired Magic Link**  
User clicks a magic link that has expired (> 1 hour): app shows an error screen "This link has expired. Return to the app to request a new one." with a deep link back to `/auth`.

**E4 — Google OAuth Cancelled**  
User cancels the Google OAuth flow: no error shown; the user is returned to the sign-in screen silently.

**E5 — Account Already Exists**  
User tries to sign in with Google using an email that already has a magic-link account: Supabase merges accounts by identity linking; no error shown; user is signed in.

---

## Security Considerations

- Access token is a Supabase JWT (RS256); stored only in FlutterSecureStorage, never in SharedPreferences or local storage
- Refresh token stored in FlutterSecureStorage with `accessible: KeychainAccessibility.whenUnlocked`
- Magic link tokens are single-use and expire in 1 hour (Supabase default)
- The `profiles` table has an RLS INSERT policy: `auth.uid() = id` — users can only create their own profile row
- The Supabase `anon` key is used in the Flutter client; it has no elevated privileges; all security enforced by RLS
- Google OAuth redirect URL is registered in Supabase Dashboard and Google Cloud Console; no open redirect is possible
- Sign-out wipes all local Drift data and PowerSync state to prevent data leakage on shared devices

---

## Human Authorship Considerations

Not applicable — no AI-generated content is involved in this story.

---

## DB Tables Touched

- `profiles` (INSERT on first sign-in via PostgreSQL trigger; READ on session restore)

---

## API Endpoints Used

- All auth operations go through Supabase Auth REST API directly via `supabase_flutter` — no LIPILY Rust API involved in this story.

---

## Riverpod Providers Required

```dart
// Auth state notifier
@riverpod
class AuthNotifier extends _$AuthNotifier {
  @override
  AuthState build() => const AuthState.loading();
  // Listens to supabase.auth.onAuthStateChange
  // Emits: AuthState.loading | AuthState.authenticated(user) | AuthState.unauthenticated
}

// Current user profile
@riverpod
Future<Profile?> currentProfile(Ref ref) async {
  final user = ref.watch(authNotifierProvider).user;
  if (user == null) return null;
  return ref.read(profileRepositoryProvider).getProfile(user.id);
}
```

---

## go_router Guards Required

```dart
redirect: (context, state) {
  final auth = ref.read(authNotifierProvider);
  final isOnAuthPage = state.matchedLocation == '/auth';
  if (auth.isLoading) return null; // Stay on current page
  if (!auth.isAuthenticated && !isOnAuthPage) return '/auth';
  if (auth.isAuthenticated && isOnAuthPage) return '/dashboard';
  return null;
}
```

---

## Test Cases

**TC-001:** Enter valid email → tap send → assert button disabled and success message shown.  
**TC-002:** Enter `not-an-email` → tap send → assert inline validation error shown, no network call made.  
**TC-003:** Simulate network error on magic link request → assert error toast appears.  
**TC-004:** Simulate successful magic link callback with valid token → assert router navigates to `/dashboard`.  
**TC-005:** Sign in as new user → assert `profiles` row created with `tier = 'free'`.  
**TC-006:** Sign in → kill app → reopen → assert user is still authenticated (session restored from secure storage).  
**TC-007:** Sign out → assert Drift DB is cleared and router navigates to `/auth`.  
**TC-008:** Simulate expired access token → assert auto-refresh happens silently without user intervention.

---

## Out of Scope

- Password-based authentication (not supported in LIPILY — magic link only)
- Apple Sign-In (Phase 2 / v2)
- Two-factor authentication (Phase 2 / v2)
- Account deletion (story-055)
- Profile avatar upload (story-040)
- Subscription tier assignment (story-048)
