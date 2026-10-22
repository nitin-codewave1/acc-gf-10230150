---
id: agent-ba/web_app/rollup.md
type: spec
project: 325c3d2f-99ab-4396-bf87-8a8b58158084
version: 0.2.0
author: acd154e9-7d7c-493d-b538-c1e75f4fcbbe
produces: agent-ba/web_app/rollup.md
depends_on: []
commit_kind: fork
confidence: agent
run_id: b3458417-e9fc-5523-818c-5a07db7110ad
---

# Business Analysis — D-COPSCAR

## Discovery

**Brief Quality:** sufficient (Problem: strong, Outcomes: strong, Stakeholders: weak)

**Declaration:** Brief quality sufficient - no declaration needed.

## Business Objectives

### OBJ-001: Enable registered users to reliably authenticate into the web app
- **Leading KPI:** % of login attempts with valid credentials that succeed and redirect to /home
- **Lagging KPI:** Authenticated session completion rate (sessions reaching /home without re-login) over 60 days
- **Target:** 99%+ of valid-credential login attempts succeed and redirect to /home within 60 days of launch
- **Impact:** Users gain dependable access; downstream authenticated features become usable

### OBJ-002: Prevent disclosure of which credential field is incorrect to reduce account enumeration risk
- **Leading KPI:** % of failed login responses returning the generic 'Invalid email or password' message (vs. field-specific hints)
- **Lagging KPI:** Number of detected account-enumeration attempts/incidents post-launch
- **Target:** 100% of failed-login responses use the generic error message from day 1; zero enumeration incidents at 60 days
- **Impact:** Attackers cannot distinguish valid emails from invalid ones, shrinking the attack surface

### OBJ-003: Protect accounts from brute-force password guessing
- **Leading KPI:** % of 6th-attempt sequences (after 5 consecutive failures within 15 min) correctly returning 'Account locked'
- **Lagging KPI:** Number of successful credential-stuffing/brute-force compromises over 60 days
- **Target:** 100% lockout trigger accuracy at 5 failed attempts/15-min window from launch; zero brute-force compromises at 60 days
- **Impact:** Repeated automated guessing is neutralized, lowering account-takeover risk

### OBJ-004: Secure session handling to limit exposure from stolen or leaked cookies
- **Leading KPI:** % of issued session cookies set with HttpOnly, Secure, and 8h expiry flags
- **Lagging KPI:** Number of session-hijacking or cookie-theft incidents over 60 days
- **Target:** 100% of session cookies correctly flagged (HttpOnly/Secure/8h expiry) from day 1; zero session-hijack incidents at 60 days
- **Impact:** Compromised cookies have a bounded, short-lived blast radius and cannot be read via client-side script

## EP-001: Login UI & Credential Authentication (→ OBJ-001)
Build login page (email/password fields, Sign in button) and POST /api/login endpoint that validates credentials against bcrypt hashes, issues session cookie, and redirects to /home on success. Directly drives reliable authentication KPI.

### US-001-01: As a registered account holder, I want to sign in with correct email and password, so that gain access to the app so that valid login attempts succeed and redirect to /home, driving the authentication success KPI
**Priority:** Must Have

**Acceptance Criteria:**
- Given a registered user enters their correct email and password on the login page, when they click "Sign in", then the server validates credentials against the stored bcrypt hash, returns a session cookie, and redirects the user to /home
- Given a registered user enters their email with different casing but correct password, when they click "Sign in", then the system behaves per the documented email-matching behavior (treat as exact match unless stated otherwise) and either succeeds or fails consistently with stored credential comparison
- Given the login form is missing a password, when the user clicks "Sign in" without entering a password, then the request is rejected before hitting credential validation and the Sign in button or form prevents submission

**Test Cases:**
- **TC-001-01**: they click "Sign in"
- **TC-001-02**: [edge] they click "Sign in"
- **TC-001-03**: [error] the user clicks "Sign in" without entering a password

### US-001-02: As a registered account holder, I want to see a generic error when entering wrong credentials, so that understand the attempt failed without leaking which field was wrong, preventing credential enumeration that could erode trust in the KPI measurement
**Priority:** Must Have

