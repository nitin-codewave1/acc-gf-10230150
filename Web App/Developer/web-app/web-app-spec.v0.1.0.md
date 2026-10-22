---
id: agent-developer/web_app/web-app/WEB-APP-SPEC.md
type: spec
project: 325c3d2f-99ab-4396-bf87-8a8b58158084
version: 0.1.0
author: acd154e9-7d7c-493d-b538-c1e75f4fcbbe
produces: agent-developer/web_app/web-app/WEB-APP-SPEC.md
depends_on: [US-001-01, US-001-06, US-002-01, US-002-04, US-003-01, US-003-05, US-004-01, US-004-04, US-001-03, US-001-04, US-001-05, US-004-03, US-004-02, US-002-03, US-002-02, US-001-02, US-003-02, US-003-03, US-003-04]
commit_kind: append
confidence: agent
run_id: ed201675-200d-5bbe-a4bc-171840d38f4b
okf_id: web-app/WEB-APP-SPEC.md
---

<!-- SECTION: web-app.overview | status: DERIVED -->
## Overview

**Purpose.** This surface delivers the client-side login experience for the web app: a login form (email field, password field, "Sign in" button) that submits to `POST /api/login` and handles every outcome the backend can return — success (session established, redirect to `/home`), generic invalid-credential failure, and account-lockout failure. Scope is drawn from EP-001 (US-001-01 … US-001-06), EP-002 (US-002-01 … US-002-04), EP-003 (US-003-01 … US-003-05), and EP-004 (US-004-01 … US-004-04) — no other epic was supplied.

**Primary actor.** Every supplied story describes one and the same actor under different framing: "registered account holder" (US-001-01, US-001-03, US-001-04, US-001-05), "registered user" (US-002-01, US-004-01, US-004-03, US-004-04), and "security-conscious user" (US-001-06, US-004-02). These are the same end user attempting authentication, not distinct roles. Three stories — US-002-03 ("developer implementing the login endpoint"), US-002-04 ("QA engineer validating anti-enumeration behavior"), and US-003-05 ("security-conscious developer validating lockout behavior") — frame the narrator as an implementer or verifier, not a web-app UI role; none of them describes a screen, navigation element, or interaction distinct from the single end-user flow above. They are treated as verification framing, not an additional role (cross-reference `web-app.authorization-ui`).

**Explicitly out of scope** (by consistent absence across `requirements.md`, `stories.md`, and `design-brief.md`):
- Registration / sign-up — no story or rule addresses it.
- Password reset — not mentioned anywhere.
- Multi-factor authentication — not mentioned anywhere.
- The content of `/home` beyond its role as the post-login redirect target (BR-002, AC-001-01-01) — no input describes what `/home` contains.
- Logout — **no input, of any kind, describes a logout flow, endpoint, or control.** Recorded as `GAP-WEB-001`.

**Note on design input.** The file supplied under the `design_brief` key (`design-brief.md`) contains only restated acceptance criteria (`AC-001-xx`, `AC-002-xx`) duplicating content already present in `requirements.md`; it names no component, token, layout, breakpoint, or frame. `figma_mcp` is `not-connected`. There is consequently no design-system input to this run at all. Per core rule 4.5 this run is treated as **behaviour-only** for every design-dependent section below (`web-app.design-system`, `web-app.responsive`). Recorded as `GAP-WEB-002`.
<!-- /SECTION: web-app.overview -->
<!-- SECTION: web-app.routing | status: DRAFT-UNVERIFIED -->
## Routing

| Route | Access | Layout | Source |
|---|---|---|---|
| `/login` | Public (unauthenticated) | Not described in any input — no shell, header, or nested layout is named anywhere. | AC-001-05-01 |
| `/home` | Protected — requires an established, valid session | Not described in any input; `/home`'s own content is out of scope (see overview). | BR-002, AC-001-01-01 |

**Protected-route rule.** AC-001-04-02 states that once the session cookie has expired, a request to an authenticated page is "no longer treated as logged in" and the user "must sign in again." No input states the behavior for a request that never carried a session cookie at all (as opposed to an expired one). `ASM-WEB-001`: the no-cookie case is assumed to be handled identically to the expired-cookie case, since nothing in the inputs suggests otherwise and no alternative is proposed anywhere.

