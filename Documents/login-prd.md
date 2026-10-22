# PRD: Email + Password Login

## Goal
Let a registered user sign in to the web app with email and password.

## Scope
- Login page with email and password fields and a "Sign in" button.
- Server endpoint POST /api/login validating credentials against stored bcrypt hashes.
- On success: return a session cookie (HttpOnly, Secure, 8h expiry) and redirect to /home.
- On failure: show "Invalid email or password" (no hint which field was wrong).
- Lock the account for 15 minutes after 5 consecutive failed attempts.

## Out of scope
Sign-up, password reset, SSO, MFA.

## Acceptance criteria
1. Valid credentials -> redirected to /home with session cookie.
2. Invalid credentials -> generic error, no cookie.
3. 6th attempt after 5 failures within 15 min -> "Account locked" message.