**Acceptance Criteria:**
- Given a registered user enters an incorrect password with a valid email, when they click "Sign in", then the system shows "Invalid email or password" and does not set a session cookie
- Given a user enters an email that is not registered, when they click "Sign in", then the system shows the same generic "Invalid email or password" message, not revealing the email is unregistered
- Given a user submits the login form with an empty email field, when they click "Sign in", then the system treats it as invalid credentials and shows the generic error without a cookie

**Test Cases:**
- **TC-001-04**: they click "Sign in"
- **TC-001-05**: [error] they click "Sign in"
- **TC-001-06**: [edge] they click "Sign in"

### US-001-03: As a registered account holder, I want to be locked out after 5 consecutive failed login attempts, so that have their account protected from brute-force attacks, preserving trust in the authentication flow measured by the KPI
**Priority:** Must Have

**Acceptance Criteria:**
- Given a user has failed login 5 consecutive times within 15 minutes, when they attempt a 6th login, then the system shows "Account locked" and rejects the attempt regardless of credential correctness
- Given a user is locked out, when they attempt to log in with the correct password during the 15-minute lockout window, then the login still fails with "Account locked" and no session cookie is issued
- Given a user's lockout period of 15 minutes has elapsed, when they attempt to log in again with correct credentials, then the lockout is cleared and the login succeeds normally

**Test Cases:**
- **TC-001-07**: they attempt a 6th login
- **TC-001-08**: [edge] they attempt to log in with the correct password during the 15-minute loc
- **TC-001-09**: [error] they attempt to log in again with correct credentials

### US-001-04: As a registered account holder, I want to receive a secure, time-limited session cookie after successful login, so that stay authenticated for a bounded session while reducing risk of session hijacking, supporting reliable and safe access to /home
**Priority:** Must Have

**Acceptance Criteria:**
- Given a user successfully authenticates without Remember me checked, when the server responds, then the response sets a session cookie flagged HttpOnly and Secure with an 8-hour expiry
- Given a user's session cookie has expired after 8 hours (or after 30 days, when Remember me was checked), when they attempt to access an authenticated page, then they are no longer treated as logged in and must sign in again
- Given a request to /api/login occurs over a non-HTTPS connection, when the server would issue a Secure cookie, then the cookie is not transmitted insecurely (Secure flag enforced per spec)

**Test Cases:**
- **TC-001-17**: the server responds
- **TC-001-18**: Remember me was checked), when they attempt to access an authenticated page
- **TC-001-19**: the server would issue a Secure cookie

### US-001-05: As a registered account holder, I want to use a visible login form with email, password, and a Sign in button, so that have a clear, usable entry point to initiate authentication, supporting successful login attempts counted in the KPI
**Priority:** Must Have

**Acceptance Criteria:**
- Given a user navigates to the login page, when the page loads, then an email field, a password field, a "Remember me" checkbox, and a "Sign in" button are displayed
- Given a user has not yet entered any text, when the page renders, then the Sign in button is present and enabled for submission (no premature disabling that blocks valid attempts)
- Given the login page loads, when the user has not interacted with it, then the "Remember me" checkbox is unchecked by default

**Test Cases:**
- **TC-001-20**: the page loads
- **TC-001-21**: the page renders
- **TC-001-22**: the user has not interacted with it

### US-001-06: As a security-conscious user, I want to have failed login attempts validated server-side against bcrypt hashes rather than client-side, so that ensure credential checks cannot be bypassed in the browser, protecting the integrity of the authentication success KPI
**Priority:** Must Have

**Acceptance Criteria:**
- Given the client submits an email and password to POST /api/login, when the server receives the request, then it compares the password against the stored bcrypt hash server-side before responding
- Given a client sends a malformed or missing credential payload to POST /api/login, when the server processes the request, then it returns an error response without attempting a bcrypt comparison or issuing a cookie

**Test Cases:**
- **TC-001-15**: the server receives the request
- **TC-001-16**: [error] the server processes the request

