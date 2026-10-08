# Eventa — Test Cases

| | |
|---|---|
| **Author** | QA Engineer |
| **Version** | 1.0 · 2026-07-23 |
| **Companion** | [test-plan.md](test-plan.md) |
| **Scope** | 340 test cases across 13 epics, tracing to the 162 user stories in the [product backlog](../01-requirements-and-features/functional-requirements.md) |

> Each test case is derived from a user story's Given/When/Then acceptance criteria, and lists its traceability (`US-*`), priority, type, preconditions, test data, steps, and expected result. Priority defaults from MoSCoW (Must→High, Should→Medium, Could→Low). See [test-plan.md](test-plan.md) for strategy, environment, defect process, and closure.

## Coverage & traceability

| # | Epic (test cases) | Stories covered | Test cases | High | Med | Low |
|---|------|:---:|:---:|:---:|:---:|:---:|
| 1 | [Accounts & Sign-in](#tc-e01) | 12 | 24 | 19 | 5 | 0 |
| 2 | [Configure the Workspace & Team](#tc-e02) | 13 | 29 | 16 | 11 | 2 |
| 3 | [Create & Manage Events](#tc-e03) | 15 | 35 | 20 | 13 | 2 |
| 4 | [Public Event Pages](#tc-e04) | 10 | 21 | 11 | 9 | 1 |
| 5 | [Sell Tickets & Run Promotions](#tc-e05) | 12 | 24 | 9 | 12 | 3 |
| 6 | [Discover & Register for Events](#tc-e06) | 15 | 35 | 25 | 8 | 2 |
| 7 | [Communicate with Attendees](#tc-e07) | 10 | 20 | 3 | 13 | 4 |
| 8 | [Manage Registrations & Admit Attendees](#tc-e08) | 14 | 30 | 17 | 13 | 0 |
| 9 | [Get Paid & Manage Finances](#tc-e09) | 14 | 27 | 20 | 7 | 0 |
| 10 | [Build the Event Program](#tc-e10) | 13 | 24 | 14 | 9 | 1 |
| 11 | [Organizer Home & Dashboard](#tc-e11) | 13 | 25 | 14 | 9 | 2 |
| 12 | [Coordinate Meetings](#tc-e12) | 9 | 21 | 8 | 11 | 2 |
| 13 | [Measure Performance (Reports)](#tc-e13) | 12 | 25 | 10 | 13 | 2 |
| | **Total** | **162** | **340** | **186** | **133** | **21** |

---

<a id="tc-e01"></a>

# Accounts & Sign-in — Test Cases

Epic E1 — Accounts & Sign-in. Test basis: user stories US-ACC-01…US-ACC-12 and their acceptance criteria.
Locale notes: bilingual EN/TH surfaces; Asia/Bangkok time for "last active" and lockout timers; neutral, non-revealing messages are a recurring security rule across this epic — except the forgot-password form, which deliberately says when no account matches (US-ACC-04) and is bounded by its own attempt lock instead.

---

## US-ACC-01 — Create an organizer account

### TC-ACC-01 — Sign up, confirm email, then activate
- **Traces:** US-ACC-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** No account exists for the test email. Access to that email inbox.
- **Test data:** Name `Somchai Prasert`; email `organizer-new@eventa-test.co.th`; password `Bangkok#2026!` (meets minimum); Terms & Privacy checkbox ticked.
- **Steps:**
  1. Open the organizer sign-up page (verify EN/TH toggle is present).
  2. Enter name, email, password; tick Terms & Privacy; submit.
  3. Observe the on-screen result and attempt to reach the console home directly.
  4. Open the confirmation link from the inbox while it is still valid.
  5. Sign in with the same email and password.
- **Expected result:** After submit, a "check your inbox" message is shown; the user is NOT signed in and cannot reach the console until confirmed. Opening the valid link activates the account; the subsequent sign-in succeeds and lands on the organizer console home.

### TC-ACC-02 — Sign up with an already-registered email (no reveal)
- **Traces:** US-ACC-01  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** An organizer account already exists for the test email.
- **Test data:** Email `existing-org@eventa-test.co.th`; password `Chiangmai#77x`; Terms ticked.
- **Steps:**
  1. Open the organizer sign-up page.
  2. Enter the already-registered email with a valid password; tick Terms; submit.
  3. Compare the resulting message with the one seen in TC-ACC-01.
  4. Check the inbox of the existing account.
- **Expected result:** The same neutral "check your inbox" confirmation is shown — the UI never states the email is already registered. No duplicate account is created; the existing account is unchanged (no new confirmation/activation email that would let a stranger act on it).

### TC-ACC-03 — Sign-up validation: Terms unchecked and weak password
- **Traces:** US-ACC-01  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** On the organizer sign-up page.
- **Test data:** Email `weakpw@eventa-test.co.th`; weak password `12345`; then a strong password `Sukhumvit#2026`; Terms left unchecked first, then ticked.
- **Steps:**
  1. Type the weak password `12345` and observe live strength guidance.
  2. Attempt to submit with the weak password and Terms unchecked.
  3. Change the password to `Sukhumvit#2026`; keep Terms unchecked; attempt to submit.
  4. Tick Terms; submit.
- **Expected result:** Live guidance appears while typing the weak password and submit is blocked until the minimum is met. With Terms unchecked, submit is blocked and the user is asked to accept Terms & Privacy. Only when both the password meets the minimum and Terms are accepted does the form submit.

---

## US-ACC-02 — Organizer sign-in

### TC-ACC-04 — Successful organizer sign-in to console home
- **Traces:** US-ACC-02  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Active, email-confirmed organizer account.
- **Test data:** Email `organizer-active@eventa-test.co.th`; password `Bangkok#2026!`.
- **Steps:**
  1. Open the organizer sign-in page.
  2. Enter the correct email and password; submit.
- **Expected result:** The user is signed in and taken to the organizer console home (not the attendee portal).

### TC-ACC-05 — Wrong credentials show one neutral message
- **Traces:** US-ACC-02  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** An active organizer account exists.
- **Test data:** (a) correct email `organizer-active@eventa-test.co.th` + wrong password `WrongPass#1`; (b) unknown email `nobody@eventa-test.co.th` + any password.
- **Steps:**
  1. Sign in with a valid email but wrong password; note the message.
  2. Sign in with an email that is not registered; note the message.
- **Expected result:** Both attempts fail with a single, identical neutral message that does not reveal whether the email exists or which field was wrong. No session is created.

### TC-ACC-06 — Sign-in blocked for unconfirmed and for suspended accounts
- **Traces:** US-ACC-02  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** One account signed up but email not yet confirmed; one account whose access has been suspended.
- **Test data:** Unconfirmed `pending@eventa-test.co.th` / `Bangkok#2026!`; suspended `suspended@eventa-test.co.th` / correct password.
- **Steps:**
  1. Sign in with the unconfirmed account using correct details; check the inbox.
  2. Sign in with the suspended account using correct details.
- **Expected result:** The unconfirmed account is not signed in — the user is asked to confirm their email first and a fresh confirmation email is sent. The suspended account is refused sign-in even though the details are correct.

---

## US-ACC-03 — Attendee sign-in without forcing accounts

### TC-ACC-07 — Attendee lands on My Tickets, never the organizer console
- **Traces:** US-ACC-03  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Confirmed attendee account with at least one past registration/ticket.
- **Test data:** Email `attendee@eventa-test.co.th`; password `Ticket#2026th`.
- **Steps:**
  1. Open the attendee portal sign-in page.
  2. Enter correct attendee credentials; submit.
  3. Inspect the destination page and available navigation.
- **Expected result:** The user lands on the "My tickets / My events" page and is never taken inside the organizer console.

### TC-ACC-08 — Organizer-only email fails at attendee sign-in; guest can still register
- **Traces:** US-ACC-03  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** An organizer account exists whose email has no attendee account. A published event open for registration.
- **Test data:** Organizer-only email `organizer-active@eventa-test.co.th` + its password; guest details `guest@eventa-test.co.th`.
- **Steps:**
  1. At the attendee portal sign-in, enter the organizer-only email and password; submit.
  2. Separately, as a signed-out guest, browse the event and complete a free/paid registration without creating an account.
- **Expected result:** The organizer-only email is rejected with the neutral failure message and no attendee session is created. The guest completes registration and receives a ticket without ever creating an account.

---

## US-ACC-04 — Reset a forgotten password

### TC-ACC-09 — Full reset: link on its way, single-use link, global sign-out, previous password rejected
- **Traces:** US-ACC-04  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** An active organizer account with a password, signed in on two devices/browsers (A and B). Inbox access.
- **Test data:** Email `reset-me@eventa-test.co.th`; current password `OldPass#2025`; new password `NewStrong#2026`.
- **Steps:**
  1. On the organizer forgot-password screen, submit the registered email; note the on-screen message.
  2. Open the reset link from the inbox and set the new password `NewStrong#2026`; confirm.
  3. From device A (previously signed in), refresh a protected page.
  4. Sign in with the new password on a fresh session.
  5. Attempt to sign in with the old password `OldPass#2025`.
- **Expected result:** Step 1 confirms a reset link is on its way to that address, and the reset email arrives. The link works once; after confirming, every device is signed out (device A must sign in again) and only the new password works. The old password no longer signs in.

### TC-ACC-10 — Forgot-password says why no link was sent and locks repeated misses; an address in two workspaces gets the link it can use; expired/used link rejected; previous password barred
- **Traces:** US-ACC-04  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** An organizer account with no attendee account; an organizer account created only through Google (no password); an organizer account whose email is not confirmed; a teammate invitation sent (US-SET-11) and not yet accepted; a suspended organizer account; an address that owns an active workspace and also has an unaccepted invitation to a second one; a used or expired reset link; the account's current (soon-to-be "previous") password known; inbox access for every address. The attempt lock at its configured limit (default: 5 misses, 15-minute cool-off).
- **Test data:** Unknown email `noone@eventa-test.co.th` — no organizer account, and not asked about in the last 15 minutes (misses carry over between runs); organizer-only `organizer-active@eventa-test.co.th`; Google-only `social-new@eventa-test.co.th` (created in TC-ACC-13); unconfirmed `pending@eventa-test.co.th`; invited `invited@eventa-test.co.th`; suspended `suspended@eventa-test.co.th`; two-workspace `two-workspaces@eventa-test.co.th` (owner of "Two Co A", invited to "Two Co B"); an already-used reset link; on the reset form, re-enter the current password `NewStrong#2026` as the "new" one.
- **Steps:**
  1. On the organizer forgot-password screen (the organizer sign-in's "Forgot password?"), submit the unknown email; check the inbox.
  2. On the attendee forgot-password screen (the portal sign-in's "Forgot password?"), submit the organizer-only email; check the inbox.
  3. On the organizer forgot-password screen, submit in turn the Google-only, unconfirmed, invited and suspended emails; check each inbox.
  4. Submit the suspended email again, more times than the attempt limit.
  5. Within 15 minutes of step 1, submit the unknown email again until it is refused, then once more.
  6. Open an expired or already-used reset link.
  7. On a valid reset form, try to set the new password to the account's current password.
  8. On the organizer forgot-password screen, submit the two-workspace email; check its inbox.
- **Expected result:** No reset email is sent in steps 1–5. Step 1 says no organizer account uses that address, and to check the spelling, try the attendee account, or create one; step 2 gives the same answer naming the attendee account and pointing to the organizer one. In step 3 each account is told why there is no link: the Google-only account has no password to reset and should use "Continue with Google" (the answer names the provider the account is actually linked to — a LinkedIn- or Apple-only account is sent to LinkedIn or Apple, never to Google); the unconfirmed account should open its confirmation link, and signing in sends a fresh one (this form sends none); the invited teammate, who has no password yet, should open the invitation email and choose a password there — not be sent to a social sign-in; the suspended account should ask a workspace admin to reactivate it. In step 4 every attempt gets the same suspended answer and is never refused as too many attempts — only an address with no account counts as a miss. In step 5 the misses up to the limit (the 5th, counting step 1) still get the no-account answer; the next is refused with "Too many password-reset attempts. Try again in 15 minutes." and no longer says whether an account exists, and the extra attempt is refused the same way. The expired/used link is refused with a clear "no longer valid" notice plus an option to request a new one. Reusing the previous/current password is rejected. Step 8 says a reset link is on its way, and exactly one reset email arrives: it names "Two Co A" and its link resets that workspace's password; nothing is sent for "Two Co B", whose invitation is still open. (Had the invitation been accepted with a password, a second email naming "Two Co B" would arrive too; the on-screen answer is the same either way.)

---

## US-ACC-05 — Change my password while signed in

### TC-ACC-11 — Change password signs out other devices, keeps this one
- **Traces:** US-ACC-05  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Account signed in on device A (the one making the change) and device B.
- **Test data:** Current password `NewStrong#2026`; new password `EvenStronger#27`.
- **Steps:**
  1. On device A, open change-password; enter the correct current password and the new strong password; save.
  2. On device A, navigate to a protected page.
  3. On device B, refresh a protected page.
- **Expected result:** The password changes; device A stays signed in; device B (and all other devices) are signed out and must sign in again with the new password.

### TC-ACC-12 — Change rejected: wrong current password and reused password
- **Traces:** US-ACC-05  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Account signed in on device A and device B.
- **Test data:** Wrong current `NotMyPassword#9`; correct current `EvenStronger#27`; new = same as current `EvenStronger#27`.
- **Steps:**
  1. Enter a wrong current password with a valid new password; save.
  2. Check that device B is still signed in.
  3. Enter the correct current password but set the new password equal to the current one; save.
- **Expected result:** Both attempts are refused. On the wrong-current-password attempt nothing is changed and no device is signed out. Setting the new password equal to the current one is rejected.

---

## US-ACC-06 — Sign in with Google, Apple or LinkedIn

### TC-ACC-13 — New social sign-in creates a confirmed account at the right home
- **Traces:** US-ACC-06  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** No existing account for the social email. Started from the organizer sign-in page (Google/LinkedIn offered there).
- **Test data:** Provider Google; social email `social-new@eventa-test.co.th`.
- **Steps:**
  1. On the organizer sign-in page, choose Google; approve the provider consent screen.
  2. Observe the destination after the provider confirms the email.
  3. Repeat from the attendee portal (Google/Apple offered) with a different new email.
- **Expected result:** An account is created already marked as email-confirmed and the user lands on the correct home for the audience — organizer console when started from an organizer page, attendee portal when started from the attendee portal. The organizer-page flow only ever creates an organizer account (never crosses audiences); provider list matches the audience.

### TC-ACC-14 — Social links to existing password account; cancelled consent signs nobody in
- **Traces:** US-ACC-06  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** A password account already exists with the same email the provider will return.
- **Test data:** Provider Google; email `existing-both@eventa-test.co.th` (already has a password account).
- **Steps:**
  1. Choose the social provider and approve consent with the matching email; check for duplicate accounts.
  2. In a fresh attempt, choose the provider and cancel/decline the consent screen; return to the app.
- **Expected result:** The social login links to the existing account — no duplicate is created and the user reaches their home. When consent is cancelled, the user is told sign-in was cancelled and is not signed in.

---

## US-ACC-07 — Extra security with two-factor sign-in

### TC-ACC-15 — 2FA challenge accepts a valid code
- **Traces:** US-ACC-07  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Account with two-factor (authenticator app) turned on.
- **Test data:** Correct password; a current 6-digit TOTP code from the paired authenticator.
- **Steps:**
  1. Sign in with the correct password.
  2. When prompted, enter the current 6-digit code; submit.
- **Expected result:** After the correct password, a 6-digit code is requested before access is granted; entering a valid current code signs the user in.

### TC-ACC-16 — Recovery code is single-use; "trust this device" skips the code for 30 days
- **Traces:** US-ACC-07  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** 2FA enabled; a set of saved single-use recovery codes; authenticator "unavailable".
- **Test data:** One recovery code `R7K2-9QX4`; "Trust this device" ticked on a subsequent sign-in.
- **Steps:**
  1. Sign in with the correct password; at the 2FA prompt enter the recovery code `R7K2-9QX4`.
  2. Sign out, sign in again, and try the same recovery code `R7K2-9QX4` a second time.
  3. Complete sign-in with a valid code and tick "Trust this device"; sign out and sign in again from the same device within 30 days.
- **Expected result:** The recovery code lets the user in and can never be used again (second attempt is rejected). With "trust this device," a repeat sign-in from that same device within 30 days is not asked for a code.

---

## US-ACC-08 — Stay signed in on my device

### TC-ACC-17 — "Remember me" keeps the session across browser restart
- **Traces:** US-ACC-08  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Valid account credentials.
- **Test data:** Email `remember@eventa-test.co.th`; password `Bangkok#2026!`; "Remember me" ticked.
- **Steps:**
  1. Sign in with "Remember me" ticked.
  2. Fully close the browser and reopen it within the remembered period.
  3. Open a protected page.
- **Expected result:** The user is still signed in and reaches the protected page without re-entering credentials.

### TC-ACC-18 — Without "Remember me," closing the browser signs out
- **Traces:** US-ACC-08  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Valid account credentials.
- **Test data:** Same account; "Remember me" left unticked.
- **Steps:**
  1. Sign in with "Remember me" unticked.
  2. Fully close and reopen the browser.
  3. Open a protected page.
- **Expected result:** The user is signed out and must sign in again. (Complementary: after the remembered period lapses, a "Remember me" session also asks for sign-in again.)

---

## US-ACC-09 — See and sign out my other devices

### TC-ACC-19 — View active sessions and sign out another device
- **Traces:** US-ACC-09  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Account signed in on the current device (A) and on another device (B).
- **Test data:** Session A = current browser; Session B = second browser/device.
- **Steps:**
  1. Open the active-sessions list on device A.
  2. Review each entry's device, rough location and last-active time; confirm the current one is marked "This device" and offers no sign-out control there.
  3. Sign out session B and confirm.
  4. On device B, try to use a protected page; recheck the list on A.
- **Expected result:** Each session shows device, rough location and how recently it was active, with the current device clearly marked "This device" (no local sign-out control). After confirming, device B loses access the next time it is used and disappears from the list.

---

## US-ACC-10 — Sign out

### TC-ACC-20 — Sign out ends only the chosen audience's session
- **Traces:** US-ACC-10  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** In one browser, signed in as BOTH an organizer and an attendee (same email registered on both sides).
- **Test data:** Organizer session + attendee session active in the same browser.
- **Steps:**
  1. Choose Sign out from the organizer console.
  2. Try to open a protected organizer page.
  3. Open a protected attendee page (e.g. My Tickets).
- **Expected result:** The organizer session ends and returns to the sign-in screen; protected organizer pages require signing in again. The attendee session stays signed in and its protected pages remain accessible.

---

## US-ACC-11 — Keep organizer and attendee accounts cleanly separated

### TC-ACC-21 — Cross-audience access is redirected to the correct sign-in
- **Traces:** US-ACC-11  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** One session signed in as attendee only; one session signed in as organizer only.
- **Test data:** Attendee session; organizer session; a protected organizer URL and a protected attendee URL.
- **Steps:**
  1. As the attendee, open a protected organizer console page directly.
  2. As the organizer, open a personal attendee page directly.
  3. With the same email registered on both sides, reset/change the password on one audience and check whether the other audience's password still works; confirm both can be signed in at once in the same browser.
- **Expected result:** The attendee is sent to the organizer sign-in; the organizer is sent to the attendee sign-in. A password change/reset on one audience leaves the other unaffected, and both can be signed into simultaneously in the same browser.

### TC-ACC-22 — Signed-out deep link returns to the intended page after sign-in
- **Traces:** US-ACC-11  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Not signed in on either audience.
- **Test data:** A protected attendee page URL (e.g. My Tickets); valid attendee credentials.
- **Steps:**
  1. While signed out, open the protected page URL directly.
  2. Complete sign-in on the sign-in screen shown.
- **Expected result:** The user is taken to the correct audience sign-in and, after signing in, is returned to the page they originally requested.

---

## US-ACC-12 — Protect accounts from password-guessing

### TC-ACC-23 — Lockout after repeated failures, recovery after cool-off
- **Traces:** US-ACC-12  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** An active account with a known password.
- **Test data:** Email `lockout@eventa-test.co.th`; wrong password `Wrong#1`; correct password `Bangkok#2026!`.
- **Steps:**
  1. Submit the wrong password repeatedly until the attempt limit is reached.
  2. Immediately try again (even with the correct password) during the cool-off.
  3. Wait out the cool-off period, then sign in with the correct password.
- **Expected result:** Once the limit is reached, further attempts are refused for a cool-off period with a message to try again later or reset the password — this holds even for the correct password during cool-off. After the cool-off, the correct password signs in successfully.

### TC-ACC-24 — Lockout message identical for a non-existent account (no reveal)
- **Traces:** US-ACC-12  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** An email that is not registered.
- **Test data:** Unknown email `ghost@eventa-test.co.th`; any password.
- **Steps:**
  1. Submit repeated failed attempts for the unknown email until blocked.
  2. Compare the blocked/failure message with the one seen in TC-ACC-23.
- **Expected result:** The block and failure messages are identical to those for a real account and never reveal whether the account exists.


---

<a id="tc-e02"></a>

# Configure the Workspace & Team — Test Cases

Epic E2 — Configure the Workspace & Team. Area code: **SET**. Locale defaults: ฿ THB, VAT 7%, Asia/Bangkok, EN/TH.

---

## US-SET-01 — My profile & preferences

### TC-SET-01 — Edit name and phone, save and cancel
- **Traces:** US-SET-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as any team member with an existing profile (name, email, phone populated).
- **Test data:** New name "Somchai Rattana", new phone "+66 81 234 5678".
- **Steps:**
  1. Open My Profile; confirm the page loads with current name, email, phone, timezone, language and photo, all editable.
  2. Change name to "Somchai Rattana" and phone to "+66 81 234 5678", then Save.
  3. Reopen the page, edit the name again, then choose Cancel.
- **Expected result:** After step 2 the new name and phone are persisted and a confirmation is shown. After step 3 all fields revert to their last-saved values ("Somchai Rattana"), with no change kept.

### TC-SET-02 — Change email keeps old sign-in and marks new address unverified
- **Traces:** US-SET-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in with a verified email on file.
- **Test data:** Current email `somchai@old.co.th`; new email `somchai@new.co.th`.
- **Steps:**
  1. Open My Profile and change email to `somchai@new.co.th`, then Save.
  2. Observe the email status shown for the new address.
  3. Sign out and sign back in using the original email `somchai@old.co.th`.
- **Expected result:** The new email is shown as "unverified" and a confirmation link is sent to `somchai@new.co.th`. The original sign-in email keeps working until the new one is verified. A confirmation of the change is displayed.

### TC-SET-03 — Timezone Bangkok and language Thai apply to dates and copy
- **Traces:** US-SET-01  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Signed in; profile currently on a non-Bangkok timezone / English.
- **Test data:** Timezone "Asia/Bangkok (GMT+7)", language "ไทย (Thai)".
- **Steps:**
  1. Set timezone to Asia/Bangkok and language to Thai, then Save.
  2. Navigate to a screen showing timestamps (e.g. an event or activity list) and open a menu.
- **Expected result:** Dates/times render in Asia/Bangkok (GMT+7) and menu labels and copy render in Thai. Change persists across navigation.

---

## US-SET-02 — Change my password

### TC-SET-04 — Successful password change signs out other sessions
- **Traces:** US-SET-02  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in; the same account is also signed in on a second device/browser.
- **Test data:** Current password `OldPass!9`; new password `NewPass#7` (8+ chars, has a number and a symbol).
- **Steps:**
  1. Open Change Password, enter current `OldPass!9` and new `NewPass#7` with matching confirmation, then Submit.
  2. On the second device, attempt any authenticated action.
- **Expected result:** Password is updated with a confirmation shown; a "password was changed" email is sent; the second device is signed out and forced to re-authenticate.

### TC-SET-05 — Weak, mismatched, or wrong-current password is refused
- **Traces:** US-SET-02  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Signed in with password `OldPass!9`.
- **Test data:** (a) new `abc` / confirm `abc` (too weak, no number+symbol, <8); (b) new `NewPass#7` / confirm `NewPass#8` (mismatch); (c) current `WrongPass!1` + valid new `NewPass#7`.
- **Steps:**
  1. Attempt case (a) and Submit.
  2. Attempt case (b) and Submit.
  3. Attempt case (c) and Submit.
- **Expected result:** Each case is rejected with a specific reason (weak password / confirmation mismatch / wrong current password). The password remains `OldPass!9` in all cases and nothing is changed.

---

## US-SET-03 — Two-factor authentication & recovery codes

### TC-SET-06 — Enable 2FA and receive 8 recovery codes
- **Traces:** US-SET-03  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in; 2FA currently off; authenticator app available.
- **Test data:** Valid current 6-digit TOTP code from the paired authenticator.
- **Steps:**
  1. Start 2FA setup, scan the displayed QR/secret with the authenticator app.
  2. Enter the current 6-digit code and confirm.
  3. Note the recovery codes presented and download them.
- **Expected result:** 2FA is turned on and exactly 8 one-time recovery codes are shown once (downloadable). Codes are not shown again after leaving the screen.

### TC-SET-07 — Wrong/expired code leaves 2FA off; disable requires re-confirmation
- **Traces:** US-SET-03  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Signed in. For part B, 2FA is already on and the workspace does not force 2FA for everyone.
- **Test data:** Wrong code `000000` / a code that has expired; valid re-confirmation credential for disable.
- **Steps:**
  1. During setup, enter `000000` (or an expired code) and try to finish.
  2. With 2FA on, choose to turn it off and complete the required identity re-confirmation.
- **Expected result:** Step 1 keeps 2FA off with a prompt to try again. Step 2 requires re-confirming identity before disabling, and an email is sent that 2FA was disabled. (If the workspace requires 2FA for everyone, the disable option is unavailable to the member.)

### TC-SET-08 — Regenerate recovery codes invalidates old set
- **Traces:** US-SET-03  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Signed in; 2FA on; running low on recovery codes.
- **Test data:** One previously issued (still-unused) recovery code.
- **Steps:**
  1. Regenerate recovery codes and save the new set.
  2. Attempt to authenticate/step-up using one of the OLD codes.
- **Expected result:** A fresh set of 8 codes is issued; all old codes stop working (the old-code attempt is rejected).

---

## US-SET-04 — Review & sign out my active sessions

### TC-SET-09 — Session list marks current device with no sign-out control
- **Traces:** US-SET-04  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in on at least two devices/browsers.
- **Test data:** Current device plus one other session.
- **Steps:**
  1. Open Active Sessions and review the list.
- **Expected result:** Each session shows device, rough location and how recently it was used. The current device is clearly marked and has no "sign out" control; other sessions do.

### TC-SET-10 — Sign out one device and sign out all other sessions
- **Traces:** US-SET-04  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in on the current device plus two other sessions.
- **Test data:** One unfamiliar device entry.
- **Steps:**
  1. Sign out the unfamiliar device and confirm; observe the list.
  2. On that device, attempt an authenticated request.
  3. Choose "Sign out all other sessions" and confirm.
- **Expected result:** The signed-out device is logged out on its next attempt and its entry disappears. After step 3 every session except the current device is signed out; the current device stays signed in.

---

## US-SET-05 — Security & access audit log

### TC-SET-11 — Role change appears newest-first and is immutable
- **Traces:** US-SET-05  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin; a teammate's role was just changed (e.g. Staff → Organizer).
- **Test data:** Teammate "Nok", change Staff → Organizer.
- **Steps:**
  1. Open the audit log and locate the role-change entry.
  2. Attempt to edit or delete any entry from anywhere in the product.
- **Expected result:** Entry shows who changed it, from Staff to Organizer, and when. Entries are ordered newest-first and no edit/delete control exists anywhere.

### TC-SET-12 — Export records a new entry and masks secrets
- **Traces:** US-SET-05  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin; log contains an entry referencing a saved payment key.
- **Test data:** Export date range 2026-07-01 to 2026-07-27.
- **Steps:**
  1. Export the audit log for the given date range and let the download complete.
  2. Reopen the log and confirm the export was logged.
  3. Open the entry that involves a saved payment key.
- **Expected result:** A file scoped to that date range downloads; the export action itself is recorded as a new audit entry. The secret-bearing entry shows only a masked hint, never the full value.

---

## US-SET-06 — Notification preferences

### TC-SET-13 — Toggle email/SMS per topic independently and honor the setting
- **Traces:** US-SET-06  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in with a phone number on file.
- **Test data:** Topics registrations, payments, reminders, product updates. Turn SMS off for "reminders" while leaving email on.
- **Steps:**
  1. Open Notification Preferences and toggle email and SMS per topic; turn SMS off for "reminders".
  2. Trigger a reminder event.
- **Expected result:** Each topic exposes independent email and SMS switches. After turning SMS off for reminders, no reminder SMS is delivered while the reminder email still sends.

### TC-SET-14 — SMS switches unavailable without a phone; receipts always send
- **Traces:** US-SET-06  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Signed in with no phone number on file; payment alerts turned off.
- **Test data:** Account with empty phone field; a successful payment event.
- **Steps:**
  1. Open Notification Preferences and inspect the SMS switches.
  2. With payment alerts off, complete a successful payment.
- **Expected result:** SMS switches are unavailable with a prompt to add a phone number. Despite payment alerts being off, the required transactional receipt (VAT 7% itemized) still sends; only the optional payment alert is suppressed.

---

## US-SET-07 — Organization profile, tax details & branding

### TC-SET-15 — Save valid org identity; VAT 7% itemized on future receipts
- **Traces:** US-SET-07  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin.
- **Test data:** Org name "Eventa Thailand Co., Ltd.", Bangkok address, website `https://eventa.co.th`, 13-digit Thai tax ID `0105558000000`.
- **Steps:**
  1. Fill in name, address, website and the 13-digit tax ID, then Save.
  2. Issue a new receipt after saving.
  3. Change the address, then view a previously issued document.
- **Expected result:** Details are saved and appear on documents issued from then on; the new receipt itemizes VAT 7%. The previously issued document is unchanged — only future documents use the new address.

### TC-SET-16 — Invalid tax ID or website is rejected
- **Traces:** US-SET-07  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Signed in as Admin with valid details already saved.
- **Test data:** (a) tax ID `12345` (not 13 digits); (b) website `not-a-url`.
- **Steps:**
  1. Enter tax ID `12345` and Save.
  2. Restore the valid tax ID, enter website `not-a-url`, and Save.
- **Expected result:** Each save is blocked with a specific validation message (tax ID must be 13 digits / invalid website). Nothing is saved and previously stored valid details remain intact.

### TC-SET-17 — Organizer/Staff cannot change organization settings
- **Traces:** US-SET-07  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Signed in as an Organizer (repeat as Staff).
- **Test data:** Any org field.
- **Steps:**
  1. Navigate to the Organization Profile page.
- **Expected result:** The page is read-only or hidden; no field is editable and no save/change action is available for non-Admin roles.

---

## US-SET-08 — Connect our payment account

### TC-SET-18 — Test connection, then go live; test-mode banner shown
- **Traces:** US-SET-08  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin; no live payment account connected yet.
- **Test data:** Valid test-mode credentials, then valid live credentials.
- **Steps:**
  1. Enter test-mode credentials; confirm the payments page shows a banner that no real charges happen until going live.
  2. Choose "Test connection".
  3. Enter valid live credentials and Save.
- **Expected result:** In test mode the reminder banner is shown. "Test connection" reports whether credentials work with no money moving (or a clear failure reason). After connecting live credentials the workspace shows "Connected" and can accept real payments. Saved sensitive details are never shown back in full.

### TC-SET-19 — Disconnect switches off paid checkout, free events unaffected
- **Traces:** US-SET-08  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin; payments connected; existing paid orders and payouts on record; at least one free event and one paid event.
- **Test data:** Disconnect confirmation.
- **Steps:**
  1. Disconnect the payment account and confirm.
  2. Attempt checkout on a paid event, then register for a free event.
  3. Review past orders and payouts.
- **Expected result:** Paid checkout is switched off; free-event registration keeps working; past orders and payouts remain untouched.

---

## US-SET-09 — Choose payment methods at checkout

### TC-SET-20 — Enable PromptPay and disable a method reflect at checkout
- **Traces:** US-SET-09  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin; payments connected; cards and PromptPay available.
- **Test data:** Enable PromptPay; disable cards.
- **Steps:**
  1. Enable PromptPay and disable cards, then save.
  2. As an attendee, start checkout on a paid ฿ event and review the payment options.
- **Expected result:** PromptPay is offered at checkout; cards no longer appear on new orders.

### TC-SET-21 — Cannot switch off the last remaining method
- **Traces:** US-SET-09  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Signed in as Admin; still selling paid tickets; only one payment method (e.g. PromptPay) currently enabled.
- **Test data:** Attempt to disable PromptPay (the last enabled method).
- **Steps:**
  1. Turn off the only remaining enabled method and save.
- **Expected result:** The action is blocked with a prompt to keep at least one method enabled; the method stays on. (A wallet method whose prerequisites are unmet stays off with guidance on what to complete.)

---

## US-SET-10 — Checkout & receipt preferences

### TC-SET-22 — Statement label ≤22 chars saved; email receipt itemizes VAT 7%
- **Traces:** US-SET-10  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin; payments connected; "email receipts" on.
- **Test data:** Statement label "EVENTA TICKETS" (14 chars); a successful ฿ payment.
- **Steps:**
  1. Set the statement label to "EVENTA TICKETS" and Save.
  2. Complete a successful paid checkout.
- **Expected result:** The label is saved and appears on the attendee's card statement for later charges; the attendee receives an email receipt with VAT 7% itemized.

### TC-SET-23 — Invalid statement label and currency mismatch handling
- **Traces:** US-SET-10  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Signed in as Admin; organization currency is THB (฿).
- **Test data:** (a) label "OUR VERY LONG COMPANY NAME LIMITED" (>22 chars) / a label with disallowed characters "EVENTA<*>#"; (b) default currency set to USD while org currency is THB.
- **Steps:**
  1. Enter an over-length / disallowed-character label and Save.
  2. Set default charge currency to USD (different from org THB) and Save.
- **Expected result:** Step 1 is rejected with a prompt to shorten or fix the label before it is accepted. Step 2 shows a warning about the currency mismatch but is not blocked (the save succeeds).

---

## US-SET-11 — Invite & manage teammates

### TC-SET-24 — Invite a teammate; re-invite is re-sent not duplicated
- **Traces:** US-SET-11  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin.
- **Test data:** New teammate "Ploy", email `ploy@eventa.co.th`, role Staff. Then re-invite the same email; then attempt to invite an already-active member `admin@eventa.co.th`.
- **Steps:**
  1. Invite Ploy with role Staff and send.
  2. Filter the team list by status "Invited".
  3. Invite `ploy@eventa.co.th` again.
  4. Invite an already-active member `admin@eventa.co.th`.
- **Expected result:** Ploy appears as "Invited" with role/last-active shown, receives a join link, and the status count updates. Re-inviting the same pending email re-sends the invitation without creating a duplicate. Inviting an already-active member is rejected as already in the workspace.

### TC-SET-25 — Suspend, remove, and last-Admin protection
- **Traces:** US-SET-11  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Signed in as Admin; one active Organizer exists; you are the only remaining Admin.
- **Test data:** Suspend the Organizer; then attempt to remove/suspend/demote yourself (the last Admin).
- **Steps:**
  1. Suspend the Organizer, then have them attempt to sign in.
  2. Remove a departing member and confirm; check their past events/exports.
  3. Attempt to remove, suspend, or demote the last remaining Admin (yourself).
- **Expected result:** The suspended Organizer cannot sign in and is signed out, with their role preserved for later reactivation. The removed member loses access immediately while their past work is kept and they can be re-invited. Removing/suspending/demoting the last Admin is blocked so the workspace always retains an Admin.

---

## US-SET-12 — Assign roles & fine-tune permissions

### TC-SET-26 — Change role takes effect immediately and is recorded
- **Traces:** US-SET-12  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin; target member "Nok" is Staff and currently using the console.
- **Test data:** Change Nok from Staff to Organizer.
- **Steps:**
  1. Change Nok's role to Organizer and save.
  2. Have Nok (without signing out and in) attempt an Organizer-only action (e.g. run an event).
  3. Open the audit record for the change.
- **Expected result:** Nok gains Organizer capabilities immediately without re-login; the change is recorded with before/after (Staff → Organizer) and who made it. Standard scoping holds: Organizer cannot issue refunds or manage users/settings.

### TC-SET-27 — Cannot self-escalate; sensitive grants are flagged
- **Traces:** US-SET-12  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Signed in as a non-Admin with limited permissions (or an Admin attempting to exceed current access).
- **Test data:** (a) Attempt to grant yourself more access than you currently hold; (b) grant a member the "export attendee data" / "issue refunds" capability.
- **Steps:**
  1. Attempt to grant yourself elevated access and Save.
  2. Grant a member a sensitive capability (export attendee data or issue refunds) and Save.
- **Expected result:** Self-escalation beyond current access is blocked. Granting a sensitive capability succeeds but is clearly flagged in the record.

---

## US-SET-13 — Create custom roles

### TC-SET-28 — Create a custom role with chosen capabilities
- **Traces:** US-SET-13  ·  **Priority:** Low  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin.
- **Test data:** New role "Volunteer" with capabilities: view registrations + check-in only.
- **Steps:**
  1. Open the roles overview; confirm each role shows member count, description and headline capabilities and is searchable.
  2. Create a role "Volunteer" with the chosen capabilities.
  3. Assign "Volunteer" to a teammate.
- **Expected result:** The roles overview lists all roles with counts/descriptions/capabilities and supports search. "Volunteer" is created and becomes assignable to teammates.

### TC-SET-29 — Duplicate role name rejected; role management never left empty
- **Traces:** US-SET-13  ·  **Priority:** Low  ·  **Type:** Negative
- **Preconditions:** Signed in as Admin; a role "Organizer" already exists.
- **Test data:** Create role named "Organizer" (duplicate); then edit the only role that can manage users/roles to remove that capability.
- **Steps:**
  1. Attempt to create a role named "Organizer" and save.
  2. Edit roles such that no role would be left able to manage users and roles, then save.
- **Expected result:** The duplicate name is rejected with a request for a unique name. Any edit that would leave the workspace with no one able to manage users and roles is blocked. Saved capability edits apply to holders on their next use.


---
<a id="tc-e03"></a>

# Create & Manage Events — Test Cases

Epic E3 · Area code: EVT · Locale: Thai market (฿, VAT 7%, PromptPay, Asia/Bangkok, EN/TH)

---

## US-EVT-01 — Find and manage my events

### TC-EVT-01 — Events list shows Active first with summary, pagination and fill progress
- **Traces:** US-EVT-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as an Event Organizer whose workspace has more events than fit on one page (e.g. 27), a mix of Active and Completed.
- **Test data:** 27 events; page size such that N > 1 page; at least one event at 45/100 registrations.
- **Steps:**
  1. Open the "My events" list.
  2. Observe the default tab, ordering, and the results summary.
  3. Locate an event row and read its capacity indicator.
  4. Use the page controls to move to the next page.
- **Expected result:** Active events are shown first with most-recent activity surfaced; a "Showing X–Y of N events" summary is visible (e.g. "Showing 1–10 of 27 events"); page controls work. The 45/100 row shows a registrations-vs-capacity progress bar reading 45% fill.

### TC-EVT-02 — Search, type filter, sort and Active/Completed switch reset to first page; empty state is friendly
- **Traces:** US-EVT-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Organizer on the events list, currently viewing page 3 of results.
- **Test data:** Search term "Songkran"; event type = "Conference"; sort = "By registrations"; a search term that matches nothing (e.g. "zzzz").
- **Steps:**
  1. From page 3, type "Songkran" in search.
  2. Change the event-type filter to "Conference" and the sort to "By registrations".
  3. Switch from Active to Completed.
  4. Replace the search with "zzzz" so nothing matches.
- **Expected result:** After each action the list narrows to matching events and returns to the first page of results. Completed switch shows completed events. When nothing matches, a friendly "No events match" message is shown instead of a blank screen.

### TC-EVT-03 — Delete and Duplicate update Active/Completed counts live without reload
- **Traces:** US-EVT-01  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Organizer on events list; a Draft event with zero registrations exists (deletable); counts visible on the Active/Completed tabs.
- **Test data:** Active count = 12; Completed count = 4; one deletable draft.
- **Steps:**
  1. Open a row's action menu and confirm all options are present: View details, Edit, Duplicate, Delete.
  2. Choose Duplicate on any event; observe the Active count.
  3. Choose Delete on the eligible draft and confirm; observe the Active count.
- **Expected result:** Action menu offers View / Edit / Duplicate / Delete. After Duplicate, Active increases by 1 immediately; after Delete, Active decreases — both without a page reload.

---

## US-EVT-02 — Create an event with a guided wizard and save drafts

### TC-EVT-04 — Wizard progress, live summary, and Save as draft resume
- **Traces:** US-EVT-02  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Event Organizer with create rights.
- **Test data:** Name "TechConf BKK 2026"; one ticket type "Early Bird ฿890 × 100"; General admission capacity 100.
- **Steps:**
  1. Start a new event; confirm the wizard shows a progress indicator and step rail (Basics, Date & location, Seating, Tickets, Review & publish).
  2. Enter the name and a ticket type; watch the live summary panel.
  3. On any step, click "Save as draft".
  4. Leave the wizard, then open the events list and reopen the event.
- **Expected result:** Wizard shows progress + step rail and a live summary reflecting name, ticket types, capacity and seating as entered. "Save as draft" stores the event as a Draft that appears in the events list; reopening restores the entered data so work resumes.

### TC-EVT-05 — Leaving with unsaved changes prompts to save as draft; autosave preserves interrupted work
- **Traces:** US-EVT-02  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Organizer partway through the wizard with unsaved edits on the current step.
- **Test data:** Partially completed Basics step (title typed, not yet saved).
- **Steps:**
  1. Enter a title but do not click Save.
  2. Attempt to navigate away / close the wizard.
  3. Also move between steps and pause to check autosave/recovery.
- **Expected result:** On leaving, the user is asked whether to save the event as a draft before leaving. Moving between steps or pausing preserves work automatically so an interrupted session is recoverable.

### TC-EVT-06 — Concurrent edit conflict prompts reload; published edit applies without going offline
- **Traces:** US-EVT-02  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Two members (A and B) can edit the same event; the event also exists in a published state for the second part.
- **Test data:** Same event opened by A and B; A changes the title after B opened it.
- **Steps:**
  1. B opens the event in the wizard.
  2. A edits and saves a change to the same event.
  3. B makes an edit and clicks Save.
  4. Separately, on an already-published event, make a valid change and Save.
- **Expected result:** B is told the event changed elsewhere and is prompted to reload the latest version — B's save does not silently overwrite A's edits. Editing a published event and saving valid changes applies them without taking the event offline.

---

## US-EVT-03 — Describe the event (Basics)

### TC-EVT-07 — Title, formatted description with live count, category, tags and highlight badges save correctly
- **Traces:** US-EVT-03  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Organizer on the Basics step; workspace has at least one category defined.
- **Test data:** Title "Bangkok Design Week"; a formatted ~120-char description; category "Conference"; tags "design, ux"; two highlight badges (icon + label) plus one empty badge; a valid JPG cover under 5 MB.
- **Steps:**
  1. Enter the title and a formatted description; watch the character counter.
  2. Pick the category and add the two tags.
  3. Add two highlight badges in order plus leave one badge empty.
  4. Upload the valid cover image and Save.
- **Expected result:** Description shows a live character count within the 250 cap; category and tags are saved. Highlight badges appear on the public page in the entered order; the empty badge is dropped on save. Valid cover is accepted.

### TC-EVT-08 — Description hard cap at 250 characters; over-paste truncates and counter turns red
- **Traces:** US-EVT-03  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Organizer on the Basics step.
- **Test data:** A clipboard string of 300 characters.
- **Steps:**
  1. Click into the description field.
  2. Paste the 300-character string.
  3. Read the counter and the retained text.
- **Expected result:** Only the first 250 characters are kept; the character counter displays in red to signal the limit was hit.

### TC-EVT-09 — Cover image rejects wrong type / over-5MB; blank title blocks publish
- **Traces:** US-EVT-03  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Organizer on the Basics step.
- **Test data:** A .pdf file; a 7 MB .jpg; event with title left blank.
- **Steps:**
  1. Attempt to upload the .pdf as cover image.
  2. Attempt to upload the 7 MB .jpg.
  3. Leave the title blank and try to publish the event.
- **Expected result:** Each invalid upload is rejected with a clear message and nothing is stored. Publishing with a blank title is blocked with "Add an event title before publishing."

---

## US-EVT-04 — Set the date, time and location

### TC-EVT-10 — In-person requires venue/address; dates display in Bangkok time
- **Traces:** US-EVT-04  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Organizer on the Date & location step.
- **Test data:** Start 2026-08-15 09:00, End 2026-08-15 17:00 (Asia/Bangkok); type In-person; venue "BITEC", address "Bangna, Bangkok".
- **Steps:**
  1. Set start and end date-times.
  2. Choose In-person and enter venue name and address.
  3. Save, then view the date wherever it displays (summary, preview).
- **Expected result:** In-person event saves with venue + address required before publishing; dates display in Bangkok time everywhere in the product.

### TC-EVT-11 — End-before-start blocked; Online hides venue and keeps meeting link private
- **Traces:** US-EVT-04  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Organizer on the Date & location step.
- **Test data:** Start 2026-08-15 17:00, End 2026-08-15 09:00 (end not after start); then type Online with meeting link "https://meet.example/xyz".
- **Steps:**
  1. Set end time earlier than start and attempt to proceed/publish.
  2. Correct the end time, then switch type to Online and enter the meeting link.
  3. Confirm venue/address are no longer required.
  4. Open the public page preview and check the meeting link is not shown.
- **Expected result:** When end is not after start, the user is blocked with "End time must be after the start time." For Online, venue/address are not required; the meeting link stays private and is never shown publicly (delivered only after registration).

---

## US-EVT-05 — Choose seating (general admission or reserved)

### TC-EVT-12 — Reserved seat map preview and total-seat count (rows × seats/row)
- **Traces:** US-EVT-05  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** In-person event on the Seating step.
- **Test data:** Reserved seating with 10 rows × 20 seats/row (expected 200 seats); alternate run: General admission headcount 150.
- **Steps:**
  1. Choose Reserved and set rows = 10, seats/row = 20.
  2. Observe the seat-map preview and total-seat count.
  3. Switch to General admission and set headcount 150.
- **Expected result:** Reserved shows a live seat-map preview with total = 200 (rows × seats/row). General admission simply captures a headcount (150) with no specific seat assignment.

### TC-EVT-13 — Seat map smaller than ticket quantities warns on publish; Online offers no seating
- **Traces:** US-EVT-05  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Two events available: (a) in-person Reserved with 100 seats but total ticket quantities of 150; (b) an Online event.
- **Test data:** (a) Reserved 5×20 = 100 seats, tickets summing to 150; (b) Online event.
- **Steps:**
  1. On event (a) proceed to publish.
  2. On event (b) open the Seating step.
- **Expected result:** Event (a) shows a warning that the seat map (100) is smaller than the ticket quantities (150). Event (b) offers no seating options and tells the user a join link is emailed after registration.

### TC-EVT-14 — Reducing rows/seats on a published reserved event never removes sold seats
- **Traces:** US-EVT-05  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Published reserved event with some seats already sold (e.g. seats in row 5 sold).
- **Test data:** Reserved map 6 rows × 20; sold seats include row 5; attempt to reduce to 4 rows.
- **Steps:**
  1. Edit seating and reduce rows from 6 to 4 (which would drop row 5).
  2. Save.
- **Expected result:** Seats already sold are never removed; the reduction cannot strip out sold seats (either blocked or those seats retained).

---

## US-EVT-06 — Set up tickets, capacity and registration rules

### TC-EVT-15 — Ticket rows require unique name, ฿ price (0 allowed) and quantity; at least one type, last row locked
- **Traces:** US-EVT-06  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Organizer on the Tickets step.
- **Test data:** Ticket "General ฿890 × 100"; a free ticket "Student ฿0 × 50"; attempt a duplicate name "General"; then attempt to remove the only remaining row.
- **Steps:**
  1. Add ticket "General" at ฿890 × 100.
  2. Add a free ticket "Student" at ฿0 × 50.
  3. Try to add a second ticket also named "General".
  4. Remove all but one row, then try to remove the last row.
- **Expected result:** Each ticket needs a name unique within the event, a ฿ price (฿0 accepted for free), and a quantity. Duplicate name "General" is rejected. At least one ticket type must exist to publish and the last row cannot be removed.

### TC-EVT-16 — ฿890 paid ticket recorded VAT-inclusive at 7% (net ฿831.78 + VAT ฿58.22)
- **Traces:** US-EVT-06  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Organizer on the Tickets step; finance/receipt view accessible.
- **Test data:** Paid ticket priced ฿890.
- **Steps:**
  1. Add and save a ticket at ฿890.
  2. Inspect the finance/receipt breakdown for that price.
- **Expected result:** ฿890 is treated as VAT-inclusive at 7% — finance records ฿831.78 net + ฿58.22 VAT — so receipts and reporting are correct.

### TC-EVT-17 — Quantity/capacity below sold is rejected; out-of-window registration closes; approval and waitlist behave
- **Traces:** US-EVT-06  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Published event with 60 tickets already sold on a type of quantity 100; registration window and toggles configurable.
- **Test data:** Try to set that ticket's quantity to 50 (below 60 sold); try event capacity below total registered; registration window 2026-08-01 to 2026-08-10 with current date 2026-07-27 (before window); Require approval ON; Waitlist ON then OFF.
- **Steps:**
  1. Set the ticket quantity to 50 and save.
  2. Set overall capacity below the already-registered count and save.
  3. Set a registration window that does not include the current Bangkok time and view the public page.
  4. Turn Require approval on; turn Waitlist on, then off, and check sold-out behaviour.
- **Expected result:** Setting quantity/capacity below what is already sold/registered is rejected with a message showing the already-committed count (e.g. 60). Outside the registration window (judged in Bangkok time) public registration is closed even though the event is published. With approval on, registrations are held for review; with waitlist on, attendees can join a waitlist once sold out; with it off, the event shows "sold out."

---

## US-EVT-07 — Pick a public page template, set visibility, and publish

### TC-EVT-18 — Template gallery auto-fills real content; preview reflects unsaved details; publish goes live
- **Traces:** US-EVT-07  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** A complete event (title, description, date/time, location, ≥1 ticket type) on the Review & publish step; user has publish rights.
- **Test data:** Templates Classic / Spotlight / Minimal / Vibrant; visibility = Public; an unsaved tweak to the title.
- **Steps:**
  1. Select each template and confirm the event's own title, date, venue, agenda, speakers, highlights and tickets fill the page.
  2. Make an unsaved change to the title, then click Preview.
  3. Set visibility to Public and click Publish.
- **Expected result:** Selecting a template auto-populates the page from the event's real content (no separate page authoring). Preview opens in a new tab reflecting entered details including unsaved ones. Publishing a complete event makes it live at its public link, discoverable because Public, and the team receives an "Event published" notice.

### TC-EVT-19 — Missing required items block publish (cover only a warning); visibility scopes; past-date confirm
- **Traces:** US-EVT-07  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** An event missing at least one required item, plus a separate complete event whose start date is already in the past.
- **Test data:** Event missing description and location and with no cover image; visibility options Public / Unlisted / Private; a complete event with start 2026-07-01 (past relative to 2026-07-27).
- **Steps:**
  1. On the incomplete event click Publish.
  2. Check whether the missing cover image blocks or only warns.
  3. Set visibility to Unlisted, then Private, and note reachability rules.
  4. On the past-dated complete event, click Publish.
- **Expected result:** Publishing is blocked and the missing required items (description, location) are flagged; a missing cover image is only a warning, not a blocker. Public lists on Discover, Unlisted is reachable only by direct link, Private is admin-only. Publishing a past-dated event asks for confirmation before it goes live.

### TC-EVT-20 — Publish disabled without publish rights
- **Traces:** US-EVT-07  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Organizer without publish rights viewing a complete, ready event.
- **Test data:** Role lacking publish permission.
- **Steps:**
  1. Open the Review & publish step.
  2. Attempt to publish.
- **Expected result:** Publish is disabled and the user is told they don't have permission to publish.

---

## US-EVT-08 — Cancel or delete an event without losing paid registrations

### TC-EVT-21 — Delete a draft with zero registrations after confirmation
- **Traces:** US-EVT-08  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Admin viewing a Draft event with zero registrations and no payment history.
- **Test data:** Empty draft event.
- **Steps:**
  1. Choose Delete.
  2. Confirm the deletion in the confirmation prompt.
- **Expected result:** Deletion always requires confirmation; on confirm the empty draft is permanently removed.

### TC-EVT-22 — Delete blocked for events with registrations/payment history; routed to Cancel
- **Traces:** US-EVT-08  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Admin viewing a published event (or any event with registrations or payment history).
- **Test data:** Published event with paid registrations.
- **Steps:**
  1. Choose Delete on the event.
- **Expected result:** Hard deletion is blocked and Cancel is offered: "This event has registrations and can't be deleted. Cancel it instead to refund and notify attendees." No confirmation-free deletion is possible.

### TC-EVT-23 — Cancel requires reason + confirm, voids waitlist, invalidates tickets, queues refunds, notifies in-language; succeeds even if refunds still processing
- **Traces:** US-EVT-08  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Admin with publish rights on a published event holding paid registrations, a waitlist, and issued tickets; attendees with EN and TH language preferences.
- **Test data:** Reason "Venue unavailable"; paid attendees (some refunds slow to process); TH-preference and EN-preference attendees.
- **Steps:**
  1. Choose Cancel; leave the reason blank and try to confirm, then enter "Venue unavailable" and confirm.
  2. Observe registration acceptance, waitlist, issued tickets, and refund queue.
  3. Simulate some refunds still processing and complete the cancel.
  4. Return to the events list.
- **Expected result:** Cancel requires a reason and confirmation. On cancel: the event stops accepting registrations, the waitlist is voided, issued tickets are invalidated, refunds are queued for paid attendees, and affected attendees receive a cancellation message in their own language (EN/TH). If some refunds are still processing the cancellation still succeeds and the admin is told refunds are still being handled. The event moves out of Active but is retained for reporting.

---

## US-EVT-09 — Manage the event program (agenda and speakers)

### TC-EVT-24 — Add/remove agenda sessions sorted by start time; overlap warns but doesn't block; add speaker card
- **Traces:** US-EVT-09  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Organizer in the event workspace on the Agenda/Speakers tabs.
- **Test data:** Session A "Keynote", Day 1, 10:00, 60 min, Room "Hall 1", speaker "Anong"; Session B same Room "Hall 1" 10:30 (overlaps A); a session with a blank title; speaker "Somchai" (name only).
- **Steps:**
  1. Add Session A; verify it appears on Day 1 sorted by start time.
  2. Add Session B overlapping A in the same room.
  3. Try to add a session with no title.
  4. Remove Session A and view the public agenda.
  5. On Speakers, add "Somchai" with name only.
- **Expected result:** Sessions appear on their chosen day sorted by start time; a title is required (blank-title session rejected). Overlapping sessions in the same room warn but are not blocked (multi-track still works). Removing a session updates the public agenda immediately. Adding a speaker (name required, rest optional) shows a card that appears on the public page.

---

## US-EVT-10 — Adjust ticket inventory after publishing

### TC-EVT-25 — Add ticket type after publishing starts at 0 sold / ฿0 and updates public list
- **Traces:** US-EVT-10  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Published event; Organizer on its Tickets tab.
- **Test data:** New ticket "VIP" at ฿1,250 × 200, optional description, "Popular" flag on.
- **Steps:**
  1. Add ticket "VIP" ฿1,250 × 200 with the Popular flag.
  2. Save and view the ticket row and the public ticket list/capacity.
- **Expected result:** The ticket appears immediately at 0/200 sold and ฿0 revenue; the public ticket list and overall capacity update to include it.

### TC-EVT-26 — Duplicate ticket name rejected; ticket with sales cannot be removed
- **Traces:** US-EVT-10  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Published event with an existing ticket "General" that has sales.
- **Test data:** Attempt new ticket named "General"; attempt to remove "General" which has sales.
- **Steps:**
  1. Add a ticket named "General" (already exists) and save.
  2. Try to remove the "General" ticket that has sales.
- **Expected result:** Duplicate name is rejected as a duplicate. Removal of a ticket with sales is blocked with "This ticket type has sales and can't be removed. Close it instead."

---

## US-EVT-11 — Organize events with categories

### TC-EVT-27 — Create/edit category with live preview and unique name; edits propagate to events
- **Traces:** US-EVT-11  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Admin in the categories view; workspace has an existing category "Conference".
- **Test data:** New category "Workshop", icon + colour, description; then edit "Workshop" colour; attempt duplicate name "Conference".
- **Steps:**
  1. Browse categories and confirm search/sort and each showing icon, colour, description, and live event count.
  2. Create "Workshop" with icon/colour/description and watch the live preview.
  3. Try to create a second category named "Conference".
  4. Edit "Workshop" colour and check an event using it.
- **Expected result:** Category create/edit shows a live preview of icon, colour and name; the name must be unique in the workspace and the duplicate "Conference" is rejected. Editing a category's name/icon/colour is reflected on every event that uses it.

### TC-EVT-28 — Delete empty category succeeds; delete category in use is blocked
- **Traces:** US-EVT-11  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Admin with one empty category and one category used by 12 events.
- **Test data:** Empty category "Old"; category "Seminar" used by 12 events.
- **Steps:**
  1. Delete the empty "Old" category and confirm.
  2. Try to delete "Seminar" (used by 12 events).
- **Expected result:** The empty category is removed after confirmation. Deleting a category still used by events is blocked until those events are moved/cleared: "This category has 12 events. Move them to another category before deleting."

---

## US-EVT-12 — See my events on a calendar and what's coming up

### TC-EVT-29 — Month calendar with header count, day roll-up, and soonest-first upcoming with Bangkok "days left"
- **Traces:** US-EVT-12  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Organizer with several events this month, including a day holding 4+ events, and future events; current date 2026-07-27 (Asia/Bangkok).
- **Test data:** Month with 6 events; one day with 4 events; an empty day; upcoming events dated 2026-07-30 and 2026-08-05.
- **Steps:**
  1. Open the calendar; read the header and each event on its start date.
  2. Inspect the busy day cell and the empty day cell.
  3. Move between months and use Today.
  4. Open the upcoming view and check ordering and "days left".
  5. Click an event on the calendar and on the upcoming view.
- **Expected result:** Header reads "{n} events this month"; each event sits on its start date; month navigation and Today work. A busy day shows up to three events with a "+N more" roll-up and picking the day lists all its events; an empty day reads "No events scheduled." Upcoming events are sorted soonest-first with correct "days left" in Bangkok time and how full each is. Clicking an event opens its detail workspace.

---

## US-EVT-13 — Duplicate an event to relaunch quickly

### TC-EVT-30 — Duplicate creates zeroed "(Copy)" draft with own link and cleared sales window
- **Traces:** US-EVT-13  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Organizer with a source event holding 312 registrations, full content (details, schedule, seating, tickets, agenda, speakers, template, visibility), and a set registration window.
- **Test data:** Source event "Annual Gala" with 312 registrations and a past open/close window.
- **Steps:**
  1. Duplicate "Annual Gala".
  2. Open the new draft and compare content, counts, public link, and registration window.
  3. Return to the events list.
- **Expected result:** A new draft "Annual Gala (Copy)" is created with all content copied but 0 registrations, 0 sold, ฿0 revenue, empty waitlist and no attendees; it gets its own unique public link and any past open/close dates are cleared so the window must be re-set. It appears next to the original and Active/Completed counts update.

### TC-EVT-31 — Double-triggering Duplicate creates only one copy
- **Traces:** US-EVT-13  ·  **Priority:** Low  ·  **Type:** Edge
- **Preconditions:** Organizer on the events list.
- **Test data:** Any source event.
- **Steps:**
  1. Trigger Duplicate on an event.
  2. Immediately trigger Duplicate again (double-click / rapid repeat) before the first completes.
- **Expected result:** Only one copy is created despite the accidental second trigger.

---

## US-EVT-14 — Monitor an event's performance from its workspace

### TC-EVT-32 — Overview headline numbers, copy public link, and registrations/attendees tabs
- **Traces:** US-EVT-14  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Organizer with full access on a published event's workspace.
- **Test data:** Event 120/300 registered; revenue ฿256,800; several registrations with statuses Paid/Pending/Refunded; confirmed attendees list.
- **Steps:**
  1. Open Overview and read registrations-vs-capacity %, revenue (฿), tickets sold, days left; click "Copy" for the public link.
  2. Open Registrations, filter by Paid, Pending, Refunded, and page through.
  3. Open Attendees, read the count badge, and click "Email all".
- **Expected result:** Overview shows live registrations-vs-capacity with a percentage (120/300 = 40%), revenue in ฿, tickets sold and days left; "Copy" copies the event's public link. Registrations filters update the list and its counts and show attendee, ticket, amount (฿) and registration time. Attendees shows confirmed attendees with a count badge; "Email all" opens a message that requires explicit confirmation before sending.

### TC-EVT-33 — Revenue hidden for members without finance access
- **Traces:** US-EVT-14  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Team member with registration access but no finance access on a published event.
- **Test data:** Role without finance permission.
- **Steps:**
  1. Open the event Overview.
- **Expected result:** Registration and attendance numbers still show, but revenue is hidden.

---

## US-EVT-15 — Promote an event with share links and a printable flyer

### TC-EVT-34 — Copy link, channel share messages, and flyer with scannable QR
- **Traces:** US-EVT-15  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Organizer on a Public published event's Share dialog.
- **Test data:** Event title "TechConf BKK 2026", date 2026-08-15, location "BITEC, Bangkok"; channels Facebook, X, LINE, WhatsApp, Email.
- **Steps:**
  1. Click "Copy link" and observe the button.
  2. Pick each channel (Facebook, X, LINE, WhatsApp, Email) and read the pre-filled message.
  3. Open the flyer preview and save it.
- **Expected result:** "Copy link" copies the public link and the button confirms "Copied!". Each channel opens a pre-filled share message in the form "Join me at {title} — {date} at {location}". Saving the flyer downloads a poster image containing a scannable QR code that leads to the event's registration page.

### TC-EVT-35 — Sharing a non-public event warns the link may not be reachable
- **Traces:** US-EVT-15  ·  **Priority:** Low  ·  **Type:** Negative
- **Preconditions:** Organizer on a Private or Unlisted event's Share dialog.
- **Test data:** Event with visibility Private.
- **Steps:**
  1. Open the Share dialog for the non-public event.
  2. Attempt to copy/share the link.
- **Expected result:** The user is warned the link may not be publicly reachable.


---

<a id="tc-e04"></a>

# Public Event Pages — Test Cases

Epic E4 — Public Event Pages. Area code: PAGE. Locale context: Thai market, ฿ (VAT 7% inclusive), PromptPay, Asia/Bangkok timezone, EN/TH bilingual.

---

## US-PAGE-01 — Open a shareable event page

### TC-PAGE-01 — Open a published event from its shared link (guest, no login)
- **Traces:** US-PAGE-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** A published in-person event exists ("Bangkok Tech Meetup") with title, date, time, location and at least one ticket tier. Tester is not signed in (guest / incognito).
- **Test data:** Event "Bangkok Tech Meetup", 15 Aug 2026, 18:00–21:00 (Asia/Bangkok), Venue "True Digital Park, Sukhumvit", ticket "General — ฿500".
- **Steps:**
  1. Open the event's public shared link in a fresh browser session with no active login.
  2. Observe the page content without interacting with any auth control.
- **Expected result:** The page renders showing title, date, time, location and the ticket(s). No login/sign-up prompt or wall appears at any point. Dates/times display in Bangkok time. (AC: shared link shows key facts with no prompt to log in.)

### TC-PAGE-02 — Link to an unpublished or non-existent event shows an "unavailable" message
- **Traces:** US-PAGE-01  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** One event exists in draft/unpublished state; a known-invalid public address is available.
- **Test data:** (a) URL of an unpublished/draft event; (b) URL with a non-existent slug e.g. `/e/does-not-exist-2026`.
- **Steps:**
  1. Open the unpublished event's public URL.
  2. Separately, open the non-existent slug URL.
- **Expected result:** In both cases a clear "this event page isn't available" message is shown. No other event's content is displayed, and no broken/error screen appears. (AC: link to no live event shows a clear unavailable message, never the wrong event or a broken screen.)

### TC-PAGE-03 — Empty sections are hidden and Thai content reads correctly in Bangkok time
- **Traces:** US-PAGE-01  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** A published event configured in Thai (TH) language with NO highlights, NO agenda, NO speakers and NO FAQs, but with title, date/time and one ticket.
- **Test data:** Event "งานสัมมนากรุงเทพ", 20 ก.ย. 2026 09:00–12:00 (Asia/Bangkok), ticket "ทั่วไป — ฿300".
- **Steps:**
  1. Open the Thai event's public page.
  2. Scroll through the full page looking for section headings.
- **Expected result:** No empty section heading appears for highlights, agenda, speakers or FAQs — each absent section is omitted entirely, not shown as an empty heading. All Thai text renders correctly, and the date/time reads correctly in Thai and Bangkok time. (AC: empty sections hidden entirely; TH text and dates/times correct in Bangkok time.)

---

## US-PAGE-02 — Event identity and one-tap Register

### TC-PAGE-04 — Register button reachable at every position and lands on registration in the same tab
- **Traces:** US-PAGE-02  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Published event with an open registration window and at least one purchasable ticket tier.
- **Test data:** Event "Bangkok Tech Meetup", ticket "General — ฿500", registration open.
- **Steps:**
  1. Open the event page; confirm a Register / Get tickets button is present in the top bar and in the hero.
  2. Scroll to the Tickets section; confirm each tier carries its own Register button.
  3. Scroll to the foot of the page; confirm a Register button is present.
  4. Click one of the Register buttons.
- **Expected result:** A Register / Get tickets button is available at the top, hero, per-ticket, and page foot. Clicking it opens the registration step for this event in the same tab (no new tab, no dead end). (AC: Register reachable at all positions; click lands on registration in the same tab.)

### TC-PAGE-05 — Registration not yet open disables Register with a hint; missing cover shows branded fallback
- **Traces:** US-PAGE-02  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Published event whose registration opening date is in the future, and whose cover image is unset or fails to load.
- **Test data:** Event "Future Expo 2027", registration opens 01 Jan 2027 (today is 2026-07-27); no cover image uploaded.
- **Steps:**
  1. Open the event page before its registration window opens.
  2. Inspect the Register button state and any accompanying hint.
  3. Observe the hero background where the cover image would appear.
- **Expected result:** The Register button is visibly unavailable (disabled) with a short "registration isn't open yet" hint; it does not lead to a dead end. The hero shows a branded background instead of a broken-image icon. (AC: registration not open → disabled Register with hint; missing/failed cover → branded background, never a broken image.)

---

## US-PAGE-03 — Clear about section and online-event handling

### TC-PAGE-06 — Online event states "Online" and promises the join link, never exposing it or an address
- **Traces:** US-PAGE-03  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** A published online event with a configured private join link.
- **Test data:** Event "Remote UX Workshop", type Online, private join link `https://meet.example.com/xyz-secret`.
- **Steps:**
  1. Open the online event's public page.
  2. Read the details/about section thoroughly.
  3. Search the visible page for the private join URL and for any physical address.
- **Expected result:** The page shows "Online event" and a note that the join link is sent after registering. No physical address is shown. The private join link is never displayed publicly — only the promise of when it will be received. (AC: online event shows "Online event", join-link-after-register note, never a physical address, never the private link.)

### TC-PAGE-07 — In-person event shows venue details and omits blank rows
- **Traces:** US-PAGE-03  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** A published in-person event with venue, date, time, category and price set, but with the street address field left blank.
- **Test data:** Event "Chiang Mai Coffee Fair", venue "TCDC Chiang Mai", address (blank), category "Food & Drink", price ฿250.
- **Steps:**
  1. Open the in-person event's public page.
  2. Review the details section row by row.
- **Expected result:** Venue, date, time, category and price are shown. The blank address row is simply left out — no empty "Address:" label — and every other detail still renders. (AC: in-person shows venue/address/date/time/category/price; a blank detail row is omitted while the rest still shows.)

---

## US-PAGE-04 — Highlights, agenda and speakers

### TC-PAGE-08 — Highlights, agenda and speakers render in organizer order with custom titles
- **Traces:** US-PAGE-04  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** A published event with 3 highlights, a 4-item agenda and 3 speakers arranged in a specific organizer order, and custom titles set for the schedule and speaker sections.
- **Test data:** Agenda order [Registration, Keynote, Break, Panel]; speakers [Anan, Bua, Chai]; custom titles "กำหนดการ" (schedule) and "วิทยากรรับเชิญ" (speakers).
- **Steps:**
  1. Open the event page and scroll to Highlights, then Schedule, then Speakers.
  2. Compare the displayed order of items against the organizer-arranged order.
  3. Check the section headings for the schedule and speakers.
- **Expected result:** Each list appears in the exact order the organizer arranged. The custom section titles ("กำหนดการ", "วิทยากรรับเชิญ") are used. A separate check on an event with these lists empty confirms each empty section is absent rather than shown blank. (AC: items in organizer order; custom titles used with sensible defaults otherwise; empty lists → section absent.)

---

## US-PAGE-05 — Ticket tiers, pricing and availability

### TC-PAGE-09 — Ticket tiers show VAT-inclusive Baht pricing, Free/RSVP labels and the recommended badge
- **Traces:** US-PAGE-05  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** A published event with multiple tiers: a paid tier, a free tier, an RSVP tier, and one tier flagged as recommended. All prices are configured VAT-inclusive.
- **Test data:** Tiers — "General ฿500" (incl. 7% VAT), "VIP ฿1,500" (marked recommended, badge "Most popular", includes ["Front seat","Swag bag"]), "Student — Free", "Waitlist — RSVP".
- **Steps:**
  1. Open the event page and scroll to the Tickets section.
  2. For each tier, read its name, price, "what's included" list and Register button.
  3. Inspect the free and RSVP tiers for any price prefix.
  4. Inspect the recommended tier for its badge/highlight.
- **Expected result:** Each tier shows name, price, any "what's included" list and its own Register button. Every Baht price is presented as already including 7% VAT (no separate VAT add-on at display). The free tier reads "Free" and the RSVP tier reads "RSVP" with no price prefix. The recommended tier stands out visually and shows its "Most popular" badge. (AC: per-tier fields; VAT-inclusive Baht; Free/RSVP labels; recommended badge stands out.)

### TC-PAGE-10 — Sold-out tier shows "Sold out" and cannot be clicked through
- **Traces:** US-PAGE-05  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** A published event whose registration is open, with one tier at zero remaining seats and another still available.
- **Test data:** "General ฿500" available; "VIP ฿1,500" sold out (0 seats).
- **Steps:**
  1. Open the event page and scroll to the sold-out VIP tier.
  2. Read the VIP tier's button label.
  3. Attempt to click through the VIP button to registration.
- **Expected result:** The sold-out tier's button reads "Sold out" and is non-actionable — it cannot be clicked through to registration. The still-available tier's Register button continues to work. (AC: sold-out tier button reads "Sold out" and cannot proceed.)

### TC-PAGE-11 — Registration closed disables all tier buttons; low-seat urgency line shows
- **Traces:** US-PAGE-05  ·  **Priority:** High  ·  **Type:** Edge
- **Preconditions:** Two scenarios available: (A) an event whose registration window has closed; (B) a still-open event with a tier at very low remaining seats.
- **Test data:** (A) Event "Closed Summit" — registration closed. (B) Event "Almost Full" — tier "General ฿500" with 3 seats left.
- **Steps:**
  1. Open event (A) and inspect every tier's Register button and any page-level message.
  2. Open event (B) and inspect the low-seat tier for an urgency line.
- **Expected result:** For (A), all tier buttons are disabled and a "registration is closed" message is shown. For (B), an urgency line such as "Going fast — only 3 left" is displayed on the low-seat tier. (AC: registration closed → all buttons disabled + closed message; very few seats → urgency line.)

---

## US-PAGE-06 — Frequently asked questions

### TC-PAGE-12 — FAQ opens the first answer, collapses the rest, and is expandable; absent when empty
- **Traces:** US-PAGE-06  ·  **Priority:** Low  ·  **Type:** Functional
- **Preconditions:** Two events: (A) published with 4 FAQ entries; (B) published with no FAQs.
- **Test data:** (A) FAQs Q1–Q4; (B) no FAQ content.
- **Steps:**
  1. Open event (A); locate the FAQ section on load.
  2. Confirm the first answer is expanded and Q2–Q4 are collapsed.
  3. Click Q3 to expand it, and confirm it opens.
  4. Open event (B) and look for a FAQ section.
- **Expected result:** On (A), the first FAQ answer is open on load and the rest are collapsed; any FAQ can be expanded on click. On (B), the FAQ section is entirely absent. (AC: first answer open, rest collapsed and expandable; no FAQs → section absent.)

---

## US-PAGE-07 — Add the event to my calendar

### TC-PAGE-13 — Add to calendar creates a Bangkok-time entry via Apple/Google/Outlook
- **Traces:** US-PAGE-07  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** A published in-person event with a clear start and end time, opened on a mobile viewport.
- **Test data:** Event "Bangkok Tech Meetup", 15 Aug 2026 18:00–21:00 (Asia/Bangkok), Venue "True Digital Park".
- **Steps:**
  1. Open the event page on a phone and tap "Add to calendar".
  2. Choose a provider (Apple, Google or Outlook).
  3. Inspect the generated calendar entry's title, location, start/end and timezone.
- **Expected result:** Apple, Google and Outlook options are offered. The resulting entry is set to Bangkok time (18:00–21:00, +07:00) with the correct event title and location. (AC: clear start/end → provider choice and a Bangkok-time entry with title and location.)

### TC-PAGE-14 — Calendar entry for online events carries no join link; option hidden when time is unclear; re-add updates not duplicates
- **Traces:** US-PAGE-07  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** (A) A published online event with a private join link and clear times; (B) a published event with a missing/unclear time; the tester already added event (A) once.
- **Test data:** (A) "Remote UX Workshop" online, join link private; (B) "TBD Networking Night" with no set time.
- **Steps:**
  1. On (A), tap Add to calendar, generate the entry and inspect it for the private join link.
  2. Add event (A) to the same calendar a second time and inspect the calendar.
  3. Open (B) and check whether the Add to calendar option is present.
- **Expected result:** The online event's calendar entry contains no private join link. Re-adding the same event updates the existing entry rather than creating a duplicate. For (B), the Add to calendar option is hidden rather than producing a broken entry. (AC: online entry has no join link; missing/unclear time → option hidden; re-add updates existing entry.)

---

## US-PAGE-08 — Rich search results and social sharing

### TC-PAGE-15 — Shared/searched link renders a rich card; share buttons copy a clean public link
- **Traces:** US-PAGE-08  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** A published event with title, description and cover image.
- **Test data:** Event "Bangkok Tech Meetup", ticket from ฿500, 15 Aug 2026 18:00 (Asia/Bangkok).
- **Steps:**
  1. Inspect the page's share preview metadata (as a social scraper / link unfurl would consume).
  2. Use the on-page share controls to copy the link and to share to Facebook, LINE and X.
  3. Examine the copied/shared URL for any personal or tracking data.
- **Expected result:** The rich card shows the correct title, description, image (or a branded default) and event details including Baht pricing and Bangkok time. Share controls allow copy-link and sharing to Facebook, LINE and X. The shared link is the clean public address with no personal data attached. (AC: rich card correct incl. ฿ and Bangkok time; share to channels; clean public link with no personal data.)

### TC-PAGE-16 — Preview and unpublished pages are marked "do not index"
- **Traces:** US-PAGE-08  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** A preview page and an unpublished event page are reachable by URL.
- **Test data:** A template preview URL and an unpublished event URL.
- **Steps:**
  1. Reach the preview page as a scraper/search engine would and inspect its indexing directives.
  2. Repeat for the unpublished event page.
- **Expected result:** Both the preview and the unpublished page are marked "do not index" (noindex), so neither can leak into public search results. (AC: preview/unpublished pages marked do-not-index.)

---

## US-PAGE-09 — Preview any template with unsaved content

### TC-PAGE-17 — Preview renders live unsaved values, saves nothing, and is marked as a preview
- **Traces:** US-PAGE-09  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** An organizer is signed in and editing a new, unsaved event.
- **Test data:** Typed title "Draft Launch Party", one highlight "Free welcome drink", event type set to "Online". Nothing saved yet.
- **Steps:**
  1. In the event editor, type the title and highlight and choose "Online" without saving.
  2. Click Preview.
  3. In the new preview tab, confirm the typed values appear and that a preview marker is shown.
  4. Return and confirm no event/page/draft was created; verify the preview is excluded from public visitor analytics and public search.
- **Expected result:** A new tab shows a live page with exactly the typed values ("Draft Launch Party", the highlight, Online handling). Nothing is saved — no event, page or draft is created — and the page is clearly marked as a preview. It is not counted in public visitor analytics and does not appear in public search. (AC: preview shows live unsaved values; nothing saved; marked preview; excluded from public analytics and search.)

### TC-PAGE-18 — Blocked pop-up on Preview shows an allow-pop-ups hint
- **Traces:** US-PAGE-09  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Organizer signed in and editing an event; browser configured to block pop-ups/new tabs.
- **Test data:** Any valid in-progress event; pop-up blocker enabled.
- **Steps:**
  1. With pop-ups blocked, click Preview.
  2. Observe the editor when no new tab opens.
- **Expected result:** Because the new tab is blocked, a clear hint is shown telling the organizer to allow pop-ups. No silent failure. (AC: browser blocks the new tab → clear hint to allow pop-ups.)

---

## US-PAGE-10 — Choose a design, brand it, and publish under a stable link

### TC-PAGE-19 — Choose a design, publish under a stable address, and switch designs with no content loss
- **Traces:** US-PAGE-10  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Organizer signed in with a complete event (title, date, at least one ticket/RSVP option) ready to publish.
- **Test data:** Event "Bangkok Tech Meetup"; designs Classic, Spotlight, Minimal, Vibrant; public address `/e/bkk-tech-meetup`.
- **Steps:**
  1. Choose the "Spotlight" design and publish.
  2. Open the public address and confirm content renders in Spotlight under the stable address.
  3. Return to the editor, switch the design to "Vibrant" and save.
  4. Reload the public page at the same address.
- **Expected result:** The event content renders in the chosen design at a stable public address. Switching from Spotlight to Vibrant re-renders the same content in the new design with no loss of content and the same public address. (AC: choose one of four designs, publish under a stable address, switch later with no content loss.)

### TC-PAGE-20 — Publishing is blocked when title, date, or a ticket/RSVP option is missing
- **Traces:** US-PAGE-10  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Organizer signed in editing an event missing one or more required fields.
- **Test data:** Case (a) no title; Case (b) no date; Case (c) no ticket/RSVP option configured.
- **Steps:**
  1. For each case, attempt to publish the event.
  2. Read the resulting message.
- **Expected result:** In each case publishing is blocked, and the message states exactly what to add first (the missing title, date, or at least one ticket/RSVP option). The event does not go live. (AC: publish blocked when missing title/date/ticket-or-RSVP with a message naming what to add.)

### TC-PAGE-21 — Duplicate public address rejected; invalid accent falls back; unpublish kills the link but preserves the page
- **Traces:** US-PAGE-10  ·  **Priority:** High  ·  **Type:** Edge
- **Preconditions:** A published event already owns the address `/e/bkk-tech-meetup`; organizer is editing a second event and a published event whose accent colour can be changed.
- **Test data:** Duplicate slug `bkk-tech-meetup`; accent colours: valid `#E91E63`, invalid `not-a-color`; brand default accent.
- **Steps:**
  1. On the second event, set the public web address to the already-taken slug and save.
  2. On a published event, set a valid accent colour and confirm buttons/highlights take that colour; then set an invalid colour value and save.
  3. Unpublish a published event, then open its public link and confirm search visibility.
  4. Re-publish the same event and confirm its original address still works.
- **Expected result:** Saving a taken address is rejected with a message that the address is in use, asking for another. A valid accent colour applies to buttons and highlights; an invalid colour quietly falls back to the brand default (no error thrown). After unpublish, the public link stops working immediately and drops out of search, while the page is preserved and can be re-published under the same address. (AC: duplicate address rejected; invalid accent → brand default fallback; unpublish disables link and de-indexes yet preserves the page for re-publish.)


---

<a id="tc-e05"></a>

# Sell Tickets & Run Promotions — Test Cases

Epic E5 — TKT. Test basis: user stories US-TKT-01 … US-TKT-12 and their acceptance criteria.
Locale: Thai market — ฿ (whole baht), 7% VAT recalculated at checkout, PromptPay, Asia/Bangkok time, EN/TH, 8-seat booking cap.

---

## US-TKT-01 — Create paid and free ticket types

### TC-TKT-01 — Create a paid ticket type with a valid sales window
- **Traces:** US-TKT-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Event Organizer; an event exists that is not cancelled.
- **Test data:** Name "General Admission"; Price ฿1,000; Quantity 500; Sales start 2026-08-01 00:00; Sales end 2026-08-31 23:59 (Asia/Bangkok); Per-order limit 4; Description "Standard entry".
- **Steps:**
  1. Open the event's Tickets setup and choose Add ticket type.
  2. Enter the name, price, quantity, sales window, per-order limit and description above.
  3. Save.
- **Expected result:** Ticket is saved and appears in the ticket list ready to sell. Because the sales window is open now (or opens on schedule) and quantity is available, status shows On sale (or Scheduled if start is in the future). Price displays as ฿1,000; attendee-facing checkout will add 7% VAT.

### TC-TKT-02 — Create a Free ticket type (no price, no payment)
- **Traces:** US-TKT-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Event Organizer; non-cancelled event exists.
- **Test data:** Name "Free Community Pass"; Type = Free; Quantity 200; Sales window open now.
- **Steps:**
  1. Add a ticket type and mark it Free.
  2. Confirm no price field is required.
  3. Save.
- **Expected result:** Ticket saves without asking for or charging a price. Attendees can confirm this ticket with no payment step and no VAT applied.

### TC-TKT-03 — Reject invalid ticket setup (end-before-start, over-cap and duplicate-name)
- **Traces:** US-TKT-01  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Signed in as Event Organizer; a ticket named "General Admission" already exists on this event; quantity available for the new ticket = 300.
- **Test data:** (a) Sales end 2026-08-01 before start 2026-08-10; (b) Per-order limit 9 (exceeds 8-seat cap) and/or per-order limit > quantity available; (c) Name reused "General Admission".
- **Steps:**
  1. Attempt to save a ticket with sales end date before start date.
  2. Attempt to save a ticket with per-order limit above the quantity available and above the 8-seat cap.
  3. Attempt to save a ticket reusing an existing name on the same event.
- **Expected result:** Each attempt is blocked with a plain-language error and the ticket is not created: (a) end-before-start error; (b) per-order limit error (capped at 8 seats / not above available quantity); (c) "a ticket with that name already exists". Organizer can correct and re-save.

---

## US-TKT-02 — Adjust a ticket type safely after it is live

### TC-TKT-04 — Increase capacity on a sold-out ticket returns it to On sale
- **Traces:** US-TKT-02  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** A paid ticket type has sold all seats and shows Sold out.
- **Test data:** Sold 500 / 500; new quantity 700.
- **Steps:**
  1. Open the sold-out ticket type for editing.
  2. Raise quantity available from 500 to 700.
  3. Save.
- **Expected result:** The 200 extra seats go on sale immediately and status returns to On sale. (If a waitlist exists, waitlisted attendees are offered the freed seats.)

### TC-TKT-05 — Cannot lower quantity below seats already sold
- **Traces:** US-TKT-02  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** A paid ticket type has sold 640 seats.
- **Test data:** Sold 640; attempt new quantity 500.
- **Steps:**
  1. Edit the ticket and set quantity to 500.
  2. Save.
- **Expected result:** Change is refused with a message that quantity cannot go below the seats already sold (640).

### TC-TKT-06 — Price / paid-free / event locked after sales begin; concurrent-edit guard
- **Traces:** US-TKT-02  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** A ticket type has sold at least one seat; a second organizer session has the same ticket open.
- **Test data:** Attempt to change price ฿1,000 → ฿1,200, and toggle Paid → Free; separately, colleague saves an edit to the same ticket first.
- **Steps:**
  1. With seats sold, attempt to change the price, the paid/free setting, or the event.
  2. Save.
  3. Separately, while both sessions have the ticket open, let the colleague save first, then attempt to save your changes.
- **Expected result:** Price/paid-free/event changes are blocked with a message that they can't change after sales began, advising to create a new ticket type instead. The description and transferable setting remain editable. On the concurrent edit, the second save is told the ticket changed since it was opened and prompted to reload — no silent overwrite. (Note: if no seats had sold, price and paid/free changes would be accepted.)

---

## US-TKT-03 — Availability that manages itself

### TC-TKT-07 — Scheduled ticket auto-opens on its start date (Bangkok time)
- **Traces:** US-TKT-03  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** A ticket type is Scheduled with a future sales start date.
- **Test data:** Sales start 2026-08-01 00:00 Asia/Bangkok; quantity available > 0.
- **Steps:**
  1. Before 2026-08-01 (Bangkok), confirm the ticket shows Scheduled and is not purchasable.
  2. At the start of 2026-08-01 Bangkok time, re-check the ticket.
- **Expected result:** When the day begins in Bangkok time, the ticket automatically becomes On sale and is purchasable.

### TC-TKT-08 — Last-seat concurrency: exactly one buyer wins, no oversell
- **Traces:** US-TKT-03  ·  **Priority:** High  ·  **Type:** Edge
- **Preconditions:** A ticket type has exactly one seat left.
- **Test data:** Two attendees attempt to buy the last seat simultaneously.
- **Steps:**
  1. Have two attendees reach checkout for the final seat at the same moment.
  2. Both submit the purchase.
- **Expected result:** Exactly one purchase succeeds; the other is refused. The ticket immediately shows Sold out to everyone else. No oversell occurs.

### TC-TKT-09 — Purchase refused outside the sales window and while paused
- **Traces:** US-TKT-03  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** (a) A ticket whose sales end date has passed; (b) a ticket inside its sales window that the organizer has Paused; (c) a paused ticket whose window has already ended.
- **Test data:** Sales end 2026-07-26 (yesterday relative to 2026-07-27); a paused On-sale ticket.
- **Steps:**
  1. As an attendee, try to buy the ticket whose sales end date has passed (even if the badge hasn't visibly refreshed).
  2. As organizer, Pause an in-window ticket; as attendee, try to buy it.
  3. As organizer, try to Resume a ticket whose sales window has already ended.
- **Expected result:** (1) Purchase refused — no sales outside the window. (2) Paused ticket is not purchasable until Resumed. (3) Resume is blocked with a message to extend the sales end date first.

---

## US-TKT-04 — See all my tickets and inventory at a glance

### TC-TKT-10 — Filter by Sold out and search by name/event
- **Traces:** US-TKT-04  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Organizer has ticket types across mixed states (On sale, Scheduled, Paused, Sold out); at least one contains "Jazz" in its name/event.
- **Test data:** Filter "Sold out"; search term "jazz".
- **Steps:**
  1. Apply the "Sold out" filter.
  2. Confirm the tab count.
  3. Clear the filter and type "jazz" in search.
- **Expected result:** The Sold out filter shows only sold-out tickets and the tab count matches. Search returns only tickets whose name or event contains "jazz" (case-insensitive). Each card shows sales progress (sold out of total) and current status. Sold/total counts are read-only.

### TC-TKT-11 — No-match empty state
- **Traces:** US-TKT-04  ·  **Priority:** Low  ·  **Type:** Edge
- **Preconditions:** Organizer has at least one ticket, none matching the query.
- **Test data:** Search "zzzz-nonexistent".
- **Steps:**
  1. Enter a search term that matches no ticket.
- **Expected result:** A clear "No matches" empty state is shown after the list refreshes.

---

## US-TKT-05 — Retire a ticket type without harming existing holders

### TC-TKT-12 — Delete an unsold ticket vs. retire a sold ticket
- **Traces:** US-TKT-05  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Two ticket types: one that has never sold; one that has sold 210 seats.
- **Test data:** Unsold ticket "Test Ticket"; sold ticket "VIP" with 210 sold.
- **Steps:**
  1. Delete the never-sold ticket and confirm.
  2. Delete/retire the ticket that has sold 210 seats and confirm.
- **Expected result:** The never-sold ticket is removed completely. The sold ticket is retired instead of removed — it disappears from the active list and can't be bought anymore, but the 210 existing holders keep valid tickets. No attendee is refunded or cancelled automatically.

### TC-TKT-13 — Delete blocked while checkouts are in progress
- **Traces:** US-TKT-05  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** A ticket type has one or more checkouts in progress.
- **Test data:** Ticket with an active in-flight checkout.
- **Steps:**
  1. Attempt to delete the ticket while a checkout is in progress.
- **Expected result:** Deletion is blocked with a message to pause it and try again shortly.

---

## US-TKT-06 — Share a ticket's registration link and QR

### TC-TKT-14 — Share view gives a working link and QR, copy and download
- **Traces:** US-TKT-06  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** The ticket belongs to a published event.
- **Test data:** Ticket "General Admission" on a published event.
- **Steps:**
  1. Open the ticket's share view.
  2. Choose Copy link.
  3. Choose Download QR.
  4. Open the copied link / scan the QR.
- **Expected result:** Both the link and the matching QR open the sign-up page with that ticket already selected. Copy link copies the URL and shows a brief "Copied" confirmation. Download QR produces a printable QR image. (Link opens registration for the ticket type; it is not a personal entry pass.)

### TC-TKT-15 — Sharing blocked for an unpublished event
- **Traces:** US-TKT-06  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** The event is not yet published.
- **Test data:** Ticket on a draft/unpublished event.
- **Steps:**
  1. Attempt to open/share the ticket's link or QR.
- **Expected result:** The organizer is prompted to publish the event first so the link actually works; no shareable live link is produced.

---

## US-TKT-07 — Create a discount code with a quick generator

### TC-TKT-16 — Generate a unique code and create a percentage promotion
- **Traces:** US-TKT-07  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin; a specific event exists.
- **Test data:** Generated code e.g. "PROMO42"; Type 25% off; Scope = specific event; Usage limit 500; Per-person limit 1; Validity start today, end 2026-08-31; Minimum order ฿500.
- **Steps:**
  1. Click Generate and note the proposed code.
  2. Click Generate again to confirm a different, unused code is proposed.
  3. Configure 25% off scoped to the event with the limits and dates above and Save.
- **Expected result:** Generator proposes a memorable, unused code and never repeats an existing one. The saved code appears in the list with 0 redemptions used and status Active (start today) — or Scheduled if the start were in the future. Code applies only to paid tickets, reducing price before 7% VAT is recalculated.

### TC-TKT-17 — Reject invalid or already-expired code setups
- **Traces:** US-TKT-07  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Signed in as Admin.
- **Test data:** (a) Per-person limit 600 with total usage limit 500; (b) End date before start date; (c) Validity window entirely in the past (e.g., 2026-07-01 to 2026-07-10).
- **Steps:**
  1. Try to save with per-person limit higher than total usage limit.
  2. Try to save with end date before start date.
  3. Try to save a code whose validity window has already ended.
- **Expected result:** (a) and (b) are blocked with plain-language errors to fix; (c) is refused as already expired. No invalid code is created.

---

## US-TKT-08 — Adjust a discount code without breaking past redemptions

### TC-TKT-18 — Value/text locked after use; unused code fully editable; usage limit floor
- **Traces:** US-TKT-08  ·  **Priority:** Low  ·  **Type:** Negative
- **Preconditions:** One code redeemed 342 times; one code never redeemed.
- **Test data:** Redeemed code: try usage limit 342 → 300, try change code text and value; unused code: change 25% → 30%.
- **Steps:**
  1. On the redeemed code, try to lower usage limit to 300 (below 342 redemptions).
  2. On the redeemed code, try to change the code text and its discount value.
  3. On the never-redeemed code, change the discount from 25% to 30% and save.
  4. Narrow the redeemed code's scope / shorten its window and save.
- **Expected result:** (1) Refused — usage limit can't drop below redemptions already made. (2) Refused — code text and value can't change after use; advised to create a new code. (3) Accepted for the unused code. (4) Accepted, and attendees who already redeemed are unaffected.

---

## US-TKT-09 — Stop a promotion safely

### TC-TKT-19 — Switch off, delete-with-history, delete-unused, and in-use guard
- **Traces:** US-TKT-09  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** An active code; a code redeemed 89 times; a never-redeemed code; a code currently in use in an active checkout.
- **Test data:** As above.
- **Steps:**
  1. Switch off the active code, then attempt to redeem it at checkout.
  2. Delete the code redeemed 89 times.
  3. Delete the never-redeemed code.
  4. Attempt to delete the code that is in use in an active checkout.
- **Expected result:** (1) Switched-off code is immediately refused at checkout; its redemption history is preserved. (2) The 89-redemption code is retired, not erased — prior orders keep their discount and it no longer works at checkout. (3) The never-redeemed code is removed completely. (4) Deleting the in-use code is blocked with a message to switch it off and try again shortly.

---

## US-TKT-10 — Codes go live and expire on schedule

### TC-TKT-20 — Auto activate / auto expire on date and on usage exhaustion
- **Traces:** US-TKT-10  ·  **Priority:** Low  ·  **Type:** Functional
- **Preconditions:** A Scheduled code with a future start; an Active code near its usage limit; an Active code near its end date; an already-expired code.
- **Test data:** Scheduled code start today at 00:00 Bangkok; code with usage limit reached on final redemption; code passing its end date.
- **Steps:**
  1. When the Scheduled code's start day begins (Bangkok), check its status and usability at checkout.
  2. Complete the final redemption that reaches the usage limit.
  3. Let an active code pass its end date.
  4. Attempt to switch an already-expired code back on.
- **Expected result:** (1) The Scheduled code automatically becomes Active and usable at checkout. (2) On the final redemption it automatically becomes Expired. (3) Past its end date it automatically becomes Expired and no longer applies. (4) Reviving an expired code is blocked with a message to update its dates or usage limit first.

---

## US-TKT-11 — Apply a discount code at checkout

### TC-TKT-21 — Percentage discount applied with VAT recalculated on reduced amount
- **Traces:** US-TKT-11  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Attendee (guest or registered) at checkout with a valid, active code for this event/paid ticket.
- **Test data:** 25%-off code; order subtotal ฿1,000.
- **Steps:**
  1. Enter the code and apply it.
- **Expected result:** ฿250 is taken off; the discounted total shows 7% VAT recalculated on the reduced ฿750 (i.e., ฿750 + ฿52.50 VAT = ฿802.50); the code is marked as applied. Only one code applies per order.

### TC-TKT-22 — Fixed discount capped so total never goes below zero
- **Traces:** US-TKT-11  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Attendee at checkout on a paid order.
- **Test data:** Fixed ฿300-off code; order subtotal ฿250.
- **Steps:**
  1. Apply the ฿300-off code to the ฿250 order.
- **Expected result:** The discount is capped at ฿250; the pre-VAT total is ฿0 and never goes below zero (VAT recalculated accordingly). The order does not become negative.

### TC-TKT-23 — Invalid, limit-reached and below-minimum codes give clear reasons
- **Traces:** US-TKT-11  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Attendee at checkout. Codes prepared: unknown; not-yet-started; expired; switched-off; scoped to a different event; a single-use code the attendee already used; a code at its overall redemption limit; a code with a ฿1,000 minimum against a ฿600 order.
- **Test data:** As above; free ticket in cart for the discountability check.
- **Steps:**
  1. Apply each of the unknown / not-started / expired / switched-off / wrong-event codes.
  2. Re-apply a single-use code already used by this attendee; then apply a code that has hit its overall limit.
  3. Apply a code whose minimum order (฿1,000) exceeds the ฿600 order.
  4. Attempt to apply any code to a free ticket only.
- **Expected result:** (1) Each gives a clear, specific reason it can't be used. (2) The already-used single-use code is refused ("you've already used it"); the limit-hit code is refused ("reached its redemption limit"). (3) The below-minimum code tells the attendee how much more to add to qualify. (4) Codes never apply to free tickets. (Note: a redemption released on cancel/refund frees the code for reuse.)

---

## US-TKT-12 — Track which promotions are working

### TC-TKT-24 — Filter, search, all-events scope visibility, and copy code
- **Traces:** US-TKT-12  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Organizer has codes in mixed statuses, including at least one scoped "All events".
- **Test data:** Filter by a specific event; filter by status Active/Scheduled/Expired/Disabled; search by code/event; copy a code.
- **Steps:**
  1. Filter the code list by a specific event.
  2. Filter by status and search by code or event text.
  3. View a code's row for its redemption count and status.
  4. Click copy on a code.
- **Expected result:** An "All events" code still shows when filtering by a specific event (it applies everywhere). Status and search filters narrow the table accordingly. Each row shows redemptions used out of its limit (live, read-only) and current status. Copy places the code on the clipboard with a brief confirmation.


---

<a id="tc-e06"></a>

# Discover & Register for Events — Test Cases

Area: DISC · Epic E6 — Discover & Register for Events (Attendee)
Locale defaults: currency ฿ (THB), VAT 7%, PromptPay, Asia/Bangkok, languages EN/TH.

---

## US-DISC-01 — Discover and browse what's on

### TC-DISC-01 — Discover grid shows upcoming events with key details, soonest-first
- **Traces:** US-DISC-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** At least 3 published, publicly visible upcoming events exist in different categories; one event has attendee ratings from US-DISC-13; attendee is a guest (not signed in).
- **Test data:** Event A "Bangkok Jazz Night", category Music, date 2026-08-15, venue GMM Live House / Bangkok, organizer "Live Nation TH", price-from ฿890, 120 going, avg rating 4.6; Event B a free workshop (price "Free"); Event C paid, no reviews yet.
- **Steps:**
  1. Open the Discover page as a guest.
  2. Inspect each event card in the grid.
  3. Note the order of the cards.
  4. Tap the Event A card.
- **Expected result:** A grid of event cards renders. Each card shows cover image, category, title, date, venue/city, organizer, count of people going, a price-from in ฿ (Event B shows "Free"), and an average rating only where feedback exists (Event A shows 4.6; Event C shows no rating). Cards are ordered soonest-first. Tapping Event A opens its public event page with a register/get-tickets action.

### TC-DISC-02 — Sold-out, selling-fast, expired, and empty-state handling
- **Traces:** US-DISC-01  ·  **Priority:** High  ·  **Type:** Edge
- **Preconditions:** One event is sold out; one event is nearly sold out (few seats remaining); one event already started/finished (start time in the past, Asia/Bangkok); a search term exists that matches no event.
- **Test data:** Sold-out event "Coldplay BKK"; nearly-sold-out event "Indie Film Fest" (e.g. 6 of 400 seats left); past event "NYE Countdown 2026" (started 2026-07-26 20:00 +07); no-match keyword "zzzxxq".
- **Steps:**
  1. Open the Discover page and locate the sold-out event card.
  2. Locate the nearly-sold-out event card.
  3. Look for the already-started/finished event in the list.
  4. Search for "zzzxxq".
- **Expected result:** The sold-out card shows a "Waitlist" badge instead of a buy-now price; the nearly-sold-out card shows a "Selling fast" badge; the started/finished event is not shown anywhere in the list; the no-match search shows a friendly empty state suggesting a different search or category.

---

## US-DISC-02 — Search and filter events

### TC-DISC-03 — Keyword and category filters combine (AND) and update the count
- **Traces:** US-DISC-02  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Published events span multiple categories; at least one Music event in Bangkok whose title contains "Jazz".
- **Test data:** Keyword "Jazz"; category "Music"; a competing "Jazz Cooking Class" event in category Food.
- **Steps:**
  1. Open the Discover page.
  2. Type "Jazz" in the search box and observe the list and result count.
  3. Select category "Music".
  4. Select "All events" to clear the category.
- **Expected result:** Typing "Jazz" narrows the list to events whose title, category, city, or venue matches and updates the result count. Adding category "Music" shows only events matching both keyword AND category (the Food "Jazz Cooking Class" is excluded). Selecting "All events" clears the category filter and the keyword-only results return.

### TC-DISC-04 — Bilingual search, trimmed whitespace, and no-match empty state
- **Traces:** US-DISC-02  ·  **Priority:** High  ·  **Type:** Edge
- **Preconditions:** An event exists with a Thai title/venue and an English equivalent (e.g. venue "ไอคอนสยาม" / "IconSiam").
- **Test data:** Thai keyword "ไอคอนสยาม"; English keyword "IconSiam"; padded keyword "  Jazz  "; combination keyword "Jazz" + category "Food" that matches nothing.
- **Steps:**
  1. Search "ไอคอนสยาม" (Thai) and note results.
  2. Clear and search "IconSiam" (English).
  3. Search "  Jazz  " with leading/trailing spaces.
  4. Set keyword "Jazz" and category "Food" (a combination with no matching event).
- **Expected result:** Thai and English searches both return the same matching event consistently (tone marks/accents do not break matching). The padded "  Jazz  " returns the same results as "Jazz" (stray spaces ignored). The keyword+category combination that matches nothing shows the friendly empty state.

---

## US-DISC-03 — Save events for later

### TC-DISC-05 — Save persists and guest saves merge into account on sign-in
- **Traces:** US-DISC-03  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** A registered attendee account exists; several published events available; attendee starts as a guest.
- **Test data:** Guest saves Event A and Event B; account already has Event B saved from a prior session (potential duplicate).
- **Steps:**
  1. As a guest, tap the save (heart) control on Event A and Event B.
  2. Reload the Discover page and confirm both remain saved.
  3. Sign in to the registered account (which already has Event B saved).
  4. Sign in on a second device/browser to the same account and open saved events.
- **Expected result:** Tapped events show as saved and stay saved after reload. On sign-in, in-session guest saves merge into the account with no duplicate for Event B. The saved events appear on the second device (saves are account-bound across devices).

### TC-DISC-06 — Save failure reverts the control with a message
- **Traces:** US-DISC-03  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Ability to simulate a save that cannot be recorded (e.g. network/service error on the save action).
- **Test data:** Event C; save action forced to fail.
- **Steps:**
  1. Open the Discover page.
  2. Tap the save (heart) control on Event C under the failing condition.
  3. Observe the control state and any message.
- **Expected result:** The save is not recorded, the heart control reverts to its unsaved state, and a brief "couldn't save right now" message is shown. No phantom saved state persists on reload.

---

## US-DISC-04 — Register and choose my tickets (guest or signed in)

### TC-DISC-07 — Reserved-seating checkout with live totals, seat hold, and guest details
- **Traces:** US-DISC-04  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** A paid event with reserved seating and an available seat map; attendee is a guest.
- **Test data:** Event "Bangkok Jazz Night"; ticket type "General" ฿890; service fee ฿50/seat; select 2 seats (A12, A13); guest name "Somchai P.", email somchai@example.com, mobile 08x-xxx-xxxx.
- **Steps:**
  1. Click "Register"/"Get tickets" on the event.
  2. Choose exactly one ticket type (General ฿890) and confirm prices show in ฿.
  3. On the seat map, select seats A12 and A13; observe held state and that taken seats are not selectable.
  4. Watch the order summary and the sticky action bar as selection changes.
  5. Enter guest name, email, and mobile number; place the order without opting into an account.
- **Expected result:** Checkout shows the event summary and a single-choice ticket type in ฿. Chosen seats are held for the attendee during checkout; already-taken seats cannot be selected. The order summary and sticky action bar update live with subtotal (฿1,780), service fee (฿100), and total (฿1,880). Guest fields are captured, and after placing the order no account is silently created for the guest.

### TC-DISC-08 — Quantity boundaries (1–8) and free-event fee/payment skip
- **Traces:** US-DISC-04  ·  **Priority:** High  ·  **Type:** Edge
- **Preconditions:** A general-admission or online event for quantity selection; a separate free event.
- **Test data:** GA/online event ฿300; free event with two tiers both ฿0.
- **Steps:**
  1. Open checkout for the GA/online event and attempt quantity 0, then 1, then 8, then 9.
  2. For an online event, note the join-link message; for GA, note the first-come seating note.
  3. Open checkout for the free event and select a tier.
  4. Proceed toward confirmation on the free event.
- **Expected result:** Quantity is constrained to 1–8: 0 and 9 are rejected/clamped, 1 and 8 are accepted. Online events state a join link will be emailed; GA events note seating is first-come. For the free event all tiers show "Free", no service fee is added, and the payment step is skipped entirely.

### TC-DISC-09 — Seat hold expiry / seat taken before confirm blocks continue without charge
- **Traces:** US-DISC-04  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Reserved-seating event; a way to let the seat hold expire or have a chosen seat taken by another buyer before confirmation.
- **Test data:** Seat A14 selected; hold timer allowed to expire (or A14 taken concurrently) before payment is confirmed.
- **Steps:**
  1. Begin checkout and select seat A14.
  2. Wait until the seat hold expires (or have another buyer take A14) before confirming.
  3. Attempt to continue/confirm.
- **Expected result:** The attendee is told the seat/hold is no longer available and is asked to re-select. No charge is made for the lapsed selection, and the released seat becomes available again.

---

## US-DISC-05 — Pay by card or PromptPay

### TC-DISC-10 — Card payment approved completes registration
- **Traces:** US-DISC-05  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** A paid order is at the payment step with a valid total in THB.
- **Test data:** Order total ฿1,880; test card that approves; method = Card.
- **Steps:**
  1. At payment, choose Card.
  2. Enter the approving test card in the secure payment provider fields and submit.
  3. Observe the result.
- **Expected result:** Attendee can choose card or PromptPay. On approval the flow moves to confirmation and the registration is completed. The charge settles in THB. (Card numbers are entered into the provider, not stored by Eventa.)

### TC-DISC-11 — Card declined shows message, issues no ticket, allows retry/switch
- **Traces:** US-DISC-05  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Paid order at payment step.
- **Test data:** Order total ฿1,880; test card that declines; a second valid card; PromptPay as fallback.
- **Steps:**
  1. Choose Card and submit the declining test card.
  2. Read the decline message and confirm no ticket was issued.
  3. Retry with the valid card, or switch to PromptPay.
- **Expected result:** A clear decline message is shown, no ticket is issued, and the attendee can try another card or switch to PromptPay. Retrying with a valid method completes the registration. The attendee is never charged for the declined attempt.

### TC-DISC-12 — PromptPay QR for exact total, single issuance, and expiry releases seats
- **Traces:** US-DISC-05  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Paid reserved-seating order at payment step.
- **Test data:** Order total ฿1,880; held seats A12/A13; method = PromptPay with time-limited QR.
- **Steps:**
  1. Choose PromptPay and continue.
  2. Verify the QR encodes the exact total ฿1,880 and shows a validity countdown.
  3. Scenario A: complete the PromptPay payment in-app and let it settle.
  4. Scenario B (fresh order): let the QR expire before paying, then generate a new code.
- **Expected result:** A PromptPay QR for the exact total is shown, valid for a limited time. On settlement (A) the registration completes and exactly one ticket is issued (no double issuance on repeated webhook/confirmation). On expiry (B) no ticket is issued, the held seats are released, and a new code can be generated. The attendee is never charged twice for the same order.

---

## US-DISC-06 — Confirm and receive my QR ticket

### TC-DISC-13 — Confirmation recap plus email with QR, VAT receipt, and calendar invite
- **Traces:** US-DISC-06  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** A valid selection with approved payment (paid event) or a free registration; attendee provided a mobile number.
- **Test data:** Paid order, 2 seats → 2 QR tickets; email somchai@example.com; mobile provided; VAT 7% on ฿1,780 base.
- **Steps:**
  1. Confirm the registration.
  2. Review the on-screen recap.
  3. Check the confirmation email contents.
  4. Check for a confirmation SMS.
- **Expected result:** The registration is placed once; one QR ticket is issued per seat/ticket (2 tickets here); a "You're registered!" recap appears with links to view tickets. The confirmation email contains the QR ticket(s), an order summary, a VAT receipt (7% breakdown in ฿) for the paid order, and a calendar invite; an online event would also include a join link. Because a mobile number was provided, a confirmation SMS is also received.

### TC-DISC-14 — Idempotent confirm and concurrent-seat race yield one registration/charge
- **Traces:** US-DISC-06  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Ability to double-submit confirm and to have two attendees confirm the same seat simultaneously.
- **Test data:** Single order double-tapped; two attendees both selecting seat B5.
- **Steps:**
  1. On one order, double-tap/retry the confirm action.
  2. Verify how many registrations and charges result.
  3. Have two attendees confirm seat B5 at the same time.
  4. Observe both outcomes.
- **Expected result:** Double-tap/retry creates only one registration and one charge. In the seat race only the first attendee succeeds; the second is not charged (or is refunded) and is asked to pick again. No duplicate ticket and no double charge occur.

---

## US-DISC-07 — View and download my QR ticket

### TC-DISC-15 — Open and download a printable QR ticket with full details
- **Traces:** US-DISC-07  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed-in attendee owns a valid ticket.
- **Test data:** Ticket for "Bangkok Jazz Night", class VIP, seat A12/Row A/Gate 3, doors 18:00 (Asia/Bangkok), admission number and ticket reference present.
- **Steps:**
  1. Open the owned ticket.
  2. Confirm the QR and ticket reference are shown.
  3. Download the ticket.
  4. Inspect the downloaded image.
- **Expected result:** The ticket displays a scannable QR and its ticket reference. The download is a printable ticket image showing the QR, event name, date, venue, doors-open time, ticket class (VIP/General), seat/row/gate where applicable, and admission number.

### TC-DISC-16 — Refunded/voided ticket is invalid; access to a non-owned ticket is refused
- **Traces:** US-DISC-07  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** One ticket has been refunded/voided; a valid ticket belongs to a different attendee.
- **Test data:** Refunded ticket T-REF-001; another attendee's ticket T-OTHER-999.
- **Steps:**
  1. Open the refunded/voided ticket.
  2. Attempt to download it as a valid pass.
  3. Attempt to open ticket T-OTHER-999 (not owned).
- **Expected result:** The refunded/voided ticket shows as no longer valid and cannot be downloaded as a valid pass. Attempting to open a ticket the attendee does not own is refused (access denied).

---

## US-DISC-08 — Sign in to my attendee account

### TC-DISC-17 — Valid sign-in lands on My Events with attendee-only access and attaches guest saves
- **Traces:** US-DISC-08  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** A registered attendee account; the same browser session has guest-saved events.
- **Test data:** Valid email/password (and/or Google/Apple); guest-saved Event A and Event B.
- **Steps:**
  1. As a guest, save Event A and Event B.
  2. Sign in with valid email/password (or via Google/Apple).
  3. Land on the account and check accessible areas.
  4. Open saved events.
- **Expected result:** Sign-in succeeds and lands on My Events with attendee access only (no organizer/admin console reachable). The guest-session saves are attached to the account.

### TC-DISC-18 — Wrong credentials show a single generic error; repeated failures trigger lockout
- **Traces:** US-DISC-08  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** A registered account exists.
- **Test data:** Correct email with wrong password; repeated wrong attempts past the lockout threshold; "Forgot password?" link.
- **Steps:**
  1. Submit a valid email with an incorrect password.
  2. Read the error message.
  3. Repeat failed attempts until the limit is passed.
  4. Choose "Forgot password?".
- **Expected result:** A single "email or password is incorrect" message appears without revealing which field was wrong. After repeated failures the attempts are temporarily blocked and the attendee is pointed to reset the password. "Forgot password?" opens the reset flow.

---

## US-DISC-09 — See my upcoming and past tickets

### TC-DISC-19 — My Events splits upcoming/past with counts, countdown, attended badge, and empty state
- **Traces:** US-DISC-09  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed-in attendee with at least one upcoming and one past registration; also a fresh account with none.
- **Test data:** Upcoming event 6 days out and one today; past event attended with feedback still open; separate empty account.
- **Steps:**
  1. Open My Events on the account with registrations.
  2. Inspect the Upcoming section (counts, countdown, ticket type, date/venue, event link, Ticket action).
  3. Inspect the Past section (Attended badge, Leave feedback action).
  4. Sign in with the empty account and open My Events.
- **Expected result:** My Events shows an Upcoming and a Past section, each with a count, listing only this attendee's registrations. Upcoming items show a countdown ("6 days left", "Today"), ticket type, date and venue, an event-page link, and a "Ticket" action to open the QR. The attended past event shows an "Attended" badge and a "Leave feedback" action while feedback is open. The empty account shows an empty state inviting the attendee to discover events.

---

## US-DISC-10 — Review my payment history and receipts

### TC-DISC-20 — Payment history tiles, transaction detail, VAT receipt, export, and refund handling
- **Traces:** US-DISC-10  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed-in attendee with several transactions including one refunded; also a fresh account with none.
- **Test data:** Paid transaction ฿1,880 (card ending 4242), invoice INV-2026-0001; PromptPay transaction ฿300; refunded transaction ฿500 (struck through); VAT 7%.
- **Steps:**
  1. Open Payment history and read the summary tiles (total spent, number of transactions, total refunded).
  2. Open a transaction row and check event, invoice number, date, masked method, amount, status.
  3. Download the receipt for a paid transaction.
  4. Export the full history.
  5. Sign in with the empty account and open Payment history.
- **Expected result:** Tiles show total spent, transaction count, and total refunded. Each row shows event, invoice number, date, payment method (masked card or PromptPay), amount, and status (Paid/Refunded); the refunded amount is struck through and excluded from total spent. The downloaded receipt is a VAT receipt showing the 7% VAT breakdown in ฿. Export returns the full transaction list. The empty account shows zero tiles and an empty state.

---

## US-DISC-11 — Manage my profile

### TC-DISC-21 — Edit and save profile, cancel discards, and details pre-fill at checkout
- **Traces:** US-DISC-11  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed-in attendee on the Profile tab.
- **Test data:** Name "Somchai Prasert", city "Bangkok", DOB 1990-05-01, bio text, valid JPG photo under 5 MB.
- **Steps:**
  1. Edit name, city, date of birth, and bio; save.
  2. Make another edit and click "Cancel".
  3. Start a new checkout and observe the contact fields.
- **Expected result:** Valid changes are saved and confirmed. "Cancel" discards the unsaved edits (previous saved values remain). At the next checkout, contact details pre-fill from the saved profile.

### TC-DISC-22 — Email/phone re-verification and photo upload validation
- **Traces:** US-DISC-11  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Signed-in attendee on the Profile tab.
- **Test data:** New email new@example.com; new phone 08x-xxx-xxxx; oversized photo 7 MB JPG; a GIF/BMP file; a valid 3 MB PNG.
- **Steps:**
  1. Change the email and save.
  2. Change the phone and save.
  3. Upload the 7 MB photo, then the GIF/BMP, then the valid PNG.
- **Expected result:** The changed email is marked unverified and a verification link is sent, while the old email keeps working until confirmed. The changed phone must be confirmed by code before it is used for texts. The 7 MB photo and the non-JPG/PNG file are rejected with guidance and the current avatar is unchanged; the valid PNG is accepted.

---

## US-DISC-12 — Manage my notifications, display preferences, and security

### TC-DISC-23 — Notification toggles, marketing exclusion, and display prefs vs Baht charges
- **Traces:** US-DISC-12  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed-in attendee on the Settings tab.
- **Test data:** Toggle marketing/promotions OFF, event reminders ON; set language = Thai, timezone = Asia/Bangkok, display currency = USD; a promotion send and an order confirmation.
- **Steps:**
  1. Toggle email notifications, event reminders, SMS alerts, and marketing/promotions; save.
  2. With marketing OFF, trigger a promotional send.
  3. Complete an order and check the confirmation/receipt still arrives.
  4. Set language, timezone, and display currency, then view an event and its price.
- **Expected result:** Toggle choices are saved and applied to future messages. With marketing off, the attendee is excluded from the promotion but still receives order confirmations and receipts (transactional always sent). Interface reflects the chosen language/timezone/currency, but all charges still settle in ฿ (THB) and event times stay anchored to each event's own timezone.

### TC-DISC-24 — Password change validation and two-factor enablement
- **Traces:** US-DISC-12  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Signed-in attendee on the Settings tab.
- **Test data:** Correct current password; a wrong current password; mismatched new/confirm passwords; 2FA setup with recovery codes.
- **Steps:**
  1. Change password using a wrong current password.
  2. Change password with correct current password but mismatched new/confirm fields.
  3. Change password correctly and save.
  4. Enable two-factor authentication and complete verification, then sign out and back in.
- **Expected result:** Wrong current password and mismatched new passwords each show a clear error and change nothing. A correct change updates the password, sends a confirmation email, and offers to sign out other sessions. Enabling 2FA turns it on with recovery codes provided, and it is required at the next sign-in.

---

## US-DISC-13 — Share post-event feedback

### TC-DISC-25 — Submit feedback contributes to average and resubmission updates it
- **Traces:** US-DISC-13  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed-in attendee who attended an event with feedback open.
- **Test data:** 4-star rating, "enjoyed the venue", heard via "friend", would recommend = Yes; later change to 5 stars.
- **Steps:**
  1. Open the feedback survey for the attended event.
  2. Give a 4-star rating and fill the optional fields; submit.
  3. Reopen the survey within the window and change the rating to 5 stars; resubmit.
  4. Check the event's average rating on Discover.
- **Expected result:** A thank-you confirmation appears and the rating contributes to the event's average shown on Discover. Resubmitting within the window updates the earlier response rather than creating a duplicate; the average reflects the single updated rating.

### TC-DISC-26 — Missing rating is blocked; ineligible/closed feedback is disallowed
- **Traces:** US-DISC-13  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** An attended event with feedback open; a second event where feedback is closed or the attendee is not eligible (e.g. did not attend).
- **Test data:** Feedback with comments filled but no star selected; a closed/ineligible event survey.
- **Steps:**
  1. Open the open survey, fill only the optional comments, leave the star rating unset, and submit.
  2. Open the survey for the closed/ineligible event.
- **Expected result:** Submitting without a star rating prompts the attendee to choose one and records nothing until a rating is chosen. For the closed/ineligible event the attendee is told feedback isn't open.

---

## US-DISC-14 — Delete my account

### TC-DISC-27 — Account deletion with confirmation, re-verification, and non-refundable warning
- **Traces:** US-DISC-14  ·  **Priority:** Low  ·  **Type:** Functional
- **Preconditions:** Signed-in attendee in the Settings danger zone; attendee holds an upcoming paid, non-refundable ticket.
- **Test data:** Upcoming paid ticket ฿1,880 marked non-refundable; identity re-verification (password/2FA); confirmation email.
- **Steps:**
  1. In the danger zone, choose to delete the account.
  2. Observe the warning about upcoming non-refundable tickets.
  3. Explicitly confirm and complete identity re-verification.
  4. Check the resulting session and email.
- **Expected result:** Deletion requires explicit confirmation and identity re-verification, and warns about non-refundable upcoming tickets before continuing. On confirmed re-verification, personal data is scheduled for removal, the attendee is signed out, and a confirmation email is received. (Financial/tax records are retained anonymized per legal retention.)

### TC-DISC-28 — Failed identity re-verification aborts deletion
- **Traces:** US-DISC-14  ·  **Priority:** Low  ·  **Type:** Negative
- **Preconditions:** Signed-in attendee attempting account deletion.
- **Test data:** Wrong password / failed 2FA at the re-verification step.
- **Steps:**
  1. Start account deletion and reach the identity re-verification step.
  2. Fail re-verification (wrong password / failed 2FA).
- **Expected result:** Deletion is aborted and the account is unchanged (still signed in, data intact, no confirmation email sent).

## US-DISC-15 — Create my account from my confirmation

### TC-DISC-29 — Follow the confirmation link and create the account with a locked email
- **Traces:** US-DISC-15  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** A guest (no account) has just confirmed a registration and is on the "You're registered!" screen. Their name, email and phone were captured during checkout.
- **Test data:** Buyer "Anong Pattana" / anong.p@example.com; one confirmed order with one QR ticket.
- **Steps:**
  1. Confirm the ticket and QR are shown on the recap, and that the account offer is a link — the recap itself asks for no credential.
  2. Follow the "Create account" link.
  3. Inspect the email field on the sign-up page.
  4. Set a password and submit.
- **Expected result:** The sign-up page opens with anong.p@example.com already filled in and the field disabled — it cannot be edited or cleared. Only a password is asked for. On submit the attendee is signed in immediately and lands on My Events, with no email-verification step, because the address was proven by the ticket being sent to it.

### TC-DISC-29b — The locked email cannot be substituted before submitting
- **Traces:** US-DISC-15  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** The sign-up page has been opened from a completed booking.
- **Steps:**
  1. Re-enable or overwrite the disabled email field in the browser, or post the form with a different email than the booking's.
  2. Submit.
- **Expected result:** The account is created for the booking's own email regardless, or the request is refused — the submitted email is never trusted. A claimed booking can never be turned into an account for a different address.

### TC-DISC-30 — Declining the account leaves the registration untouched
- **Traces:** US-DISC-15  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** A guest has just confirmed a registration.
- **Steps:**
  1. Skip or ignore the account offer.
  2. Check the ticket, QR and confirmation email.
  3. Follow the ticket link in the confirmation email while signed out.
- **Expected result:** The registration, ticket and receipt are entirely unaffected. The emailed ticket link opens without signing in. The offer never blocks the ticket.

### TC-DISC-31 — Past registrations appear once the account exists
- **Traces:** US-DISC-15  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** The same email has TWO earlier confirmed registrations, made as a guest with no account.
- **Steps:**
  1. Create the account from a third registration's confirmation screen.
  2. Open My Events.
- **Expected result:** All three registrations are listed, including the two made before the account existed — registrations are matched to the account by the email the order was placed with.

### TC-DISC-32 — An email that already has an account is offered sign-in, not a duplicate
- **Traces:** US-DISC-15  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** An attendee account already exists for the email used at checkout.
- **Steps:**
  1. Complete a registration with that email as a signed-out guest.
  2. Follow the "Create account" link from the confirmation.
- **Expected result:** The page invites signing in instead of offering a second account. No duplicate account is created.

### TC-DISC-32b — The sign-up link cannot be guessed to harvest a buyer's email
- **Traces:** US-DISC-15  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** A real confirmed order exists, with reference ORD-GAZJG4J0 printed on its ticket.
- **Steps:**
  1. Open the sign-up page using the order's human-readable REFERENCE as the identifier.
  2. Open it using a plausible neighbouring reference.
  3. Open it using a random value.
- **Expected result:** None of these reveal an email address. The page accepts only the order's unguessable id; anything else opens the page in its ordinary mode with an empty, editable email field. The buyer's address is never disclosed to someone who merely guessed an identifier.

### TC-DISC-33 — The account can be created later from the attendee sign-in page
- **Traces:** US-DISC-15  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** A person skipped the offer at confirmation, or never registered for anything.
- **Steps:**
  1. Open the attendee sign-in page and choose "Create one".
  2. Complete the form and verify the email.
  3. Open My Events.
- **Expected result:** The email field is empty and editable, because no booking was in hand. The account is created in the single platform-wide realm with no workspace named, and because nothing proved the address, it is verified by email before it can be used. Any registrations previously made with that email are then listed.


---

<a id="tc-e07"></a>

# Communicate with Attendees — Test Cases

Epic E7 — Communicate with Attendees. Area code: MSG. Locale: Thai market (฿, VAT 7%, PromptPay, Asia/Bangkok, EN/TH).

---

## US-MSG-01 — Attendees automatically get the right message at the right moment

### TC-MSG-01 — Confirmation message fires on successful payment
- **Traces:** US-MSG-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** A published paid event with the registration-confirmation message enabled; PromptPay enabled as a payment method.
- **Test data:** Attendee "Somchai Jai"; 1 x General ticket ฿1,000 (VAT 7% inclusive/added per event config); PromptPay payment.
- **Steps:**
  1. As the attendee, complete registration for the event and reach the payment step.
  2. Pay ฿1,000 via PromptPay and let the payment succeed.
  3. Open the attendee's inbox / message channel.
- **Expected result:** A confirmation message is received containing the ticket and order details (event name, ticket type, ฿ amount, order reference). No personalization field is left as a raw placeholder. Ties to the confirmation-on-payment acceptance criterion.

### TC-MSG-02 — Reminder is sent when the reminder time arrives
- **Traces:** US-MSG-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Attendee holds a confirmed ticket; reminder message enabled; event scheduled roughly one day out in Asia/Bangkok time.
- **Test data:** Event start 2026-07-28 18:00 (Asia/Bangkok); reminder configured for one day before.
- **Steps:**
  1. Advance to (or wait for) the configured reminder time on 2026-07-27.
  2. Open the attendee's inbox.
- **Expected result:** A reminder message is received stating the event's time and place (date/time shown in Asia/Bangkok, venue). Ties to the day-before reminder acceptance criterion.

### TC-MSG-03 — Language preference honoured with fallback; disabled message not sent
- **Traces:** US-MSG-01  ·  **Priority:** High  ·  **Type:** Edge
- **Preconditions:** Two attendees on the same event: one with language preference Thai, one with no preference set; event default language English. Cancellation message is turned OFF by the organizer.
- **Test data:** Attendee A preference = TH; Attendee B preference = none; event default = EN.
- **Steps:**
  1. Trigger the confirmation message for both attendees on successful payment.
  2. Inspect each attendee's received message language.
  3. Organizer cancels the event so the cancellation trigger fires.
  4. Check whether a cancellation message is delivered.
- **Expected result:** Attendee A receives the message in Thai; Attendee B receives it in English (event default fallback). No cancellation message is sent to either attendee because that message type is turned off. Ties to the language-preference and message-off acceptance criteria.

---

## US-MSG-02 — Keep automated messages on-brand and in the organizer's control

### TC-MSG-04 — Edit wording and insert a personalization field
- **Traces:** US-MSG-02  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Organizer signed in; registration-confirmation template open in the editor (EN and TH versions present).
- **Test data:** New body text "Welcome [first name] to [event name]!"; personalization fields = first name, event name.
- **Steps:**
  1. Place the cursor in the body and insert the "first name" personalization field.
  2. Insert the "event name" field at a second cursor position.
  3. Edit surrounding wording and Save.
  4. Trigger a new confirmation to a test recipient "Nok"; also inspect a previously-sent message.
- **Expected result:** Each field appears exactly where the cursor was; the newly-sent message shows the fields filled ("Welcome Nok to ..."). Messages sent before the edit are unchanged. Ties to the edit-wording and personalization-field acceptance criteria.

### TC-MSG-05 — SMS live segment count and second-segment cost warning (Thai encoding)
- **Traces:** US-MSG-02  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Organizer editing an SMS message; both EN and TH bodies editable.
- **Test data:** Thai body text long enough to cross from one segment into a second (Thai/UCS-2 uses shorter ~70-char segments); an English body just under one GSM-7 segment.
- **Steps:**
  1. Type Thai text and watch the live character/segment counter as it approaches the single-segment limit.
  2. Continue typing until the message crosses into a second segment.
  3. Compare the segment threshold against the English GSM-7 body.
- **Expected result:** A live character/segment count is displayed and reflects the actual text; when the Thai text crosses into a second segment a warning appears explaining the added cost. Thai text triggers the second segment at a lower character count than English (shorter Thai segments). Ties to the SMS segment-count/warning acceptance criterion.

### TC-MSG-06 — Disable warning for legally-expected message; save validation blocks
- **Traces:** US-MSG-02  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Organizer editing messages; payment-receipt message currently active with EN and TH versions.
- **Test data:** (a) Attempt to turn off the payment-receipt message. (b) Attempt to save an active message with an empty subject. (c) Attempt to save with body referencing unsupported field "[loyalty_points]". (d) Attempt to save with the TH language version left blank.
- **Steps:**
  1. Toggle the payment-receipt message off.
  2. Separately, clear the subject of an active message and Save.
  3. Insert an unsupported personalization field and Save.
  4. Clear the Thai body of an active channel and Save.
- **Expected result:** (a) A warning states attendees won't receive the receipt and requires explicit confirmation before it is disabled. (b) Save is blocked citing empty subject. (c) Save is blocked citing the unsupported field. (d) Save is blocked because an active channel cannot have a blank language version. Ties to the disable-warning and save-validation acceptance criteria.

---

## US-MSG-03 — Triage everything happening across my events in one feed

### TC-MSG-07 — Feed shows All/Unread counts; mark all read persists
- **Traces:** US-MSG-03  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Organizer with several notifications, some unread, spanning registrations, payments, and feedback.
- **Test data:** 12 notifications total, 5 unread.
- **Steps:**
  1. Open the notification feed.
  2. Note the All and Unread counts and which activity is expanded.
  3. Switch to the Unread filter.
  4. Click "mark all read".
  5. Reload the page and reopen the feed.
- **Expected result:** All=12 and Unread=5 shown; most recent activity is expanded with older activity available on demand. After "mark all read" the unread count drops to 0. On the Unread filter with nothing unread, a "you're all caught up" message shows. After reload the unread count stays 0. Ties to the counts, empty-state, and persistence acceptance criteria.

### TC-MSG-08 — Staff/Team member sees only own feed and has no compose/broadcast/log access
- **Traces:** US-MSG-03  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** A Staff/Team member account without finance access, part of an organizer's workspace.
- **Test data:** Staff member "Aran"; workspace has payment and payout notifications belonging to finance-access members.
- **Steps:**
  1. Sign in as the Staff member and open the notification feed.
  2. Look for other members' activity, payment/payout items, and controls to compose messages, edit templates, broadcast, view the delivery log, or manage feedback.
- **Expected result:** Only the staff member's own personal activity is shown; payment/payout items (needing finance access) are not visible. No controls to compose, edit templates, broadcast, view the delivery log, or manage feedback are available. Ties to the Staff-scope acceptance criterion.

---

## US-MSG-04 — Broadcast a one-off announcement to the right audience

### TC-MSG-09 — Send now to checked-in attendees; recipient count matches
- **Traces:** US-MSG-04  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Event with a mix of registrants; some checked in, some with missing/invalid contact details. Email and SMS channels enabled.
- **Test data:** 50 registrants; 20 checked in; of those 20, 2 have no valid contact; announcement over Email + SMS.
- **Steps:**
  1. Compose an announcement (subject + body), choose audience "checked-in attendees", select Email and SMS.
  2. Send now.
  3. Double-click / retry the send button once.
  4. Review the recorded recipient count.
- **Expected result:** Only the 18 checked-in attendees with a valid contact receive it; the recorded recipient count is 18 and matches who was addressed. The double-click/retry does not double-send to any recipient. Ties to the checked-in-audience and no-double-send acceptance criteria.

### TC-MSG-10 — Schedule for future Bangkok time; validation and empty/marketing edge cases
- **Traces:** US-MSG-04  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Organizer composing an announcement; audiences available (all registrants, checked-in, waitlist); marketing-class option available.
- **Test data:** (a) Schedule 2026-08-01 09:00 Asia/Bangkok. (b) No channel selected / empty body / empty email subject. (c) Audience that resolves to nobody (empty waitlist). (d) Marketing-class announcement with one opted-out recipient among the audience.
- **Steps:**
  1. Schedule the announcement for the future Bangkok date/time and send.
  2. Separately, attempt to send with no channel selected, then with an empty body, then with an empty email subject.
  3. Choose the empty waitlist audience and send.
  4. Send a marketing-class announcement to an audience that includes an opted-out attendee.
- **Expected result:** (a) Announcement is recorded as Scheduled and goes out at the specified Bangkok time. (b) Each attempt is blocked with a clear reason (no channel / empty message / empty subject). (c) Nothing is sent and the organizer is told no recipients matched. (d) Delivered marketing message includes an unsubscribe option; the opted-out attendee is excluded and counted as skipped. Ties to the schedule, validation, no-recipients, and marketing-opt-out acceptance criteria.

---

## US-MSG-05 — Change my mind on a scheduled announcement

### TC-MSG-11 — Cancel and reschedule a not-yet-sent announcement
- **Traces:** US-MSG-05  ·  **Priority:** Low  ·  **Type:** Functional
- **Preconditions:** Two scheduled announcements still in the future.
- **Test data:** Announcement X scheduled 2026-08-05 10:00 BKK (to cancel); Announcement Y scheduled 2026-08-05 10:00 BKK, rescheduled to 2026-08-06 15:00 BKK.
- **Steps:**
  1. Cancel Announcement X.
  2. Verify no messages are sent for X and check the history.
  3. Reschedule Announcement Y to the later valid time and let that time pass.
- **Expected result:** X sends no messages and no longer shows as Scheduled, but remains visible in history. Y goes out at the new time (2026-08-06 15:00 BKK) to a freshly resolved audience. Ties to the cancel and reschedule acceptance criteria.

### TC-MSG-12 — Cannot change an announcement that has started sending
- **Traces:** US-MSG-05  ·  **Priority:** Low  ·  **Type:** Negative
- **Preconditions:** An announcement whose send has already begun.
- **Test data:** Announcement Z currently in the process of sending.
- **Steps:**
  1. Attempt to cancel or reschedule Announcement Z while it is sending.
- **Expected result:** The organizer is told the announcement can no longer be changed; no cancel/reschedule takes effect. Ties to the already-sending acceptance criterion.

---

## US-MSG-06 — Prove messages were delivered and diagnose failures

### TC-MSG-13 — Delivery log per-recipient with statuses; failures surfaced
- **Traces:** US-MSG-06  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Admin with attendee-view access; messages have gone out across email and SMS, including a provider-reported SMS failure, an opened email, and a failed critical message (ticket/receipt).
- **Test data:** Recipients: email opened (Nok), SMS undelivered (Anan), receipt email failed (Ploy).
- **Steps:**
  1. Open the delivery log.
  2. Locate the entries for Nok, Anan, and Ploy.
  3. Review how critical-message failures are presented.
- **Expected result:** One entry per recipient shows recipient, message type, channel, status, and time. Anan's entry shows SMS channel with Failed status; Nok's entry advances to Opened once confirmed; Ploy's failed ticket/receipt is surfaced for follow-up. Statuses reflect only what the provider confirmed. Ties to the log-contents, failed-SMS, opened, and critical-failure acceptance criteria.

### TC-MSG-14 — Delivery log access restricted; Staff denied
- **Traces:** US-MSG-06  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** A Staff member without attendee-view access; the log exposes attendee contact details.
- **Test data:** Staff member "Aran".
- **Steps:**
  1. Sign in as the Staff member.
  2. Attempt to open the delivery log.
- **Expected result:** Access is denied; the log (containing attendee contact details) is available only to organizers/admins with attendee-view access. Ties to the log-access-restriction note.

---

## US-MSG-07 — Export the delivery log for records

### TC-MSG-15 — Export permitted with audit; denied / empty-view edge cases
- **Traces:** US-MSG-07  ·  **Priority:** Low  ·  **Type:** Negative
- **Preconditions:** (a) Organizer with attendee-export permission and a non-empty log view; (b) Staff member or user lacking export permission; (c) A log view filtered to zero rows.
- **Test data:** Current log view = 25 entries (case a); filtered view = 0 entries (case c).
- **Steps:**
  1. As the permitted organizer, export the current log view.
  2. As a Staff / no-export-permission user, view the log and look for the export option.
  3. As the permitted organizer, filter the view to zero rows and attempt to export.
- **Expected result:** (a) A file of the current log view (25 entries) downloads and the export is recorded for audit. (b) The export option is unavailable. (c) The organizer is told there is nothing to export. Ties to the export-with-audit, permission, and nothing-to-export acceptance criteria.

---

## US-MSG-08 — Measure attendee satisfaction across all my events

### TC-MSG-16 — Portfolio KPIs weighted by responses; no-responses events handled
- **Traces:** US-MSG-08  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Organizer with several events; some have collected feedback, at least one has none.
- **Test data:** Event A 100 responses avg 4.5; Event B 10 responses avg 3.0; Event C 0 responses.
- **Steps:**
  1. Open the feedback overview.
  2. Read the portfolio KPIs (total responses, average rating, NPS, completion rate) and the per-event cards.
  3. Inspect Event C's card.
- **Expected result:** Portfolio KPIs shown; the average is weighted by each event's response count (A weighted more heavily than B) and Event C does not distort the averages. Event C's card clearly shows "no responses yet" rather than misleading numbers. Ties to the portfolio-KPI, weighting, and no-responses acceptance criteria.

### TC-MSG-17 — Event search filter and drill-down rating-bar filtering
- **Traces:** US-MSG-08  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Organizer on the feedback overview with multiple events, one named "Bangkok Tech Meetup".
- **Test data:** Search term "Bangkok"; then a non-matching term "zzzz"; drill into an event and click the 5-star rating bar.
- **Steps:**
  1. Type "Bangkok" in the event search.
  2. Type a term matching no event.
  3. Open an event's detail; review its KPIs, rating distribution, and surveys; click the 5-star rating bar.
- **Expected result:** The grid filters to matching events for "Bangkok"; a clear empty state shows when none match. The event detail shows its KPIs, rating distribution, and surveys; clicking the 5-star bar filters the responses to that rating. Ties to the search-filter and drill-down acceptance criteria.

---

## US-MSG-09 — Build and manage surveys for my events

### TC-MSG-18 — Create survey as Draft; validation blocks invalid questions
- **Traces:** US-MSG-09  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Organizer with an event to attach a survey to.
- **Test data:** Valid survey: title "Post-Event Feedback" + one rating question. Invalid: a multiple-choice question with one option; a question with blank text.
- **Steps:**
  1. Create a survey with a title and one rating question against the event and Save.
  2. Confirm where it appears and whether it collects responses.
  3. Add a multiple-choice question with only one option and Save.
  4. Add a question with blank text and Save.
- **Expected result:** The valid survey is created as a Draft under that event, appears in its surveys list, and collects nothing until made live. Saving is blocked with a clear reason when a multiple-choice question has fewer than two options, and when a question's text is blank. Ties to the create-draft and question-validation acceptance criteria.

### TC-MSG-19 — Survey lifecycle: duplicate, close/reopen, delete with confirm
- **Traces:** US-MSG-09  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** An existing live survey with collected responses.
- **Test data:** Survey "Post-Event Feedback" with 30 responses.
- **Steps:**
  1. Duplicate the survey.
  2. Close the live survey, then attempt to submit a response (via the attendee portal) and confirm it is rejected.
  3. Reopen the survey and confirm responses are accepted again.
  4. Delete the survey (which has responses).
- **Expected result:** Duplicating produces a fresh Draft copy with zero responses next to the original. Closing stops it accepting responses; reopening resumes acceptance. Deleting a survey with responses requires explicit confirmation before removal. Ties to the duplicate, close/reopen, and delete-confirmation acceptance criteria.

---

## US-MSG-10 — Browse and filter individual feedback responses

### TC-MSG-20 — Filter responses, paginate range, and empty-state
- **Traces:** US-MSG-10  ·  **Priority:** Low  ·  **Type:** Functional
- **Preconditions:** Organizer with attendee-view access on an event that has many responses across multiple surveys and ratings.
- **Test data:** 47 responses; page size 20; filter by rating = 5; then a filter combination matching zero responses.
- **Steps:**
  1. Filter responses by rating = 5 (and/or a specific survey) and review the list.
  2. Move from page 1 to page 2 to page 3 and read the "showing X–Y of N" range.
  3. Apply a filter combination that matches no responses.
- **Expected result:** The list narrows to matching responses and paginates, each row showing respondent, survey, rating, comment, and date. The "showing X–Y of N" range updates correctly across pages (e.g. 1–20, 21–40, 41–47 of 47). When no responses match, a clear "no responses match these filters" message shows. Ties to the filter, pagination-range, and empty-state acceptance criteria.


---

<a id="tc-e08"></a>

# Manage Registrations & Admit Attendees — Test Cases

Area: REG · Epic E8 · Locale: Thai market (฿, VAT 7% inclusive, PromptPay, Asia/Bangkok, EN/TH)

Roles referenced: Organizer, Admin (full), Staff (view + door check-in), Attendee (subject only, never operates screens).

---

## US-REG-01 — Review the registrations queue

### TC-REG-01 — Queue loads with tabs, live counts, status badges and VAT-inclusive amounts
- **Traces:** US-REG-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in as Organizer with finance access. Registrations exist across events in mixed states: Confirmed (paid), Pending, Waitlisted, Cancelled, plus at least one free-ticket registration.
- **Test data:** Paid registration total ฿1,070 (VAT-inclusive); one free-ticket registration; event "Bangkok Tech Meetup".
- **Steps:**
  1. Open the registrations workbench.
  2. Observe the tabs and their counts.
  3. Note the status badge and amount on each visible row.
  4. Approve or cancel one registration in another tab/session, then return.
- **Expected result:** List shows tabs All, Pending, Waitlist, Cancelled, each with a live count. Each row carries a clear status badge (Confirmed / Pending / Waitlisted / Cancelled). Paid rows show the amount as a VAT-inclusive Baht figure (฿1,070); free rows show a "Free" badge. Pending rows offer Approve and Reject; non-pending rows offer only View plus an overflow menu. After a status change elsewhere, the tab counts update to reflect it.

### TC-REG-02 — Search and combined event + ticket-type filters, plus empty state
- **Traces:** US-REG-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in as Organizer. Multiple events and ticket types with registrations exist. Attendee "สมชาย ใจดี" (booking ref RG-2026-0042) is registered for a "VIP" ticket on Event A.
- **Test data:** Search terms: name "สมชาย", email "somchai@example.com", booking ref "RG-2026-0042". Filter: Event A + ticket type "General" (a combination with no matching rows).
- **Steps:**
  1. In the workbench, search by attendee name "สมชาย"; confirm results, then repeat with the email and again with the booking reference.
  2. Clear search. Filter by Event A only, then additionally by ticket type "VIP".
  3. Change the ticket-type filter to "General" (no registrations for that combination).
- **Expected result:** Each search narrows the list to matching rows and resets to the first page. Applying Event A and ticket type "VIP" together shows only rows satisfying both conditions (filters combine — AND logic). The Event A + "General" combination returns a clear "No matches" empty state rather than an error or stale rows.

### TC-REG-03 — Staff without finance access sees monetary amounts masked
- **Traces:** US-REG-01  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Logged in as a Staff member with view access but no finance access. Paid registrations exist.
- **Test data:** A confirmed paid registration with total ฿2,140.
- **Steps:**
  1. Open the registrations workbench as Staff.
  2. Inspect a paid registration row's amount.
  3. Open that registration's detail panel and inspect the payment/amount fields.
- **Expected result:** Monetary amounts are masked (not shown as ฿ figures) in both the list and the detail panel for the Staff member. Read-only browsing of status, ticket and history remains available, but the actual Baht amount is not disclosed.

---

## US-REG-02 — Approve or reject a pending registration

### TC-REG-04 — Approve a pending free registration issues ticket + QR and notifies (idempotent)
- **Traces:** US-REG-02  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in as Organizer. A pending registration exists for a free ticket on an event with seats available.
- **Test data:** Attendee "Ploy S." (email + Thai mobile on file), free ticket, 1 seat, event with remaining capacity > 0.
- **Steps:**
  1. Open the pending registration and click Approve.
  2. Verify the resulting status, seat assignment and issued ticket.
  3. Confirm notifications are queued.
  4. Click Approve again (simulate a retry/double-click).
- **Expected result:** Registration becomes Confirmed, the attendee is seated (capacity decremented by 1), a ticket with a QR code is issued, and a confirmation is sent by both email and SMS. On the repeated Approve, only one ticket exists and only one confirmation was sent (idempotent — no duplicate ticket or message).

### TC-REG-05 — Approving a paid-but-unpaid registration is blocked
- **Traces:** US-REG-02  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Logged in as Organizer. A pending registration exists for a paid ticket whose payment is not yet complete (PromptPay not settled).
- **Test data:** Paid ticket total ฿1,605 (VAT-inclusive), payment state = unpaid/pending.
- **Steps:**
  1. Open the pending paid registration.
  2. Click Approve.
- **Expected result:** Approval is blocked with the message "Payment isn't complete yet, so this registration can't be approved." The registration stays Pending, no ticket/QR is issued, and no confirmation is sent.

### TC-REG-06 — Reject a pending registration releases the seat and cannot be re-approved
- **Traces:** US-REG-02  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Logged in as Organizer. A pending (or waitlisted) registration with no funds captured is holding a seat on an event that is at/near capacity with a waitlist.
- **Test data:** Free or unpaid registration holding 1 seat; a waitlist exists for the same ticket.
- **Steps:**
  1. Open the registration and click Reject.
  2. When the confirmation prompt appears, confirm the rejection.
  3. Reopen the now-rejected registration and look for an Approve action.
  4. Check the freed seat is available to the waitlist.
- **Expected result:** After confirming, the registration becomes Rejected, the held seat is released (may trigger waitlist promotion), and a rejection notice is emailed to the attendee. The confirmation step is required (a single misclick alone does not terminate it). On the rejected registration no Approve action is offered and it cannot be re-approved. (Note: for a Confirmed registration with money captured, Reject is not offered — the user is directed to cancel and refund instead.)

---

## US-REG-03 — Add a registration by hand

### TC-REG-07 — Manually add a registration with auto VAT amount and confirmation delivery
- **Traces:** US-REG-03  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in as Organizer. An event is open for registration with remaining capacity and both a free and a paid ticket type.
- **Test data:** Attendee name "Nattaya P.", valid email, Thai mobile; paid ticket priced so total = ฿1,070 (VAT-inclusive); seat quantity = 2; "Send confirmation" toggle ON.
- **Steps:**
  1. Open "Add registration", enter name, email, event, paid ticket type and quantity 2.
  2. Observe the amount field (do not type a price).
  3. Save with "Send confirmation" ON.
  4. Repeat for a free ticket type and observe the amount and resulting status.
  5. Repeat once with "Send confirmation" OFF.
- **Expected result:** Seats are reserved against capacity and the attendee is matched to an existing directory record or added as new. The paid amount is calculated automatically as a VAT-inclusive Baht figure (never typed). With the toggle ON, ticket + QR are delivered by email and SMS; with it OFF, the ticket is created quietly for later delivery. For the free ticket the amount shows "Free" and the registration is confirmed immediately.

### TC-REG-08 — Seat quantity boundary: over 8 per booking and over remaining capacity are rejected
- **Traces:** US-REG-03  ·  **Priority:** High  ·  **Type:** Edge
- **Preconditions:** Logged in as Organizer. Event open for registration with exactly 3 seats remaining.
- **Test data:** Attempt A: quantity 9 (over max). Attempt B: quantity 8 (allowed max). Attempt C: quantity 5 (exceeds the 3 remaining). Also a draft/completed/cancelled event for the final check.
- **Steps:**
  1. Try to save a booking with quantity 9.
  2. Change quantity to 8 and observe acceptance behaviour (capacity permitting).
  3. On the event with 3 seats left, try to save quantity 5.
  4. Switch the chosen event to a draft (or completed/cancelled) event and try to add a registration.
- **Expected result:** Quantity 9 is rejected with a prompt to choose between 1 and 8 seats and nothing is reserved; quantity 8 is within the allowed maximum. Quantity 5 against 3 remaining is refused with a message stating exactly how many seats are left (3) and no partial booking is made. Adding to a draft/completed/cancelled event is refused with "the event isn't open for new registrations." Missing required fields or an invalid email produce inline errors while the panel keeps entered values.

---

## US-REG-04 — Manage the waitlist & promote attendees

### TC-REG-09 — Promote the front waitlisted attendee with a time-limited offer
- **Traces:** US-REG-04  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Logged in as Organizer. A seat is free and a waitlist exists (first-come-first-served order).
- **Test data:** Front-of-queue attendee for a paid ticket (฿1,070); default offer window 24 hours. Also a free-ticket waitlist for the second part.
- **Steps:**
  1. Promote the person at the front of the waitlist for the paid ticket.
  2. Observe the offer and notification.
  3. Deliberately promote a different person out of order and check what is recorded.
  4. For a free-ticket waitlist, promote the front person and observe status.
- **Expected result:** The paid-ticket promotee receives a time-limited offer (24 hours by default) to complete payment and is notified; the offer holds the seat. When promoting out of order, the out-of-order choice is recorded. Free-ticket promotions confirm immediately (ticket + QR + confirmation). If the offer lapses unused, the seat passes automatically to the next in line and the attendee is told the offer expired; if the promotee pays within the window the registration confirms just like a normal approval.

### TC-REG-10 — Promotion refused when no seat is free
- **Traces:** US-REG-04  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Logged in as Organizer. The ticket is fully seated (0 free) with people on the waitlist.
- **Test data:** Sold-out ticket, waitlist length ≥ 1.
- **Steps:**
  1. Attempt to promote the front waitlisted attendee while capacity is full.
- **Expected result:** Promotion is refused and the organizer is told to free capacity first. No offer is sent and the waitlist order is unchanged.

---

## US-REG-05 — Browse the attendee directory & profiles

### TC-REG-11 — Directory loads with segments, live counts and sort options
- **Traces:** US-REG-05  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in as Organizer. Attendees exist with varied activity, ticket counts and event counts; at least one VIP-tagged and one checked-in attendee.
- **Test data:** Sort options: name, most events, most tickets. Segments: All, New, Checked-in, VIP.
- **Steps:**
  1. Open the attendee directory and note default order and segment counts.
  2. Sort by name, then by most events, then by most tickets.
  3. Switch between the All / New / Checked-in / VIP segments.
- **Expected result:** The directory shows segments All, New, Checked-in, VIP each with a live count, sorted by most recent activity by default. Sorting by name / most events / most tickets re-orders the list accordingly, with ties broken by most recent activity. Segment switching filters to the matching attendees.

### TC-REG-12 — Search / tag filter combine, badges shown, and profile detail opens
- **Traces:** US-REG-05  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in as Organizer. Tagged attendees exist (VIP, Speaker, Sponsor, Student) plus untagged ones.
- **Test data:** Search name "อนันต์" / email; tag filter "VIP"; an attendee registered for 3 events with mixed statuses and one free + one ฿1,605 ticket.
- **Steps:**
  1. Search by name, then by email.
  2. Filter by tag "VIP" and combine with the search term.
  3. Observe tag badges on rows (tagged vs untagged).
  4. Open an attendee's profile panel.
  5. Search for a value that matches no attendee.
- **Expected result:** Search and tag filter narrow the list and conditions combine (AND). Tagged attendees show a coloured badge; untagged attendees show a dash. The profile panel shows contact details, tags, every event registered for with its status (Checked-in / Registered / Past), tickets with VAT-inclusive amounts or "Free", and a dated activity timeline newest first. A no-match search shows a "No matches" empty state.

---

## US-REG-06 — Invite people to an event

### TC-REG-13 — Send an invitation with a personal note
- **Traces:** US-REG-06  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in as Organizer/Admin. An event open for registration exists.
- **Test data:** Recipient name "Kanya T.", valid email, exactly one event; personal note "Hope you can join us!" (≤ 250 chars).
- **Steps:**
  1. Open the invite form; enter recipient name, email and select exactly one event.
  2. Add the personal note.
  3. Send the invite.
- **Expected result:** A tracked invite record is created and exactly one invitation email is queued to that person containing a link into the event's registration flow, with the personal note appearing at the top of the email. The invite does not reserve a seat (it is a promise to attend, not a booking).

### TC-REG-14 — Invite validation, note length limit and duplicate suppression
- **Traces:** US-REG-06  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Logged in as Organizer/Admin.
- **Test data:** Case A: missing name / invalid email "bad@" / no event selected. Case B: note of 251 characters. Case C: two identical invites (same person + same event) sent within a short window.
- **Steps:**
  1. Submit with a missing name, an invalid email and no event chosen.
  2. Correct fields, enter a 251-character note and submit.
  3. Send a valid invite, then immediately resend an identical invite to the same recipient + event.
- **Expected result:** Case A shows inline errors and nothing is sent. Case B is refused with a prompt to shorten the note to ≤ 250 characters. Case C suppresses the duplicate — no second (duplicate) email is dispatched. (Staff cannot invite — the invite action is unavailable to Staff.)

---

## US-REG-07 — Tag & segment attendees

### TC-REG-15 — Apply a VIP tag and see the segment count update
- **Traces:** US-REG-07  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Logged in as Organizer/Admin. An untagged attendee exists; current VIP segment count noted.
- **Test data:** Tag = VIP; VIP segment count before = N.
- **Steps:**
  1. Open the untagged attendee and set the tag to VIP.
  2. Observe the row badge and the VIP segment count.
- **Expected result:** A coloured VIP badge appears on the attendee's row and the VIP segment count increases to N+1 live.

### TC-REG-16 — Single-tag rule, re-apply no-op, and concurrent last-write-wins
- **Traces:** US-REG-07  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Logged in as Organizer/Admin. An attendee already tagged VIP.
- **Test data:** Change VIP → Speaker; then re-apply Speaker; concurrent edit by another team member setting Sponsor at the same moment.
- **Steps:**
  1. Change the tag from VIP to Speaker and observe badge and both segment counts (VIP −1, Speaker +1).
  2. Re-apply the Speaker tag the attendee already has and confirm.
  3. Simulate another team member changing the same attendee's tag at the same time as your change; submit yours.
- **Expected result:** An attendee carries at most one tag — changing VIP to Speaker moves the badge and adjusts both segment counts live. Re-applying the existing tag changes nothing. On a concurrent edit, the most recent change wins and the user is told the attendee was just updated. (Staff can view tags but not change them.)

---

## US-REG-08 — Keep attendee contact details current

### TC-REG-17 — Edit name, email and phone; audit entry and new routing
- **Traces:** US-REG-08  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Logged in as Organizer/Admin. An attendee profile exists with an upcoming event (future confirmations/reminders due).
- **Test data:** New valid name, new valid email, new Thai mobile "+66 8x-xxx-xxxx".
- **Steps:**
  1. Open the profile and edit name, email and phone to valid new values; save.
  2. Reopen the directory row and the profile.
  3. Check the activity timeline.
  4. Confirm the routing target for future confirmations/reminders.
- **Expected result:** The directory row and profile show the new details; the change is recorded for audit and an "Updated contact details" entry appears in the activity timeline. Future confirmations and reminders go to the new phone number.

### TC-REG-18 — Duplicate email blocked with merge prompt; invalid input rejected
- **Traces:** US-REG-08  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Logged in as Organizer/Admin. Two distinct attendee records exist; attendee B's email is "b@example.com".
- **Test data:** Editing attendee A's email to "b@example.com" (already owned by B); also an invalid email "no-at-sign" and an empty name.
- **Steps:**
  1. Edit attendee A's email to "b@example.com" and save.
  2. Separately, try to save an invalid email and an empty name.
- **Expected result:** The duplicate-email change is blocked and the user is prompted to merge the two records rather than overwrite. Invalid name/email produce inline errors and the change is not applied.

---

## US-REG-09 — Email an attendee directly

### TC-REG-19 — Send a one-off email; empty subject/message blocked; failure retryable
- **Traces:** US-REG-09  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Logged in as Organizer/Admin. An attendee profile is open.
- **Test data:** Valid subject "About your VIP seat" + message body; then an empty subject; then an empty message.
- **Steps:**
  1. Enter a valid subject and message and send.
  2. Check the attendee's activity timeline.
  3. Attempt to send with an empty subject, then with an empty message.
  4. (If a send fails) observe the error and retry.
- **Expected result:** With a valid subject and message, one email is queued to the attendee and the send is logged on the activity timeline. An empty subject or empty message is refused with a prompt to add both before sending. If a send fails, the user is told it couldn't be sent and can retry. (Staff cannot send attendee emails.)

---

## US-REG-10 — Export registration & attendee lists

### TC-REG-20 — Export exactly the filtered set with audit record and correct Thai names
- **Traces:** US-REG-10  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Logged in as Organizer/Admin with export permission. A filter is active that yields more rows than one page.
- **Test data:** Filter: Event A + tab "Confirmed" + tag "VIP", yielding e.g. 120 rows across multiple pages; attendees include Thai names "ศิริพร ทองดี".
- **Steps:**
  1. Apply the filter (tab, event, ticket, tag, search as available).
  2. Trigger Export.
  3. Open the produced file and inspect scope, columns and Thai-character rendering.
  4. Check the export audit record.
- **Expected result:** The file contains exactly the filtered set (all 120 rows, not just the current page) with attendee, event, ticket, date, amount and status columns, and Thai names render correctly (proper UTF-8, no mojibake). The export is recorded (who exported, row count, which filters) because it discloses attendee personal data.

### TC-REG-21 — Staff has no Export control; empty filter yields headers-only file
- **Traces:** US-REG-10  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Case A: logged in as Staff. Case B: logged in as Organizer with a filter that matches zero rows.
- **Test data:** Case B filter combination returning no registrations.
- **Steps:**
  1. As Staff, open the registrations/attendee list and look for the Export control.
  2. As Organizer, apply a zero-match filter and trigger Export.
- **Expected result:** The Export control is not available to Staff. For the zero-match export, an empty file with headers only is produced (no rows), not an error.

---

## US-REG-11 — Run the manual check-in list

### TC-REG-22 — Roster loads with live progress bar; checking in advances the count
- **Traces:** US-REG-11  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in with check-in permission (Organizer/Admin/Staff). An event with 29 confirmed attendees, 7 already checked in, and the check-in window open.
- **Test data:** Progress readout "7 / 29"; search terms name/email/ticket; ticket-type filter; a not-yet attendee to admit.
- **Steps:**
  1. Select the event and load the roster; note the progress readout and segments (All, Checked-in, Not-yet).
  2. Search by name, email and ticket, then filter by ticket type.
  3. Check in a not-yet attendee.
- **Expected result:** The roster shows confirmed attendees, a live progress readout "7 / 29" with a matching percentage bar, and segments All / Checked-in / Not-yet. Searching narrows the roster and resets to the first page. Checking in the attendee stamps a check-in time (HH:MM, Bangkok), raises the checked-in count to 8, and advances the progress bar immediately.

### TC-REG-23 — Undo a check-in; re-checking an already-in attendee keeps earliest time
- **Traces:** US-REG-11  ·  **Priority:** High  ·  **Type:** Edge
- **Preconditions:** Logged in with check-in permission. An event with at least one checked-in attendee (arrival time recorded).
- **Test data:** Attendee checked in at 09:12 (Bangkok).
- **Steps:**
  1. Undo the check-in for the checked-in attendee.
  2. Observe the time field and the Not-yet count.
  3. Check the same attendee in again, then attempt to check them in a second time.
- **Expected result:** Undo clears the check-in time and increases the Not-yet count by one. Re-checking records a time; a further duplicate check-in changes nothing and the original/earliest arrival time is kept (manual and QR check-ins stay consistent — no double count).

### TC-REG-24 — Check-in blocked for invalid registration or closed check-in window
- **Traces:** US-REG-11  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Logged in with check-in permission. Case A: a cancelled registration appears/searchable for the event. Case B: an event whose check-in window is not yet open.
- **Test data:** Case A: cancelled registration. Case B: event with check-in window closed.
- **Steps:**
  1. Attempt to check in the cancelled (invalid) registration.
  2. On the event whose window is closed, attempt to check anyone in.
- **Expected result:** Case A is refused with "the registration isn't valid for check-in." Case B is refused with "check-in isn't open for this event." No count changes in either case. (View access is open to anyone; only members with check-in permission can mark or undo.)

---

## US-REG-12 — Scan tickets at the live QR check-in station

### TC-REG-25 — Scan a valid unused ticket for the bound event admits the attendee
- **Traces:** US-REG-12  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in with check-in permission on a device with a working camera on a secure page. Station bound to Event A.
- **Test data:** A valid, unused ticket QR issued for Event A.
- **Steps:**
  1. Confirm the station is bound to Event A; start the camera.
  2. Hold a valid unused Event A ticket QR steady in view.
  3. Observe the result, live count and feed.
  4. Re-bind the station to a different event and confirm the scanner/stats/feed re-point.
- **Expected result:** A green "Checked in" result shows, the attendee is admitted, and the live count and feed update. Holding the code steady produces exactly one check-in — immediate re-reads of the same code are ignored. Switching the bound event re-points the scanner, stats and feed to the new event and starts fresh.

### TC-REG-26 — Duplicate, wrong-event, cancelled and invalid scan outcomes
- **Traces:** US-REG-12  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Station bound to Event A; camera running.
- **Test data:** (a) The Event A ticket already checked in at 09:12; (b) a valid ticket for Event B; (c) a cancelled/refunded ticket for Event A; (d) a non-ticket QR (e.g. a random URL).
- **Steps:**
  1. Scan the already-checked-in Event A ticket again.
  2. Scan the valid Event B ticket.
  3. Scan the cancelled/refunded ticket.
  4. Scan the unrecognisable non-ticket QR.
- **Expected result:** (a) "Already checked in" showing the original arrival time (09:12), count unchanged; (b) "Wrong event", no admission; (c) "Ticket cancelled — entry denied", no admission; (d) "Invalid ticket", no admission. In every negative case no one is admitted and the live count does not change.

### TC-REG-27 — Camera unavailable falls back gracefully without crashing
- **Traces:** US-REG-12  ·  **Priority:** High  ·  **Type:** Edge
- **Preconditions:** Station bound to an event. Camera cannot be opened (permission denied, no camera, insecure page, or camera in use by another app).
- **Test data:** Simulated camera-permission denial.
- **Steps:**
  1. Attempt to start the camera at the station.
  2. Observe the message and offered alternatives.
  3. Use the fallback: upload a QR image and/or search manually.
- **Expected result:** A plain-language reason is shown and the user is pointed to upload an image or search manually; the station never crashes. Decoding an uploaded QR image and manual search remain usable. (Torch, camera flip and a simulate control — a demo aid, not a real entry method — are available to support real door conditions.)

---

## US-REG-13 — Check attendees in manually when a QR won't scan

### TC-REG-28 — Manual lookup at the station checks in a not-yet attendee like a scan
- **Traces:** US-REG-13  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Station bound to Event A; logged in with check-in permission. A not-yet-checked-in attendee exists for Event A.
- **Test data:** Search by name / email / ticket for a valid not-yet attendee on Event A.
- **Steps:**
  1. Open "Can't scan? Find attendee manually" and search.
  2. Select the matching not-yet attendee and check them in.
  3. Observe the live count and feed.
- **Expected result:** The attendee is admitted, the live count goes up, and they appear at the top of the "Just checked in" feed — exactly as a scan would.

### TC-REG-29 — Already-checked-in attendee shows no action; empty search shows no-match message
- **Traces:** US-REG-13  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Station bound to Event A. One attendee already checked in; a search term matching nobody.
- **Test data:** Already-checked-in attendee "Somsak"; search term "zzz-no-match".
- **Steps:**
  1. In the manual lookup, find the already-checked-in attendee.
  2. Search for a value matching no attendee.
- **Expected result:** The already-checked-in attendee's row shows a "Checked in" badge and no check-in action, so they cannot be admitted twice. A search with no matches shows "No attendees match your search."

---

## US-REG-14 — Watch live turnout at the door

### TC-REG-30 — Live counter and feed update on each check-in, capped at capacity, per event
- **Traces:** US-REG-14  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Logged in as Organizer viewing the live door view. Check-ins occurring via mixed methods (scan, uploaded image, simulate, manual). Event A total capacity known.
- **Test data:** Event A capacity 200; check-ins arriving from different methods; a second event (Event B) to switch to.
- **Steps:**
  1. Observe the counter (checked / total, percentage, on-site, remaining) as check-ins land.
  2. Confirm a successful check-in from each method places the attendee atop the "Just checked in" feed marked "just now", with earlier arrivals ageing below.
  3. Drive check-ins toward capacity and observe the counter at the ceiling.
  4. Switch the station to Event B and observe the counter and feed.
- **Expected result:** The counter updates immediately on each successful check-in and never counts past the event's total capacity (200). Successful check-ins from any method (scan, uploaded image, simulate, manual) appear at the top of the feed marked "just now". Switching to Event B refreshes the counter and feed to that event's figures.


---

<a id="tc-e09"></a>

# Get Paid & Manage Finances — Test Cases

Epic E9 — Get Paid & Manage Finances. Area code: **FIN**. Locale defaults: ฿ (VAT-inclusive), VAT 7%, Asia/Bangkok time, EN/TH, PromptPay / Card / Bank transfer. RBAC baseline: Admin = full control; Organizer = view/invoice/export only; Staff & Attendee = no finance access.

---

## US-FIN-01 — See every payment

### TC-FIN-01 — Filter, search and live status counts on the payment ledger
- **Traces:** US-FIN-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in as Organizer (or Admin). Ledger has payments across Paid, Pending, Refunded and Failed, on Card, PromptPay and Bank transfer methods.
- **Test data:** Attendee "Somchai Jaidee", txn ref "TXN-90210", amounts e.g. ฿1,284.00 (VAT-inclusive).
- **Steps:**
  1. Open the payments ledger.
  2. Select status filter = Paid and method filter = PromptPay.
  3. In the search box type part of a name ("Somch") or part of a transaction reference ("90210").
  4. Read the count shown on each status tab.
- **Expected result:** Only payments matching status Paid + method PromptPay + the typed name/ref are listed. Each status tab (Paid/Pending/Refunded/Failed) shows a live count reflecting the whole ledger, not just the filtered view. Amounts display as VAT-inclusive Baht (฿).

### TC-FIN-02 — Newest-first order and disabled actions on Pending/Failed rows
- **Traces:** US-FIN-01  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Ledger contains at least one Pending and one Failed payment plus several Paid payments with differing timestamps.
- **Test data:** A Pending PromptPay charge and a Failed Card charge.
- **Steps:**
  1. Load the ledger and note the ordering of rows by date/time.
  2. Locate the Pending payment row; inspect "View invoice" and "Refund" controls.
  3. Locate the Failed payment row; inspect the same controls.
- **Expected result:** Payments are listed newest first. On both the Pending and Failed rows, "View invoice" and "Refund" are unavailable (disabled/hidden), each explaining why — no completed charge (View invoice) and not eligible for refund (Refund).

### TC-FIN-03 — PromptPay charge auto-updates to Paid on refresh
- **Traces:** US-FIN-01  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** A PromptPay charge is currently Pending and is expected to settle at the provider.
- **Test data:** Pending PromptPay charge, amount ฿980.00.
- **Steps:**
  1. View the ledger while the PromptPay charge is Pending.
  2. Allow the charge to settle at the provider.
  3. Refresh the ledger.
- **Expected result:** The row now shows Paid, with no manual editing by the user. Status tab counts update accordingly (Pending −1, Paid +1).

---

## US-FIN-02 — Issue a refund

### TC-FIN-04 — Admin issues a full refund of a Paid charge
- **Traces:** US-FIN-02  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in as Admin. A Paid payment exists with an admitted attendee/ticket and a mobile number on file. A completed invoice exists behind the payment.
- **Test data:** Paid charge ฿2,140.00, attendee has email + Thai mobile 08x-xxx-xxxx.
- **Steps:**
  1. Open the Paid payment and click Refund.
  2. Review the refund dialog (amount restated, finality warning).
  3. Confirm the refund.
  4. Check the payment status, the attendee's ticket/seat, the related invoice, and the current VAT period.
- **Expected result:** Dialog restates the exact amount (฿2,140.00) and warns the refund is final before confirm. After confirm: payment shows Refunded; the attendee's ticket is cancelled and its seat returned to availability; buyer receives a refund confirmation email plus SMS (mobile on file). The related invoice is marked Void and drops out of outstanding receivables; the VAT on that sale is backed out of the current tax period.

### TC-FIN-05 — Organizer cannot refund a paid payment
- **Traces:** US-FIN-02  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Logged in as Organizer (not Admin). A Paid payment exists.
- **Test data:** Paid charge ฿2,140.00.
- **Steps:**
  1. As Organizer, open a Paid payment.
  2. Look for the Refund action.
- **Expected result:** The Refund action is not available to the Organizer (hidden/disabled). No refund can be initiated.

### TC-FIN-06 — Double-submitted refund issues only one refund
- **Traces:** US-FIN-02  ·  **Priority:** High  ·  **Type:** Edge
- **Preconditions:** Logged in as Admin. A Paid payment exists.
- **Test data:** Paid charge ฿5,000.00.
- **Steps:**
  1. Open the refund dialog and confirm.
  2. Immediately trigger the same confirmation again (double click / retry the request).
  3. Inspect the payment and any refund records.
- **Expected result:** Only one refund is ever issued for the charge; the second submission is deduplicated (no double refund to the buyer). Payment ends in Refunded with a single refund reflected.

---

## US-FIN-03 — Track balances and payout history

### TC-FIN-07 — Three headline balances with pending equal to in-progress payout
- **Traces:** US-FIN-03  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in as Organizer. Bank account connected. Settled sales exist, past payouts exist, and exactly one payout is currently in progress. Payout history spans Paid/Processing/Scheduled/Failed.
- **Test data:** In-progress payout ฿48,000.00; masked account "•••• 6789".
- **Steps:**
  1. Open Payouts and read the three headline balances (available, pending, paid out to date) in Baht.
  2. Note the pending balance and its label.
  3. Filter payout history by status = Processing, then by Paid, Scheduled and Failed in turn.
- **Expected result:** Three balances display in ฿. Pending balance equals the in-progress payout amount (฿48,000.00) and is labelled as pending/in progress. Each status filter shows matching payouts with amount, masked destination bank account (last-4 only), and requested + completed dates.

### TC-FIN-08 — No bank connected: balances unavailable, prompted to connect
- **Traces:** US-FIN-03  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Logged in as Organizer/Admin with no bank/payout account connected.
- **Test data:** — (no account).
- **Steps:**
  1. Open Payouts.
- **Expected result:** The three balances read as unavailable (not ฿0 as if earned), and the user is prompted to connect payouts. No full bank/card number is shown anywhere.

---

## US-FIN-04 — Follow and recover a payout

### TC-FIN-09 — Payout timeline, in-progress note and Admin retry of a failed payout
- **Traces:** US-FIN-04  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Logged in as Admin. Payouts exist including one in progress, one failed, and one completed.
- **Test data:** Failed payout ฿48,000.00, masked "•••• 6789"; completed payout with reference "PO-2026-0031".
- **Steps:**
  1. Open the in-progress payout; read amount, status, masked destination, dates and the timeline/note.
  2. Open the failed payout and choose Retry.
  3. Open the completed payout and download its receipt.
- **Expected result:** Each payout shows amount, status, masked destination, dates and a plain-language timeline (requested → processing → paid); the in-progress note states funds typically arrive within 1–3 business days. After Admin retry is accepted, the same failed payout moves to Processing with no duplicate payout created, and the retry is recorded for audit. The completed payout's receipt downloads showing payout reference, amount and completion date (a transfer record, not a tax document).

### TC-FIN-10 — Organizer cannot retry a failed payout
- **Traces:** US-FIN-04  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Logged in as Organizer (not Admin). A failed payout exists.
- **Test data:** Failed payout ฿48,000.00.
- **Steps:**
  1. As Organizer, open the failed payout.
  2. Look for the Retry action.
- **Expected result:** The retry action is not available to the Organizer. The Organizer can view the payout details but cannot re-trigger settlement.

---

## US-FIN-05 — Manage bank and payout settings

### TC-FIN-11 — Admin manages payouts via secure provider / guided connect
- **Traces:** US-FIN-05  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in as Admin.
- **Test data:** Scenario A — account already connected; Scenario B — no account connected.
- **Steps:**
  1. (A) As Admin choose "Manage payouts"; observe where you are taken and what is available (bank details, payout schedule, tax forms/statements).
  2. (B) With no account connected, start the flow.
- **Expected result:** (A) Admin is taken securely to the payments provider to update bank details, set payout schedule, and access tax forms and statements — this data lives with the provider, not in Eventa. (B) With no account, the Admin is guided through connecting a payout account first before other management is possible.

### TC-FIN-12 — Organizer denied bank/payout management
- **Traces:** US-FIN-05  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Logged in as Organizer (not Admin).
- **Test data:** —
- **Steps:**
  1. As Organizer, open Payouts.
  2. Look for bank/payout/tax-form management controls.
- **Expected result:** Bank and payout management is not available to the Organizer (view-only where applicable, no manage entry point).

---

## US-FIN-06 — Invoice ledger with ageing

### TC-FIN-13 — Ageing: Overdue vs Issued based on the 14-day term (Bangkok time)
- **Traces:** US-FIN-06  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in as Organizer. Two unpaid invoices exist: one whose due date passed 3 days ago, one due in 9 days. "Today" evaluated on Asia/Bangkok time.
- **Test data:** Invoice A due date = today − 3 days; Invoice B due date = today + 9 days.
- **Steps:**
  1. Render the invoice ledger.
  2. Inspect Invoice A's status and ageing label.
  3. Inspect Invoice B's status and ageing label.
- **Expected result:** Invoice A shows Overdue with "3 days overdue" highlighted. Invoice B shows Issued with "in 9 days". Ageing is judged on Bangkok time against the fixed 14-day term.

### TC-FIN-14 — Filter/search invoices with whole-ledger tab counts; paid/void detail
- **Traces:** US-FIN-06  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Ledger has invoices across Paid, Issued, Overdue and Void, across multiple events.
- **Test data:** Invoice number "INV-2026-0142", buyer "Naruemon Co., Ltd.".
- **Steps:**
  1. Filter by status (e.g. Overdue), then by a specific event.
  2. Search by invoice number, then by buyer name.
  3. Read the status tab counts.
  4. Open a Paid invoice row, then a Void invoice row.
- **Expected result:** Only matching invoices show for each filter/search; tab counts reflect the whole ledger regardless of the active filter. The Paid invoice row shows how and when it was paid; the Void invoice shows "Voided".

---

## US-FIN-07 — Issue a VAT invoice

### TC-FIN-15 — Issue a compliant invoice with sequential number, 14-day term and 7% VAT
- **Traces:** US-FIN-07  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in as Organizer/Admin. A billable order with a valid buyer email exists. The last issued invoice number is known.
- **Test data:** Order total ฿48,000.00 (VAT-inclusive). Today = 2026-07-27 (Bangkok) → due 2026-08-10. Expected subtotal ฿44,859.81 + VAT ฿3,140.19 = ฿48,000.00.
- **Steps:**
  1. Issue an invoice for the ฿48,000 order today.
  2. Inspect the assigned invoice number against the sequence.
  3. Inspect the due date, subtotal, 7% VAT, total and status.
  4. Repeat for an already-paid order and check its status.
- **Expected result:** Invoice gets the next number in an unbroken sequence, due date 14 days out (2026-08-10), subtotal ฿44,859.81 and VAT ฿3,140.19 that add back exactly to ฿48,000.00, status Issued. For the already-paid order, the invoice appears Paid with how and when it was paid.

### TC-FIN-16 — Validation and duplicate-issue guard
- **Traces:** US-FIN-07  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Logged in as Organizer/Admin.
- **Test data:** Order 1 — buyer email missing. Order 2 — amount ฿0.00 / −฿100.00. Order 3 — valid order with issuing triggered twice.
- **Steps:**
  1. Attempt to issue an invoice for Order 1 (no buyer email).
  2. Attempt to issue for Order 2 (zero, then negative amount).
  3. Trigger issuing for Order 3 twice in quick succession.
- **Expected result:** Orders 1 and 2 are blocked — the user is asked to provide a valid buyer email and a valid (positive) amount, and no invoice is created (no sequence number consumed). For Order 3, only one invoice exists for the order despite the double trigger.

---

## US-FIN-08 — View and download a tax invoice

### TC-FIN-17 — Invoice detail reconciles to total; refunded payment's invoice is Void
- **Traces:** US-FIN-08  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in as Organizer/Admin. A ฿32,500 paid invoice exists; a completed payment with a behind-it invoice exists; a refunded payment with its invoice exists.
- **Test data:** Paid invoice ฿32,500.00 → subtotal ฿30,373.83 + VAT ฿2,126.17 = ฿32,500.00.
- **Steps:**
  1. Open the ฿32,500 invoice detail; check buyer, order reference, event line item, issued + due dates, subtotal, 7% VAT and payment info.
  2. From a completed payment, open the invoice behind it and check the breakdown.
  3. Open the invoice behind a refunded payment.
- **Expected result:** Detail shows buyer, order reference, event line item, issued and due dates, and subtotal ฿30,373.83 + VAT ฿2,126.17 reconciling to ฿32,500.00, plus how it was paid. The invoice reached from a completed payment shows the same reconciling breakdown; the refunded payment's invoice is marked Void.

### TC-FIN-18 — Download single-page A4 tax-invoice PDF with legal identity
- **Traces:** US-FIN-08  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in as Organizer/Admin. An invoice is open. A due-but-unpaid invoice also exists.
- **Test data:** Invoice for ฿32,500.00; a due-but-unpaid invoice due in 9 days and one 3 days overdue.
- **Steps:**
  1. From the invoice detail, download the PDF.
  2. Inspect the PDF layout and content.
  3. View the due-but-unpaid invoice and note its ageing label.
- **Expected result:** PDF is a single A4 page carrying Eventa's legal name, address and tax ID, the buyer, the event service line, and the reconciling 7% VAT breakdown. The due-but-unpaid invoice shows its due date with ageing (e.g. "in 9 days" or "3 days overdue").

---

## US-FIN-09 — Chase unpaid invoices

### TC-FIN-19 — Resend, automatic overdue reminders (EN/TH), stop-on-settle and flood limit
- **Traces:** US-FIN-09  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Logged in as Organizer/Admin. An invoice with a valid buyer email exists; an unpaid invoice reaching its due date exists; buyer language set to TH for a second buyer.
- **Test data:** Invoice "INV-2026-0142" ฿12,000.00; buyer language EN and (second) TH.
- **Steps:**
  1. Resend the invoice to its buyer; check number/amount/status afterwards.
  2. Advance an unpaid invoice to its due date and run the daily reminder; check the buyer message and invoice status.
  3. Mark an invoice Paid (and another Void); run the next reminder cycle.
  4. Resend the same invoice repeatedly beyond a sensible limit.
- **Expected result:** Resend delivers the invoice again with number, amount and status unchanged. On the due date the daily reminder sends a due/overdue reminder (rendered in the buyer's EN/TH language) and the invoice shows Overdue thereafter. Once an invoice is Paid or Void, no further reminders are sent. Repeated resends beyond the limit are held back so the buyer isn't flooded. Every send is recorded in the messaging delivery log.

---

## US-FIN-10 — Void an invoice

### TC-FIN-20 — Admin voids an Issued/Overdue invoice; sequence preserved; VAT reversed
- **Traces:** US-FIN-10  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in as Admin. An Issued (or Overdue) invoice exists that was raised in error.
- **Test data:** Invoice "INV-2026-0150" ฿12,000.00, optional reason "Duplicate order".
- **Steps:**
  1. Open the Issued/Overdue invoice and confirm the void (add optional reason).
  2. Check the invoice status, outstanding receivables, the current open tax period, and the audit record.
  3. Inspect the invoice number sequence and issue a correcting invoice.
- **Expected result:** Invoice becomes Void, drops out of outstanding receivables, its VAT is reversed from the current open tax period, and the action is recorded for audit. Its number is preserved and never reused; the correction is a brand-new invoice with the next sequential number.

### TC-FIN-21 — Void blocked for paid/already-void invoices and for Organizers
- **Traces:** US-FIN-10  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** A Paid invoice and an already-Void invoice exist. Two roles available: Admin and Organizer.
- **Test data:** Paid invoice ฿9,000.00; already-Void invoice.
- **Steps:**
  1. As Admin, attempt to void the Paid invoice.
  2. As Admin, attempt to void the already-Void invoice.
  3. As Organizer, attempt to void an Issued invoice.
- **Expected result:** Voiding the Paid invoice is refused with guidance that a paid invoice is only voided by refunding its payment. Voiding an already-Void invoice is refused (can't be voided). The Organizer's void attempt is denied (action not available).

---

## US-FIN-11 — Monthly VAT ledger

### TC-FIN-22 — VAT collected at 7% and headline reconciliation for a year
- **Traces:** US-FIN-11  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in as Organizer/Admin. VAT ledger has periods across the chosen year with taxable sales, remittances and withholding figures.
- **Test data:** A period with taxable sales ฿3,012,800 → VAT collected ฿210,896 (7%). Year = 2026.
- **Steps:**
  1. Open the VAT ledger and scope to the year.
  2. Inspect the VAT collected value on the ฿3,012,800 period row.
  3. Compare the headline figures (VAT collected, remitted, payable) to the sum of the rows.
- **Expected result:** VAT collected shows ฿210,896 for that period (7% of sales). Headline VAT payable equals VAT collected minus VAT remitted for the year, and the headlines never disagree with the rows beneath them.

### TC-FIN-23 — Filter by filing status; withholding tracked separately with due dates
- **Traces:** US-FIN-11  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Ledger has periods with statuses Filed, Due and Upcoming and with withholding-tax figures for the chosen year.
- **Test data:** June 2026 period due 2026-07-15; periods carrying withholding (PND) figures.
- **Steps:**
  1. Filter by filing status = Filed, then Due, then Upcoming.
  2. Check each period's due date (the 15th of the following month) and status.
  3. Read the withholding headline for the year.
- **Expected result:** Each filter shows only matching periods, each with its due date on the 15th of the following month and its status. The withholding headline sums the periods' withholding figures and is tracked separately from VAT payable.

---

## US-FIN-12 — File the VAT return each period

### TC-FIN-24 — Admin records a PP30 filing on a Due period
- **Traces:** US-FIN-12  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in as Admin. A Due period with computed VAT exists.
- **Test data:** Due period with VAT ฿210,896.00.
- **Steps:**
  1. Open the Due period and record its filing (PP30) on or before its due date.
  2. Check the period status, remitted amount and VAT payable.
- **Expected result:** Period becomes Filed, its remitted amount shows ฿210,896.00, and VAT payable drops by that amount. The PP30 filing is recorded.

### TC-FIN-25 — Filing guards: not-yet-due, late flag, post-filing adjustment, Organizer denied
- **Traces:** US-FIN-12  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Roles available: Admin and Organizer. An Upcoming (not-yet-Due) period, a Due period filed after its due date, and an already-Filed period that is later hit by a refund/void.
- **Test data:** Upcoming period; a Due period recorded on 2026-07-20 with due date 2026-07-15; a Filed period affected by a later refund.
- **Steps:**
  1. As Admin, attempt to mark the Upcoming (not-yet-Due) period as filed.
  2. As Admin, record a filing after the due date; check how it's recorded.
  3. Trigger a refund/void that affects an already-Filed period; check where the adjustment lands.
  4. As Organizer, attempt to record a filing.
- **Expected result:** (1) Blocked — only a Due period can be marked filed. (2) The filing is saved but flagged as late for the record. (3) The adjustment carries into the next open period rather than changing the filed return. (4) The Organizer's attempt is denied.

---

## US-FIN-13 — Export finance ledgers

### TC-FIN-26 — Export respects on-screen filters; Thai names intact; empty-set guard
- **Traces:** US-FIN-13  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Logged in as Organizer/Admin. Payments ledger filterable to Refunded (Should); invoice ledger to Overdue and VAT ledger by year/status (Could). At least one record contains Thai text. An empty filtered set is reproducible.
- **Test data:** Refunded payment "Somchai Jaidee" ฿2,140.00; Overdue invoices; VAT periods for 2026; a filter combination yielding zero rows.
- **Steps:**
  1. Filter payments to Refunded and export; open the file.
  2. Filter invoices to Overdue and export; check columns.
  3. Scope the VAT ledger to a year + status and export.
  4. Open an exported file containing Thai names in a spreadsheet.
  5. Apply a filter that yields zero rows and attempt to export.
- **Expected result:** The Refunded export contains exactly those refunded payments with a VAT-inclusive amount. The Overdue invoice export contains only overdue invoices with reconciling subtotal and VAT columns. The VAT export lists the scoped periods with reconciling VAT and remitted figures. Thai names render correctly in the spreadsheet (no mojibake). Exporting an empty filtered set tells the user there's nothing to export (no empty/broken file produced).

---

## US-FIN-14 — Keep finance to finance roles

### TC-FIN-27 — Finance-area role visibility and action matrix
- **Traces:** US-FIN-14  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Test accounts for Staff, Attendee, Organizer and Admin in the same workspace.
- **Test data:** One user per role.
- **Steps:**
  1. Sign in as Staff, then as Attendee; look for the Finance area in navigation and attempt to reach it directly.
  2. Sign in as Organizer; confirm which finance actions are available.
  3. Sign in as Admin; confirm which finance actions are available.
- **Expected result:** For Staff and Attendee, the Finance area is not visible or reachable. The Organizer can view, invoice and export, but cannot issue refunds, move payout money, void invoices or record VAT filings. The Admin has all finance actions available.


---

<a id="tc-e10"></a>

# Build the Event Program — Test Cases

Epic E10 — Build the Event Program. Area code: PROG. Locale context: Asia/Bangkok timezone, EN/TH bilingual content.

---

## US-PROG-01 — See the event's schedule at a glance

### TC-PROG-01 — Agenda shows only the selected event's sessions in correct slots
- **Traces:** US-PROG-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Organizer signed in. Event "Bangkok Tech Summit" has sessions across 3 days; a second event "HR Forum" also has sessions.
- **Test data:** Session "Keynote: AI in ASEAN", Day 1 09:00–10:00, type Keynote (blue); Session "Panel: FinTech", Day 2 14:00–15:00, type Panel (green); speakers assigned to each.
- **Steps:**
  1. Select "Bangkok Tech Summit" and open its Agenda (week view).
  2. Inspect each session block's day/time placement.
  3. Read the block contents (title, type colour, speaker).
- **Expected result:** Every session appears in its correct day/time position; each block shows title, type colour and speaker name; no session from "HR Forum" is shown. (AC: correct positions, block detail, event isolation.)

### TC-PROG-02 — Empty agenda shows a friendly prompt and event switch reloads correctly
- **Traces:** US-PROG-01  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Organizer signed in. Event "New Meetup 2026" has zero sessions; another event has sessions.
- **Test data:** —
- **Steps:**
  1. Select "New Meetup 2026" and open its Agenda.
  2. Observe the calendar body.
  3. Switch to an event that has sessions and let the agenda reload.
- **Expected result:** For the empty event a friendly "no agenda yet" prompt inviting the first session is shown (not an error); after switching, only the newly selected event's sessions display. (AC: empty-state prompt, reload isolation.)

---

## US-PROG-02 — Add a session to the schedule

### TC-PROG-03 — Add a valid session and speaker count increments
- **Traces:** US-PROG-02  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Organizer signed in on a chosen event. Speaker "Somsak P." exists on that event with current session count = 2. Room "Hall A" is free at the target slot.
- **Test data:** Title "Cloud Native Workshop"; Day 2; start 13:00, end 14:30 (Asia/Bangkok); type Workshop; room Hall A; speaker Somsak P.; description optional.
- **Steps:**
  1. Open the add-session form for the chosen event.
  2. Fill all fields with the test data and save.
  3. Observe the calendar and then open speaker "Somsak P.".
- **Expected result:** Session appears immediately at Day 2 13:00–14:30, coloured by Workshop type, showing Somsak P.; the speaker's session count reads 3. (AC: immediate placement + colour + speaker, count +1.)

### TC-PROG-04 — Reject blank title and invalid time ranges
- **Traces:** US-PROG-02  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Organizer signed in on a chosen event whose schedulable hours are 08:00–18:00.
- **Test data:** (a) Title blank, valid time; (b) Title "Late Session", start 16:00, end 15:00; (c) Title "Off-hours", start 19:00, end 20:00 (outside 08:00–18:00).
- **Steps:**
  1. Attempt to save case (a) with the title left blank.
  2. Attempt to save case (b) with end time not after start.
  3. Attempt to save case (c) with a time outside the day's schedulable hours.
- **Expected result:** (a) Prompted to enter a session title, nothing saved; (b) told the time is invalid, nothing saved; (c) told the time is invalid / outside schedulable hours, nothing saved. No block appears on the calendar in any case. (AC: title required, end-after-start, within schedulable hours.)

---

## US-PROG-03 — Update a session

### TC-PROG-05 — Move a session to a new slot with no clash
- **Traces:** US-PROG-03  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Organizer signed in. Session "Design Sprint" exists Day 1 10:00–11:00 in Room B; target slot Day 1 15:00–16:00 in Room C is free of clashes.
- **Test data:** New day Day 1, new time 15:00–16:00, new room Room C; type changed Panel → Workshop.
- **Steps:**
  1. Open "Design Sprint".
  2. Change day/time/room to the target and change type to Workshop.
  3. Save.
- **Expected result:** The block relocates to Day 1 15:00–16:00 in Room C, its colour updates to the Workshop type, and the change is recorded. (AC: relocate on no-clash save, colour updates with type.)

### TC-PROG-06 — Concurrent edit is blocked with reload prompt (optional attendee notify)
- **Traces:** US-PROG-03  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Live event. Session "Opening Keynote" open in two browser sessions by two organizers. Session has attendees who added it to their schedule.
- **Test data:** Colleague B changes the room and saves first; Organizer A then changes the start time (a material change) and saves.
- **Steps:**
  1. Both users open "Opening Keynote".
  2. Colleague B changes the room and saves successfully.
  3. Organizer A, working on the stale copy, changes the start time and tries to save.
  4. (Follow-up) After reloading, A re-applies the material change and saves, observing the notify option.
- **Expected result:** A's save is refused with a message that the session changed and a prompt to reload — B's change is not silently overwritten. On the subsequent valid material change (day/time/room) to the live event, A is optionally offered to notify attendees who bookmarked it, with no message sent by default. (AC: optimistic-lock reload, optional notify defaulting to off.)

---

## US-PROG-04 — Remove a session

### TC-PROG-07 — Confirmed removal drops speaker count and clears public agenda
- **Traces:** US-PROG-04  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Live event. Session "Startup Pitches" has speaker "Anong K." (session count = 3) and is visible on the public agenda.
- **Test data:** —
- **Steps:**
  1. Choose to remove "Startup Pitches".
  2. Read the confirmation dialog, then confirm.
  3. Check speaker "Anong K." and the public event agenda.
- **Expected result:** Confirmation warns that removal won't auto-notify bookmarking attendees and removes only on explicit confirm; after confirm the session is gone, Anong K. remains in the directory with count 2, and the session no longer appears on the public agenda. (AC: warning + explicit confirm, speaker retained with count −1, removed from public agenda.)

### TC-PROG-08 — Cancelling the confirmation changes nothing
- **Traces:** US-PROG-04  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Session "Networking Break" exists with an assigned speaker.
- **Test data:** —
- **Steps:**
  1. Choose to remove "Networking Break".
  2. In the confirmation dialog, cancel/dismiss.
  3. Return to the agenda and check the speaker's session count.
- **Expected result:** The session still exists in its original slot and the speaker's session count is unchanged. (AC: cancel leaves everything unchanged.)

---

## US-PROG-05 — Keep the schedule physically feasible

### TC-PROG-09 — Room double-booking is blocked with a clear message
- **Traces:** US-PROG-05  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Organizer signed in. Room "Hall A" already booked Day 1 09:00–10:30.
- **Test data:** New session "Second Talk", Day 1 10:00–11:00, room Hall A (overlaps existing 09:00–10:30).
- **Steps:**
  1. Add a new session in Hall A overlapping the existing booking.
  2. Save.
- **Expected result:** Save is blocked with a message naming the conflicting room (Hall A) and the clashing time; the session is not created. (AC: block on room overlap, message identifies room + time.)

### TC-PROG-10 — Speaker overlap warns and needs confirm; parallel tracks allowed; no self-clash
- **Traces:** US-PROG-05  ·  **Priority:** High  ·  **Type:** Edge
- **Preconditions:** Speaker "Ploy S." already assigned to a session Day 1 11:00–12:00 in Room A. Room C is free 11:00–12:00.
- **Test data:** (a) New session Day 1 11:30–12:30 in Room B with speaker Ploy S. (overlaps her existing session); (b) New session Day 1 11:00–12:00 in Room C with a different speaker "Wichai T." (parallel track); (c) Open the existing 11:00–12:00 session and re-save without changes.
- **Steps:**
  1. Create case (a) and save.
  2. On the "assign anyway?" warning, confirm.
  3. Create case (b) and save.
  4. Edit case (c) and save with the same time/room/speaker.
- **Expected result:** (a) A speaker double-booking warning appears and the session saves only after confirming "assign anyway?"; (b) both parallel sessions in different rooms with different speakers save without warning; (c) the session is not flagged as clashing with itself. (AC: speaker overlap warn+confirm, parallel tracks allowed, no self-clash.)

---

## US-PROG-06 — Find sessions in a busy programme

### TC-PROG-11 — Keyword search filters by title/type (EN & TH) and clears cleanly
- **Traces:** US-PROG-06  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Selected event has sessions including "Keynote: AI", a Thai-titled session "เวิร์กช็อปการตลาด" (Marketing Workshop), and several Panels.
- **Test data:** Search terms: "keynote"; "เวิร์กช็อป"; "zznomatch".
- **Steps:**
  1. Type "keynote" in the agenda search box.
  2. Clear the box, then type the Thai term "เวิร์กช็อป".
  3. Clear, then type "zznomatch".
- **Expected result:** "keynote" leaves only sessions matching by title or type visible; clearing restores all; the Thai term matches the Thai-titled session (EN/TH supported); "zznomatch" shows the calendar layout with nothing highlighted — not an error. Search stays scoped to the selected event. (AC: title/type keyword filter, clear restores, no-match not an error, EN/TH.)

---

## US-PROG-07 — Switch between week and month views

### TC-PROG-12 — Toggle week/month and navigate weeks stay on the same event
- **Traces:** US-PROG-07  ·  **Priority:** Low  ·  **Type:** Functional
- **Preconditions:** Selected multi-week event with sessions in one week only.
- **Test data:** —
- **Steps:**
  1. From week view, switch to month view and observe the day-by-day overview; drill back into a day to return to week view.
  2. From week view, navigate to the previous and next week and read the date/month labels.
  3. Navigate to a week beyond the event's scheduled weeks.
- **Expected result:** Month view shows a day-by-day overview drillable back to the time-slot week layout; navigating weeks shifts dates and the month label while staying on the same event; a week beyond the schedule shows an empty week with correct dates and never sessions from another event. (AC: week/month toggle, week navigation labels, empty out-of-range week.)

---

## US-PROG-08 — Browse and filter the speaker directory

### TC-PROG-13 — Directory shows key details with card/list layouts and event filter
- **Traces:** US-PROG-08  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Organizer signed in. Multiple speakers across two events; some rated, some unrated.
- **Test data:** Filter by event "Bangkok Tech Summit".
- **Steps:**
  1. Open the speaker directory.
  2. Inspect a speaker entry for name, role, contact, session count and rating.
  3. Toggle between card and list layout.
  4. Apply the event filter "Bangkok Tech Summit".
- **Expected result:** Each speaker shows name, role, contact, session count and rating; card/list layouts both render; after filtering, only "Bangkok Tech Summit" speakers appear and the displayed count reflects the filtered total. (AC: key details visible, layout switch, event filter + count.)

### TC-PROG-14 — Search by name/role/email; no-match shows prompt not error
- **Traces:** US-PROG-08  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Directory has speakers spanning multiple pages, including "Napat Chai" (role Speaker, email napat@example.co.th).
- **Test data:** Search terms: "napat"; "nonexistent@nowhere".
- **Steps:**
  1. Navigate to page 2 of the directory.
  2. Search "napat".
  3. Clear and search "nonexistent@nowhere".
- **Expected result:** Searching "napat" shows only matching speakers and the list returns to the first page; the no-match search shows a "no speakers found" prompt, not an error. (AC: name/role/email search, reset to first page, no-match prompt.)

---

## US-PROG-09 — Add a speaker to the line-up

### TC-PROG-15 — Add a valid speaker starting at zero sessions and unrated
- **Traces:** US-PROG-09  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Organizer signed in with a chosen event selected.
- **Test data:** Photo valid JPG within size limit; name "Kanya Wong"; role "CTO, ThaiCloud"; email kanya@thaicloud.co.th; bio short text; topic "Serverless"; event = chosen event; website https://thaicloud.co.th; social https://linkedin.com/in/kanyawong.
- **Steps:**
  1. Open the add-speaker form.
  2. Fill all fields with valid data and save.
  3. Open the new speaker's card.
- **Expected result:** A speaker profile is created and assigned to the chosen event, session count = 0, and shown with an unrated indicator (not a zero score). (AC: profile created + assigned, zero sessions, unrated until feedback.)

### TC-PROG-16 — Reject blank/invalid/duplicate email, oversize photo and malformed links
- **Traces:** US-PROG-09  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** A speaker with email taken@example.com already exists.
- **Test data:** (a) Name blank; (b) email "not-an-email"; (c) email "taken@example.com" (duplicate); (d) photo a 20 MB .bmp (wrong type / too large); (e) website "htp:/broken link".
- **Steps:**
  1. Attempt to save with the name blank (a).
  2. Attempt to save with an invalid email (b).
  3. Attempt to save with the duplicate email (c).
  4. Attempt to save with the oversize/wrong-type photo (d).
  5. Attempt to save with the malformed website link (e).
- **Expected result:** (a)/(b) asked to correct the name/email, nothing saved; (c) told a speaker with that email already exists, nothing saved; (d) told the accepted format and size limit, photo not accepted; (e) asked to enter a valid link rather than have it silently dropped. No speaker is created in any case. (AC: required name+email, valid email, duplicate email, photo type/size, valid links.)

---

## US-PROG-10 — Keep a speaker's profile up to date

### TC-PROG-17 — Edit a speaker's role and see it reflected on the live public page
- **Traces:** US-PROG-10  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Speaker "Anon Ratana" (role "Engineer, OldCo") appears on a live event's public speakers section.
- **Test data:** New role "VP Engineering, NewCo".
- **Steps:**
  1. Open speaker "Anon Ratana" and change the role/company.
  2. Save.
  3. View the live event's public speakers section.
- **Expected result:** The card shows the new role, the change is recorded, and the public speakers section reflects the update. (AC: card updates + recorded, public page reflects change.)

### TC-PROG-18 — Duplicate email and concurrent edit are both refused
- **Traces:** US-PROG-10  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Speaker A being edited; speaker B already uses email dup@example.com. Speaker A is also open in a second organizer's session.
- **Test data:** (a) Change A's email to dup@example.com; (b) Colleague changes A and saves first, then this organizer saves the stale copy.
- **Steps:**
  1. Change speaker A's email to dup@example.com and save.
  2. Separately, have a colleague edit and save speaker A first, then attempt to save this stale copy.
- **Expected result:** (a) The duplicate-email message appears and nothing is saved; (b) told the speaker changed and asked to reload, so no work is silently lost. (AC: duplicate-email guard, optimistic-lock reload.)

---

## US-PROG-11 — Remove a speaker

### TC-PROG-19 — Confirmed deletion keeps sessions and clears the live public page
- **Traces:** US-PROG-11  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Speaker "Preeya M." assigned to 3 sessions and shown on a live event's public agenda/speakers section.
- **Test data:** —
- **Steps:**
  1. Choose to delete "Preeya M.".
  2. Read the confirmation warning, then confirm.
  3. Check the 3 sessions and the public event pages.
- **Expected result:** Confirmation warns the speaker will be removed from all their sessions and the action cannot be undone; after confirm the speaker disappears from the directory, the 3 sessions remain but no longer list Preeya M., and she no longer appears in the public agenda or speakers section. (AC: sessions retained without speaker, irreversible warning, removed from public.)

### TC-PROG-20 — Cancelling deletion leaves speaker and assignments intact
- **Traces:** US-PROG-11  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Speaker "Preeya M." assigned to 3 sessions.
- **Test data:** —
- **Steps:**
  1. Choose to delete "Preeya M.".
  2. Cancel the confirmation dialog.
  3. Re-check the speaker and her session assignments.
- **Expected result:** The speaker and all 3 assignments are unchanged. (AC: cancel leaves speaker and assignments intact.)

---

## US-PROG-12 — Assign speakers to their sessions and balance the line-up

### TC-PROG-21 — Assigning a second session updates count and shows initials
- **Traces:** US-PROG-12  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Speaker "Ratcha B." (initials RB) currently carries 1 session on the selected event; a second non-overlapping session exists.
- **Test data:** Assign Ratcha B. to the second session.
- **Steps:**
  1. Open the second session and assign speaker Ratcha B. (add to existing speakers).
  2. Save.
  3. Check the speaker's session count and the session's displayed speaker.
- **Expected result:** Assignment is saved, the speaker's session count reads two, the session shows the speaker (initials RB), and the session's displayed speaker updates. (AC: count reads two + initials, assignments saved + displayed speaker updates.)

### TC-PROG-22 — Cross-event assignment refused; overlapping assignment warns
- **Traces:** US-PROG-12  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Speaker "Guest X" belongs to a different event. On the current event, speaker "Malee T." already has a session Day 1 10:00–11:00.
- **Test data:** (a) Assign Guest X to a session on the current event; (b) assign Malee T. to a session Day 1 10:30–11:30 (overlaps her existing).
- **Steps:**
  1. Attempt to assign Guest X to a current-event session and save.
  2. Assign Malee T. to the overlapping session and save.
- **Expected result:** (a) Told Guest X belongs to another event and the assignment is refused; (b) the double-booking warning appears and the save proceeds only after confirming. (AC: cross-event assignment refused, overlap warn + confirm.)

---

## US-PROG-13 — See how speakers were rated

### TC-PROG-23 — Rated speaker shows aggregate score with reviews and reflects new feedback
- **Traces:** US-PROG-13  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Admin signed in. Speaker "Chai N." has post-session feedback averaging 4.9.
- **Test data:** —
- **Steps:**
  1. View Chai N.'s card and read the rating.
  2. Open the speaker's reviews.
  3. After new feedback arrives, re-open the speaker's card.
- **Expected result:** The card shows "4.9"; the reviews show the underlying feedback that produced the score; after new feedback the rating reflects the latest results. Rating is read-only (not hand-editable). (AC: aggregate score display, underlying reviews, latest feedback reflected.)

### TC-PROG-24 — Brand-new speaker shows an unrated indicator, not zero
- **Traces:** US-PROG-13  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Admin signed in. Speaker "Fern S." has no feedback yet.
- **Test data:** —
- **Steps:**
  1. View Fern S.'s card.
  2. Inspect the rating area.
- **Expected result:** The card shows an unrated indicator, not a zero score. (AC: no feedback shows unrated, not 0.)


---

<a id="tc-e11"></a>

# Organizer Home & Dashboard — Test Cases

Area: DASH · Epic E11 — Organizer Home & Dashboard
Locale defaults: Currency ฿ (THB), VAT 7% tracked separately, timezone Asia/Bangkok, languages EN/TH.

---

## US-DASH-01 — Daily operations home

### TC-DASH-01 — Home greets organizer by name in chosen language, using Bangkok time
- **Traces:** US-DASH-01 · **Priority:** High · **Type:** Functional
- **Preconditions:** Organizer account "Somchai" exists; UI language set to Thai; device locale/timezone set to a non-Bangkok zone (e.g. Europe/Berlin).
- **Test data:** Force Bangkok wall-clock to 08:15 (morning) while the device clock reads 03:15 CEST.
- **Steps:**
  1. Sign in as the organizer.
  2. Land on the home screen.
  3. Read the greeting text.
- **Expected result:** A time-of-day greeting for MORNING is shown, addressed to the organizer by name, rendered in Thai. The greeting reflects Bangkok time (morning) even though the device is in an earlier timezone. Home only summarizes/links — no record is modified.

### TC-DASH-02 — "New event" launches the event creation flow
- **Traces:** US-DASH-01 · **Priority:** High · **Type:** Functional
- **Preconditions:** Signed in as organizer on the home screen.
- **Test data:** —
- **Steps:**
  1. Choose "New event".
- **Expected result:** The event creation flow opens. No event record is created merely by opening it (home never changes records itself).

### TC-DASH-03 — Attendee is refused access to the admin home
- **Traces:** US-DASH-01 · **Priority:** High · **Type:** Negative
- **Preconditions:** A pure Attendee account (no team-member role) is signed in.
- **Test data:** Direct navigation to the admin home URL.
- **Steps:**
  1. As the Attendee, attempt to open the organizer/admin home.
- **Expected result:** Access is refused (not authorized / redirected away). No organizer home data, greeting, or panels are rendered.

---

## US-DASH-02 — Today's registrations at a glance

### TC-DASH-04 — Today's sign-up count and newest previews shown for team member with registration access
- **Traces:** US-DASH-02 · **Priority:** High · **Type:** Functional
- **Preconditions:** Signed in as a team member permitted to view registrations. Several registrations exist for events, some created today (Bangkok calendar day), some earlier.
- **Test data:** 3 registrations today at 09:05, 11:40, 14:20 (Bangkok); 1 registration from yesterday 23:50 Bangkok that must NOT count as today.
- **Steps:**
  1. Open home and read the registrations panel count badge.
  2. Inspect the previewed newest sign-ups.
  3. Choose "View all registrations".
- **Expected result:** Count badge shows 3 (today only, Bangkok calendar day; yesterday's late entry excluded). Newest sign-ups preview each show attendee name, their event + ticket type, and the time (Bangkok, in the UI language). "View all registrations" navigates to the registrations list.

### TC-DASH-05 — Empty state when no one has registered today
- **Traces:** US-DASH-02 · **Priority:** Medium · **Type:** Edge
- **Preconditions:** Signed in as a team member with registration access. No registrations exist with a Bangkok "today" timestamp (older registrations may exist).
- **Test data:** Last registration was yesterday 22:00 Bangkok.
- **Steps:**
  1. Open home and view the today's-registrations panel.
- **Expected result:** Panel clearly shows "no registrations yet today" (not a zero-count list of previews and not stale entries).

---

## US-DASH-03 — Today's meetings

### TC-DASH-06 — Today's meetings previewed earliest-first with empty-state and schedule shortcut verified
- **Traces:** US-DASH-03 · **Priority:** Medium · **Type:** Functional
- **Preconditions:** Signed in as organizer.
- **Test data:** Two meetings scheduled today (Bangkok): "Sponsor sync" 15:00–15:30 with counterpart "Nok"; "Vendor call" 10:00–10:45 with counterpart "Arun". No meetings scheduled after these for today.
- **Steps:**
  1. Open home and read the meetings count badge and preview order.
  2. Verify each preview shows title, time range, and counterpart.
  3. Choose "Schedule meeting".
- **Expected result:** Count badge shows 2; meetings are previewed earliest-first ("Vendor call" 10:00–10:45 before "Sponsor sync" 15:00–15:30), each with title, time range (Bangkok), and counterpart. "Schedule meeting" opens the meeting creation flow. (If no meetings existed today, the panel would instead show "no meetings scheduled today".)

---

## US-DASH-04 — Upcoming events progress

### TC-DASH-07 — Upcoming event card shows days-remaining chip, attendee preview, and capacity progress
- **Traces:** US-DASH-04 · **Priority:** Medium · **Type:** Functional
- **Preconditions:** Signed in as organizer. Multiple upcoming events exist. Today's Bangkok date is 2026-07-27.
- **Test data:** Event A starts 2026-08-01 (5 days out), capacity 100, sold 83; Event B starts 2026-08-10; some attendees registered on Event A.
- **Steps:**
  1. Open home and view the upcoming-events panel.
  2. Read Event A's days-remaining chip, attendee preview, and progress bar.
  3. Confirm ordering of cards.
- **Expected result:** Event A card shows a days-remaining chip ("5 days" / Thai equivalent), a preview of who's attending, and a progress bar reading 83% (83 of 100 seats). Soonest-starting event (A) appears before B; only a small number of cards shown, with a link to the full events list.

### TC-DASH-08 — Event starting today shows "Today" chip; empty state when no upcoming events
- **Traces:** US-DASH-04 · **Priority:** Medium · **Type:** Edge
- **Preconditions:** Signed in as organizer. Bangkok "today" is well defined.
- **Test data:** Case A: Event C starts today (Bangkok). Case B: no upcoming events at all.
- **Steps:**
  1. Case A: open home, read Event C's chip.
  2. Case B: with no upcoming events, open home and view the panel.
- **Expected result:** Case A — Event C's chip reads "Today" (localized). Case B — panel shows "no upcoming events".

---

## US-DASH-05 — Active-events activity share

### TC-DASH-09 — Activity-share ring shows per-event share, legend, and total; empty state
- **Traces:** US-DASH-05 · **Priority:** Low · **Type:** Functional
- **Preconditions:** Signed in as organizer.
- **Test data:** Case A: 3 active events with differing sign-up activity. Case B: 0 active events.
- **Steps:**
  1. Case A: open home; inspect the ring, its color-keyed legend of event names, and the center number.
  2. Choose "See all".
  3. Case B: with no active events, view the panel.
- **Expected result:** Case A — ring shows each event's share of activity, a color-keyed legend of event names, and the total count of active events (3) in the center; "See all" navigates to the events list. Case B — panel shows "no active events".

---

## US-DASH-06 — Operational alerts

### TC-DASH-10 — Alerts listed most-urgent-first, each linking to its module; resolved alert clears on refresh
- **Traces:** US-DASH-06 · **Priority:** High · **Type:** Functional
- **Preconditions:** Signed in as organizer with full access. Multiple unresolved actionable issues exist (e.g. a pending approval, an unconfirmed speaker, a pending reply).
- **Test data:** At least three alerts of differing severity.
- **Steps:**
  1. Open home and read the alerts panel: severity indicators, text, and ordering.
  2. Click one alert and confirm it links to the correct resolving module.
  3. Resolve that issue in its owning module, return to home, and refresh.
- **Expected result:** Each alert shows a clear severity indicator + text, most urgent first, with a link to the right module. Resolving is done only in the module (never on home). After the issue is resolved, the alert is gone on the next home refresh.

### TC-DASH-11 — Finance alert hidden from user without finance access; all-caught-up empty state
- **Traces:** US-DASH-06 · **Priority:** High · **Type:** Negative
- **Preconditions:** A declined-payment (finance) alert exists in the workspace.
- **Test data:** Case A: signed in as a staff member WITHOUT finance access. Case B: a user with no outstanding actionable items.
- **Steps:**
  1. Case A: open home; scan the alerts panel for the declined-payment (finance) alert.
  2. Case B: with nothing outstanding, view the alerts panel.
- **Expected result:** Case A — the finance/declined-payment alert is NOT shown to the user without finance access (non-finance alerts still show normally). Case B — panel shows "you're all caught up".

---

## US-DASH-07 — Website template shortcuts

### TC-DASH-12 — Template shortcut opens landing-pages area; shortcuts reflect current template set on reload
- **Traces:** US-DASH-07 · **Priority:** Low · **Type:** Functional
- **Preconditions:** Signed in as organizer; templates panel is shown with the current set of landing-page templates.
- **Test data:** Available templates set changes (one added/removed) between the two loads.
- **Steps:**
  1. Choose a template shortcut from the panel.
  2. Change the available template set, then reload home.
  3. Re-inspect the templates panel.
- **Expected result:** Choosing a template navigates to the landing-pages area to work with it. After reload, the shortcuts reflect the current set (added template appears, removed template disappears).

---

## US-DASH-08 — Performance KPIs at a glance

### TC-DASH-13 — KPI cards render five headline metrics with period-over-period change
- **Traces:** US-DASH-08 · **Priority:** High · **Type:** Functional
- **Preconditions:** Signed in as Admin with finance access. Data exists for the current and previous comparable periods.
- **Test data:** Registrations up vs prior period; ticket revenue e.g. ฿128,500 (net of VAT and refunds); some upcoming events; check-in rate lower than prior period; capacity filled with a value.
- **Steps:**
  1. Open the dashboard and wait for KPI cards to load.
  2. Read each card: total registrations, ticket revenue, upcoming events, check-in rate, capacity filled.
  3. Read the change indicator on each.
- **Expected result:** All five KPI cards render. Ticket revenue is shown in ฿, net of VAT and refunds (7% VAT not surfaced). Registrations show an upward, positive-colored change; check-in rate (worsened) shows a downward, warning-colored change. Each card shows change versus the previous comparable period.

### TC-DASH-14 — Ticket revenue KPI withheld from user without finance access
- **Traces:** US-DASH-08 · **Priority:** High · **Type:** Negative
- **Preconditions:** Signed in as a team member WITHOUT finance access.
- **Test data:** Revenue data exists for the period.
- **Steps:**
  1. Open the dashboard and inspect the KPI cards.
- **Expected result:** The ticket-revenue figure is not disclosed (hidden entirely — never shown as ฿0 or a placeholder). The other four KPI cards render normally.

### TC-DASH-15 — Metric with no data shows neutral empty state, not a misleading value
- **Traces:** US-DASH-08 · **Priority:** Medium · **Type:** Edge
- **Preconditions:** Signed in as Admin. At least one KPI (e.g. check-in rate) has no data for the selected period (no events required check-in yet).
- **Test data:** Period with zero check-in-eligible activity.
- **Steps:**
  1. Open the dashboard and read the affected KPI card.
- **Expected result:** The card shows a neutral, empty state rather than a misleading value (e.g. not a fabricated 0% or 100%, and not a false change arrow).

---

## US-DASH-09 — Revenue trend with time-range toggle

### TC-DASH-16 — Revenue chart defaults to yearly and toggles Week/Month without page reload
- **Traces:** US-DASH-09 · **Priority:** High · **Type:** Functional
- **Preconditions:** Signed in as organizer with finance access. Revenue exists across weeks, months, and the year.
- **Test data:** Yearly total e.g. ฿1,240,000 with change vs previous year; distinct Week and Month totals.
- **Steps:**
  1. Open the dashboard and observe the default revenue view.
  2. Note the period total and change indicator.
  3. Switch to Month, then to Week.
- **Expected result:** Revenue view defaults to Yearly, showing the period total (฿) and its change vs the previous period. Switching to Month then Week updates the chart, headline total, and change to that range without a full page reload. The yearly total agrees with the ticket-revenue KPI for the same scope/period.

### TC-DASH-17 — Revenue section entirely hidden without finance access; empty period shows zero/neutral
- **Traces:** US-DASH-09 · **Priority:** High · **Type:** Negative
- **Preconditions:** Case A: signed in WITHOUT finance access. Case B: signed in WITH finance access but the selected period has zero revenue.
- **Test data:** Case B: a Week with no ticket sales.
- **Steps:**
  1. Case A: open the dashboard and look for the revenue section.
  2. Case B: select the empty Week and view the chart.
- **Expected result:** Case A — the revenue section is not shown at all (not zeroed, not a locked placeholder). Case B — the chart shows an empty (zero) result with a neutral change.

---

## US-DASH-10 — Registrations by ticket type

### TC-DASH-18 — Ticket-type breakdown shows shares, counts, percentages, and center total, largest-first; reconciles with KPI
- **Traces:** US-DASH-10 · **Priority:** Medium · **Type:** Functional
- **Preconditions:** Signed in as organizer. Registrations exist across multiple ticket types for the current scope.
- **Test data:** Total 200 registrations — General 120 (60%), VIP 50 (25%), Early Bird 30 (15%).
- **Steps:**
  1. Open the dashboard and view the ticket-type breakdown.
  2. Check each segment's count and percentage and the center total.
  3. Compare the center total against the total-registrations KPI for the same scope.
- **Expected result:** Breakdown lists each ticket type's share with count and percentage, largest share first (General → VIP → Early Bird), and shows 200 in the center. The center total equals the total-registrations KPI for the same scope. (If the mix later changes and the dashboard refreshes, the breakdown updates to match.)

### TC-DASH-19 — Empty state when there are no registrations
- **Traces:** US-DASH-10 · **Priority:** Medium · **Type:** Edge
- **Preconditions:** Signed in as organizer for a scope with zero registrations.
- **Test data:** New workspace / event with no sign-ups.
- **Steps:**
  1. Open the dashboard and view the ticket-type panel.
- **Expected result:** Panel shows "no registrations yet" (no empty ring with misleading segments).

---

## US-DASH-11 — Tickets selling fast

### TC-DASH-20 — Low-inventory ticket types listed scarcest-first with urgency coloring and Manage link
- **Traces:** US-DASH-11 · **Priority:** Medium · **Type:** Functional
- **Preconditions:** Signed in as organizer. Several ticket types are low on remaining inventory.
- **Test data:** VIP 2 left (critical), General 6 left (warning), Workshop 15 left (normal), each with its event context.
- **Steps:**
  1. Open the dashboard and view the selling-fast panel.
  2. Check ordering, event context, and the color of each remaining count.
  3. Choose "Manage" on a low ticket type.
- **Expected result:** Ticket types are listed scarcest-first (VIP 2 → General 6 → Workshop 15) with event context and a remaining count colored by urgency; the critically low VIP (2 left) is highlighted in the most urgent color. "Manage" navigates to ticket management for that inventory.

### TC-DASH-21 — 8-seat booking boundary flags possible single-booking sell-out; empty state
- **Traces:** US-DASH-11 · **Priority:** Medium · **Type:** Edge
- **Preconditions:** Signed in as organizer. Booking can hold up to 8 seats.
- **Test data:** Case A: Ticket type with exactly 8 remaining, and another with 9 remaining. Case B: no ticket type close to selling out (all comfortably above threshold).
- **Steps:**
  1. Case A: open the dashboard; check whether the 8-remaining and 9-remaining types are flagged.
  2. Case B: view the selling-fast panel with nothing low.
- **Expected result:** Case A — the type with 8 remaining is flagged as a possible sell-out within a single booking (at/below 8); the 9-remaining type is not flagged at that threshold. Case B — panel shows "no tickets running low".

---

## US-DASH-12 — Recent registrations table

### TC-DASH-22 — Recent registrations table shows attendee, event, ฿ amount, status badge, and time, newest-first
- **Traces:** US-DASH-12 · **Priority:** High · **Type:** Functional
- **Preconditions:** Signed in as a team member permitted to view registrations, WITH finance access. Recent registrations exist, including one on-site walk-in.
- **Test data:** Row 1 newest: online reg ฿1,500 Paid at 14:20; Row 2: online reg ฿0 (free) at 13:05; Row 3: walk-in ฿800 Pending at 11:50; plus a Refunded row.
- **Steps:**
  1. Open the dashboard and view the recent-registrations table.
  2. Check ordering and each column: attendee, event, amount (฿), status badge, time.
  3. Locate the walk-in row.
  4. Choose "View all".
- **Expected result:** Rows are newest-first, each with attendee, event, amount in ฿, a Paid / Pending / Refunded badge, and the time (Bangkok, localized). The walk-in follows the same amount/status rules and may show Pending until settled. "View all" navigates to the registrations list.

### TC-DASH-23 — Amount hidden in table for user without finance access
- **Traces:** US-DASH-12 · **Priority:** High · **Type:** Negative
- **Preconditions:** Signed in as a team member permitted to view registrations but WITHOUT finance access.
- **Test data:** Recent registrations with non-zero ฿ amounts exist.
- **Steps:**
  1. Open the dashboard and view the recent-registrations table.
- **Expected result:** The amount column is hidden while attendee, event, status badge, and time still show. Amount is not shown as ฿0 or a masked placeholder — it is absent.

---

## US-DASH-13 — Trustworthy, always-current overview

### TC-DASH-24 — New confirmed registration updates and reconciles total, ticket-type breakdown, and recent table
- **Traces:** US-DASH-13 · **Priority:** High · **Type:** Functional
- **Preconditions:** Signed in as organizer with finance + registration access, viewing the dashboard. Known starting totals.
- **Test data:** Confirm a new registration for ticket type "VIP" for ฿1,500 in another session/module.
- **Steps:**
  1. Note the current registration total, VIP share in the ticket-type breakdown, and the top of the recent-registrations table.
  2. Confirm a new VIP registration in the owning module.
  3. Within the refresh window (or after a manual refresh), re-read the three surfaces.
- **Expected result:** Within a short refresh window the total registrations KPI increments, the VIP share in the ticket-type breakdown increases, and the new row appears at the top of the recent table — and all three agree with each other. Home/dashboard themselves changed no data.

### TC-DASH-25 — Single panel failure is isolated; refresh cadences and permission-based hiding hold
- **Traces:** US-DASH-13 · **Priority:** High · **Type:** Edge
- **Preconditions:** Signed in as organizer. Simulate one panel's data source failing (e.g. selling-fast) while others succeed.
- **Test data:** One panel forced into an error/unavailable condition; user lacks finance access for a revenue figure and lacks rights to attendee personal data in one panel.
- **Steps:**
  1. Load the surface with one panel's source failing.
  2. Observe the failed panel vs the others.
  3. Trigger a manual refresh and compare operational feeds vs analytics behavior.
  4. Inspect a finance/personal-data figure the user is not permitted to see.
- **Expected result:** Only the failing panel shows an "unavailable / retry" state; every other panel works normally. Operational feeds (today's sign-ups, alerts, selling-fast) refresh near-real-time while analytics refresh on a slightly longer cadence, and a manual refresh is available. Any figure the user isn't permitted to see (finance or attendee personal data) is hidden — never shown as zero or a placeholder. All dates, "today", days-remaining, and times read in Bangkok time and in the user's language (EN/TH).


---

<a id="tc-e12"></a>

# Coordinate Meetings — Test Cases

Area code: MTG · Epic E12 — Coordinate Meetings
Locale assumptions: Asia/Bangkok timezone, EN/TH content, ฿ (VAT not applicable to this epic — no money fields).

---

## US-MTG-01 — See meetings organized by Today / Upcoming / Past

### TC-MTG-01 — Meetings group correctly into Today / Upcoming / Past with live counts
- **Traces:** US-MTG-01 · **Priority:** High · **Type:** Functional
- **Preconditions:** Logged in as Event Organizer. Google Calendar connected. At least one meeting scheduled for today, two for a future day, and one in the past, all in Bangkok time.
- **Test data:** Meeting A "Venue walkthrough" today 14:00–15:00; Meeting B "Sponsor sync" tomorrow 09:00–09:30; Meeting C "Speaker brief" tomorrow 11:00–11:30; Meeting D "Post-event review" 3 days ago 10:00–11:00.
- **Steps:**
  1. Open the Meetings console.
  2. Read the Today, Upcoming and Past tab counts.
  3. Open the Today tab and inspect Meeting A's time label.
  4. Open the Upcoming tab and read the order of Meetings B and C.
  5. Open the Past tab.
- **Expected result:** Today count = 1 (Meeting A, shown as "Today · 14:00 – 15:00" and marked as today); Upcoming count = 2 with Meeting B (09:00) listed before Meeting C (11:00), earliest-start first; Past count = 1 (Meeting D). Each tab count matches the number of rows shown.

### TC-MTG-02 — Grouping is computed in Bangkok time regardless of viewer timezone
- **Traces:** US-MTG-01 · **Priority:** Medium · **Type:** Edge
- **Preconditions:** Logged in as Event Organizer. One meeting scheduled for late today Bangkok time.
- **Test data:** Meeting "Late vendor call" today 23:30–23:59 Asia/Bangkok. Viewer device/browser set to a non-Thai timezone (e.g., America/New_York, where that instant is still the previous day).
- **Steps:**
  1. Set the local device timezone to America/New_York.
  2. Open the Meetings console and view the Today tab.
- **Expected result:** The meeting still appears under Today (its Bangkok date), not under Past or Upcoming, confirming grouping is derived from Bangkok time and independent of the viewer's location.

### TC-MTG-03 — Meeting row shows all at-a-glance details
- **Traces:** US-MTG-01 · **Priority:** Medium · **Type:** Functional
- **Preconditions:** Logged in as Event Organizer. One video, one in-person and one phone meeting exist.
- **Test data:** Video meeting with Google Meet link; In-person meeting on event "Tech Summit BKK" (venue "BITEC Hall 2"); Phone meeting.
- **Steps:**
  1. Open the Meetings console.
  2. Inspect each meeting row.
- **Expected result:** Each row shows title, guest name and their role, related event, meeting type, mode (video / in person / phone), and location — join link for video, venue name ("BITEC Hall 2") for in person, and "Phone call" for phone.

---

## US-MTG-02 — Find a specific meeting in a long list

### TC-MTG-04 — Search by title/person/role/event/type matches and resets to first page
- **Traces:** US-MTG-02 · **Priority:** High · **Type:** Functional
- **Preconditions:** Logged in as Event Organizer. Many meetings booked spanning multiple pages. On the All tab, currently viewing page 3.
- **Test data:** Search term "Somchai" (a guest name appearing on 2 meetings). Also verify a Thai-language term "ผู้สนับสนุน" (sponsor) matches Thai content.
- **Steps:**
  1. From page 3, type "Somchai" into the search box and apply it.
  2. Observe the result list and current page.
  3. Clear the search, type the Thai term "ผู้สนับสนุน" and apply.
- **Expected result:** Only meetings whose title, person, role, event or type match "Somchai" remain, and the list returns to page 1. The Thai search returns matching Thai-content meetings — search works equally for Thai and English.

### TC-MTG-05 — Type filter combines with the active time tab
- **Traces:** US-MTG-02 · **Priority:** Medium · **Type:** Functional
- **Preconditions:** Logged in as Event Organizer. Upcoming tab has meetings of several types (Venue, Sponsor, Vendor, Speaker, Internal).
- **Test data:** Filter type = "Sponsor". Active tab = Upcoming.
- **Steps:**
  1. Open the Upcoming tab.
  2. Set the type filter to Sponsor.
- **Expected result:** Only upcoming Sponsor-type meetings are shown; meetings of other types and non-upcoming Sponsor meetings are excluded — the type filter is combined with the current time tab.

### TC-MTG-06 — No results message and paging summary
- **Traces:** US-MTG-02 · **Priority:** Medium · **Type:** Negative
- **Preconditions:** Logged in as Event Organizer with a long meeting list.
- **Test data:** (a) Search "zzzznomatch"; (b) rows-per-page = 10 on a list of 23 meetings.
- **Steps:**
  1. Search for "zzzznomatch" that matches nothing.
  2. Clear the search; set rows per page to 10 and move to page 2.
- **Expected result:** (a) A clear "No meetings match your filters" message is shown, no rows. (b) The summary reads "showing 11–20 of 23 meetings" on page 2 and updates correctly as rows-per-page and page change.

---

## US-MTG-03 — Schedule a meeting with a speaker, sponsor, venue or vendor

### TC-MTG-07 — Schedule a valid meeting and see it in the correct group
- **Traces:** US-MTG-03 · **Priority:** High · **Type:** Functional
- **Preconditions:** Logged in as Event Organizer. Calendar connected.
- **Test data:** Title "Keynote speaker briefing"; date tomorrow; start 10:00, end 10:45; type Speaker; mode Video; guest "Anong Phromma", email anong@example.com; related event "Tech Summit BKK"; notes "Confirm slides".
- **Steps:**
  1. Open the schedule panel.
  2. Fill in all required fields with the test data.
  3. Save.
- **Expected result:** A new meeting is created with status Scheduled and appears under Upcoming (tomorrow), earliest-start ordered. Confirms the happy-path creation criterion.

### TC-MTG-08 — Validation blocks bad title, past date and non-forward end time
- **Traces:** US-MTG-03 · **Priority:** High · **Type:** Negative
- **Preconditions:** Logged in as Event Organizer. Schedule panel open.
- **Test data:** (a) Title blank / single character "x"; (b) date = yesterday; (c) start 14:00, end 13:30 (end not after start).
- **Steps:**
  1. Leave the title blank (then try one character), attempt to save.
  2. Set the date to yesterday, attempt to save.
  3. Set start 14:00 and end 13:30, attempt to save.
- **Expected result:** Save is blocked in each case with a specific, field-level message telling exactly what to fix (title too short/required; date cannot be in the past; end time must be after start). No meeting is created.

### TC-MTG-09 — Location derives from mode; double-submit creates only one meeting
- **Traces:** US-MTG-03 · **Priority:** Medium · **Type:** Edge
- **Preconditions:** Logged in as Event Organizer. Event "Tech Summit BKK" has venue "BITEC Hall 2".
- **Test data:** Meeting X mode In person on event "Tech Summit BKK"; Meeting Y mode Phone. Rapid double-click on Save for one submission.
- **Steps:**
  1. Create Meeting X with mode In person tied to "Tech Summit BKK" and save.
  2. Create Meeting Y with mode Phone and save.
  3. On a third valid meeting, click Save twice in rapid succession.
- **Expected result:** Meeting X shows location = event venue "BITEC Hall 2"; Meeting Y shows location = "Phone call". The double-submitted meeting is created exactly once, not duplicated.

---

## US-MTG-04 — Have invites, video links and reminders handled automatically

### TC-MTG-10 — Video meeting auto-generates Meet link, invite and 15-min reminder
- **Traces:** US-MTG-04 · **Priority:** High · **Type:** Functional
- **Preconditions:** Logged in as Event Organizer. Google Calendar connected and healthy.
- **Test data:** Video meeting "Sponsor kickoff", tomorrow 15:00–15:30 Bangkok, guest email partner@example.com.
- **Steps:**
  1. Schedule the video meeting and save.
  2. Inspect the saved meeting and the guest-facing invite.
- **Expected result:** A Google Meet link is created and attached to the meeting and included in the invite; the guest receives a calendar invite for 15:00–15:30 Asia/Bangkok; both organizer and guest have a 15-minute-before reminder set.

### TC-MTG-11 — Meeting saved when calendar not connected; prompted to connect
- **Traces:** US-MTG-04 · **Priority:** High · **Type:** Negative
- **Preconditions:** Logged in as Event Organizer. Workspace Google Calendar NOT connected.
- **Test data:** Any valid video meeting.
- **Steps:**
  1. Schedule a valid meeting and save.
- **Expected result:** The meeting is still saved (nothing lost) and flagged as not synced; the organizer is prompted to connect the calendar so the invite and Meet link can be sent. No error discards the meeting.

### TC-MTG-12 — Calendar service briefly unavailable — meeting kept, marked not synced, retry sends invite
- **Traces:** US-MTG-04 · **Priority:** Medium · **Type:** Edge
- **Preconditions:** Logged in as Event Organizer. Calendar connected but the calendar service is temporarily unreachable at save time.
- **Test data:** Valid video meeting scheduled during the outage; service restored afterward.
- **Steps:**
  1. Save the meeting while the calendar service is unavailable.
  2. Observe the meeting's sync state and the retry option.
  3. Restore the service and trigger/allow retry.
- **Expected result:** The meeting is kept and marked "not yet synced" with a retry option; nothing is lost. Once the service is reachable and retry runs, the invite (and Meet link) go out and the meeting shows as synced.

---

## US-MTG-05 — Reschedule or edit a meeting

### TC-MTG-13 — Reschedule updates the existing invite, shifts reminder, re-notifies guest
- **Traces:** US-MTG-05 · **Priority:** High · **Type:** Functional
- **Preconditions:** Logged in as Event Organizer. An existing synced video meeting with a guest invite already sent.
- **Test data:** Move "Sponsor kickoff" from tomorrow 15:00–15:30 to tomorrow 17:00–17:30.
- **Steps:**
  1. Open the meeting and change start/end to 17:00–17:30.
  2. Save.
  3. Inspect the guest invite and reminder.
- **Expected result:** The guest's existing invite is updated in place (not duplicated), the 15-minute reminder shifts to the new 17:00 start, and the guest is re-notified of the new time.

### TC-MTG-14 — Changing mode video → in person removes Meet link and shows venue
- **Traces:** US-MTG-05 · **Priority:** Medium · **Type:** Functional
- **Preconditions:** Logged in as Event Organizer. A synced video meeting with a Meet link, tied to event "Tech Summit BKK" (venue "BITEC Hall 2").
- **Test data:** Change mode from Video to In person.
- **Steps:**
  1. Open the meeting and switch mode to In person.
  2. Save and inspect the meeting.
- **Expected result:** The Google Meet link is removed and the location shows the event's venue "BITEC Hall 2"; no stale join link remains and no Join button is offered.

### TC-MTG-15 — Concurrent edit conflict and cancelled-meeting edit block
- **Traces:** US-MTG-05 · **Priority:** Medium · **Type:** Negative
- **Preconditions:** Logged in as Event Organizer. Two sessions (or Admin + Organizer) have the same meeting open. Separately, one cancelled meeting exists.
- **Test data:** Session 1 and Session 2 both open Meeting M; Session 2 saves a change first. Cancelled Meeting N.
- **Steps:**
  1. In Session 2, change and save Meeting M.
  2. In Session 1, change and try to save Meeting M.
  3. Attempt to edit cancelled Meeting N.
- **Expected result:** Session 1 is told the meeting changed and asked to reload before saving — no silent overwrite of Session 2's edit. Cancelled Meeting N cannot be edited (edit is not offered / is blocked).

---

## US-MTG-06 — Cancel a meeting

### TC-MTG-16 — Cancel with reason marks Cancelled, removes from calendar, notifies guest
- **Traces:** US-MTG-06 · **Priority:** Medium · **Type:** Functional
- **Preconditions:** Logged in as Event Organizer. A scheduled, not-yet-happened meeting with an invite already sent.
- **Test data:** Cancel reason "Speaker unavailable".
- **Steps:**
  1. Open the scheduled meeting and choose Cancel.
  2. Enter the reason "Speaker unavailable" and confirm.
  3. Reload the Meetings list and inspect counts and row actions.
- **Expected result:** The meeting is marked Cancelled, removed from the guest's calendar, and the guest receives a cancellation notice that includes the reason "Speaker unavailable". After reload it no longer counts toward Today or Upcoming and offers no Join or Edit; the record is retained as Cancelled.

### TC-MTG-17 — Dismissing the cancel confirmation changes nothing
- **Traces:** US-MTG-06 · **Priority:** Low · **Type:** Negative
- **Preconditions:** Logged in as Event Organizer. A scheduled meeting exists.
- **Test data:** N/A.
- **Steps:**
  1. Open the meeting and click Cancel.
  2. Dismiss/close the confirmation without confirming.
- **Expected result:** The meeting remains Scheduled and unchanged; no cancellation notice is sent and it still counts in its time group.

---

## US-MTG-07 — Join a video meeting in one click

### TC-MTG-18 — Join opens link in a new tab; unavailable for past and not-ready links
- **Traces:** US-MTG-07 · **Priority:** High · **Type:** Functional
- **Preconditions:** Logged in as Event Organizer. A today video meeting with a ready Meet link; a past video meeting; an upcoming video meeting whose link is not yet ready (unsynced).
- **Test data:** Today video meeting (ready link); Past video meeting; Upcoming video meeting (link pending).
- **Steps:**
  1. On the today video meeting with a ready link, click Join.
  2. Inspect the past video meeting row.
  3. Inspect the upcoming video meeting whose link is not ready.
- **Expected result:** Join opens the meeting in a new browser tab for the ready link; the past video meeting shows no Join button; the not-ready meeting's Join is unavailable with a hint to retry the calendar sync.

---

## US-MTG-08 — Connect and manage the workspace Google Calendar

### TC-MTG-19 — Admin connects the workspace Google Calendar; header shows connected
- **Traces:** US-MTG-08 · **Priority:** Medium · **Type:** Functional
- **Preconditions:** Logged in as Admin. Workspace calendar currently disconnected.
- **Test data:** Valid Google account authorization for the workspace.
- **Steps:**
  1. Open Meetings and start the Google Calendar connection.
  2. Complete the Google authorization successfully.
  3. Observe the Meetings header status.
- **Expected result:** The workspace is marked connected and the Meetings header shows a connected status. Subsequently scheduled meetings sync normally.

### TC-MTG-20 — Non-Admin sees no connect/disconnect control; expired connection prompts reconnect
- **Traces:** US-MTG-08 · **Priority:** Medium · **Type:** Negative
- **Preconditions:** (a) Logged in as Event Organizer (non-Admin). (b) Separately, workspace connection has expired/been revoked.
- **Test data:** N/A.
- **Steps:**
  1. As a non-Admin Organizer, open Meetings and look for calendar connect/disconnect controls.
  2. With the connection expired, as Admin/Organizer trigger a sync (e.g., schedule a meeting).
- **Expected result:** The non-Admin sees no connect/disconnect control. When the connection is expired/revoked, the user is prompted to reconnect rather than sync failing silently; meetings scheduled while disconnected are saved but flagged not synced.

---

## US-MTG-09 — Be warned about overlapping meetings

### TC-MTG-21 — Overlap warning appears but never blocks saving
- **Traces:** US-MTG-09 · **Priority:** Low · **Type:** Edge
- **Preconditions:** Logged in as Event Organizer. An existing meeting tomorrow 10:00–11:00.
- **Test data:** New meeting tomorrow 10:30–11:15 (overlaps the existing one).
- **Steps:**
  1. Schedule the new overlapping meeting and click Save.
  2. Observe the warning.
  3. Choose to continue/schedule anyway.
- **Expected result:** A "this overlaps another meeting — schedule anyway?" warning is shown; choosing to continue saves the meeting normally. The warning is advisory and never blocks the save — the organizer stays in control.


---

<a id="tc-e13"></a>

# Measure Performance (Reports) — Test Cases

Area: RPT · Epic E13 — Measure Performance (Reports)
Locale notes applied throughout: currency ฿ (THB), VAT 7%, Asia/Bangkok timezone, EN/TH text, bank accounts masked to last 4 digits.

---

## US-RPT-01 — Workspace health at a glance

### TC-RPT-01 — Overview loads all health metrics with correct change direction and colour
- **Traces:** US-RPT-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in as Event Organizer with at least two events that have revenue, registrations, and check-ins in both the current and previous equal period.
- **Test data:** Default period = 30 days. Current period: revenue ฿120,000, registrations 400, attendance 80%, avg ticket ฿300, refund rate 4%. Previous period: revenue ฿100,000, refund rate 6%.
- **Steps:**
  1. Open the reports overview.
  2. Read the five headline tiles: revenue, registrations, attendance rate, average ticket price, refund rate.
  3. Read the "vs previous period" change on each tile, noting its arrow and colour.
- **Expected result:** All five metrics show for the 30-day default period. Revenue ฿120,000 shows an up change (+20%) in the positive colour; refund rate 4% vs 6% shows a down change in the positive colour (lower refund is "good"). Each tile compares against the previous equal 30-day window, so direction-of-good fits the metric rather than only the numeric sign.

### TC-RPT-02 — Revenue trend range switch recomputes; empty window renders neutral, not an error
- **Traces:** US-RPT-01  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Logged in as Event Organizer; account has revenue history spanning the year but no revenue in the most recent 7 days.
- **Test data:** Trend selector options: 7 days, 30 days, 90 days, Year. Last-7-days revenue = ฿0.
- **Steps:**
  1. On the overview, set the revenue trend selector to "Year" and note the chart, its headline total, and the "vs previous period" change.
  2. Switch the selector to "7 days".
  3. Observe the chart, headline total, and change.
- **Expected result:** On switching to 7 days the chart, headline total, and "vs previous period" change all recompute for the shorter window. Because there was no revenue in the last 7 days, the panel shows a zero total (฿0) and a neutral "—" change instead of an error or a misleading percentage.

---

## US-RPT-02 — Filter and search every report

### TC-RPT-03 — Single-event + search recomputes totals, charts, and table, and resets to page one
- **Traces:** US-RPT-02  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in as Admin; multiple events exist; the active report has more than one page of rows.
- **Test data:** Event = "Bangkok Tech Summit"; search term = "VIP".
- **Steps:**
  1. Open any report and page forward to page 2 of results.
  2. In the filter bar select the single event "Bangkok Tech Summit".
  3. Type "VIP" in the free-text search and apply.
  4. Observe the summary totals, charts, and the results table.
- **Expected result:** The totals, charts, and table all recompute to show only rows matching "VIP" within "Bangkok Tech Summit" — the whole view narrows, not just the table. After the filter change the results return to the first page.

### TC-RPT-04 — End date before start date is rejected with a clear message
- **Traces:** US-RPT-02  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Logged in as Admin; a report is open.
- **Test data:** Start date = 2026-07-01, End date = 2026-06-01.
- **Steps:**
  1. Set the date range start to 2026-07-01 and end to 2026-06-01.
  2. Apply the filter.
- **Expected result:** The message "End date must be on or after the start date" is shown, the range is not applied, and no broken or partial result is rendered.

### TC-RPT-05 — Over-24-month span is trimmed with a note; a no-match filter shows a friendly empty state
- **Traces:** US-RPT-02  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Logged in as Admin; a report is open.
- **Test data:** Date range = 2024-01-01 to 2026-12-31 (36 months); then filter combination event="Bangkok Tech Summit" + search="zzzzz" that matches nothing.
- **Steps:**
  1. Set a date span longer than 24 months and apply.
  2. Note the applied range and any explanatory note.
  3. Now apply a filter/search combination that matches no rows.
- **Expected result:** The span is trimmed to 24 months with a note explaining the limit. The no-match combination shows a friendly empty message with zeroed figures — not an error or a blank crash.

---

## US-RPT-03 — Registration mix and sales-channel breakdown

### TC-RPT-06 — Ticket-type and channel breakdowns total correctly and reconcile to headline; single-event refresh
- **Traces:** US-RPT-03  ·  **Priority:** Low  ·  **Type:** Functional
- **Preconditions:** Logged in as Event Organizer with registrations across multiple ticket types, events, and channels.
- **Test data:** Headline registrations = 400; ticket types General/VIP/Student; channels Website/Email/Social/Partner; single-event filter = "Bangkok Tech Summit".
- **Steps:**
  1. Open the overview and view the ticket-type breakdown (donut/percentage view).
  2. Sum the ticket-type shares and read the centre figure.
  3. View "registrations by event" and "sales by channel".
  4. Filter to the single event "Bangkok Tech Summit" and re-observe all breakdowns.
- **Expected result:** Each ticket type shows its share of total registrations and the shares add up to the whole; the donut centre shows 400, matching the headline registrations. "Registrations by event" lists top events with a relative-size bar. Channel shares add up to the whole. After filtering to one event, every breakdown reflects only that event.

---

## US-RPT-04 — Rank and explore event performance

### TC-RPT-07 — Event performance ranked best-first with revenue, attendance, and status badge
- **Traces:** US-RPT-04  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in as Event Organizer with a mix of Upcoming, Live, and Completed events.
- **Test data:** Events with registrations 400 / 250 / 90; revenue in ฿; statuses across all three.
- **Steps:**
  1. Open the full event-performance report.
  2. Read the row order and the columns registrations, revenue, attendance rate, and status badge.
- **Expected result:** Events are listed best-first by registrations (400, then 250, then 90). Each row shows registrations, revenue (฿), attendance rate, and a status badge of Upcoming, Live, or Completed.

### TC-RPT-08 — Status filter, upcoming-event attendance dash, and drill-in navigation
- **Traces:** US-RPT-04  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Logged in as Event Organizer; at least one Live event and one Upcoming event that has not happened yet; report is multi-page. Overview "top events" summary is visible.
- **Test data:** Status filter = "Live"; an Upcoming event "Future Expo" with no attendance yet.
- **Steps:**
  1. On the full report, apply status filter "Live".
  2. Observe which rows remain, their order, and the current page.
  3. Locate the Upcoming event row and read its attendance cell.
  4. Click an event's name.
  5. Return to the overview and click "View all" on the top-events summary.
- **Expected result:** Only Live events remain, still ranked by registrations, starting from the first page. The Upcoming event shows attendance "—" (not a misleading 0%). Clicking the event name opens that event's detail page. "View all" reaches this full ranked report.

---

## US-RPT-05 — Income and reconciliation report

### TC-RPT-09 — Net = Gross − Refunds − Fees per row and overall; VAT portion visible; ties to overview revenue
- **Traces:** US-RPT-05  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in as Admin; events with paid ticket sales including at least one refund and processing fees, same period/scope also viewable on the overview.
- **Test data:** Event row: Gross ฿100,000, Refunds ฿5,000, Fees ฿3,000 → expected Net ฿92,000. Prices are 7% VAT-inclusive → VAT portion of ฿100,000 gross = ฿6,542.06.
- **Steps:**
  1. Open the income report for the chosen period and events.
  2. For the event row, verify Net against Gross − Refunds − Fees.
  3. Read the summary Net tile and compare to the overview's revenue figure for the same period/scope.
  4. Locate and read the VAT portion of gross.
- **Expected result:** Row Net = ฿92,000 (Gross − Refunds − Fees). Summary Net tile equals Gross − Refunds − Fees overall and matches the overview's revenue for the same period and events. The VAT portion of gross is shown for reconciliation (≈฿6,542.06 on ฿100,000 VAT-inclusive).

### TC-RPT-10 — Free event shows ฿0 money columns, not hidden or errored
- **Traces:** US-RPT-05  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Logged in as Admin; at least one free event with registrations but no revenue.
- **Test data:** Free event "Community Meetup", price ฿0.
- **Steps:**
  1. Open the income report including the free event.
  2. Read the free event's Gross, Refunds, Fees, and Net columns.
- **Expected result:** All money columns show ฿0. The row is not hidden and does not error.

### TC-RPT-11 — No matching income shows friendly empty state and zeroed tiles
- **Traces:** US-RPT-05  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Logged in as Admin.
- **Test data:** Filter combination that matches no income (e.g. a free-only event plus a date range with no paid sales).
- **Steps:**
  1. Open the income report and apply the no-match filter.
- **Expected result:** The message "No income for this selection" is shown and all summary tiles read zero (฿0), with no error.

---

## US-RPT-06 — Transaction ledger for investigations

### TC-RPT-12 — Refund renders negative with label; payments positive; summary tiles correct; reference drills to payment detail
- **Traces:** US-RPT-06  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Logged in as Admin; ledger contains successful payments and at least one refund.
- **Test data:** Refund of ฿1,250; several positive payments; a known transaction reference.
- **Steps:**
  1. Open the transaction ledger.
  2. Locate the refund row and read its amount, colour, and label.
  3. Read the summary tiles: number of transactions, total successful payments, number of refunds, payment success rate.
  4. Click a transaction reference.
- **Expected result:** The refund shows as "-฿1,250" in the negative colour with a "Refund" label; payments show as positive amounts. Summary tiles show transaction count, total successful payments (฿), refund count, and success rate. Clicking the reference opens that payment's detail page (where refund actions live); the ledger itself changes no money.

### TC-RPT-13 — Failed charge excluded from payments total but counted in success rate; refunds never rewrite the original
- **Traces:** US-RPT-06  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Logged in as Admin; ledger contains at least one failed charge and one payment that was later refunded.
- **Test data:** Failed charge ฿800; original payment ฿1,250 subsequently refunded ฿1,250.
- **Steps:**
  1. Open the ledger and find the failed charge row.
  2. Confirm whether it is included in the total successful payments and in the success-rate calculation.
  3. Find the refunded payment and its refund entry.
- **Expected result:** The failed charge is clearly marked failed, is excluded from the total successful payments, but is still counted in the denominator for the success rate. The refund appears as a separate new entry and the original payment is marked "Refunded"; the original ฿1,250 amount is never overwritten.

---

## US-RPT-07 — Payouts and settlement visibility

### TC-RPT-14 — Payout rows and settlement tiles render with masked bank account
- **Traces:** US-RPT-07  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in as Event Organizer; payouts exist in Paid, In transit, and Pending states.
- **Test data:** Payout ref PO-2026-014, bank account ending 6789, amount ฿85,000, status Paid.
- **Steps:**
  1. Open the payouts report.
  2. Read a payout row: reference, date, bank, events covered, amount, status.
  3. Read the summary tiles: total paid out, total pending, total in transit, average payout.
  4. Inspect how the bank account is displayed.
- **Expected result:** Each payout shows reference, date, bank, covered events, amount (฿), and status (Paid / In transit / Pending). Summary tiles show total paid out, total pending, total in transit, and average payout. The bank shows only the masked last four digits (e.g. ••••6789) — never the full account number.

### TC-RPT-15 — Multi-event payout still appears when filtered to one of its events; empty state message
- **Traces:** US-RPT-07  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Logged in as Event Organizer; one payout covers several events; a separate filter can produce no matches.
- **Test data:** Payout covering events A, B, C; filter to event B; then a filter matching no payouts.
- **Steps:**
  1. In the payouts report, filter to a single event (B) that is one of several the payout covers.
  2. Confirm the multi-event payout still appears.
  3. Apply a filter that matches no payouts.
- **Expected result:** The payout covering A/B/C still appears when filtered to event B. When nothing matches, "No payouts for this selection" is shown, with no error.

---

## US-RPT-08 — Registrations report by event and status

### TC-RPT-16 — Status counts sum to the event total and reconcile to the overview
- **Traces:** US-RPT-08  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Logged in with access to the registrations report; events have a mix of confirmed, pending, waitlist, and cancelled registrations; same scope viewable on the overview.
- **Test data:** Event row: confirmed 300, pending 40, waitlist 30, cancelled 30 → total 400.
- **Steps:**
  1. Open the registrations report for a chosen period/scope.
  2. For an event row, add confirmed + pending + waitlist + cancelled and compare to the row total.
  3. Read the summary total and compare with the overview's registrations figure for the same period and events.
- **Expected result:** 300 + 40 + 30 + 30 = 400 equals the row total. The summary total matches the overview's registrations figure for the same period and events.

### TC-RPT-17 — Staff member can fully view the registrations report; empty state message
- **Traces:** US-RPT-08  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Logged in as a Staff/Team member with registration access only.
- **Test data:** Staff account; a filter combination that matches no registrations.
- **Steps:**
  1. As Staff, open the registrations report and confirm the data renders in full.
  2. Apply a filter that matches nothing.
- **Expected result:** The Staff member can view the report fully (registration data is available to the role). When nothing matches, "No registrations for this selection" is shown, not an error.

---

## US-RPT-09 — Attendance and no-show report

### TC-RPT-18 — No-shows and attendance rate computed correctly; tiles reconcile to overview
- **Traces:** US-RPT-09  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in as Event Organizer; at least one completed event with check-in data; same scope viewable on overview.
- **Test data:** Event with 400 registered and 320 checked in → expected no-shows 80, attendance 80%.
- **Steps:**
  1. Open the attendance/no-show report for the scope.
  2. Read the event row's no-shows and attendance rate.
  3. Read the summary tiles: total checked in, overall attendance rate, total no-shows, on-time share.
  4. Compare the overall attendance rate with the overview's attendance figure for the same scope.
- **Expected result:** No-shows show as 80 and attendance rate as 80%. Tiles show total checked in, overall attendance rate, total no-shows, and on-time share; the overall attendance rate matches the overview's attendance figure for the same scope.

### TC-RPT-19 — Upcoming event excluded from attendance rate; empty state message
- **Traces:** US-RPT-09  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Logged in as Event Organizer; an upcoming event with registrations but no check-ins yet; a filter that can match no attendance.
- **Test data:** Upcoming event "Future Expo", registered 150, checked in 0.
- **Steps:**
  1. Open the attendance report including the upcoming event.
  2. Read the upcoming event's attendance cell and confirm whether it contributes to the overall attendance rate.
  3. Apply a filter that matches no attendance.
- **Expected result:** The upcoming event's attendance shows "—" and it is left out of the overall attendance-rate calculation (it does not drag the rate down to 0%). When nothing matches, "No attendance for this selection" is shown.

---

## US-RPT-10 — Promotion and discount payback

### TC-RPT-20 — Discount rows and tiles correct; scheduled code reads zero; expired code still counts historically
- **Traces:** US-RPT-10  ·  **Priority:** Low  ·  **Type:** Functional
- **Preconditions:** Logged in as Event Organizer; discount codes exist that are Active, Scheduled, and Expired, with redemption history on the active and expired ones.
- **Test data:** Active "SUMMER25" = 25% off, 40 redemptions, ฿12,000 discount given, ฿90,000 revenue influenced; Scheduled "EARLYBIRD" = ฿200 off, not yet started; Expired "LAUNCH200" = ฿200 off with historical redemptions.
- **Steps:**
  1. Open the discounts report.
  2. Read the active code row: type, redemptions, total discount given, revenue influenced, status.
  3. Read the summary tiles: count of active codes, total redemptions, total discount given, total revenue influenced.
  4. Read the scheduled code row and the expired code row.
- **Expected result:** The active code shows its type ("25% off"), redemptions, ฿ discount given, ฿ revenue influenced, and status. Tiles show active-code count, total redemptions, total discount given (฿), and total revenue influenced (฿). The scheduled code reads zero redemptions/discount/revenue and is not counted among active codes. The expired code is marked Expired but its historical redemptions, discount, and revenue still contribute to the totals.

---

## US-RPT-11 — Export the current view

### TC-RPT-21 — Export respects applied filters and produces a correctly formatted Excel workbook
- **Traces:** US-RPT-11  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Logged in as Admin with export permission; a report filtered to a single event.
- **Test data:** Report filtered to event "Bangkok Tech Summit"; export format = Excel (.xlsx).
- **Steps:**
  1. Filter the report to the single event and note the on-screen summary figures and rows.
  2. Choose Export → Excel.
  3. Open the produced file.
- **Expected result:** The file contains only that event's rows plus the summary figures shown on screen — nothing more. The Excel file includes a summary of the key figures plus the detailed rows, with money (฿) and percentages properly formatted. The exported figures match the on-screen figures at the moment of request.

### TC-RPT-22 — CSV preserves Thai text and machine-friendly values; PDF is branded with filters and timestamp
- **Traces:** US-RPT-11  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Logged in as Admin with export permission; report contains Thai attendee/event names; filters applied.
- **Test data:** Thai event name "มหกรรมเทคโนโลยีกรุงเทพ"; export CSV then PDF.
- **Steps:**
  1. Export the filtered report to CSV and open it (UTF-8).
  2. Inspect Thai text, dates, and raw numbers.
  3. Export the same view to PDF and open it.
- **Expected result:** In the CSV, Thai text reads correctly (no mojibake), and dates and numbers are machine-friendly (raw, unformatted values). The PDF is a tidy, branded, readable document showing the key figures, the table, the applied filters, and the generation time (Asia/Bangkok).

### TC-RPT-23 — Large export is prepared asynchronously; rapid double-click returns the same file
- **Traces:** US-RPT-11  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Logged in as Admin with export permission; a very large result set is loaded.
- **Test data:** All-events, 24-month range export; then two export clicks within ~2 seconds.
- **Steps:**
  1. Request the large export.
  2. Observe the response and, once ready, the notification/download.
  3. On a normal export, click the export button twice within a few seconds.
- **Expected result:** For the large export the user is told it is being prepared and is later notified with a secure, time-limited download link when ready. On the rapid double-click, the second click yields the same file (idempotent) rather than producing a duplicate export.

---

## US-RPT-12 — Role-appropriate access and safe finance sharing

### TC-RPT-24 — Staff blocked from finance reports and from exporting attendee data
- **Traces:** US-RPT-12  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Logged in as a Staff/Team member with registration access only (no finance permission).
- **Test data:** Finance reports = income, transactions, payouts, discounts, event performance; a registrations/attendance report containing attendee names.
- **Steps:**
  1. As Staff, look for the finance reports (income, transactions, payouts, discounts, event performance) in navigation and attempt to open one.
  2. Open the registrations or attendance report (permitted) and attempt to export a view containing attendee names.
- **Expected result:** Finance reports are not shown to Staff — the options are hidden or disabled and cannot be opened. On the permitted report, attempting to export attendee data is refused with a message that the user lacks permission to export attendee data, and no file is produced.

### TC-RPT-25 — Attendee refused admin console; exports are audited; cross-report totals reconcile
- **Traces:** US-RPT-12  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** One attendee (non-admin) account; one Admin account with export permission; same period/scope viewable across overview, registrations, and income reports.
- **Test data:** Attendee attempts to reach any report URL; Admin exports the income report for a known scope.
- **Steps:**
  1. As the attendee (non-admin), attempt to reach any report / the admin console.
  2. As Admin, export a report and then check the export/audit record.
  3. For the same period and events, compare overview registrations vs the registrations report total, and overview revenue vs income Net.
- **Expected result:** The attendee is refused access to the admin console entirely. The Admin's export creates an audit record of who exported what, when, and for which scope. Overview registrations equals the registrations report total, and overview revenue matches income Net for the same period and events — reports are read-only and the numbers agree across screens.


---