**Unauthenticated redirect.** AC-001-04-02 establishes *that* an unauthenticated user accessing `/home` is denied, but not *how* — automatic client-side redirect to `/login`, a blocked render with an inline prompt, or a server-level redirect are all consistent with the cited text. Recorded as `GAP-WEB-003`.

**Layout nesting.** No input names an app shell, navigation bar, or any nested layout for either route. Recorded as `GAP-WEB-004`.
<!-- /SECTION: web-app.routing -->
<!-- SECTION: web-app.module-map | status: DERIVED -->
## Module Map

Exactly one feature area is described by the supplied inputs:

| Module | Routes owned | Basis |
|---|---|---|
| Authentication | `/login` (full ownership); the access rule gating `/home` (partial — `/home`'s own content belongs to an undefined, out-of-scope module) | EP-001 – EP-004, all supplied stories |

No second module (e.g., a "home"/dashboard module, a profile module) is described in any input — there is nothing to map besides the above.
<!-- /SECTION: web-app.module-map -->
<!-- SECTION: web-app.state-management | status: DRAFT-UNVERIFIED -->
## State Management

**Server state.**
- *Session/authentication status.* The session is carried entirely by a server-issued, `HttpOnly` cookie (AC-001-04-01, BR-009). Because the cookie is `HttpOnly`, client-side script cannot read it (US-004-02: "the session cookie is not returned because it is marked HttpOnly"). The web-app therefore has **no client-readable signal** of session validity; it can only infer status from how the server responds to a given request (e.g., being treated as logged out per AC-001-04-02). This is a hard constraint on any state-management design, not a gap.
- *Lockout status.* Failed-attempt counters and lockout timers are stored and evaluated server-side only, and NFR-012 requires them to be "resilient to client manipulation... not reliant on cookies/local storage." The web-app must not compute, cache, or persist a lockout counter or countdown itself; it only ever displays the message the server returns for the current attempt.

**Client state.** Form field values (email, password); the client-side rule blocking submission when the password field is empty (AC-001-01-03); a submit-in-flight flag; and the currently displayed error-message text, which must be rendered verbatim from the server's response rather than reworded or augmented client-side (BR-003, NFR-002 require character-identical generic messaging).

**Cache / invalidation.** No input describes fetching or caching any entity data before or after login — the only server interaction in scope is the single `POST /api/login` call. There is consequently no cache-invalidation policy beyond the login attempt itself. The precise UI mechanics on a repeated failed attempt (does the password field clear? does a stale error message visibly change state before the new one renders?) are not specified by any input. Recorded as `GAP-WEB-005`.

No client-side state-management library, HTTP client, or caching layer is named anywhere. Recorded as `GAP-WEB-006`.
<!-- /SECTION: web-app.state-management -->
<!-- SECTION: web-app.auth-flows | status: DRAFT-UNVERIFIED -->
## Auth Flows

**Login.** The user enters email and password and clicks "Sign in," which is present and enabled on page load with no premature disabling (AC-001-05-01, AC-001-05-02). Client-side, submission is blocked specifically when the **password** field is empty, before any credential validation occurs (AC-001-01-03). No equivalent client-side block exists for an empty **email** field: AC-001-02-03 and AC-002-01-02 show an empty-email submission is instead sent to the server and treated as invalid credentials, returning the same generic message. This asymmetry is explicit in the cited inputs and is preserved as written, not resolved as an inconsistency.

The request is submitted to `POST /api/login` (BR-001, AC-001-06-01). Credential comparison happens server-side against a stored bcrypt hash; the web-app performs no client-side credential comparison of any kind (US-001-06, NFR-001, NFR-011).

**Success.** On valid credentials, the server sets a session cookie flagged `HttpOnly` and `Secure` with an 8-hour expiry (AC-001-04-01, BR-009, NFR-004) and the client redirects to `/home` (BR-002, AC-001-01-01).

**Failure — invalid credentials.** Wrong password, unregistered email, and empty email all produce the identical message "Invalid email or password," with no session cookie set (AC-001-02-01, AC-001-02-02, AC-001-02-03, US-002-01, US-002-02, BR-003, BR-004, NFR-002, NFR-005). The client renders this text exactly as received and must not surface any field-specific hint (e.g., "email not found") — US-002-02 requires the response to contain "no field-specific hint... and match the generic error exactly."

**Failure — lockout.** After the 5th consecutive failed attempt within 15 minutes, any further attempt — including one with the correct password — returns "Account locked" instead of the generic message, and no cookie is issued (AC-001-03-01, AC-001-03-02, BR-006, BR-007, BR-008, NFR-003). This message is distinct from, and never substitutes for, the generic invalid-credentials message (US-003-05). Once 15 minutes elapse, the next correct-credential attempt succeeds normally and the counter resets (AC-001-03-03). The client's responsibility is limited to rendering whichever message the server returns; it must not display a remaining-attempts count or a lockout countdown — US-002-04's criterion explicitly requires failed attempts to stay generic "without revealing attempt count or remaining tries," and no input specifies a countdown UI for the locked state either.

**Refresh.** No token-refresh, silent-renewal, or cookie-extension behavior is described anywhere. Recorded as `GAP-WEB-008`.

**Session expiry.** Once the 8-hour cookie expires, the next authenticated-page access is treated as unauthenticated and the user must sign in again (AC-001-04-02). The exact client-side transition (silent redirect vs. an explicit "your session expired" notice) is unspecified — see `GAP-WEB-003`.

**Mid-flow expiry.** No input describes what happens if the session expires while the user is actively interacting with an authenticated page. Recorded as `GAP-WEB-009`.

**Logout.** Absent from every supplied input — no button, endpoint, or behavior is described. This flow cannot be specified from what was supplied (`GAP-WEB-001`).
<!-- /SECTION: web-app.auth-flows -->
<!-- SECTION: web-app.authorization-ui | status: DRAFT-UNVERIFIED -->
## Authorization UI

Exactly one role can sign in to this surface per the supplied inputs. "Registered account holder," "registered user," and "security-conscious user" are the same actor described with different motivating framing (see overview) — not distinct roles.

| Role | Visible navigation | Enabled actions |
|---|---|---|
| Unauthenticated visitor | `/login` only | Submit the login form |
| Registered user (authenticated) | `/home` (contents out of scope, undefined by any input) | None specified beyond reaching `/home` — no further in-app action is described anywhere |

**Excluded roles.** No role beyond the single one above (no admin, operator, guest-with-partial-access, etc.) appears in `requirements.md`, `stories.md`, or `design-brief.md`. There is accordingly nothing else to exclude by name. This differs from a case where a backend role/permission matrix exists and specific entries are deliberately left out: here, `RUN-CONTEXT.upstream.contract_source` is `null` — no backend role matrix was supplied to this run at all, so there is no matrix to check this table's completeness against. Recorded as `GAP-WEB-010`.
<!-- /SECTION: web-app.authorization-ui -->
<!-- SECTION: web-app.design-system | status: BLOCKED -->
## Design System

**Blocked.** The only input supplied under the `design_brief` key contains restated acceptance criteria, not design content — no color, typography, spacing value, token, breakpoint, or component is named anywhere in it. `figma_mcp` is `not-connected`. There is therefore no source from which to state a type scale, spacing scale, token set, or breakpoint list without inventing one, which is forbidden outright (core rule 1 / I-2; role rule W10: "Do not hardcode a visual value. Token reference, or BLOCKED.").

**Would be unblocked by:** a design brief or connected Figma source containing an actual token set, type scale, and spacing scale, or an explicit statement that this surface reuses a named, existing design system. (`GAP-WEB-002`)
<!-- /SECTION: web-app.design-system -->
<!-- SECTION: web-app.responsive | status: BLOCKED -->
## Responsive

**Blocked.** No breakpoint, target viewport, or layout change at any viewport size is named in `requirements.md`, `stories.md`, or `design-brief.md`.

**Would be unblocked by:** a design brief or Figma source stating breakpoints and the layout change at each, per core rule 4.5. (`GAP-WEB-011`)
<!-- /SECTION: web-app.responsive -->
<!-- SECTION: web-app.browser-support | status: BLOCKED -->
## Browser Support

**Blocked.** No input names a supported browser, a minimum version, or a browser-support policy of any kind.

**Would be unblocked by:** a platform or requirements document stating a target browser matrix. (`GAP-WEB-012`)
<!-- /SECTION: web-app.browser-support -->
<!-- SECTION: web-app.accessibility | status: DRAFT-UNVERIFIED -->
## Accessibility

**Target standard.** NFR-010 states the target metric verbatim as "WCAG 2.1 AA compliance for login page," and requires the login form's "fields, button, and error messages" to be "keyboard-navigable and screen-reader accessible." This target is scoped explicitly to the login page by NFR-010's own wording — no input extends it to any other route.

**Verification.** NFR-010 states the target but not a verification method (tool, automated audit, or manual process). Recorded as `GAP-WEB-013`. `ASM-WEB-002`: pending confirmation, an automated accessibility scan (e.g., axe-core or equivalent) plus a manual keyboard-only pass on `/login`, run in CI, is proposed as the default verification method — this is not confirmed by any supplied input and should be treated as a default, not a decision.
<!-- /SECTION: web-app.accessibility -->
<!-- SECTION: web-app.performance | status: BLOCKED -->
## Performance

**Blocked, with one exclusion noted.** NFR-006 ("POST /api/login must respond... p95 response time ≤ 500ms") is a backend-latency NFR describing the API, not a frontend bundle or payload budget, and is referenced here only to exclude it — per core rule 6, backend behavior is referenced, not restated as a frontend performance commitment.

No input states a JavaScript bundle-size budget, a route-level code-splitting policy, or identifies a largest-payload route. Given `web-app.module-map` names only two routes (`/login`, `/home`) with no component inventory for either, there is no basis from which to derive a "largest payload route" either.

**Would be unblocked by:** a platform or performance requirements document stating a bundle budget and a code-splitting policy. (`GAP-WEB-014`)
<!-- /SECTION: web-app.performance -->
<!-- SECTION: web-app.coverage | status: DRAFT-UNVERIFIED -->
## Coverage

No backend API contract was supplied to this run (`RUN-CONTEXT.upstream.contract_source` is `null`), so no module-level `WEB-<M>-FR-nnn` functional requirements have been minted in this project-level deliverable — those, with their Given/When/Then and verification entries, belong in the module-level spec. The table below instead maps every supplied story to the business rules (`BR-`) and non-functional requirements (`NFR-`) that cover it — the only "requirements" artifacts this run received — and states, for each, whether the behavior is a web-app responsibility or is entirely backend-owned (in which case the web-app's only obligation is to not interfere with or duplicate it).

| Story | Covering requirement(s) | Web-app responsibility |
|---|---|---|
| US-001-01 | BR-001, BR-002 | Submit form; redirect to `/home` on success |
| US-001-02 | BR-003, BR-004, NFR-002, NFR-005 | Render generic message verbatim; no FE-side cookie logic |
| US-001-03 | BR-006, BR-007, BR-008, NFR-003, NFR-012 | Render "Account locked" verbatim; counting/timing is backend-owned |
| US-001-04 | BR-009, NFR-004 | Backend-owned (cookie issuance); no FE requirement |
| US-001-05 | NFR-009, NFR-010 | Render email field, password field, enabled Sign-in button |
| US-001-06 | BR-001, NFR-001, NFR-011 | Backend-owned; FE constraint: never perform client-side credential comparison |
| US-002-01 | BR-003, BR-004, NFR-002 | Render whatever generic text the server returns, verbatim |
| US-002-02 | BR-003, NFR-002 | FE constraint: never render a field-specific hint on failure |
| US-002-03 | BR-004, NFR-005 | Backend-owned (Set-Cookie header); no FE requirement |
| US-002-04 | NFR-002 | Verification-only story; covered by US-002-01/US-002-02 behavior |
| US-003-01 | BR-005, BR-006, NFR-007, NFR-012 | Backend-owned entirely; no FE requirement |
| US-003-02 | BR-006, BR-007, BR-008, NFR-003 | Render "Account locked" (shared with US-001-03) |
| US-003-03 | BR-007, NFR-003 | Post-lockout success behaves as a normal login (US-001-01); no distinct FE requirement |
| US-003-04 | BR-005, NFR-012 | Backend-owned entirely; no FE requirement |
| US-003-05 | BR-008 | FE must render "Account locked" as a visually/semantically distinct state from the generic error |
| US-004-01 | BR-009, NFR-004 | Backend-owned (cookie issuance); FE performs the accompanying redirect (shared with US-001-01) |
| US-004-02 | NFR-004 | Backend-owned entirely; informs `web-app.state-management` (session state is not client-readable) |
| US-004-03 | BR-009, NFR-004, AC-001-04-02 | Backend-owned (expiry enforcement) + FE behavior on next access (see `GAP-WEB-003`) |
| US-004-04 | BR-004, NFR-005 | Backend-owned entirely; no FE requirement |

All 19 supplied story IDs appear above; none is uncovered.
<!-- /SECTION: web-app.coverage -->