## EP-002: Generic Failure Messaging (→ OBJ-002)
Ensure all invalid-credential responses return the single generic 'Invalid email or password' message with no cookie issued, regardless of which field was wrong, to prevent account enumeration.

### US-002-01: As a registered user who mistyped their password, I want to receive the same generic 'Invalid email or password' error as any other failed login, so that attackers cannot distinguish a wrong password from a wrong email, reducing account enumeration risk
**Priority:** Must Have

**Acceptance Criteria:**
- Given a registered email with an incorrect password, when the user submits the login form, then the API returns the generic 'Invalid email or password' message and no session cookie is set
- Given an email that does not exist in the system, when the user submits the login form, then the API returns the identical 'Invalid email or password' message (same text, status code, and timing profile as a wrong-password case)

**Test Cases:**
- **TC-002-01**: the user submits the login form
- **TC-002-02**: [edge] the user submits the login form

### US-002-02: As a security-conscious user probing the login form, I want to be unable to infer whether an email is registered by observing error responses, so that prevents account enumeration, directly improving the % of failed responses returning the generic message
**Priority:** Must Have

**Acceptance Criteria:**
- Given the login endpoint is called with an unregistered email and any password, when the request is processed, then the response contains no field-specific hint (e.g. 'email not found' or 'wrong password') and matches the generic error exactly
- Given an empty or malformed email/password field submitted to POST /api/login, when validation fails, then the response still returns the generic 'Invalid email or password' message rather than a field-validation-specific error

**Test Cases:**
- **TC-002-03**: [error] the request is processed
- **TC-002-04**: [edge] validation fails

### US-002-03: As a developer implementing the login endpoint, I want to ensure no session cookie is ever issued on any failure path, so that guarantees failed attempts cannot result in an authenticated session, supporting accurate measurement of the generic-message KPI
**Priority:** Must Have

**Acceptance Criteria:**
- Given a login request with invalid credentials, when the server responds with the generic error, then the response contains no Set-Cookie header for a session cookie
- Given a login request with a locked account and invalid credentials, when the server rejects the request, then the response still contains no session cookie, consistent with all other failure cases

**Test Cases:**
- **TC-002-05**: the server responds with the generic error
- **TC-002-06**: [error] the server rejects the request

### US-002-04: As a QA engineer validating anti-enumeration behavior, I want to verify failure response consistency across wrong-email and wrong-password scenarios, so that confirms the generic-message KPI is met before release, protecting against enumeration risk
**Priority:** Should Have

**Acceptance Criteria:**
- Given two separate login attempts, one with a wrong email and one with a wrong password for a valid email, when both are submitted to POST /api/login, then both responses return identical message text and HTTP status code
- Given a login attempt with correct email but incorrect password repeated multiple times (below lockout threshold), when each attempt fails, then each response remains the generic message without revealing attempt count or remaining tries

**Test Cases:**
- **TC-002-07**: both are submitted to POST /api/login
- **TC-002-08**: [edge] each attempt fails

## EP-003: Account Lockout on Repeated Failures (→ OBJ-003)
Track consecutive failed login attempts per account and lock the account for 15 minutes after 5 failures, returning 'Account locked' on the 6th attempt, to neutralize brute-force guessing.

### US-003-01: As a registered user attempting repeated sign-ins, I want to have failed login attempts counted consecutively per account, so that so brute-force guessing against their account is tracked and stopped, supporting the lockout accuracy KPI
**Priority:** Must Have

**Acceptance Criteria:**
- Given an account with 0 failed attempts, when the user submits an incorrect password, then the failed attempt counter for that account increments to 1
- Given an account with 3 consecutive failed attempts, when the user submits a correct email and password, then the failed attempt counter resets to 0 and login succeeds
- Given an account with 4 consecutive failed attempts, when the user submits another incorrect password, then the failed attempt counter increments to 5 and the account becomes locked

**Test Cases:**
- **TC-003-01**: the user submits an incorrect password
- **TC-003-02**: [edge] the user submits a correct email and password
- **TC-003-03**: [error] the user submits another incorrect password

### US-003-02: As a registered user who has triggered 5 consecutive failures, I want to be blocked from signing in for 15 minutes after the 5th failure, so that so the account is protected from further brute-force guesses, directly satisfying the lockout KPI
**Priority:** Must Have

**Acceptance Criteria:**
- Given an account has reached 5 consecutive failed attempts within 15 minutes, when the user submits a 6th login attempt with any credentials, then the server returns 'Account locked' and no session cookie is issued
- Given an account has reached 5 consecutive failed attempts, when the user submits the 6th attempt with the correct password, then the server still returns 'Account locked' and does not authenticate
- Given an account is locked, when the user submits a login attempt before the 15-minute lockout period has elapsed, then the server returns 'Account locked' and does not reset the lockout timer

**Test Cases:**
- **TC-003-04**: the user submits a 6th login attempt with any credentials
- **TC-003-05**: [edge] the user submits the 6th attempt with the correct password
- **TC-003-06**: [error] the user submits a login attempt before the 15-minute lockout period has

### US-003-03: As a registered user who was previously locked out, I want to regain the ability to sign in after the 15-minute lockout expires, so that so legitimate access is restored without manual intervention, balancing security with usability
**Priority:** Must Have

**Acceptance Criteria:**
- Given an account was locked and 15 minutes have elapsed since the lockout started, when the user submits a login attempt with correct credentials, then the server authenticates the user, issues a session cookie, and redirects to /home
- Given an account's lockout period has just expired, when the user submits an incorrect password, then the failed attempt counter restarts at 1 instead of continuing the previous lockout

**Test Cases:**
- **TC-003-07**: the user submits a login attempt with correct credentials
- **TC-003-08**: [edge] the user submits an incorrect password

### US-003-04: As a registered user attempting to log in, I want to have failed attempt counts tracked per account rather than reset by unrelated activity, so that so the 5-consecutive-failure rule is applied accurately per account, ensuring KPI correctness
**Priority:** Should Have

**Acceptance Criteria:**
- Given two different user accounts each with failed login attempts, when one account reaches 5 failures and locks, then the other account's failure count and lock status remain unaffected
- Given a login attempt is made for an email address with no matching account, when the credentials are submitted repeatedly, then the system does not create or track a lockout state for a non-existent account

**Test Cases:**
- **TC-003-09**: one account reaches 5 failures and locks
- **TC-003-10**: [error] the credentials are submitted repeatedly

### US-003-05: As a security-conscious developer validating lockout behavior, I want to receive a consistent 'Account locked' message distinct from the generic invalid-credentials error, so that so locked-out status is clearly observable and testable against the KPI without leaking account validity
**Priority:** Must Have

**Acceptance Criteria:**
- Given an account is currently locked, when any login attempt is made against it, then the response message is exactly 'Account locked' and never 'Invalid email or password'
- Given an account is locked, when the user attempts to log in with a different, unregistered email in the same browser session, then the lockout message does not appear and the generic invalid-credentials error is returned instead

**Test Cases:**
- **TC-003-11**: any login attempt is made against it
- **TC-003-12**: [edge] the user attempts to log in with a different, unregistered email in the s

## EP-004: Secure Session Cookie Issuance (→ OBJ-004)
Issue session cookies with HttpOnly, Secure, and 8h expiry flags on successful login to bound exposure from stolen/leaked cookies.

### US-004-01: As a registered web app user, I want to receive a session cookie immediately after signing in with valid credentials, so that my session is established securely, limiting exposure if the cookie is ever stolen or leaked
**Priority:** Must Have

**Acceptance Criteria:**
- Given a user submits a valid email and password, when POST /api/login succeeds, then the response sets a session cookie with HttpOnly, Secure, and an 8h expiry flag (or a 30-day Max-Age when Remember me is checked)
- Given a valid login, when the session cookie is issued, then the response also redirects the user to /home
- Given a valid login response, when the cookie attributes are inspected, then the Secure flag is present even if the request arrived over HTTPS behind a proxy, preventing transmission over plain HTTP

**Test Cases:**
- **TC-004-10**: POST /api/login succeeds
- **TC-004-11**: the session cookie is issued
- **TC-004-12**: the cookie attributes are inspected

### US-004-02: As a security-conscious user, I want to have my session cookie be inaccessible to client-side scripts, so that my session token can't be exfiltrated via XSS, reducing the risk from a stolen/leaked cookie
**Priority:** Must Have

**Acceptance Criteria:**
- Given a successful login has set the session cookie, when client-side JavaScript attempts to read document.cookie, then the session cookie is not returned because it is marked HttpOnly
- Given the session cookie is set, when the cookie is transmitted on subsequent requests, then it is only sent over HTTPS connections due to the Secure flag

**Test Cases:**
- **TC-004-04**: client-side JavaScript attempts to read document.cookie
- **TC-004-05**: [edge] the cookie is transmitted on subsequent requests

### US-004-03: As a registered user, I want to have my session automatically expire 8 hours after login, so that a stolen or leaked cookie has a bounded window of usefulness to an attacker
**Priority:** Must Have

**Acceptance Criteria:**
- Given a user logged in without Remember me and received a session cookie, when exactly 8 hours have elapsed since issuance, then the cookie expires and is no longer accepted as a valid session
- Given a user logged in with Remember me checked, when exactly 30 days have elapsed since issuance, then the cookie expires and is no longer accepted as a valid session
- Given an expired session cookie is presented, when the user accesses a protected route, then the server treats the request as unauthenticated and does not grant access

**Test Cases:**
- **TC-004-13**: exactly 8 hours have elapsed since issuance
- **TC-004-14**: exactly 30 days have elapsed since issuance
- **TC-004-15**: the user accesses a protected route

### US-004-04: As a registered user, I want to not receive any session cookie when my login attempt fails, so that unauthenticated attempts can never establish a session, keeping exposure to zero unless credentials are valid
**Priority:** Must Have

**Acceptance Criteria:**
- Given a user submits invalid credentials, when POST /api/login responds with the generic error, then no session cookie (Set-Cookie header) is included in the response
- Given an account is locked after 5 failed attempts, when the user attempts a 6th login, then no session cookie is issued and the response shows the account-locked message

**Test Cases:**
- **TC-004-08**: [error] POST /api/login responds with the generic error
- **TC-004-09**: [error] the user attempts a 6th login

### US-004-05: As a registered web app user who wants to stay signed in, I want to check a Remember me option on the login page so that my session cookie persists for 30 days, so that I stay signed in across browser restarts without re-entering credentials, while the default remains a bounded 8-hour session
**Priority:** Should Have

**Acceptance Criteria:**
- Given the Remember me checkbox is checked, when the user signs in with valid credentials, then the session cookie is set with Max-Age=2592000 (30 days) and keeps the HttpOnly and Secure flags
- Given the Remember me checkbox is unchecked (default), when the user signs in with valid credentials, then the session cookie uses the 8-hour expiry and no 30-day Max-Age
- Given the Remember me checkbox is checked and the credentials are invalid or the account is locked, when the login fails, then no session cookie is issued

**Test Cases:**
- **TC-004-16**: the user signs in with valid credentials
- **TC-004-17**: the user signs in with valid credentials
- **TC-004-18**: the login fails

## Business Rules

### BR-001: Credentials (email + password) must be validated server-side against stored bcrypt password hashes; no client-side credential validation is permitted.
**Rationale:** Prevents bypass of authentication logic via browser manipulation | **Enforcement:** POST /api/login server handler only

### BR-002: Successful authentication redirects the user to /home and establishes a session.
**Rationale:** Defines the authentication success path required for the KPI | **Enforcement:** Login UI redirect logic + session issuance on 200 response

### BR-003: Any failed login attempt (wrong email, wrong password, or locked account distinction aside) must return the generic message 'Invalid email or password'.
**Rationale:** Prevents account enumeration by not revealing which field was incorrect | **Enforcement:** POST /api/login error-response construction

### BR-004: No session cookie may be issued on any failure path, including invalid credentials and lockout.
**Rationale:** Guarantees unauthenticated attempts can never establish a session | **Enforcement:** POST /api/login response handling; cookie set only on success branch

### BR-005: Failed login attempts are counted consecutively per account; a successful login resets the counter to zero.
**Rationale:** Ensures brute-force detection is accurate per account and not affected by unrelated activity | **Enforcement:** Server-side failed-attempt counter keyed by account identifier

### BR-006: An account is locked after 5 consecutive failed login attempts.
**Rationale:** Protects accounts from brute-force password guessing | **Enforcement:** POST /api/login attempt-counter check before credential validation

### BR-007: A locked account remains locked for 15 minutes from the 5th failed attempt, after which the failed-attempt counter resets and sign-in is allowed again.
**Rationale:** Balances security (brute-force protection) with usability (automatic recovery) | **Enforcement:** Lockout timestamp check in POST /api/login

### BR-008: A locked-out login attempt must return a distinct 'Account locked' message, separate from the generic invalid-credentials message.
**Rationale:** Makes lockout state observable/testable without leaking whether credentials were valid | **Enforcement:** POST /api/login error-response construction, lockout branch

### BR-009: Session cookies must be issued only immediately following successful credential validation, marked HttpOnly, and expire 8 hours after login.
**Rationale:** Limits exposure window and prevents client-side script access if the cookie is stolen | **Enforcement:** Set-Cookie header configuration on successful POST /api/login response

### BR-010: 
**Rationale:**  | **Enforcement:** 

## Non-Functional Requirements

### NFR-001: Passwords must be validated against bcrypt hashes server-side only; plaintext passwords are never logged or stored.
**Category:** Security | **Target:** 0 occurrences of plaintext password persistence/logging | **Priority:** Must Have

### NFR-002: All failed login responses (wrong email, wrong password) must return an identical generic error message and HTTP status.
**Category:** Security | **Target:** 100% of failure responses match generic-message format | **Priority:** Must Have

### NFR-003: Account lockout triggers after exactly 5 consecutive failed attempts and lasts exactly 15 minutes before auto-reset.
**Category:** Security | **Target:** Lockout trigger accuracy = 100% at 5th failure; lockout duration = 15 min ± 5 sec | **Priority:** Must Have

### NFR-004: Session cookies must be set with HttpOnly, Secure, and SameSite attributes, with 8-hour expiry.
**Category:** Security | **Target:** 100% of issued cookies include HttpOnly+Secure+SameSite flags; expiry = 8h ± 1 min | **Priority:** Must Have

### NFR-005: No session cookie is ever present in any failed-login (invalid credentials or locked) HTTP response.
**Category:** Security | **Target:** 0% of failure responses contain Set-Cookie header | **Priority:** Must Have

### NFR-006: POST /api/login must respond within acceptable latency for both success and failure paths under normal load.
**Category:** Performance | **Target:** p95 response time ≤ 500ms | **Priority:** Should Have

### NFR-007: Login endpoint and lockout-state store must support concurrent login attempts without race conditions in attempt counting.
**Category:** Scalability | **Target:** Support ≥ 100 concurrent login requests with correct per-account counter increments | **Priority:** Should Have

### NFR-008: Login service (UI + API) must be available to serve authentication requests reliably.
**Category:** Availability | **Target:** Uptime ≥ 99.9% monthly | **Priority:** Should Have

### NFR-009: Login form must present email field, password field, and Sign in button with clear focus states and inline error display.
**Category:** Usability | **Target:** Error message rendered within 1s of failed response receipt | **Priority:** Must Have

### NFR-010: Login form fields, button, and error messages must be keyboard-navigable and screen-reader accessible.
**Category:** Accessibility | **Target:** WCAG 2.1 AA compliance for login page | **Priority:** Should Have

### NFR-011: Credential storage and verification must use bcrypt with an industry-standard work factor; no credential checks occur client-side.
**Category:** Compliance | **Target:** bcrypt cost factor ≥ 10; 0 client-side credential comparisons | **Priority:** Must Have

### NFR-012: Failed-attempt counters and lockout timestamps must be stored server-side and resilient to client manipulation (e.g., not reliant on cookies/local storage).
**Category:** Security | **Target:** 0 client-controlled inputs affect lockout state | **Priority:** Must Have
