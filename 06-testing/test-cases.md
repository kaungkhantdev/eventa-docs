# Eventa — Test Cases

| | |
|---|---|
| **Author** | QA Engineer |
| **Version** | 1.0 · 2026-07-23 |
| **Companion** | [test-plan.md](test-plan.md) |
| **Scope** | 374 test cases across 13 epics, tracing to the 165 user stories in the [product backlog](../01-requirements-and-features/functional-requirements.md) |

> Each test case is derived from a user story's Given/When/Then acceptance criteria, and lists its traceability (`US-*`), priority, type, preconditions, test data, steps, and expected result. Priority defaults from MoSCoW (Must→High, Should→Medium, Could→Low); a **withdrawn** case (its story is Won't, or what it tested is not in the web app) keeps its id, takes priority Won't and is not run. See [test-plan.md](test-plan.md) for strategy, environment, defect process, and closure.

## Coverage & traceability

| # | Epic (test cases) | Stories covered | Test cases | High | Med | Low | Won't |
|---|------|:---:|:---:|:---:|:---:|:---:|:---:|
| 1 | [Accounts & Sign-in](#tc-e01) | 12 | 24 | 19 | 3 | 0 | 2 |
| 2 | [Configure the Workspace & Team](#tc-e02) | 13 | 29 | 16 | 10 | 2 | 1 |
| 3 | [Create & Manage Events](#tc-e03) | 15 | 39 | 24 | 13 | 1 | 1 |
| 4 | [Public Event Pages](#tc-e04) | 10 | 22 | 12 | 7 | 1 | 2 |
| 5 | [Sell Tickets & Run Promotions](#tc-e05) | 12 | 25 | 9 | 13 | 2 | 1 |
| 6 | [Discover & Register for Events](#tc-e06) | 15 | 42 | 29 | 9 | 2 | 2 |
| 7 | [Communicate with Attendees](#tc-e07) | 10 | 21 | 3 | 13 | 4 | 1 |
| 8 | [Manage Registrations & Admit Attendees](#tc-e08) | 14 | 30 | 17 | 8 | 0 | 5 |
| 9 | [Get Paid & Manage Finances](#tc-e09) | 14 | 27 | 18 | 6 | 0 | 3 |
| 10 | [Build the Event Program](#tc-e10) | 13 | 24 | 14 | 7 | 0 | 3 |
| 11 | [Organizer Home & Dashboard](#tc-e11) | 16 | 44 | 21 | 18 | 3 | 2 |
| 12 | [Coordinate Meetings](#tc-e12) | 9 | 22 | 8 | 12 | 1 | 1 |
| 13 | [Measure Performance (Reports)](#tc-e13) | 12 | 25 | 10 | 13 | 2 | 0 |
| | **Total** | **165** | **374** | **200** | **132** | **18** | **24** |

---

<a id="tc-e01"></a>

# Accounts & Sign-in — Test Cases

Epic E1 — Accounts & Sign-in. Test basis: user stories US-ACC-01…US-ACC-12 and their acceptance criteria.
Locale notes: bilingual EN/TH surfaces; Asia/Bangkok time for "last active" and lockout timers; neutral, non-revealing messages are a recurring security rule across this epic — except the forgot-password form, which deliberately says when no account matches (US-ACC-04) and is bounded by its own attempt lock instead.

---

## US-ACC-01 — Create an organizer account

### TC-ACC-01 — Sign up, confirm email, then sign in
- **Traces:** US-ACC-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** No organizer account uses the test email; access to its inbox.
- **Test data:** Workspace name `Somchai Events`; full name `Somchai Prasert`; email `organizer-new@eventa-test.co.th`; password `Bangkok#2026!` (typed twice).
- **Steps:**
  1. Open "Create your account", fill in every field and tick the Terms & Privacy Policy box; choose "Create account".
  2. Open Home by its address.
  3. Open the confirmation link from the email.
  4. Choose "Sign in" and sign in with the same email and password.
- **Expected result:** Step 1 opens "Check your email" naming the address. Step 2 sends me to the organizer sign-in (not signed in yet). Step 3 shows "Email confirmed". Step 4 lands on Home.

### TC-ACC-02 — Already-registered email gets the same "Check your email"
- **Traces:** US-ACC-01  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** An active organizer account already uses the test email, with password `Bangkok#2026!`.
- **Test data:** Email `existing-org@eventa-test.co.th`; workspace name `Chiang Mai Tours`; full name `Niran Chaiyo`; password `Chiangmai#77x`.
- **Steps:**
  1. On "Create your account", sign up with the existing email and the test data; choose "Create account".
  2. Compare the page with TC-ACC-01 step 1.
  3. Sign in with `Chiangmai#77x`, then with `Bangkok#2026!`.
- **Expected result:** The same "Check your email" page shows; nothing says the email is registered. `Chiangmai#77x` gets "Invalid email or password" (no second account was made); `Bangkok#2026!` signs in to the original workspace.

### TC-ACC-03 — Password meter, form refusals and "Send it again"
- **Traces:** US-ACC-01  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** On "Create your account"; workspace `Somchai Events` exists (TC-ACC-01).
- **Test data:** Email `weakpw@eventa-test.co.th`; full name `Malee Srisuk`; passwords `12345`, `bangkok2026`, `Sukhumvit#2026`; mismatched confirm `Sukhumvit#2027`.
- **Steps:**
  1. Type `12345`, then `bangkok2026`, then `Sukhumvit#2026` in Password, watching the meter.
  2. With workspace `Weakpw Events`, password `bangkok2026` in both boxes and Terms ticked, choose "Create account".
  3. Try `Sukhumvit#2026` with confirm `Sukhumvit#2027`; then matching passwords with Terms unticked; then Terms ticked with workspace `Somchai Events`.
  4. Use workspace `Weakpw Events`; choose "Create account".
  5. On "Check your email", choose "Send it again", then try it again at once.
- **Expected result:** The meter reads "Use 8+ characters with a number and a symbol" for the first two and "Strong password" for the third. Step 2 is refused: "Password must be at least 8 characters and include a number and a symbol." Step 3 is refused each time: "Those passwords do not match.", the browser asks to tick the box, then "That workspace name is already taken. Try another." Step 4 opens "Check your email". In step 5 a fresh link arrives, then the button waits about 60 seconds ("in 59s").

---

## US-ACC-02 — Organizer sign-in

### TC-ACC-04 — Organizer signs in to Home
- **Traces:** US-ACC-02  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Active, confirmed organizer account.
- **Test data:** Email `organizer-active@eventa-test.co.th`; password `Bangkok#2026!`.
- **Steps:**
  1. Open the organizer sign-in ("Welcome back").
  2. Enter the email and password; choose "Sign in".
- **Expected result:** I land on Home in the organizer console.

### TC-ACC-05 — Wrong credentials show one neutral message
- **Traces:** US-ACC-02  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** An active organizer account exists.
- **Test data:** (a) `organizer-active@eventa-test.co.th` with `WrongPass#1`; (b) unknown `nobody@eventa-test.co.th` with `WrongPass#1`.
- **Steps:**
  1. Sign in with (a); note the message.
  2. Sign in with (b); note the message.
- **Expected result:** Both show exactly "Invalid email or password"; neither signs in.

### TC-ACC-06 — Unconfirmed and suspended accounts are refused
- **Traces:** US-ACC-02  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** One organizer signed up but never confirmed; one organizer suspended by an admin on the Users page; inbox access.
- **Test data:** Unconfirmed `pending@eventa-test.co.th` / `Bangkok#2026!`; suspended `suspended@eventa-test.co.th` / `Bangkok#2026!`.
- **Steps:**
  1. Sign in with the unconfirmed account's right password; check its inbox.
  2. Sign in with the suspended account's right password.
- **Expected result:** Step 1 shows "Please confirm your email — we’ve sent you a fresh link.", a new confirmation email arrives, and I am not signed in. Step 2 shows "Your access has been suspended." and I am not signed in.

---

## US-ACC-03 — Attendee sign-in without forcing accounts

### TC-ACC-07 — Attendee lands on "My Account", never the console
- **Traces:** US-ACC-03  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Confirmed attendee account with one booked ticket.
- **Test data:** Email `attendee@eventa-test.co.th`; password `Ticket#2026th`.
- **Steps:**
  1. Open the attendee sign-in ("Welcome back" in the portal).
  2. Enter the email and password; choose "Sign in".
- **Expected result:** "My Account" opens on the "My Events" tab showing the booked ticket; no organizer console page or menu appears.

### TC-ACC-08 — Organizer-only email refused; a guest registers without an account
- **Traces:** US-ACC-03  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** An organizer account whose email has no attendee account; a published free event open for registration, with no approval step; signed out.
- **Test data:** Organizer `organizer-active@eventa-test.co.th` / `Bangkok#2026!`; guest `Malee Srisuk`, `guest@eventa-test.co.th`.
- **Steps:**
  1. At the attendee sign-in, sign in with the organizer's email and password.
  2. On "What's on in Bangkok", open the free event and register as the guest; choose "Confirm registration".
  3. Check the guest's inbox.
- **Expected result:** Step 1 shows "Invalid email or password" and does not sign in. Step 2 completes with no account step. Step 3: a "You're going to <event>" email arrives with a link to the ticket.

---

## US-ACC-04 — Reset a forgotten password

### TC-ACC-09 — Reset by email: new password, every device signed out, link spent
- **Traces:** US-ACC-04  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Active organizer account with a password, signed in on browsers A and B; inbox access.
- **Test data:** Email `reset-me@eventa-test.co.th`; old password `OldPass#2025`; new password `NewStrong#2026`.
- **Steps:**
  1. On the organizer sign-in choose "Forgot password?", enter the email and choose "Send reset link".
  2. Within the hour, open the emailed link, enter the new password twice and choose "Set new password".
  3. On browser A, open any console page.
  4. Sign in with `OldPass#2025`, then with `NewStrong#2026`.
  5. Open the same reset link again.
- **Expected result:** Step 1 shows "Check your inbox — a reset link is on its way to reset-me@eventa-test.co.th" and the email arrives. Step 2 shows "Password reset". Step 3 sends A to sign-in. In step 4 the old password gets "Invalid email or password"; the new one lands on Home. Step 5 shows "That link didn’t work" with "Request a new link".

### TC-ACC-10 — No link: unknown address, blocked accounts, too many misses, same password
- **Traces:** US-ACC-04  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Organizer accounts that are unconfirmed, invited but not yet accepted (US-SET-11), and suspended; an organizer with no attendee account; an address with no organizer account, not tried in the last 15 minutes; inbox access.
- **Test data:** Unknown `noone@eventa-test.co.th`; organizer-only `organizer-active@eventa-test.co.th`; `pending@eventa-test.co.th`, `invited@eventa-test.co.th`, `suspended@eventa-test.co.th`; `reset-me@eventa-test.co.th` with current password `NewStrong#2026`.
- **Steps:**
  1. On the organizer "Reset your password", submit `noone@eventa-test.co.th`.
  2. Choose "Reset your attendee account" and submit `organizer-active@eventa-test.co.th`.
  3. On the organizer form, submit the unconfirmed, invited and suspended emails in turn.
  4. Submit `noone@eventa-test.co.th` 4 more times, then twice again.
  5. Request a link for `reset-me@eventa-test.co.th`, open it and set `NewStrong#2026` as the new password.
- **Expected result:** No email is sent in steps 1–4. Step 1: "No organizer account uses that email address. Check the spelling, try your attendee account, or create one." Step 2: the attendee wording ("No attendee account uses… try your organizer account…"). Step 3: each says what unblocks it: open the confirmation email (signing in sends a fresh one); open the invitation email and choose a password there; ask a workspace admin to reactivate it. Step 4: the 5th miss still gets the no-account answer; the next two read "Too many password-reset attempts. Try again in 15 minutes." Step 5: "Please choose a password different from your current one." and the password is unchanged.

---

## US-ACC-05 — Change my password while signed in

### TC-ACC-11 — Change password: other devices out, this one stays, email sent
- **Traces:** US-ACC-05  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Organizer signed in on browser A (making the change) and browser B; inbox access.
- **Test data:** Email `reset-me@eventa-test.co.th`; current password `NewStrong#2026`; new password `EvenStronger#27`.
- **Steps:**
  1. On A, open Settings → "Security" and choose "Change" next to Password.
  2. Enter the current password and the new one twice; choose "Update password".
  3. On A, open another console page.
  4. On B, open any console page.
  5. Check the inbox.
- **Expected result:** "Password changed." shows. A stays signed in; B is sent to sign-in. A "Your Eventa password was changed" email arrives, with the Bangkok time of the change and how many other sign-ins were signed out.

### TC-ACC-12 — Change refused: wrong current, same password, mismatch
- **Traces:** US-ACC-05  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Attendee signed in on browsers A and B.
- **Test data:** Email `attendee@eventa-test.co.th`; current password `Ticket#2026th`; wrong current `NotMyPassword#9`; new `Sathorn#2027`; mismatched confirm `Sathorn#2028`.
- **Steps:**
  1. On A, open "My Account" → "Settings" and choose "Change" next to Password.
  2. Enter the wrong current password and `Sathorn#2027` twice; choose "Update password".
  3. On B, reload "My Account".
  4. Enter the right current password and `Ticket#2026th` as the new one twice; choose "Update password".
  5. Enter the right current password, `Sathorn#2027` and confirm `Sathorn#2028`; choose "Update password".
- **Expected result:** Step 2: "Your current password is incorrect." Step 3: B is still signed in. Step 4: "Please choose a password different from your current one." Step 5: "Those two do not match. Retype the new password." The password never changes.

---

## US-ACC-06 — Sign in with Google, Apple or LinkedIn

### TC-ACC-13 — New social sign-in creates a confirmed account at the right home
- **Traces:** US-ACC-06  ·  **Priority:** Won't  ·  **Type:** Functional
_Withdrawn: social sign-in is not in the web app; the id stays valid — not run._

### TC-ACC-14 — Social links to existing password account; cancelled consent signs nobody in
- **Traces:** US-ACC-06  ·  **Priority:** Won't  ·  **Type:** Negative
_Withdrawn: social sign-in is not in the web app; the id stays valid — not run._

---

## US-ACC-07 — Extra security with two-factor sign-in

### TC-ACC-15 — Code step refuses a wrong code and accepts the current one
- **Traces:** US-ACC-07  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Organizer account with two-factor on, paired with an authenticator app.
- **Test data:** Email `twofa@eventa-test.co.th`; password `Bangkok#2026!`; a code that is not the current one; the app's current 6-digit code.
- **Steps:**
  1. Sign in with the email and password.
  2. Enter the wrong code; choose "Verify".
  3. Enter the current code; choose "Verify".
- **Expected result:** Step 1 switches the form to "Enter your code" with "Authentication code"; I am not signed in yet. Step 2 shows "That code didn't work. Try the next one, or a recovery code." Step 3 lands on Home.

### TC-ACC-16 — A recovery code works once; the code step expires
- **Traces:** US-ACC-07  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Same account; its recovery codes were saved when two-factor was turned on.
- **Test data:** One saved recovery code (shaped like `3F9A2-C71B0`).
- **Steps:**
  1. Sign in with the password; on "Enter your code" enter the recovery code; choose "Verify".
  2. Sign out, sign in with the password again and enter the same recovery code.
  3. Sign in with the password, wait more than 5 minutes, then enter a current app code.
- **Expected result:** Step 1 lands on Home. Step 2 shows "That code didn't work. Try the next one, or a recovery code." and does not sign in. Step 3 shows "Your sign-in challenge expired — please sign in again."

---

## US-ACC-08 — Stay signed in on my device

### TC-ACC-17 — "Remember me" keeps me signed in for 7 days
- **Traces:** US-ACC-08  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Active organizer account.
- **Test data:** Email `remember@eventa-test.co.th`; password `Bangkok#2026!`; "Remember me" ticked.
- **Steps:**
  1. Sign in with "Remember me" ticked.
  2. Close the browser fully, reopen it and open Home.
  3. Two days after sign-in, open Home.
  4. Just over 7 days after sign-in, open Home.
- **Expected result:** Steps 2 and 3 open Home without signing in. Step 4 sends me to the organizer sign-in.

### TC-ACC-18 — Without "Remember me", the sign-in lasts a day
- **Traces:** US-ACC-08  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Active organizer account.
- **Test data:** Same account; "Remember me" left unticked.
- **Steps:**
  1. Sign in with "Remember me" unticked.
  2. Close the browser fully, reopen it and open Home.
  3. Just over 24 hours after sign-in, open Home.
- **Expected result:** Step 2 opens Home: closing the browser does not sign me out. Step 3 sends me to the organizer sign-in.

---

## US-ACC-09 — See and sign out my other devices

### TC-ACC-19 — See sessions and sign out other devices
- **Traces:** US-ACC-09  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Organizer signed in on browser A (current) and browser B; attendee signed in on browser C (current) and browser D.
- **Test data:** Organizer `organizer-active@eventa-test.co.th`; attendee `attendee@eventa-test.co.th`.
- **Steps:**
  1. On A, open Settings → "Security" and read "Active sessions".
  2. Choose "Revoke" on B's row; on B, open any console page.
  3. Sign in again on B; on A, reload and choose "Sign out everywhere else"; on B, open a console page.
  4. On C, open "My Account" → "Settings", read "Other devices" and choose "Sign out".
  5. On D, open "My Account"; on C, reload "Settings".
- **Expected result:** Step 1: each row shows its device (e.g. "Chrome on macOS"), IP address and sign-in date and time; A is marked "This device" with no "Revoke". Step 2: B leaves the list and is sent to sign-in. Step 3: B is sent to sign-in and only A remains. Step 4: "Signed in on 1 other device", then "Signed out on your other devices." Step 5: D is sent to the attendee sign-in; C reads "This device only".

---

## US-ACC-10 — Sign out

### TC-ACC-20 — Sign out from the console and from "My Account"
- **Traces:** US-ACC-10  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Organizer and attendee accounts on the same email, one browser.
- **Test data:** `both@eventa-test.co.th`; organizer password `Bangkok#2026!`; attendee password `Ticket#2026th`.
- **Steps:**
  1. Sign in as the organizer; open the user menu and choose "Sign out"; then open Home by its address.
  2. Sign in as the organizer again; then sign in at the attendee sign-in in the same browser.
  3. Open Home by its address.
  4. Open "My Account" (still signed in as the attendee), choose "Sign out", then open "My Account" by its address.
- **Expected result:** Step 1 opens the organizer sign-in, and Home asks me to sign in again. Step 3 sends me to the organizer sign-in: the attendee sign-in replaced the organizer's in this browser. Step 4 opens the attendee sign-in, and "My Account" asks me to sign in again.

---

## US-ACC-11 — Keep organizer and attendee accounts cleanly separated

### TC-ACC-21 — Each audience goes to its own sign-in; passwords stay separate
- **Traces:** US-ACC-11  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** The same email has an organizer and an attendee account with different passwords.
- **Test data:** `both@eventa-test.co.th`; organizer `Bangkok#2026!`; attendee `Ticket#2026th`; new organizer password `Silom#2027x`.
- **Steps:**
  1. Sign in as the attendee; open Home by its address.
  2. Sign in as the organizer; open "My Account" by its address.
  3. Go back to the console; in Settings → "Security", change the organizer password to `Silom#2027x`.
  4. At the attendee sign-in, sign in with `Ticket#2026th`.
- **Expected result:** Step 1 opens the organizer sign-in. Step 2 opens the attendee sign-in. Step 4 signs in to "My Account": the attendee password is unchanged.

### TC-ACC-22 — A signed-out link returns to its page after sign-in
- **Traces:** US-ACC-11  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed out of both audiences.
- **Test data:** The Registrations page's address; organizer `organizer-active@eventa-test.co.th` / `Bangkok#2026!`.
- **Steps:**
  1. Open the Registrations page by its address.
  2. Sign in on the page shown.
- **Expected result:** The organizer sign-in opens, and after signing in Registrations opens, not Home.

---

## US-ACC-12 — Protect accounts from password-guessing

### TC-ACC-23 — Five wrong passwords lock sign-in for 15 minutes
- **Traces:** US-ACC-12  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Active organizer account with no failed sign-ins in the last 15 minutes.
- **Test data:** Email `lockout@eventa-test.co.th`; wrong password `Wrong#2026x`; right password `Bangkok#2026!`.
- **Steps:**
  1. Sign in with the wrong password 5 times.
  2. Sign in with the right password.
  3. A few minutes later, try the right password again.
  4. 15 minutes after the 5th failure, sign in with the right password.
- **Expected result:** Step 1 shows "Invalid email or password" each time. Step 2 shows "Too many sign-in attempts. Try again in 15 minutes." and does not sign in. Step 3 shows fewer minutes left. Step 4 lands on Home.

### TC-ACC-24 — An unknown email locks the same way
- **Traces:** US-ACC-12  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** An email with no account, not tried in the last 15 minutes.
- **Test data:** Unknown email `ghost@eventa-test.co.th`; password `Wrong#2026x`.
- **Steps:**
  1. Sign in with the unknown email 6 times.
  2. Compare the messages with TC-ACC-23.
- **Expected result:** The first 5 show "Invalid email or password" and the 6th "Too many sign-in attempts. Try again in 15 minutes.", exactly as for the real account.


---

<a id="tc-e02"></a>

# Configure the Workspace & Team — Test Cases

Epic E2 — Configure the Workspace & Team. Area code: **SET**. Locale defaults: ฿ THB, VAT 7%, Asia/Bangkok, EN/TH.

---

## US-SET-01 — My profile & preferences

### TC-SET-01 — Save my name and confirm a phone by code
- **Traces:** US-SET-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as any team member; the test SMS inbox for the new number is readable.
- **Test data:** Full name "Somchai Rattana"; phone "081 234 5678".
- **Steps:**
  1. Open Settings → "Profile"; check the photo (or initials), the email badge and the editable fields.
  2. Change "Full name" to "Somchai Rattana" and choose "Save changes".
  3. Type the phone, choose "Send code", enter the texted 6-digit code and choose "Confirm".
  4. Reload the page.
- **Expected result:** Step 1 shows the email with "Verified" or "Unverified" and name, phone, city, timezone, language and "About you" ready to edit. Step 2 shows "Profile saved." Step 3 shows "Code sent. Check your phone." then "Number confirmed." After reload the name is "Somchai Rattana" and the number shows as confirmed.

### TC-SET-02 — Change email: link to the new address, old one works until then
- **Traces:** US-SET-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as `somchai@eventa-test.co.th`; both test mailboxes are readable.
- **Test data:** New email `somchai.r@eventa-test.co.th`.
- **Steps:**
  1. On Settings → "Profile", enter the new address under "New email" and choose "Send confirmation".
  2. Check both mailboxes.
  3. Sign out, then sign in with `somchai@eventa-test.co.th`.
  4. Open the link in the new mailbox.
- **Expected result:** Step 1 shows "Check the new address for a link." and a hint that `somchai.r@eventa-test.co.th` is waiting to be confirmed. The new address gets the link; the current one gets a heads-up with no link. Step 3 signs in. Step 4 shows "Email confirmed" — "You now sign in with somchai.r@eventa-test.co.th."

### TC-SET-03 — Language and timezone are kept; account emails follow the language
- **Traces:** US-SET-01  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Signed in; profile language English, timezone Asia/Bangkok.
- **Test data:** Language "ไทย", timezone "Europe/London"; password `OldPass!9` → `NewPass#7`.
- **Steps:**
  1. Set Language to "ไทย" and Timezone to "Europe/London"; choose "Save changes"; reload.
  2. Change the password on Settings → "Security" (US-SET-02).
  3. Open the "password changed" email, then read a session's time on Settings → "Security".
- **Expected result:** Both choices are still selected after reload. The email is in Thai. The console still shows times in Bangkok time, not London time.

---

## US-SET-02 — Change my password

### TC-SET-04 — Password change signs out my other devices
- **Traces:** US-SET-02  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in on browser A; the same account is signed in on browser B.
- **Test data:** Current password `OldPass!9`; new password `NewPass#7`.
- **Steps:**
  1. On browser A, open Settings → "Security" and choose "Change" beside Password.
  2. Enter `OldPass!9`, then `NewPass#7` in "New password" and "Confirm new password"; choose "Update password".
  3. On browser B, open any page.
  4. Check the account's mailbox.
- **Expected result:** Browser A shows "Password changed." and stays signed in. Browser B is sent back to sign in. An email says the password was changed.

### TC-SET-05 — Mismatched, weak or wrong-current password is refused
- **Traces:** US-SET-02  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Signed in with password `OldPass!9`; the "Change password" panel is open.
- **Test data:** (a) new `NewPass#7`, confirm `NewPass#8`; (b) new and confirm `abcdefgh`; (c) current `WrongPass!1`, new and confirm `NewPass#7`.
- **Steps:**
  1. Enter case (a) and choose "Update password".
  2. Enter case (b) and choose "Update password".
  3. Enter case (c) and choose "Update password".
  4. Sign out and sign in with `OldPass!9`.
- **Expected result:** (a) "Those two do not match. Retype the new password." (b) "Password must be at least 8 characters and include a number and a symbol." (c) "Your current password is incorrect." Step 4 signs in: nothing changed.

---

## US-SET-03 — Two-factor authentication & recovery codes

### TC-SET-06 — Turn on 2FA and get 8 recovery codes
- **Traces:** US-SET-03  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in; two-factor reads "Off"; an authenticator app on a test phone.
- **Test data:** The current 6-digit code from the app.
- **Steps:**
  1. On Settings → "Security", switch Two-factor authentication on and choose "Start setup".
  2. Scan the QR code (or type the key under "Can't scan? Enter this key").
  3. Enter the app's code and choose "Enable 2FA".
  4. Choose "Save as file", reload the page and select the two-factor switch again.
- **Expected result:** Step 3 shows exactly 8 codes under "Save these now — they will not be shown again." The saved file holds the same 8. After reload two-factor reads "On" and the switch opens "Turn off two-factor authentication", with no codes.

### TC-SET-07 — Wrong code keeps 2FA off; turning it off needs a code and sends an email
- **Traces:** US-SET-03  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Part A: two-factor off, setup started. Part B: two-factor on.
- **Test data:** Wrong code `000000`; a current code from the app.
- **Steps:**
  1. Part A: enter `000000` and choose "Enable 2FA".
  2. Part B: switch Two-factor authentication off, enter the current code and choose "Turn off".
  3. Check the account's mailbox.
- **Expected result:** Step 1 shows "That code is wrong or has expired — try the current one." and two-factor stays "Off". Step 2 shows "Two-factor turned off." and the badge reads "Off". An email says two-factor was turned off.

### TC-SET-08 — Regenerate recovery codes invalidates old set
- **Traces:** US-SET-03  ·  **Priority:** Won't  ·  **Type:** Edge
_Withdrawn: regenerating recovery codes is not in the web app; the id stays valid — not run._

---

## US-SET-04 — Review & sign out my active sessions

### TC-SET-09 — Sessions show device, IP and sign-in time; this device has no Revoke
- **Traces:** US-SET-04  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in on two browsers (for example Chrome on macOS and Firefox on Windows).
- **Test data:** The current session plus one other.
- **Steps:**
  1. Open Settings → "Security" and read "Active sessions".
- **Expected result:** Each row shows its device (for example "Chrome on macOS"), IP address and sign-in date and time in Bangkok time. The current one is marked "This device" with no "Revoke"; the other has "Revoke".

### TC-SET-10 — Revoke one device, then sign out everywhere else
- **Traces:** US-SET-04  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in on browser A plus browsers B and C.
- **Test data:** —
- **Steps:**
  1. On browser A, choose "Revoke" on browser B's row.
  2. On browser B, open any page.
  3. On browser A, choose "Sign out everywhere else".
  4. On browser C, open any page.
- **Expected result:** B's row leaves the list and B is sent back to sign in. After step 3 only "This device" remains and the button is gone; C is sent back to sign in; A stays signed in.

---

## US-SET-05 — Security & access audit log

### TC-SET-11 — Role change shows newest first and can't be edited
- **Traces:** US-SET-05  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Admin "Anong" has just changed "Nok" from Staff to Organizer.
- **Test data:** Teammate "Nok", Staff → Organizer.
- **Steps:**
  1. As Anong, open Settings → "Security" and choose "View log".
  2. Read the top entry; look for any edit or delete control.
  3. As Nok, open the same log.
- **Expected result:** The top entry starts "Changed Nok's role from Staff to Organizer", with Anong's name, IP and the Bangkok time; entries run newest first and none can be edited or deleted. Nok sees only Nok's own entries, not Anong's change.

### TC-SET-12 — Export a date range; the export is logged
- **Traces:** US-SET-05  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin; the log has entries in and outside July 2026.
- **Test data:** From 2026-07-01, To 2026-07-27; then From 2026-07-27, To 2026-07-01.
- **Steps:**
  1. In "Audit log", set From and To and choose "Export log".
  2. Open the downloaded file.
  3. Look at the top of the log.
  4. Swap the dates and choose "Export log".
- **Expected result:** `audit-2026-07-01-to-2026-07-27.csv` downloads with only entries from those Bangkok days. A new top entry reads "Exported N audit entries". Step 4 shows "The start date is after the end date." and nothing downloads.

---

## US-SET-06 — Notification preferences

### TC-SET-13 — Each topic's Email and SMS switch saves on its own
- **Traces:** US-SET-06  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in with a confirmed phone; preferences never changed.
- **Test data:** SMS on for "Payments"; Email off for "Feedback".
- **Steps:**
  1. Open Settings → "Notification preferences".
  2. Switch SMS on for "Payments" and Email off for "Feedback".
  3. Reload the page.
- **Expected result:** Seven topics are listed (Registrations, Payments, Sales milestones, Feedback, Payouts, Operational alerts, Tasks), each with Email on and SMS off at first. After reload only the two switches changed in step 2 differ.

### TC-SET-14 — SMS needs a confirmed phone
- **Traces:** US-SET-06  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Signed in with no phone on file.
- **Test data:** Phone "081 234 5678" confirmed by texted code.
- **Steps:**
  1. Open Settings → "Notification preferences" and read the SMS column.
  2. Confirm the phone on Settings → "Profile" (US-SET-01), then reopen preferences.
- **Expected result:** Step 1 shows "—" instead of a switch in the SMS column for every topic. After step 2 every topic has an SMS switch, off.

---

## US-SET-07 — Organization profile, tax details & branding

### TC-SET-15 — Save valid details; new documents use them, old ones don't change
- **Traces:** US-SET-07  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin; payments connected in test mode; a paid ticket on sale.
- **Test data:** "Ban Suan Events Co., Ltd.", "99 Sukhumvit Rd, Khlong Toei, Bangkok 10110", website `https://eventa-test.co.th`, Tax ID `0105558000000`; later address "1 Silom Rd, Bang Rak, Bangkok 10500"; a PNG logo.
- **Steps:**
  1. On Settings → "Organization", fill in the details and choose "Save changes".
  2. Complete a paid order and open its receipt.
  3. Change the address to the Silom one and save.
  4. Complete a second paid order; open both receipts.
  5. Choose "Upload" and pick the PNG logo.
- **Expected result:** Step 1 shows "Organization saved." The first receipt names "Ban Suan Events Co., Ltd.", the Sukhumvit address and Tax ID 0105558000000, and still does after step 3; the second shows the Silom address. Step 5 shows "Logo updated." and the logo replaces the initials at the top of the side rail.

### TC-SET-16 — Invalid tax ID or website is refused
- **Traces:** US-SET-07  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Signed in as Admin with valid details saved.
- **Test data:** (a) Tax ID `12345`; (b) website `ftp://files.eventa-test.co.th`.
- **Steps:**
  1. Enter Tax ID `12345` and choose "Save changes".
  2. Restore the valid Tax ID, enter the ftp website and choose "Save changes".
  3. Reload the page.
- **Expected result:** (a) Under Tax ID: "Tax ID must be the 13-digit Thai VAT registration number." (b) Under Website: "Website must be a valid http(s) address." After reload the earlier valid details are unchanged.

### TC-SET-17 — Organizer and Staff see the page read-only
- **Traces:** US-SET-07  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Signed in as an Organizer (repeat as Staff).
- **Test data:** —
- **Steps:**
  1. Open Settings → "Organization".
- **Expected result:** Every field is disabled, there is no "Save changes", "Upload", "Replace" or "Remove", and the page says "Only an Admin can change organization settings."

---

## US-SET-08 — Connect our payment account

### TC-SET-18 — Save test keys, see the test banner, test the connection
- **Traces:** US-SET-08  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin on a test environment; status "Not connected".
- **Test data:** The project's Stripe test-mode publishable key, secret key and webhook signing secret; a made-up secret `sk_test_invalid000`.
- **Steps:**
  1. On Settings → "Payments", with "Test" selected, paste the three keys and choose "Save keys".
  2. Read the status, the banner and the "Secret key" box.
  3. Choose "Test connection".
  4. Paste `sk_test_invalid000` as "Secret key" and choose "Save keys".
- **Expected result:** Step 1 shows "Payment keys saved."; the status reads "Connected · test", the banner says "You're using Test keys — no real charges are processed.", and "Secret key" is empty, showing only a masked tail. Step 3 shows "Connection is working." Step 4 shows "That key could not be used: …" under "Secret key"; the saved keys stay.

### TC-SET-19 — Disconnect after confirming; free events keep working
- **Traces:** US-SET-08  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin; payments connected; past paid orders and payouts exist; one free and one paid event on sale.
- **Test data:** —
- **Steps:**
  1. Choose "Disconnect", then "Keep it connected" in "Disconnect Stripe?".
  2. Choose "Disconnect" again and confirm with "Disconnect".
  3. As an attendee, try to buy a paid ticket, then register for the free event.
  4. Open the payments and payouts lists.
- **Expected result:** Step 1 changes nothing. Step 2 shows "Payment account disconnected.", the status reads "Not connected" and the key boxes are empty. The paid ticket can't be paid for; the free registration completes. Past orders and payouts are unchanged.

---

## US-SET-09 — Choose payment methods at checkout

### TC-SET-20 — A method switched off disappears at checkout
- **Traces:** US-SET-09  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin; payments connected in test mode; Card and PromptPay on; a ฿500 ticket on sale.
- **Test data:** Switch Card off, then on again.
- **Steps:**
  1. On Settings → "Payments" → "Payment methods", switch Card off.
  2. As an attendee, start checkout for the ฿500 ticket.
  3. Switch Card back on; start a new checkout.
- **Expected result:** Step 1 shows "Payment methods updated." Step 2 offers only PromptPay. Step 3 offers Card and PromptPay.

### TC-SET-21 — The last method can't be switched off; wallets need a connected account
- **Traces:** US-SET-09  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Part A: connected; only PromptPay on; a paid ticket type exists. Part B: not connected.
- **Test data:** —
- **Steps:**
  1. Part A: switch PromptPay off.
  2. Part B: switch Apple Pay on.
- **Expected result:** Step 1 shows "Keep at least one payment method enabled while you sell paid tickets." and PromptPay stays on. Step 2 shows "Connect a payment account before enabling Apple Pay." and Apple Pay stays off.

---

## US-SET-10 — Checkout & receipt preferences

### TC-SET-22 — Statement label is used; receipt is emailed with VAT
- **Traces:** US-SET-10  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin; test keys connected; "Email receipts" on; a ฿1,070 ticket (VAT 7% included) on sale.
- **Test data:** Label "EVENTA TICKETS"; buyer `ploy@eventa-test.co.th`; Stripe's published test card.
- **Steps:**
  1. Under "Checkout preferences", enter "EVENTA TICKETS" as "Statement descriptor" and choose "Save changes".
  2. As the buyer, pay for the ticket by card.
  3. Check the charge in the Stripe test dashboard, then the buyer's mailbox.
- **Expected result:** Step 1 shows "Payment preferences saved." The charge's statement label ends with "EVENTA TICKE" (the first 12 characters). The receipt email itemizes "VAT 7% (included)" as ฿70 of the ฿1,070 total.

### TC-SET-23 — Disallowed label characters are refused; receipts off sends none
- **Traces:** US-SET-10  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Signed in as Admin; test keys connected; a paid ticket on sale.
- **Test data:** (a) label "EVENTA<*>#"; (b) label "OUR VERY LONG COMPANY NAME LTD" (30 characters).
- **Steps:**
  1. Enter label (a) and choose "Save changes".
  2. Type label (b) into "Statement descriptor".
  3. Switch "Email receipts" off, then complete a paid order as a buyer.
- **Expected result:** Step 1 shows "Statement descriptor may use letters, numbers, spaces and . , ' -" and nothing is saved. In step 2 the box stops at 22 characters. Step 3 shows "Payment preferences saved." and no receipt email arrives.

---

## US-SET-11 — Invite & manage teammates

### TC-SET-24 — Invite a teammate; re-invite re-sends, never duplicates
- **Traces:** US-SET-11  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin; `anong@eventa-test.co.th` is an Active member.
- **Test data:** "Ploy Siriwan", `ploy@eventa-test.co.th`, role Staff.
- **Steps:**
  1. On Settings → "Users", choose "Invite user", fill in Ploy and choose "Send invitation"; open the "Invited" tab.
  2. Invite `ploy@eventa-test.co.th` again.
  3. Invite `anong@eventa-test.co.th`.
  4. Open Ploy's latest link, set a password and choose "Accept invitation".
- **Expected result:** Step 1 shows "Invitation sent."; Ploy is listed as Staff, "Invited", the Invited count rises by 1, and a join link arrives. Step 2 sends a fresh link; there is still one Ploy row. Step 3 shows "A member with this email already exists in the workspace." Step 4 shows "You're in" and Ploy's status becomes "Active".

### TC-SET-25 — Suspend, reactivate, remove, and the last-Admin guard
- **Traces:** US-SET-11  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Signed in as the only Admin; Organizer "Nok" is signed in on another browser; Organizer "Mali" created the event "Bangkok Food Fair".
- **Test data:** —
- **Steps:**
  1. Suspend Nok; on Nok's browser open any page, then try to sign in.
  2. Reactivate Nok.
  3. Choose "Remove from workspace" on Mali's row; open the events list.
  4. Try to suspend, remove and change the role of yourself.
- **Expected result:** Step 1: Nok reads "Suspended", Nok's browser is signed out and sign-in says "Your access has been suspended." Step 2: Nok is "Active" as Organizer again. Step 3: Mali leaves the list; "Bangkok Food Fair" is still there. Step 4: each is refused with "This is the last Admin — promote someone else first."

---

## US-SET-12 — Assign roles & fine-tune permissions

### TC-SET-26 — Role change takes effect at once and is recorded
- **Traces:** US-SET-12  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin "Anong"; "Nok" is Staff and signed in on another browser.
- **Test data:** Nok, Staff → Organizer.
- **Steps:**
  1. On Settings → "Users", pick Organizer in Nok's Role column.
  2. Without signing out, Nok creates a draft event.
  3. Nok opens Settings → "Users".
  4. Anong opens the audit log (Settings → "Security" → "View log").
- **Expected result:** Step 2 succeeds without signing in again. Step 3 is refused: Organizer can't manage users. The top audit entry reads "Changed Nok's role from Staff to Organizer — sensitive access granted" (Organizer gains "Export attendee data"), with Anong's name and the time.

### TC-SET-27 — No granting access you don't hold; sensitive grants are flagged
- **Traces:** US-SET-12  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** A custom role "Team lead" holds only "Manage users & roles" and "View registrations & attendees"; "Kanya" holds it. Admin "Anong" is available.
- **Test data:** —
- **Steps:**
  1. As Kanya, pick Admin in Ploy's Role column.
  2. As Kanya, on Settings → "Roles", edit "Team lead", switch on "Issue refunds" and choose "Save changes".
  3. As Anong, edit Staff, switch on "Export attendee data", save, then open the audit log.
- **Expected result:** Steps 1 and 2 are refused with "You cannot grant access you do not have…" and nothing changes. Step 3 shows "Role saved." and the entry "Changed the permissions of the Staff role — sensitive access granted".

---

## US-SET-13 — Create custom roles

### TC-SET-28 — Roles overview and creating "Volunteer"
- **Traces:** US-SET-13  ·  **Priority:** Low  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin; only the built-in roles exist; teammate "Ploy" is Staff.
- **Test data:** Name "Volunteer"; description "Door check-in at weekend events"; permissions "View registrations & attendees" and "Check attendees in".
- **Steps:**
  1. Open Settings → "Roles"; read the tiles; type "check" in "Search roles…", then clear it.
  2. Choose "New role", fill in the test data, switch on the two permissions and choose "Create role".
  3. On Settings → "Users", pick Volunteer in Ploy's Role column; reopen Roles.
- **Expected result:** Each tile shows the role's member count, "Built-in" on Admin, Organizer and Staff, its description and first three permissions ("+N more" beyond that); the search leaves only Staff ("Check-in & registration view"). Step 2 shows "Role saved." and a "Volunteer" tile with "No members yet". After step 3 it reads "1 member".

### TC-SET-29 — Duplicate name refused; the last user-manager role is protected
- **Traces:** US-SET-13  ·  **Priority:** Low  ·  **Type:** Negative
- **Preconditions:** Signed in as Admin; only the Admin role has "Manage users & roles"; Staff member "Ploy" is signed in elsewhere.
- **Test data:** —
- **Steps:**
  1. Choose "New role", name it "Organizer" and choose "Create role".
  2. Choose "Edit role" on Admin, switch off "Manage users & roles" and choose "Save changes".
  3. Edit Staff, switch on "Create & edit events" and save; without signing out, Ploy creates a draft event.
- **Expected result:** Step 1 shows "A role called "Organizer" already exists — choose a unique name." Step 2 shows "This is the only role that can manage users and roles — grant it to another role first."; the "Role name" box is read-only. Step 3 shows "Role saved." and Ploy's draft is created.


---
<a id="tc-e03"></a>

# Create & Manage Events — Test Cases

Epic E3 · Area code: EVT · Locale: Thai market (฿, VAT 7%, PromptPay, Asia/Bangkok, EN/TH)

---

## US-EVT-01 — Find and manage my events

### TC-EVT-01 — Active tab, summary line, paging and fill bar
- **Traces:** US-EVT-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as an Event Organizer whose workspace has 27 active and 4 completed events.
- **Test data:** Active event "Bangkok Design Week" at 45 of 100 registrations.
- **Steps:**
  1. Open Events.
  2. Read the tabs, the summary line and the pager.
  3. Find "Bangkok Design Week" and read its Registrations cell.
  4. Press "Next".
- **Expected result:** "Overview" opens on "Active" (27) beside "Completed" (4), with "Showing 1–10 of 27 events" and "Rows per page" at 10. The row shows name, type, date, status and "45/100" with a 45% fill bar. "Next" shows "Showing 11–20 of 27 events".

### TC-EVT-02 — Search, type, sort and tab go back to page 1; no match is explained
- **Traces:** US-EVT-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Organizer on Events, page 3 of the Active tab.
- **Test data:** Search "Songkran"; type "Conference"; sort "Name A–Z"; search "zzzz".
- **Steps:**
  1. Type "Songkran" in "Search events…".
  2. Pick "Conference" and sort by "Name A–Z".
  3. Switch to "Completed".
  4. Replace the search with "zzzz", then press "Clear filters".
- **Expected result:** Each change narrows the list and returns it to page 1; Completed shows only finished or cancelled events. "zzzz" shows "No events match" with "Clear filters", tabs and search still on screen; "Clear filters" brings back the full Active list.

### TC-EVT-03 — Row menu actions; duplicate and delete update the counts without a reload
- **Traces:** US-EVT-01  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Organizer with create and publish rights; Active 12, Completed 4; a draft "Test Draft" with no registrations. A second member with create rights but no publish rights.
- **Test data:** "Test Draft".
- **Steps:**
  1. Open any row's "More actions" menu and read it.
  2. Choose "Duplicate"; read the Active count.
  3. On "Test Draft" choose "Delete" and confirm "Delete"; read the Active count.
  4. As the second member, open a row's menu.
- **Expected result:** The menu offers "View details", "Edit", "Duplicate" and "Delete". After the duplicate, Active reads 13; after the delete, "Event deleted." shows and Active reads 12, both without a reload. The second member's menu has no "Delete".

---

## US-EVT-02 — Create an event with a guided wizard and save drafts

### TC-EVT-04 — Six-step rail, progress bar, Save as draft and reopen
- **Traces:** US-EVT-02  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Organizer with create rights; an existing online draft "Online Meetup".
- **Test data:** Title "TechConf BKK 2026", in person, start 2026-08-15 09:00.
- **Steps:**
  1. Choose "New event"; read the rail, the "% completed" bar and the header's "Save as draft".
  2. Open Seating, choose "Reserved seating" then "General admission", reading the rail each time; open Assign seats and press "Go to Seating".
  3. Enter the title and start date, then press "Save as draft".
  4. Go back to Events, find the draft and choose "Edit".
  5. Open "Online Meetup" in the wizard and open Assign seats.
- **Expected result:** The rail always lists Basics, Date & location, Seating, Tickets, Assign seats, Review & publish; the bar reads 17% on Basics and 50% on Seating, and no seating choice adds or removes a step. Assign seats says "General admission has no seats to assign. To sell seats by ticket type, choose Reserved seating on the Seating step."; "Go to Seating" opens Seating. "Save as draft" can't be pressed until title and start date are set; then the event is a Draft under Active, reopens with its title, date and General admission, and the header reads "Save changes". "Online Meetup" has the same six steps and says "Online events have no seats to assign."

### TC-EVT-05 — Each Next saves the step being left
- **Traces:** US-EVT-02  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Organizer editing an existing draft, on Basics.
- **Test data:** New title "Bangkok Design Week 2026"; new start time 10:00.
- **Steps:**
  1. Change the title and press "Next".
  2. Reload the page.
  3. On Date & location change the start time and press the header "Save changes".
- **Expected result:** "Next" saves the title without a message: after the reload it reads "Bangkok Design Week 2026". The header "Save changes" saves the open step and shows "Changes saved."

### TC-EVT-06 — A stale save is refused; a published event saves without going offline
- **Traces:** US-EVT-02  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Members A and B can edit the same draft; a separate published event.
- **Test data:** A changes the title; B changes the description; on the published event, a new venue name "Riverside Hall".
- **Steps:**
  1. B opens the draft in the wizard.
  2. A opens it, changes the title and presses "Save changes".
  3. B changes the description and presses "Save changes".
  4. On the published event, change the venue name, go to Review & publish and press "Save changes".
- **Expected result:** B is refused with "This event changed elsewhere. Reload the latest version and try again." and A's title stays. The published event's last button reads "Save changes", not "Publish event"; it saves, opens the event's workspace, and the event is still published with its page live.

---

## US-EVT-03 — Describe the event (Basics)

### TC-EVT-07 — Basics fields, highlight order and a valid cover save
- **Traces:** US-EVT-03  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Organizer on Basics of a draft; the workspace has a category "Design & Creative".
- **Test data:** Title "Bangkok Design Week"; a 120-character description with one bold word; type "Exhibition"; highlights "20+ speakers", "Free coffee" and one with no label; a 2 MB JPG cover.
- **Steps:**
  1. Enter the title and description; read the counter.
  2. Pick the type and the category, and upload the cover.
  3. Add the three highlights and press "Save changes"; reload.
  4. Press "Preview" and read the highlights.
- **Expected result:** The counter reads "120/5000" and the bold word stays bold. Type, category and cover are saved (the cover shows "Replace" and "Remove"). After the reload two highlights remain, in order; the empty one is gone. The preview shows "20+ speakers" then "Free coffee".

### TC-EVT-08 — Description stops at 5,000 characters and the counter turns red
- **Traces:** US-EVT-03  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Organizer on Basics.
- **Test data:** A 5,200-character text on the clipboard.
- **Steps:**
  1. Click into "Description" and paste the text.
  2. Read the counter and the end of the text.
- **Expected result:** Only the first 5,000 characters are kept, and the counter reads "5000/5000" in red.

### TC-EVT-09 — A wrong cover is refused; an empty title blocks publishing
- **Traces:** US-EVT-03  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Organizer on Basics of a draft that is otherwise ready to publish.
- **Test data:** A .pdf file; a 7 MB .jpg.
- **Steps:**
  1. Choose the .pdf as cover image.
  2. Choose the 7 MB .jpg.
  3. Clear the title and open Review & publish.
- **Expected result:** The .pdf is refused with "That has to be a PNG or JPG image." and the .jpg with "That image is over 5MB. Try a smaller one, or export it at 1600×900."; no cover is kept. The checklist reads "Still needs a title" and "Publish event" can't be pressed.

---

## US-EVT-04 — Set the date, time and location

### TC-EVT-10 — Bangkok time and an in-person venue
- **Traces:** US-EVT-04  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Organizer on Date & location of a draft with a title, a description and a saved ticket type.
- **Test data:** Start 2026-08-15 09:00, end 2026-08-15 17:00; In-person; venue "Riverside Hall", address "88 Charoen Krung Rd, Bang Rak, Bangkok".
- **Steps:**
  1. Read "Timezone"; set the start and end.
  2. Choose "In-person", leave the venue empty and open Review & publish.
  3. Back on Date & location, enter the venue and address.
  4. Press "Save changes" and open the event's workspace.
- **Expected result:** "Timezone" reads "GMT+7 Bangkok" and can't be edited. Without a venue the checklist reads "Still needs a venue or an online link"; with one it reads "Location added" and a map of the venue shows. The workspace shows the event at 09:00 on 15 August 2026, marked "Asia/Bangkok · GMT+7".

### TC-EVT-11 — End before start is refused; an online event emails its link
- **Traces:** US-EVT-04  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Organizer on Date & location of a draft with a title, a description and a free ticket type.
- **Test data:** Start 2026-08-15 17:00, end 2026-08-15 09:00; Online, "Meeting link" https://meet.example.com/design-week; attendee "Nattaya Srisuk", nattaya@example.com.
- **Steps:**
  1. Set the end before the start and press "Save changes".
  2. Fix the end, choose "Online" and enter the meeting link.
  3. Open Seating, then Review & publish.
  4. Publish, register Nattaya on the public page and open her confirmation email.
- **Expected result:** Step 1 is refused with "End time must be after the start time." and nothing is saved. Online shows no venue fields; Seating says "Online event — no seating. A join link is emailed after registration."; the checklist reads "Location added". Nattaya's confirmation email carries the meeting link.

---

## US-EVT-05 — Choose seating (general admission or reserved)

### TC-EVT-12 — Reserved map preview and total; general admission has no map
- **Traces:** US-EVT-05  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** In-person draft on the Seating step, on a wide screen (the Summary panel shows).
- **Test data:** Reserved 10 rows × 20 seats; General admission with Capacity 150.
- **Steps:**
  1. Choose "Reserved seating"; set "Rows" 10 and "Seats / row" 20.
  2. Read the preview and "Total seats".
  3. Choose "General admission".
  4. On Tickets set "Capacity" to 150 and read the Summary panel.
- **Expected result:** "Total seats" reads 200 and the preview shows rows 1–10 under "Stage", with "After Tickets, give the seats ticket types in Assign seats." General admission hides the map. The Summary reads Seating "General admission" and Capacity 150.

### TC-EVT-12b — Row selects give whole rows a type; each type's quantity is its seats
- **Traces:** US-EVT-05  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** In-person reserved draft, 7 rows × 16 seats (112); ticket types VIP ฿2,500, Normal ฿890 and Free ฿0 saved in that order; nothing sold.
- **Test data:** Rows 1–2 VIP, rows 3–6 Normal, row 7 Free; then row 6 changed to VIP.
- **Steps:**
  1. On Tickets, look for a Quantity column.
  2. On Assign seats, read "Ticket types by row".
  3. Give rows 1–2 VIP, 3–6 Normal and 7 Free; read the legend and save.
  4. Change row 6 to VIP; read the legend before saving, then save.
  5. Open the Tickets screen and edit Normal.
- **Expected result:** Tickets has no Quantity column, only "Each type's quantity is the seats it holds — give seats a type in the next step, Assign seats." There is one select per row, "Row 1" to "Row 7", offering VIP, Normal, Free and "No ticket type". The legend reads "VIP · Rows 1–2 · 32 seats", "Normal · Rows 3–6 · 64 seats", "Free · Row 7 · 16 seats", and after the change, before saving, "VIP · Rows 1–2, 6 · 48 seats", "Normal · Rows 3–5 · 48 seats". The Tickets screen shows VIP 48, Normal 48, Free 16; Normal's quantity is read-only, "Set by the seat map".

### TC-EVT-12c — Part of a row takes a type by clicking or dragging
- **Traces:** US-EVT-05  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** As TC-EVT-12b, with rows 1–2 VIP and rows 3–7 Normal saved.
- **Test data:** Free for row 7 seats 1–5.
- **Steps:**
  1. On Assign seats, read "Paint seats with" and the hint under the map.
  2. Choose Free; click row 7 seat 1, then drag from seat 2 to seat 5.
  3. Read row 7's select and the legend; save, reload and reopen Assign seats.
  4. Pick Normal in row 7's select and read the legend.
- **Expected result:** The picker offers VIP, Normal, Free and "No ticket type"; the hint reads "Pick a ticket type, then click or drag across seats to give just those seats that type." Seats 1–5 of row 7 take Free's colour and the select reads "Mixed"; the legend reads "VIP · Rows 1–2 · 32 seats", "Normal · Rows 3–6, row 7 seats 6–16 · 75 seats", "Free · Row 7 seats 1–5 · 5 seats", and is unchanged after the reload. Step 4 gives all of row 7 Normal: "Normal · Rows 3–7 · 80 seats", "Free · 0 seats".

### TC-EVT-13 — No ticket types points to Tickets; seats without a type stay neutral
- **Traces:** US-EVT-05  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** A new in-person reserved draft, 5 rows × 20 seats, with no ticket types.
- **Test data:** Ticket types VIP and Normal; rows 1–2 VIP, rows 3–4 Normal, row 5 left on "No ticket type".
- **Steps:**
  1. Open Assign seats and press "Go to Tickets".
  2. Add VIP and Normal and press "Next".
  3. Give rows 1–4 their types, leave row 5, and save.
  4. Read the preview and legend, then open Review & publish.
- **Expected result:** Assign seats first says "Add your ticket types in the Tickets step first, then give the seats one here."; "Go to Tickets" opens Tickets. Row 5's seats stay a neutral colour; the legend reads "VIP · Rows 1–2 · 40 seats", "Normal · Rows 3–4 · 40 seats", "No ticket type · 20 seats". The checklist reads "Still needs a ticket type for every seat".

### TC-EVT-13b — Publishing waits for every seat and every ticket type
- **Traces:** US-EVT-05  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** A complete in-person draft (title, description, venue), reserved 7 × 16, ticket types VIP, Normal and Free; publish rights.
- **Test data:** Run A: rows 1–2 VIP, 3–6 Normal, row 7 seats 1–5 Free and 6–16 "No ticket type". Run B: rows 1–2 VIP, 3–7 Normal. Run C: as A, with row 7 seats 6–16 Normal.
- **Steps:**
  1. Save Run A and open Review & publish.
  2. Save Run B and press "Publish event".
  3. Save Run C and press "Publish event".
- **Expected result:** Run A: the checklist reads "Still needs a ticket type for every seat" and "Publish event" can't be pressed. Run B: the checklist is complete, but publishing is refused with "Free has no seats. Give it some seats or remove it."; the event stays a draft. Run C: the event publishes and its workspace opens.

### TC-EVT-14 — A published map keeps sold seats; new seats wait for a type
- **Traces:** US-EVT-05  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Published reserved event, 6 rows × 20, rows 1–3 VIP and 4–6 Normal; seats in row 5 sold.
- **Test data:** Rows reduced to 4, then raised to 7.
- **Steps:**
  1. On Seating set "Rows" to 4 and press "Save changes".
  2. Reload; set "Rows" to 7, read the step and press "Save changes".
  3. Set "Rows" to 7 again, press "Next" to Assign seats, leave row 7 on "No ticket type" and save.
  4. Give row 7 Normal, save and read the legend.
- **Expected result:** Step 1 is refused with "Seats that are already sold cannot be removed from a published event."; the map keeps 6 rows. Step 2 shows "New seats need a ticket type. The seat map is saved when you give them one in Assign seats." and "Saved. Give the new seats a ticket type in Assign seats to save the seat map."; the map still has 6 rows. Step 3 is refused with "Give every seat in row 7 a ticket type. Every seat of a published event needs one." Step 4 saves 7 rows: "VIP · Rows 1–3 · 60 seats", "Normal · Rows 4–7 · 80 seats".

### TC-EVT-14b — A booked or held seat keeps its ticket type
- **Traces:** US-EVT-05  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Published reserved event, 7 × 16, rows 1–2 VIP, 3–6 Normal, 7 Free. Row 3 seat 2 is booked; row 4 seat 8 is held in an unexpired checkout.
- **Test data:** VIP for row 3 by its select; VIP painted on row 4 seat 8; VIP painted on row 3 seats 1 and 3–16.
- **Steps:**
  1. On Assign seats, set row 3 to VIP with its select and save.
  2. Reload; choose VIP under "Paint seats with", click row 4 seat 8 and save.
  3. Reload; paint row 3 seats 1 and 3–16 VIP and save.
- **Expected result:** Step 1 is refused with "Row 3 seat 2 is booked or held, so its ticket type can't change." and nothing is saved. Step 2 is refused the same way, naming row 4 seat 8. Step 3 saves: row 3 reads "Mixed", seat 2 stays Normal, and the legend counts VIP 47 and Normal 49.

---

## US-EVT-06 — Set up tickets, capacity and registration rules

### TC-EVT-15 — Unique names, free price, and the last ticket type stays
- **Traces:** US-EVT-06  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Organizer on Tickets of a general-admission draft.
- **Test data:** "General" ฿890 × 100; "Student" ฿0 × 50; a second "General" ฿500 × 20.
- **Steps:**
  1. Add General and Student and press "Next"; go back to Tickets.
  2. Add the second "General" and press "Save changes".
  3. Remove the second "General" row, then remove Student.
  4. Remove General.
- **Expected result:** General and Student save; Student's ฿0 is accepted, and the event's Tickets tab shows it as "Free". The second "General" is refused with "A ticket type with this name already exists for this event." Student is removed. Removing General is refused with "An event must keep at least one ticket type — this is the last one."

### TC-EVT-16 — A ฿890 ticket's receipt shows the 7% VAT inside the price
- **Traces:** US-EVT-06  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Published event with ticket "General" at ฿890, in a workspace with 7% VAT and the standard 5% service fee; attendee "Kanya Thongdee" (kanya@example.com) with an attendee account; payments in test mode.
- **Test data:** One "General" ticket paid with a test card.
- **Steps:**
  1. As Kanya, choose one "General" ticket and read the "Order summary".
  2. Pay, then open "My events" and choose "Download receipt".
- **Expected result:** The summary reads "1 × General" ฿890, "Service fee" ฿44.50 and "Total" ฿934.50: no VAT is added on top. The "VAT Receipt" shows "Amount excluding VAT" ฿873.36, "VAT 7% (included)" ฿61.14 and "Total paid" ฿934.50.

### TC-EVT-17 — Quantity below sold is refused; approval and waitlist
- **Traces:** US-EVT-06  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Published general-admission event with "General" at quantity 100 and 60 sold; a second general-admission event "Small Workshop" whose only ticket is sold out; both with approval and waitlist off.
- **Test data:** Quantity 50; registrant "Somsak Chaiyo", somsak@example.com.
- **Steps:**
  1. In the wizard set General's quantity to 50 and press "Save changes".
  2. Turn on "Require approval", save, and register Somsak on the public page; open the Registrations screen.
  3. Open "Small Workshop" checkout; then turn on "Enable waitlist", save and reopen checkout.
  4. On an in-person draft choose "Reserved seating" and read the waitlist switch.
- **Expected result:** Step 1 is refused with "Cannot set the quantity (50) below the 60 already sold." Somsak's registration reads "Awaiting approval". With the waitlist off the ticket reads "Sold out"; on, "Sold out · join the waitlist". With reserved seating the switch is off and says "Waitlists are for general admission — a freed seat can’t be offered without choosing it."

---

## US-EVT-07 — Pick a public page template, set visibility, and publish

### TC-EVT-18 — Template fills from the event; preview shows unsaved text; publish goes live
- **Traces:** US-EVT-07  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** A complete draft (title, description, venue, a saved ticket type) on Review & publish; publish rights.
- **Test data:** Template "Spotlight"; unsaved title "Riverside Jazz Night"; visibility "Public".
- **Steps:**
  1. Read the template choices and choose "Spotlight".
  2. On Basics change the title without saving; press "Preview".
  3. On Review & publish, keep "Public" and press "Publish event".
  4. Open the public page, then Discover.
- **Expected result:** The wizard offers Classic, Spotlight and Minimal, with "Browse templates". The preview opens in a new tab in Spotlight, titled "Riverside Jazz Night". Publishing opens the event's workspace; the public page uses Spotlight with the event's own details, and the event is listed on Discover.

### TC-EVT-19 — The checklist blocks publishing; cover optional; only Public has a page
- **Traces:** US-EVT-07  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** An in-person draft with a title and a saved ticket type, but no description, venue or cover; publish rights.
- **Test data:** Description "An evening of live jazz."; venue "Riverside Hall"; visibility "Unlisted".
- **Steps:**
  1. Open Review & publish and read the checklist and "Publish event".
  2. Add the description and venue, still no cover; read them again.
  3. Choose "Unlisted" and publish; open its public address and search Discover.
  4. Edit the event again and open Review & publish.
- **Expected result:** The checklist reads "Still needs a description" and "Still needs a venue or an online link", and "Publish event" can't be pressed. Once both are added every item is met and it can be pressed without a cover. The Unlisted event has no public page at its address and is not on Discover. "Visibility" can no longer be changed.

### TC-EVT-20 — Publishing without publish rights is refused
- **Traces:** US-EVT-07  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Member with create rights but no publish rights; a complete draft.
- **Test data:** Role without publish rights.
- **Steps:**
  1. Open Review & publish.
  2. Press "Publish event".
- **Expected result:** The publish is refused with a permission message above the wizard, and the event stays a Draft.

---

## US-EVT-08 — Cancel or delete an event without losing paid registrations

### TC-EVT-21 — An empty draft is deleted only after confirming
- **Traces:** US-EVT-08  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Admin with publish rights; draft "Test Draft" with no registrations or payments.
- **Test data:** "Test Draft".
- **Steps:**
  1. Open its "More actions" menu and choose "Delete".
  2. Read the dialog, then press "Delete".
- **Expected result:** The dialog "Delete Test Draft?" says "This permanently removes the draft and can’t be undone." and nothing is deleted until "Delete" is pressed. Then "Event deleted." shows and the row is gone.

### TC-EVT-22 — Deleting a published event is refused and Cancel is offered
- **Traces:** US-EVT-08  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Admin with publish rights; published event "Riverside Jazz Night" with paid registrations.
- **Test data:** "Riverside Jazz Night".
- **Steps:**
  1. Choose "Delete" on the event and confirm "Delete".
- **Expected result:** The dialog shows "A published event cannot be deleted. Cancel it instead to refund and notify attendees.", the button becomes "Cancel event instead", and the event stays.

### TC-EVT-23 — Cancelling needs a reason, closes registration and emails attendees in their language
- **Traces:** US-EVT-08  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Admin with publish rights; published event with confirmed attendees "Nattaya Srisuk" (English, nattaya@example.com) and "Somchai Rakdee" (Thai, somchai@example.com), and "Malee Sukjai" (English, malee@example.com) awaiting approval.
- **Test data:** Reason "Venue unavailable".
- **Steps:**
  1. Choose "Delete", confirm, then press "Cancel event instead".
  2. With "Reason" empty, try "Cancel event"; then enter the reason and press it.
  3. Read the Events tabs and open the event's public page.
  4. Open the three emails.
- **Expected result:** "Cancel event" can't be pressed until a reason is given; then "Event cancelled. Refunds and attendee notices are being processed." shows. The event leaves Active for Completed as Cancelled, and its public page is gone. Each person gets a cancellation email with "Venue unavailable" and a refund line: Nattaya and Malee in English, Somchai in Thai, Malee's line about her pending registration.

---

## US-EVT-09 — Manage the event program (agenda and speakers)

### TC-EVT-24 — Add and remove sessions; a room clash is refused; speakers via Manage speakers
- **Traces:** US-EVT-09  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Organizer on the Agenda tab of a published event that has speaker "Anong Saelim".
- **Test data:** "Opening Keynote", Day 1, 10:00, 60 min, Keynote, Hall A; "Design Panel", Day 1, 10:30, 30 min, Panel, in Hall A then Hall B; a session with no title.
- **Steps:**
  1. "Add session" for Opening Keynote, then for Design Panel in Hall A.
  2. Add Design Panel in Hall B, then a session with no title.
  3. Remove Opening Keynote, confirm "Remove", and open the public page's agenda.
  4. Open the Speakers tab and press "Manage speakers".
- **Expected result:** The keynote shows under "Day 1" at 10:00. Design Panel in Hall A is refused with "Hall A is already booked for "Opening Keynote" from 10:00 to 11:00. Choose another room or time."; in Hall B it is added after the keynote. The untitled session is refused. "Remove session?" warns attendees won't be told; the keynote then leaves the tab and the public agenda. Anong shows as a card; "Manage speakers" opens the Speakers page for this event.

---

## US-EVT-10 — Adjust ticket inventory after publishing

### TC-EVT-25 — A ticket type added after publishing starts at 0 sold
- **Traces:** US-EVT-10  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Published general-admission event; Organizer on its Tickets tab.
- **Test data:** "VIP", ฿1,250, quantity 200.
- **Steps:**
  1. Press "Add ticket", enter VIP and press "Add ticket".
  2. Read the new card.
  3. Open the public page.
- **Expected result:** The card reads "VIP", "On sale", "฿1,250" and "0/200 sold" at 0%. The public page lists VIP at ฿1,250.

### TC-EVT-26 — A duplicate name is refused; a reserved event's new type gets its quantity from seats
- **Traces:** US-EVT-10  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Published general-admission event with ticket "General"; a published reserved event whose seats have ticket types.
- **Test data:** New ticket named "General".
- **Steps:**
  1. On the first event, "Add ticket" named "General".
  2. On the reserved event, open "Add ticket" and read "Quantity".
- **Expected result:** Step 1 is refused with "A ticket type with this name already exists for this event." On the reserved event "Quantity" reads 0, read-only, with "Set by the seat map".

---

## US-EVT-11 — Organize events with categories

### TC-EVT-27 — Browse, create and rename categories; a duplicate name is refused
- **Traces:** US-EVT-11  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Admin with settings access; category "Conference" used by published event "TechConf BKK 2026".
- **Test data:** New category "Workshop" with an icon, a colour and a description; duplicate "Conference"; rename "Conference" to "Conferences & Summits".
- **Steps:**
  1. Open Event categories; search "Work", sort "Most events" and switch to list.
  2. Press "New category", fill in "Workshop" while watching "Preview", and press "Save category".
  3. Create another category named "Conference".
  4. Rename "Conference" to "Conferences & Summits"; open TechConf's public page.
- **Expected result:** Each category shows its icon, colour, name, description and event count; search, sort and list view work. The preview follows the name, icon and colour as they change; "Category saved." shows. The duplicate is refused with "A category with this name already exists in your workspace." The public page shows "Conferences & Summits".

### TC-EVT-28 — An empty category is deleted; one in use is refused
- **Traces:** US-EVT-11  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Admin with one empty category and one used by 12 events.
- **Test data:** Empty "Old"; "Seminar" used by 12 events.
- **Steps:**
  1. Delete "Old" and confirm "Delete".
  2. Delete "Seminar" and confirm.
- **Expected result:** "Delete Old?" says "This permanently removes the category."; then "Category deleted." shows and it is gone. "Seminar" is refused with "This category has 12 events. Move them to another category before deleting."

---

## US-EVT-12 — See my events on a calendar and what's coming up

### TC-EVT-29 — Month calendar, day roll-up and upcoming countdown
- **Traces:** US-EVT-12  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Organizer; today is 2026-07-27 in Bangkok; 6 events in July 2026, 4 of them on 30 July and none on 20 July; "Riverside Jazz Night" on 30 July at 83 of 100.
- **Test data:** 30 July (busy); 20 July (empty).
- **Steps:**
  1. Open Events › "Calendar" and read the header.
  2. Read the 30 July square, choose it, then choose 20 July.
  3. Go to the next month, then press "Today".
  4. Open "Upcoming events".
  5. Click an event on the calendar, and one on Upcoming events.
- **Expected result:** The header reads "July 2026" and "6 events this month". 30 July shows three events and "+1 more"; choosing it lists all four by time; 20 July reads "No events scheduled." "Today" returns to July. Upcoming lists events soonest first; Riverside Jazz Night reads "3 days left" and "Filled 83%". Each click opens that event's workspace.

---

## US-EVT-13 — Duplicate an event to relaunch quickly

### TC-EVT-30 — Duplicate makes a fresh "(Copy)" draft
- **Traces:** US-EVT-13  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Published "Annual Gala" with 312 registrations, an agenda, speakers, a reserved seat map with typed seats, the Spotlight template, Public visibility, and ticket types with sales windows.
- **Test data:** "Annual Gala".
- **Steps:**
  1. Choose "Duplicate" in its row menu.
  2. Open "Annual Gala (Copy)" in its workspace and in the wizard.
  3. Read its public address, its ticket types' sales windows on the Tickets screen, and its Assign seats.
- **Expected result:** A Draft "Annual Gala (Copy)" appears under Active with the same details, date, seat layout, ticket types, agenda, speakers, template and visibility, and 0 registrations and 0 sold. It has its own public address, its ticket types have no sales window, and its seats read "No ticket type".

### TC-EVT-31 — One press of Duplicate makes one copy
- **Traces:** US-EVT-13  ·  **Priority:** Low  ·  **Type:** Edge
- **Preconditions:** Organizer on Events with create rights.
- **Test data:** Any event.
- **Steps:**
  1. Choose "Duplicate" on an event.
  2. Immediately press it again.
- **Expected result:** The item reads "Working…" and can't be pressed until the copy exists; exactly one copy is made and Active goes up by one.

---

## US-EVT-14 — Monitor an event's performance from its workspace

### TC-EVT-32 — Overview numbers, copy link, Registrations and Attendees tabs
- **Traces:** US-EVT-14  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Organizer with finance and registration access; published event at 120 of 300, revenue ฿256,800; registrations that are Paid, Pending and Refunded; confirmed attendees.
- **Test data:** The event above.
- **Steps:**
  1. On Overview read the four tiles; press "Copy" by the public link.
  2. Open Registrations; choose each filter and page through.
  3. Open Attendees.
- **Expected result:** Tiles read "120/300" with 40%, "฿256,800", Tickets sold and Days left; "Copy" turns to "Copied" and the link is on the clipboard. Each filter shows its count and rows with attendee and reference, tickets, amount in ฿, registered date and time, and status. Attendees shows "{n} confirmed" with each name, email, registrations and seats.

### TC-EVT-33 — Revenue is hidden without finance access
- **Traces:** US-EVT-14  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Member with registration access but no finance access; published event.
- **Test data:** Role without finance access.
- **Steps:**
  1. Open the event's Overview.
- **Expected result:** "Revenue" shows "—"; Registrations, Tickets sold and Days left still show.

---

## US-EVT-15 — Promote an event with share links and a printable flyer

### TC-EVT-34 — Copy link, share channels and flyer download
- **Traces:** US-EVT-15  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Organizer on a published, Public event's workspace.
- **Test data:** "TechConf BKK 2026", 15 August 2026, "Riverside Hall, Bangkok".
- **Steps:**
  1. Press "Share", then "Copy link".
  2. Choose Facebook, X, LINE, WhatsApp and Email in turn.
  3. Point at the flyer and press "Save".
- **Expected result:** The label reads "Copied!" and the public link is on the clipboard. Facebook and LINE open their share pages with the link; X, WhatsApp and Email also carry "Join me at TechConf BKK 2026 — {date} at Riverside Hall, Bangkok"; Email opens a message titled "TechConf BKK 2026". "Save" downloads a PNG flyer showing the title, date, place and link.

### TC-EVT-35 — Sharing a non-public event warns the link may not be reachable
- **Traces:** US-EVT-15  ·  **Priority:** Won't  ·  **Type:** Negative
_Withdrawn: a warning when sharing a non-public event is not in the web app; the id stays valid — not run._


---

<a id="tc-e04"></a>

# Public Event Pages — Test Cases

Epic E4 — Public Event Pages. Area code: PAGE. Locale context: Thai market, ฿ (VAT 7% inclusive), PromptPay, Asia/Bangkok timezone, EN/TH bilingual.

---

## US-PAGE-01 — Open a shareable event page

### TC-PAGE-01 — Open a published event from its shared link as a guest
- **Traces:** US-PAGE-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** A published, public in-person event with one ticket type. Tester is not signed in (private window).
- **Test data:** "Bangkok Tech Meetup" at /e/bkk-tech-meetup, 15 Aug 2026 18:00–21:00 Bangkok, venue "True Digital Park", ticket "General" ฿500.
- **Steps:**
  1. Open the event's link in a private window.
  2. Read the page top to bottom without signing in.
- **Expected result:** The page shows "Bangkok Tech Meetup", "Aug 15, 2026", "18:00 – 21:00", "True Digital Park" and the "General" ticket at "฿500". No sign-in or sign-up prompt appears anywhere.

### TC-PAGE-02 — A link to an event that isn't live shows "This event page isn't available."
- **Traces:** US-PAGE-01  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** In one workspace: a draft event, an event published as Unlisted, and a cancelled event.
- **Test data:** (a) draft "Launch Party" at /e/launch-party; (b) Unlisted "Team Offsite" at /e/team-offsite; (c) cancelled "Rainy Day Fair" at /e/rainy-day-fair; (d) /e/does-not-exist-2026.
- **Steps:**
  1. In a private window, open each of the four links.
  2. On one of them, click "Back to home".
- **Expected result:** Each link shows "Not found" with "This event page isn't available." — never another event's content or a broken screen. "Back to home" opens the Discover page ("What's on in Bangkok").

### TC-PAGE-03 — Empty sections are left out; dates read in English and Bangkok time
- **Traces:** US-PAGE-01  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** A published event written in Thai, with a description and one ticket type but no highlights, agenda, speakers or FAQs.
- **Test data:** "งานสัมมนากรุงเทพ", 20 Sep 2026 09:00–12:00 Bangkok, ticket "ทั่วไป" ฿300.
- **Steps:**
  1. Open the event's page as a guest.
  2. Scroll the whole page, reading every section heading.
  3. Set the computer's time zone to Europe/London and reload.
- **Expected result:** The title and ticket name show in Thai exactly as typed. The date reads "Sep 20, 2026" and the time "09:00 – 12:00" both times, whatever the device's zone. There is no "Highlights", "Agenda", "Speakers" or "Frequently asked" heading: those sections are absent, not empty.

---

## US-PAGE-02 — Event identity and one-tap Register

### TC-PAGE-04 — Register is in reach everywhere and opens the Register page in the same tab
- **Traces:** US-PAGE-02  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Two published events with a ticket on sale: one in the Classic design, one in Vibrant.
- **Test data:** "Bangkok Tech Meetup" (Classic), 15 Aug 2026 18:00–21:00, venue "True Digital Park"; "Songkran Beats" (Vibrant); each with ticket "General" ฿500.
- **Steps:**
  1. Open "Bangkok Tech Meetup" and read the title block and the buttons in the top bar and hero.
  2. Scroll to "Tickets" and then to the foot of the page, watching the top bar.
  3. Click any Register button.
  4. Open "Songkran Beats" and read the same buttons.
- **Expected result:** The top shows "Bangkok Tech Meetup" with "Aug 15, 2026", "18:00 – 21:00" and "True Digital Park". Classic shows "Register" in the top bar, which stays visible while scrolling, "Register now" in the hero, "Register" on each ticket and "Register now" at the foot. Any of them opens the "Register" page for Bangkok Tech Meetup in the same tab. Vibrant labels the same buttons "Get tickets".

### TC-PAGE-05 — A missing or broken cover shows the brand gradient
- **Traces:** US-PAGE-02  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Two published events in the Classic design: (A) with no cover image; (B) with a cover image.
- **Test data:** (A) "Future Expo 2027"; (B) "Riverside Run", its cover image blocked in the browser's developer tools.
- **Steps:**
  1. Open (A) and look at the hero.
  2. Open (B) with its cover image blocked and look at the hero.
- **Expected result:** Both heroes show the green brand gradient behind the title. Neither shows a broken-image icon or an empty frame.

---

## US-PAGE-03 — Clear about section and online-event handling

### TC-PAGE-06 — An online event says "Online event" and never shows the meeting link
- **Traces:** US-PAGE-03  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** A published online event whose "Meeting link" was filled in the event wizard, with a free ticket. Tester is a guest.
- **Test data:** "Remote UX Workshop", Meeting link https://meet.example.com/xyz-secret, guest "Nok Srisuk" nok@example.com.
- **Steps:**
  1. Open the event's page and read the hero and the details list under About.
  2. Search the page for "meet.example.com" and look for any address or map.
  3. Register as Nok and open the confirmation email.
- **Expected result:** The page shows "Online event" (hero and Format row) and "Join link sent after you register". "meet.example.com" appears nowhere on it, and there is no address, map or "Get directions". The confirmation email carries the meeting link.

### TC-PAGE-07 — An in-person event lists its details and leaves out a blank address
- **Traces:** US-PAGE-03  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** A published in-person event with a description, venue, category and one ticket type, but no address.
- **Test data:** "Chiang Mai Coffee Fair", venue "TCDC Chiang Mai", address blank, category "Food & Drink", ticket "General" ฿250.
- **Steps:**
  1. Open the page as a guest and find the About section.
  2. Read the details list row by row and look beneath it.
- **Expected result:** The organizer's description shows under About. The details list shows Date, Time, Venue "TCDC Chiang Mai", Category "Food & Drink" and Price "From ฿250"; there is no Address row and no empty label. A map with "Get directions" shows beneath the list.

---

## US-PAGE-04 — Highlights, agenda and speakers

### TC-PAGE-08 — Highlights, agenda and speakers show in the organizer's order
- **Traces:** US-PAGE-04  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** (A) A published event with 3 highlights (wizard "Highlights"), a 4-session agenda ("Agenda" page) and 3 speakers ("Speakers" page). (B) A published event with none of them.
- **Test data:** Highlights "Free WiFi", "Lunch included", "Certificate". Agenda: 09:00–09:30 Registration; 09:30–10:15 Keynote, Room A, Anan; 10:15–10:30 Break; 10:30–11:30 Panel, Room A, Bua and Chai. Speakers added in the order Anan, Bua, Chai, each with a talk title.
- **Steps:**
  1. Open (A) and read the highlights.
  2. Read the "Agenda" section, then the "Speakers" section.
  3. Open (B).
- **Expected result:** The highlights show with their icons in the order entered. "Agenda" lists Registration, Keynote, Break, Panel, with times like "09:30 – 10:15" and lines like "Room A · Anan". "Speakers" shows Anan, Bua, Chai in that order, each with initials and talk title. (B) has no highlights, "Agenda" or "Speakers" section.

---

## US-PAGE-05 — Ticket tiers, pricing and availability

### TC-PAGE-09 — Ticket cards run cheapest first with VAT-inclusive prices and Free/RSVP labels
- **Traces:** US-PAGE-05  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** A published general-admission event whose ticket types were created in this order: "VIP", "General", "Student", "Guest list".
- **Test data:** "VIP" ฿1,500; "General" ฿500; "Student" marked Free; "Guest list" priced ฿0 and not marked Free.
- **Steps:**
  1. Open the page as a guest and scroll to "Tickets".
  2. Read each card's name, price and button, in order.
- **Expected result:** The cards run "Student" "Free", "Guest list" "RSVP", "General" "฿500", "VIP" "฿1,500". No VAT is added on top of any price. Each card has its own "Register" button.

### TC-PAGE-10 — A sold-out ticket says "Sold out" and can't be bought
- **Traces:** US-PAGE-05  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** A published event on sale with one ticket available and one sold out.
- **Test data:** "General" ฿500 with 50 left; "VIP" ฿1,500 with all 20 sold.
- **Steps:**
  1. Open the page and read the "VIP" card.
  2. Click VIP's "Register".
  3. On the "Register" page, try to choose VIP, then "General".
- **Expected result:** The VIP card says "Sold out". Its Register opens the event's "Register" page, where VIP can't be bought (it can't be chosen, or offers "Sold out · join the waitlist" when its waitlist is open). "General" can still be chosen.

### TC-PAGE-11 — A ticket with 10 or fewer left says "Going fast"
- **Traces:** US-PAGE-05  ·  **Priority:** High  ·  **Type:** Edge
- **Preconditions:** A published event with a ticket type capped at 100.
- **Test data:** "General" ฿500, sold down to 11 left, then to 3 left.
- **Steps:**
  1. With 11 left, open the page and read the "General" card.
  2. Sell 8 more, reload and read the card again.
- **Expected result:** At 11 left the card shows no urgency line. At 3 left it says "Going fast — only 3 left".

### TC-PAGE-11b — A reserved event's page shows which seats each ticket sells; other events show no seat map
- **Traces:** US-PAGE-05  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** (A) A published in-person event with reserved seating, 7 rows of 16 seats; ticket types created in the order VIP, Normal, Free; on "Assign seats", rows 1–2 VIP, rows 3–6 Normal, row 7 Free. (B) A published general-admission event. (C) A published online event. The organizer of (A) is signed in on a second browser.
- **Test data:** (A) "Bangkok Symphony Night": VIP ฿2,500, Normal ฿890, Free.
- **Steps:**
  1. As a guest, open (A) and read the ticket cards, the "Seat map" and its key; compare the colours with the organizer's preview on "Assign seats".
  2. Narrow the window to phone width (375 px).
  3. As the organizer, on "Assign seats" give row 6 seats 1–8 to VIP and save; as the guest, reload (A).
  4. Open (B) and (C).
- **Expected result:** (1) Cards run Free "Row 7", Normal "Rows 3–6", VIP "Rows 1–2". The seat map shows "Stage" and rows 1–7, each seat in its ticket's colour, the same colours as the organizer's preview; the key reads "Free · Row 7 · 16 seats", "Normal · Rows 3–6 · 64 seats", "VIP · Rows 1–2 · 32 seats". No seat shows as sold or held. (2) The seat map scrolls inside its own block; the page never scrolls sideways. (3) Normal reads "Rows 3–5, row 6 seats 9–16" and VIP "Rows 1–2, row 6 seats 1–8"; the key reads "Normal · Rows 3–5, row 6 seats 9–16 · 56 seats" and "VIP · Rows 1–2, row 6 seats 1–8 · 40 seats". (4) Neither shows a seat map, and no card names rows or seats.

---

## US-PAGE-06 — Frequently asked questions

### TC-PAGE-12 — The first FAQ answer is open, the rest open on click; no FAQs, no section
- **Traces:** US-PAGE-06  ·  **Priority:** Low  ·  **Type:** Functional
- **Preconditions:** (A) A published event with 4 FAQs, added outside the web app (it has no FAQ editor yet). (B) A published event with none.
- **Test data:** (A) Q1 "Is parking available?", Q2 "Is lunch included?", Q3 "Can I bring a guest?", Q4 "Is there a dress code?".
- **Steps:**
  1. Open (A) and find "Frequently asked".
  2. Note which answers are open.
  3. Click Q3, then click Q1.
  4. Open (B).
- **Expected result:** On load only Q1's answer is open. Clicking Q3 opens it; clicking Q1 closes it. (B) has no FAQ section.

---

## US-PAGE-07 — Add the event to my calendar

### TC-PAGE-13 — Add to calendar gives a Bangkok-time entry
- **Traces:** US-PAGE-07  ·  **Priority:** Won't  ·  **Type:** Functional
_Withdrawn: Add to calendar is not in the web app; the id stays valid — not run._

### TC-PAGE-14 — An online event's calendar entry has no join link; adding again updates it
- **Traces:** US-PAGE-07  ·  **Priority:** Won't  ·  **Type:** Edge
_Withdrawn: Add to calendar is not in the web app; the id stays valid — not run._

---

## US-PAGE-08 — Rich search results and social sharing

### TC-PAGE-15 — Every way of sharing gives the same clean public address
- **Traces:** US-PAGE-08  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** A published public event. The organizer is signed in on the event's page in the console.
- **Test data:** "Bangkok Tech Meetup" at /e/bkk-tech-meetup.
- **Steps:**
  1. Click "View page" and read the address of the tab it opens.
  2. On the "Landing page" card, read the link and click "Copy".
  3. Click "Share", then "Copy link"; then choose Facebook, X and LINE and read the link each one shares.
  4. Open the copied link in a private window and read the browser tab.
- **Expected result:** Every link is the same address ending /e/bkk-tech-meetup, with nothing added: no name, email or tracking code. The page opens for the guest, and the tab reads "Bangkok Tech Meetup · Eventa".

### TC-PAGE-16 — Previews are marked "noindex"; an unpublished event has no page
- **Traces:** US-PAGE-08  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** The organizer is signed in. One published public event and one draft.
- **Test data:** Published "Bangkok Tech Meetup"; draft "Launch Party" at /e/launch-party.
- **Steps:**
  1. On "Landing Pages", preview "Bangkok Tech Meetup" in any design and check the page's search-engine tags.
  2. Open /e/bkk-tech-meetup and check the same tags.
  3. Open /e/launch-party.
- **Expected result:** (1) The preview carries a robots "noindex" tag. (2) The real page carries none. (3) The draft shows "Not found" with "This event page isn't available.", so there is nothing to index.

---

## US-PAGE-09 — Preview any template with unsaved content

### TC-PAGE-17 — Preview shows typed, unsaved values and saves nothing
- **Traces:** US-PAGE-09  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** The organizer is signed in. (A) A draft "Launch Party", in person at "Siam Hall", no highlights. (B) A published public event "Bangkok Tech Meetup" with tickets and an agenda.
- **Test data:** Typed title "Draft Launch Party", highlight "Free welcome drink", location "Online".
- **Steps:**
  1. Open (A) in the event wizard and, using the step rail rather than "Next", type the title and highlight and switch to "Online"; click "Preview".
  2. Read the new tab, then close it and reload the wizard.
  3. Open (B) in the wizard, change its title to "Draft Launch Party", go to "Review & publish" by the step rail (nothing saved) and click "Preview landing page".
  4. In a private window, open /e/bkk-tech-meetup.
  5. On "Landing Pages", choose "Bangkok Tech Meetup" under "Preview with" and click "Preview" on Minimal.
- **Expected result:** (1–2) The tab shows the chosen design with "Draft Launch Party", "Online event" and "Free welcome drink", and no tickets, agenda or speakers; it is marked "noindex". After the reload the wizard still shows "Launch Party", in person, no highlight. (3) B's real page opens, with its tickets and agenda, under the typed title. (4) The public page still shows "Bangkok Tech Meetup". (5) A new tab shows B in the Minimal design with its saved details.

### TC-PAGE-18 — A blocked pop-up says how to allow it
- **Traces:** US-PAGE-09  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** The organizer is signed in and editing an event in the wizard. The browser blocks pop-ups for the site.
- **Test data:** Any event; pop-up blocker on.
- **Steps:**
  1. Click "Preview" in the top bar.
  2. Go to "Review & publish" and click "Preview landing page".
- **Expected result:** No tab opens; each time the message "Your browser blocked the preview — allow pop-ups for this site and try again." appears, and everything typed stays in the wizard.

---

## US-PAGE-10 — Choose a design, brand it, and publish under a stable link

### TC-PAGE-19 — Publish in a chosen design, then switch design with nothing lost
- **Traces:** US-PAGE-10  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** The organizer is signed in. A draft with a title, description, venue and one saved ticket type, and the public address bkk-tech-meetup.
- **Test data:** "Bangkok Tech Meetup", ticket "General" ฿500.
- **Steps:**
  1. On "Review & publish", pick "Spotlight" and click "Publish event".
  2. Open /e/bkk-tech-meetup in a private window.
  3. On "Landing Pages", choose "Bangkok Tech Meetup" under "Preview with" and click "Use template" on Vibrant.
  4. Reload /e/bkk-tech-meetup.
- **Expected result:** (1) The event is published. (2) The page is in Spotlight: a register card beside the title shows "Starting from", "฿500" and "Register now". (3) Vibrant's button now reads "In use". (4) The same address now shows Vibrant ("Get tickets"), with the same title, description, venue and ticket.

### TC-PAGE-20 — "Publish event" stays disabled until the checklist is complete
- **Traces:** US-PAGE-10  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** The organizer is signed in. Draft (A) saved with only a title and a start date. Draft (B) with reserved seating and one seat left without a ticket type.
- **Test data:** (A) "Chiang Mai Coffee Fair", in person, no description, venue or ticket type; later: a description, venue "TCDC Chiang Mai", ticket "General" ฿250. (B) "Bangkok Symphony Night".
- **Steps:**
  1. Open (A) on "Review & publish" and read the "Publish checklist".
  2. Point at "Publish event".
  3. Add the description, venue and ticket type (saving each step with "Next"), then return to "Review & publish".
  4. Open (B) on "Review & publish".
- **Expected result:** (1) The list shows "Title added", "Still needs a description", "Still needs a venue or an online link" and "Still needs at least one ticket type". (2) "Publish event" is disabled and its tip names what is missing; the event stays a draft. (3) Every line is ticked ("Description added", "Location added", "At least one ticket type") and "Publish event" is enabled. (4) The list shows "Still needs a ticket type for every seat" and "Publish event" is disabled.

### TC-PAGE-21 — A taken address is refused; the accent colour is kept or replaced by the default
- **Traces:** US-PAGE-10  ·  **Priority:** High  ·  **Type:** Edge
- **Preconditions:** The organizer is signed in. In the same workspace, "Bangkok Tech Meetup" uses bkk-tech-meetup, and the published "Chiang Mai Coffee Fair" uses cm-coffee-fair.
- **Test data:** Addresses "BKK Tech Meetup" and "Chiang Mai Coffee Fair 2026"; accent colours "#E91E63" and "pink".
- **Steps:**
  1. Open "Chiang Mai Coffee Fair" on "Review & publish", set "Public address" to "BKK Tech Meetup" and click "Save changes" in the top bar.
  2. Set it to "Chiang Mai Coffee Fair 2026" and save; open the new and the old address.
  3. Set "Accent colour" to "#E91E63" and save.
  4. Set it to "pink" and save.
- **Expected result:** (1) The message "The address "bkk-tech-meetup" is already in use — pick another." appears and nothing is saved. (2) The box shows "chiang-mai-coffee-fair-2026" and the hint "The link you share: /e/chiang-mai-coffee-fair-2026"; the new address opens the page and the old one shows "Not found". (3) The box keeps "#E91E63". (4) The box shows "#2563EB", with no error.


---

<a id="tc-e05"></a>

# Sell Tickets & Run Promotions — Test Cases

Epic E5 — TKT. Test basis: user stories US-TKT-01 … US-TKT-12 and their acceptance criteria.
Locale: Thai market — ฿ (whole baht), 7% VAT recalculated at checkout, PromptPay, Asia/Bangkok time, EN/TH, 8-seat booking cap.

---

## US-TKT-01 — Create paid and free ticket types

### TC-TKT-01 — Create a paid ticket type with a sales window
- **Traces:** US-TKT-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as an Event Organizer; the general-admission event "Bangkok Jazz Night 2026" exists.
- **Test data:** Ticket name "General Admission"; Paid; Price 1000; Quantity available 500; Sales start tomorrow; Sales end in 30 days; Per-order limit 4.
- **Steps:**
  1. On Tickets, choose "New ticket type".
  2. Enter the data above with Event "Bangkok Jazz Night 2026" and choose "Save ticket type".
  3. Choose Edit on the new card, set Sales start to today and save.
- **Expected result:** (2) "Ticket type saved." shows and a card reads General Admission · Bangkok Jazz Night 2026, ฿1,000, "0 / 500 sold", Scheduled. (3) The card reads On sale.

### TC-TKT-02 — Create a Free ticket type (no price, no payment)
- **Traces:** US-TKT-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as an Event Organizer; the published event "Chiang Mai Design Week" has no paid ticket on sale.
- **Test data:** Ticket name "Community Pass"; Free; Quantity available 200; no sales dates. Guest buyer "Nok Srisuk", nok@eventa-test.co.th.
- **Steps:**
  1. On Tickets, choose "New ticket type", pick the event and choose "Free".
  2. Enter the name and quantity and choose "Save ticket type".
  3. As a guest, open the event's checkout, choose "Community Pass" and enter Nok's details.
  4. Choose "Confirm registration".
- **Expected result:** (1) The Price box disappears. (2) The card shows a "Free" badge. (3) There is no Payment step and the bar reads "Free admission". (4) "You're registered!" shows; nothing is charged.

### TC-TKT-03 — Reject invalid ticket setup (end-before-start, over-cap and duplicate-name)
- **Traces:** US-TKT-01  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Signed in as an Event Organizer; "Bangkok Jazz Night 2026" already has a ticket type "General Admission".
- **Test data:** (a) Sales start in 10 days, Sales end in 2 days; (b) Quantity available 4, Per-order limit 6; (c) Ticket name "General Admission"; (d) Per-order limit 9.
- **Steps:**
  1. In "New ticket type" for this event, try each of (a)–(d) on its own and choose "Save ticket type".
- **Expected result:** (a) "The sales end date must come after the sales start date." (b) "The per-order limit (6) can't exceed the 4 seats available." (c) "A ticket type with this name already exists for this event." (d) The Per-order limit box takes at most 8, so the form doesn't save. Each time nothing is created and the panel stays open to fix.

---

## US-TKT-02 — Adjust a ticket type safely after it is live

### TC-TKT-04 — Increase capacity on a sold-out ticket returns it to On sale
- **Traces:** US-TKT-02  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Paid ticket "General Admission" on the published "Bangkok Jazz Night 2026" has sold 500 of 500; checkout shows it as "Sold out".
- **Test data:** New Quantity available 700.
- **Steps:**
  1. On Tickets, choose Edit on the card.
  2. Set Quantity available to 700 and choose "Save ticket type".
  3. Open the event's checkout as a guest.
- **Expected result:** The card reads "500 / 700 sold" and On sale. At checkout "General Admission" can be chosen and bought again.

### TC-TKT-05 — Cannot lower quantity below seats already sold
- **Traces:** US-TKT-02  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** A paid ticket type has sold 640 of 800.
- **Test data:** New Quantity available 500.
- **Steps:**
  1. Choose Edit on the card, set Quantity available to 500 and save.
- **Expected result:** The panel shows "Cannot set the quantity (500) below the 640 already sold."; the card still reads "640 / 800 sold".

### TC-TKT-06 — Price / paid-free / event locked after sales begin; concurrent-edit guard
- **Traces:** US-TKT-02  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** On "Bangkok Jazz Night 2026", "General Admission" (฿1,000) has sold at least one; "Early Bird" has sold none. The organizer is signed in on two browsers, A and B.
- **Test data:** Price 1200; Free.
- **Steps:**
  1. Edit "General Admission", set Price to 1200 and save; then choose "Free" and save.
  2. Look at the Event field in the panel.
  3. Edit "Early Bird", set Price to 1200 and save.
  4. Open "General Admission" for editing in A and B; save a new name in A, then save in B.
- **Expected result:** (1) Both saves show "The price and paid/free setting can't change after sales began — create a new ticket type instead."; nothing changes. (2) Event is locked with "A tier belongs to its event and cannot be moved." (3) Saved; the card reads ฿1,200. (4) B sees "This ticket type changed elsewhere. Reload and try again."; A's name stands.

---

## US-TKT-03 — Availability that manages itself

### TC-TKT-07 — Scheduled ticket can't be bought before its sales start
- **Traces:** US-TKT-03  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** "Early Bird" on the published "Bangkok Jazz Night 2026" has Sales start tomorrow.
- **Test data:** —
- **Steps:**
  1. On Tickets, read the "Early Bird" card.
  2. As a guest, open the event's checkout.
- **Expected result:** The card reads Scheduled. At checkout "Early Bird" reads "Not on sale yet" and can't be chosen.

### TC-TKT-08 — Last-seat concurrency: exactly one buyer wins, no oversell
- **Traces:** US-TKT-03  ·  **Priority:** High  ·  **Type:** Edge
- **Preconditions:** Paid general-admission ticket "General Admission" has sold 499 of 500; the event's waitlist is off.
- **Test data:** Buyers "Somchai Jaidee" (somchai@example.com) and "Nok Srisuk" (nok@eventa-test.co.th), 1 ticket each, PromptPay.
- **Steps:**
  1. Open the checkout in two browsers, choose the ticket and fill in each buyer.
  2. Choose pay in both at the same moment.
  3. Complete the winner's PromptPay payment.
  4. Reload the checkout.
- **Expected result:** (2) One buyer gets an order and a PromptPay QR; the other sees "Only 0 left — please reduce your quantity." and no order. (4) "General Admission" reads "Sold out" and its card on Tickets reads "500 / 500 sold" — no oversell.

### TC-TKT-09 — Purchase refused outside the sales window and while paused
- **Traces:** US-TKT-03  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** On the published "Bangkok Jazz Night 2026": "Late Pass" had Sales start 10 days ago and Sales end yesterday; "General Admission" is on sale inside its window.
- **Test data:** Guest buyer "Somchai Jaidee", somchai@example.com.
- **Steps:**
  1. As the guest, choose "Late Pass" at checkout and pay.
  2. On Tickets, choose "Pause sales" on "General Admission"; then open the checkout.
  3. Choose "Resume sales" on "General Admission".
  4. Choose "Pause sales", then "Resume sales", on "Late Pass".
- **Expected result:** (1) "Ticket sales have ended."; no order. (2) The card reads Paused; checkout shows "General Admission" as "Paused" and it can't be chosen. (3) The card reads On sale again. (4) Resume is refused with "Sales for this ticket type have already ended — extend the sales end date first."; it stays Paused.

---

## US-TKT-04 — See all my tickets and inventory at a glance

### TC-TKT-10 — Filter by status and search by name or event
- **Traces:** US-TKT-04  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** The organizer has 6 ticket types: 3 On sale, 1 Scheduled, 2 Paused. "General Admission" on "Bangkok Jazz Night 2026" has sold 640 of 800; "Jazz Brunch" belongs to "Hua Hin Food Fair".
- **Test data:** Tab "Paused"; search "JAZZ"; event filter "Bangkok Jazz Night 2026".
- **Steps:**
  1. Open Tickets and read the "General Admission" card.
  2. Choose the "Paused" tab.
  3. Choose "All" and type "JAZZ" in "Search ticket types…".
  4. Clear the search and pick "Bangkok Jazz Night 2026" in the event filter.
- **Expected result:** (1) The card shows its name, event, ฿ price, On sale and "640 / 800 sold" with an 80% bar. (2) Only the 2 paused cards show and the tab reads 2. (3) Every ticket of Bangkok Jazz Night 2026 and "Jazz Brunch" show, whatever the letter case. (4) Only that event's tickets show.

### TC-TKT-11 — No-match empty state
- **Traces:** US-TKT-04  ·  **Priority:** Low  ·  **Type:** Edge
- **Preconditions:** The organizer has at least one ticket type, none matching the search.
- **Test data:** Search "zzzz-nonexistent".
- **Steps:**
  1. Type the search term in "Search ticket types…".
  2. Choose "Clear filters".
- **Expected result:** (1) "No ticket types match" shows with "Clear filters". (2) All cards come back.

---

## US-TKT-05 — Retire a ticket type without harming existing holders

### TC-TKT-12 — Delete an unsold ticket vs. retire a sold ticket
- **Traces:** US-TKT-05  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** "Bangkok Jazz Night 2026" has four ticket types, including "Test Ticket" (never sold) and "VIP" (210 sold); nobody is checking out.
- **Test data:** —
- **Steps:**
  1. On Tickets, choose Delete on "Test Ticket" and confirm "Delete".
  2. Do the same for "VIP".
  3. Open the event's checkout, then look up a VIP holder in Registrations.
- **Expected result:** Each delete shows "Ticket type deleted." and the card leaves the list. Checkout offers neither ticket. All 210 VIP holders still have valid tickets; no order was refunded or cancelled.

### TC-TKT-13 — Delete blocked during a checkout, or for the last ticket type
- **Traces:** US-TKT-05  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** A guest has chosen to pay for "General Admission" by PromptPay and the QR is still showing. "Chiang Mai Design Week" has one ticket type, "Community Pass".
- **Test data:** —
- **Steps:**
  1. Choose Delete on "General Admission" and confirm.
  2. Choose Delete on "Community Pass" and confirm.
- **Expected result:** (1) The dialog shows "Someone is checking out with this ticket type — pause it and try again shortly." (2) The dialog shows "An event must keep at least one ticket type — this is the last one." Both cards stay.

### TC-TKT-13b — A ticket type that still has seats can't be deleted or retired
- **Traces:** US-TKT-05  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Published reserved event, 7 rows × 16 seats: rows 1–2 VIP, rows 3–6 Normal, row 7 Free. Free and Normal have never sold; VIP has sold 10; nobody is checking out.
- **Test data:** —
- **Steps:**
  1. On Tickets, delete Free and confirm.
  2. Delete VIP and confirm.
  3. On the event's "Assign seats" step, set row 7 to Normal under "Ticket types by row" and save.
  4. Delete Free again and confirm.
- **Expected result:** (1) "Free still has seats. Give them to another ticket type first."; Free stays with its 16 seats. (2) The same for VIP; its 10 holders are untouched. (3) Free reads "0 / 0 sold" and Normal "0 / 80 sold". (4) "Ticket type deleted."; Free leaves the list.

---

## US-TKT-06 — Share a ticket's registration link and QR

### TC-TKT-14 — Share view gives a link and matching QR, copy and download
- **Traces:** US-TKT-06  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** "General Admission" belongs to the published, public "Bangkok Jazz Night 2026".
- **Test data:** —
- **Steps:**
  1. On the card, choose "Share link & QR".
  2. Choose "Copy link" and paste it into a note.
  3. Choose "Download QR".
  4. Scan the QR with a phone.
- **Expected result:** (1) "Share ticket type" shows "General Admission · Bangkok Jazz Night 2026", a QR and the registration link, with no warning. (2) The button reads "Copied" for a moment; the pasted link matches the one shown. (3) An SVG image of the QR downloads. (4) The QR reads the same link.

### TC-TKT-15 — Unpublished event shows a publish-first warning
- **Traces:** US-TKT-06  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** "Early Bird" belongs to a draft event.
- **Test data:** —
- **Steps:**
  1. On the card, choose "Share link & QR".
- **Expected result:** The link and QR show with "This event isn't published yet — publish it first, or the link and QR won't open for anyone."

---

## US-TKT-07 — Create a discount code with a quick generator

### TC-TKT-16 — Generate a unique code and create a percentage promotion
- **Traces:** US-TKT-07  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in as an Event Organizer; "Bangkok Jazz Night 2026" exists.
- **Test data:** Percent 25; Applies to "Bangkok Jazz Night 2026"; Usage limit 500; Per-user limit 1; Start date today; End date in 30 days.
- **Steps:**
  1. On Discounts, choose "New code", then "Generate".
  2. Enter the data above and choose "Save code".
  3. Create a second code the same way, with a generated code and Start date tomorrow.
- **Expected result:** (1) The Code box fills with a word plus digits (e.g. "PROMO42") that no code in the workspace uses. (2) "Discount code saved."; the row reads the code, "25% off", "Bangkok Jazz Night 2026", "0 / 500", its dates and Active. (3) The second row reads Scheduled.

### TC-TKT-17 — Reject invalid or already-expired code setups
- **Traces:** US-TKT-07  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Signed in as an Event Organizer, in the "New discount code" panel with a generated code and 25% off.
- **Test data:** (a) Usage limit 500, Per-user limit 600; (b) Start date in 10 days, End date in 5 days; (c) Start date 10 days ago, End date yesterday.
- **Steps:**
  1. Try each of (a)–(c) on its own and choose "Save code".
- **Expected result:** (a) "The per-person limit (600) can't exceed the total usage limit (500)." (b) "The end date must come after the start date." (c) "That validity window has already expired — pick an end date in the future." No code is created; the panel stays open to fix.

---

## US-TKT-08 — Adjust a discount code without breaking past redemptions

### TC-TKT-18 — Value/text locked after use; unused code fully editable; usage limit floor
- **Traces:** US-TKT-08  ·  **Priority:** Won't  ·  **Type:** Negative

_Withdrawn: editing a discount code is not in the web app; the id stays valid — not run._

---

## US-TKT-09 — Stop a promotion safely

### TC-TKT-19 — Switch off, switch on and delete a code
- **Traces:** US-TKT-09  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Active code "JAZZ25" (25% off, all events) has been redeemed 89 times; "Bangkok Jazz Night 2026" has a paid ticket on sale.
- **Test data:** —
- **Steps:**
  1. On Discounts, choose "Disable code" on JAZZ25, then type JAZZ25 in "Discount code" at the event's checkout.
  2. Choose "Enable code" and try the code at checkout again.
  3. Choose Delete on JAZZ25, confirm "Delete", and try it at checkout again.
  4. Open the order page of one of the 89 earlier orders.
- **Expected result:** (1) The row reads Disabled and still "89" used; checkout says '"JAZZ25" is no longer available.' (2) The row reads Active; the discount applies. (3) "Discount code deleted."; the row is gone; checkout says '"JAZZ25" isn't a code we recognise for this event.' (4) The order still shows its discount.

---

## US-TKT-10 — Codes go live and expire on schedule

### TC-TKT-20 — Auto activate / auto expire on date and on usage exhaustion
- **Traces:** US-TKT-10  ·  **Priority:** Low  ·  **Type:** Functional
- **Preconditions:** For "Bangkok Jazz Night 2026" (paid ticket on sale): "EARLY10" is Scheduled with Start date tomorrow; "LUCKY1" is Active with Usage limit 1, unused; "SPRING20" is Active with End date tomorrow; "TREAT15" is Disabled and its End date has passed.
- **Test data:** —
- **Steps:**
  1. Tomorrow (Bangkok), open Discounts and try EARLY10 at checkout.
  2. Complete one paid order with LUCKY1; open Discounts and try LUCKY1 again.
  3. The day after SPRING20's End date, open Discounts and try it at checkout.
  4. Choose "Enable code" on TREAT15.
- **Expected result:** (1) EARLY10 reads Active and its discount applies. (2) LUCKY1 reads Expired and "1 / 1"; checkout says '"LUCKY1" has reached its redemption limit.' (3) SPRING20 reads Expired; checkout says '"SPRING20" has expired.' (4) "This code has expired — update its dates or usage limit to revive it."; TREAT15 stays Disabled.

---

## US-TKT-11 — Apply a discount code at checkout

### TC-TKT-21 — Percentage discount shown in the Order summary
- **Traces:** US-TKT-11  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** A guest is at the checkout of "Bangkok Jazz Night 2026"; "General Admission" costs ฿1,000; the workspace has the default 5% service fee; Active code "JAZZ25" (25% off) applies to this event.
- **Test data:** Code typed as "jazz25".
- **Steps:**
  1. Choose "General Admission", 1 ticket.
  2. Type the code in "Discount code".
- **Expected result:** The box shows "JAZZ25". The Order summary reads "1 × General Admission ฿1,000", "Discount ฿250", "Service fee ฿37.50" and "Total ฿787.50"; the pay button shows the same total.

### TC-TKT-22 — Fixed discount capped at the order; free tickets take no code
- **Traces:** US-TKT-11  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** A guest is at checkout; a paid ticket costs ฿250 and a free "Student Pass" is also on sale; Active code "SAVE300" takes ฿300 off.
- **Test data:** 1 ticket; code "SAVE300".
- **Steps:**
  1. Choose the ฿250 ticket and type SAVE300 in "Discount code".
  2. Choose "Student Pass" instead.
- **Expected result:** (1) The Order summary reads "Discount ฿250", "Service fee ฿0" and "Total ฿0" — never below zero. (2) A red message says "There's nothing to discount — this order is already free."

### TC-TKT-23 — Invalid and limit-reached codes give clear reasons
- **Traces:** US-TKT-11  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** A guest is at the checkout of "Bangkok Jazz Night 2026" with 1 "General Admission". Codes: "NOPE99" doesn't exist; "SOON10" starts next week; "OLD10" ended last week; "OFF10" is Disabled; "OTHER10" is for another event; "FULL10" has used its whole Usage limit; "ONCE10" has Per-user limit 1 and was already used by nok@eventa-test.co.th.
- **Test data:** Buyer "Nok Srisuk", nok@eventa-test.co.th.
- **Steps:**
  1. Type NOPE99, SOON10, OLD10, OFF10, OTHER10 and FULL10 in "Discount code", one at a time.
  2. Type ONCE10, enter Nok's details and pay.
- **Expected result:** (1) In turn: '"NOPE99" isn't a code we recognise for this event.', '"SOON10" isn't active yet.', '"OLD10" has expired.', '"OFF10" is no longer available.', '"OTHER10" isn't a code we recognise for this event.', '"FULL10" has reached its redemption limit.' (2) 'You've already used "ONCE10".'; no order is placed.

---

## US-TKT-12 — Track which promotions are working

### TC-TKT-24 — Filter, search, all-events scope visibility, and copy code
- **Traces:** US-TKT-12  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Codes: "JAZZ25" (Bangkok Jazz Night 2026, 342 of 500 used, Active), "WELCOME10" (All events, Active), "DESIGN15" (Chiang Mai Design Week, Scheduled), "OLD10" (Hua Hin Food Fair, Expired), "OFF10" (Hua Hin Food Fair, Disabled).
- **Test data:** Search "chiang".
- **Steps:**
  1. Open Discounts and read the JAZZ25 row.
  2. Pick "Bangkok Jazz Night 2026" in the event filter.
  3. Return to "All events", choose each status tab, then type "chiang" in "Search codes…".
  4. Choose "Copy code" on JAZZ25 and paste it.
- **Expected result:** (1) "JAZZ25", "25% off", "Bangkok Jazz Night 2026", "342 / 500" with a bar, its Valid dates and Active. (2) JAZZ25 and WELCOME10 show; DESIGN15 doesn't. (3) Each tab shows only its codes; "chiang" finds DESIGN15 by its event's name. (4) The icon briefly shows a tick and the paste reads "JAZZ25".


---

<a id="tc-e06"></a>

# Discover & Register for Events — Test Cases

Area: DISC · Epic E6 — Discover & Register for Events (Attendee)
Locale defaults: currency ฿ (THB), VAT 7%, PromptPay, Asia/Bangkok, languages EN/TH.

---

## US-DISC-01 — Discover and browse what's on

### TC-DISC-01 — Discover cards show the key facts, soonest first
- **Traces:** US-DISC-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** 14 published, public, upcoming events exist; the visitor is signed out.
- **Test data:** "Bangkok Jazz Night" (category Music, 15 Aug 2026 19:00 Bangkok, Riverside Hall, Bangkok, by "Eventa Test Organizer", from ฿890, 120 going) is the soonest; "Thai Cooking Workshop" is free; "Online Design Talk" is online.
- **Steps:**
  1. Open Discover.
  2. Read the "Bangkok Jazz Night" card, then the workshop and online cards.
  3. Check the order of the cards and choose "Next".
  4. Tap the "Bangkok Jazz Night" card.
- **Expected result:** The page is headed "What's on in Bangkok". Cards run soonest first, 12 on page 1 and 2 on page 2. The jazz card shows its cover, Music, title, date, "Riverside Hall, Bangkok", "by Eventa Test Organizer", "120 going" and "From ฿890"; the workshop shows "Free" and the talk "Online". Tapping opens the event's page with its "Register" (or "Get tickets") button.

### TC-DISC-02 — Waitlist and Selling fast badges, started events, nothing coming up
- **Traces:** US-DISC-01  ·  **Priority:** High  ·  **Type:** Edge
- **Preconditions:** "Riverside Indie Fest" has every ticket type sold out; "Lantern Market Tour" has 360 of 400 seats sold; "Morning Yoga Class" started 30 minutes ago (Bangkok time). For step 4, a test environment with no upcoming public events.
- **Test data:** The three events above.
- **Steps:**
  1. Open Discover and find "Riverside Indie Fest".
  2. Find "Lantern Market Tour".
  3. Look for "Morning Yoga Class".
  4. With no upcoming public events, open Discover.
- **Expected result:** (1) A "Waitlist" badge and "—" where the price would be. (2) A "Selling fast" badge. (3) Not listed. (4) "Nothing on just yet", with no category strip or count.

---

## US-DISC-02 — Search and filter events

### TC-DISC-03 — Keyword and category combine and update the count
- **Traces:** US-DISC-02  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Upcoming events "Bangkok Jazz Night" (Music), "Jazz Cooking Class" (Food) and "Riverside Indie Fest" (Music).
- **Test data:** Keyword "jazz"; category "Music".
- **Steps:**
  1. On Discover, type "jazz" in "Search events, cities, categories…".
  2. Pick the "Music" chip.
  3. Pick "All events".
- **Expected result:** (1) Only the two jazz events remain and the count reads "2 events". (2) Only "Bangkok Jazz Night" remains: "1 event". (3) Both jazz events are back.

### TC-DISC-04 — Accents, Thai tone marks, stray spaces and no matches
- **Traces:** US-DISC-02  ·  **Priority:** High  ·  **Type:** Edge
- **Preconditions:** Upcoming events "Café Live Sessions" and "คอนเสิร์ตริมน้ำ", plus a "Workshop" category with no jazz event.
- **Test data:** "cafe"; "คอนเสิรต" (typed without the ์ mark); "  jazz  "; "jazz" with "Workshop".
- **Steps:**
  1. Search "cafe".
  2. Search "คอนเสิรต".
  3. Search "  jazz  ".
  4. Search "jazz" and pick "Workshop".
  5. Choose "Clear filters".
- **Expected result:** (1) "Café Live Sessions" is found. (2) "คอนเสิร์ตริมน้ำ" is found. (3) Same results as "jazz". (4) "No events match" with "Try a different search or category." (5) Search and category are cleared and every event is back.

---

## US-DISC-03 — Save events for later

### TC-DISC-05 — Hearts stay, and guest saves join the account
- **Traces:** US-DISC-03  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Attendee account anong.p@example.com already has "Thai Cooking Workshop" saved; the visitor starts signed out.
- **Test data:** "Bangkok Jazz Night" and "Thai Cooking Workshop".
- **Steps:**
  1. As a guest, tap the heart on both events, then reload Discover.
  2. Sign in as anong.p@example.com and open Discover.
  3. Sign in to the same account in a second browser and open Discover.
  4. Tap the "Bangkok Jazz Night" heart again and reload.
- **Expected result:** (1) Both hearts stay filled. (2) Both hearts are filled; the account holds "Thai Cooking Workshop" once, not twice. (3) The second browser shows both hearts filled. (4) "Bangkok Jazz Night" is no longer saved.

### TC-DISC-06 — A save that fails puts the heart back
- **Traces:** US-DISC-03  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Signed in as an attendee; saving is made to fail (for example, the network drops when the heart is tapped).
- **Test data:** "Riverside Indie Fest".
- **Steps:**
  1. Open Discover.
  2. Tap the heart on "Riverside Indie Fest" while saving fails.
  3. Restore the network and reload.
- **Expected result:** The heart fills on the tap, then goes back, and "Couldn't save right now." shows under the card. After reload the event is not saved.

---

## US-DISC-04 — Register and choose my tickets (guest or signed in)

### TC-DISC-07 — Reserved checkout: seats, live total and guest details
- **Traces:** US-DISC-04  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** "Bangkok Jazz Night" is published with reserved seating, ticket type "General" ฿890, a 5% service fee, and Card and PromptPay on. Row 1 seat 15 is sold. The visitor is signed out.
- **Test data:** Row 1 seats 12 and 13; guest "Somchai Prasert", somchai.p@example.com, +66 81 234 5678.
- **Steps:**
  1. On the event page choose "Register" and pick "General".
  2. Pick row 1 seats 12 and 13, then try row 1 seat 15.
  3. Read "Order summary" and the bottom bar.
  4. Fill "3 · Your details" and choose "Continue to pay ฿1,869".
  5. Try to sign in to the attendee portal as somchai.p@example.com.
- **Expected result:** Checkout shows the event's name, date, place and organizer, and General at ฿890. Seat 15 reads "Booked" and can't be picked. The summary shows "2 × General" ฿1,780, Service fee ฿89 and Total ฿1,869; the bar reads "2 tickets · General" and "Total · ฿1,869". The order is placed for Somchai. No account exists for somchai.p@example.com, so sign-in fails.

### TC-DISC-07b — Other ticket types' seats can't be picked and drop on a type switch
- **Traces:** US-DISC-04  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** A reserved event with 12 rows of 10 seats: rows 1–2 VIP ฿2,500; row 3 seats 1–5 VIP and seats 6–10 Normal ฿890; rows 4–12 Normal. Nothing is sold.
- **Test data:** VIP: row 1 seat 5, row 3 seat 5. Normal: row 3 seats 6 and 7.
- **Steps:**
  1. Open checkout and choose VIP.
  2. Try row 3 seat 6 and a seat in row 12.
  3. Pick row 1 seat 5 and row 3 seat 5.
  4. Switch to Normal.
  5. Pick row 3 seats 6 and 7.
- **Expected result:** With VIP, row 3 seats 6–10 and rows 4–12 show as "Other ticket type" (named in the legend) and nothing happens when tapped. Step 3 gives a ฿5,000 subtotal. Switching to Normal drops both VIP seats, and the VIP seats become "Other ticket type". Step 5 gives a ฿1,780 subtotal.

### TC-DISC-08 — Quantity runs 1 to 8; a free event skips payment
- **Traces:** US-DISC-04  ·  **Priority:** High  ·  **Type:** Edge
- **Preconditions:** General-admission event "Thai Cooking Workshop" (฿300, no lower per-booking limit); online event "Online Design Talk"; free event "Community Fun Run" with two free ticket types.
- **Test data:** The three events above.
- **Steps:**
  1. Open checkout for the workshop; press "−" at 1, then "+" until it stops.
  2. Read the note above the stepper; repeat for the online talk.
  3. Open checkout for the fun run and pick a type.
  4. Fill your details and choose "Confirm registration".
- **Expected result:** (1) "−" is disabled at 1 and "+" stops at 8 ("tickets · up to 8"). (2) The workshop says "General admission — no assigned seats, first come first served."; the talk says "Online event — a join link is emailed after you register." (3) Both types read "Free", there is no "4 · Payment" step and the bar reads "Free admission". (4) The registration is placed without any payment.

### TC-DISC-09 — A seat someone else bought can't be picked
- **Traces:** US-DISC-04  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Reserved event "Bangkok Jazz Night"; row 1 seat 14 was bought by another buyer.
- **Test data:** Row 1 seat 14.
- **Steps:**
  1. Open checkout and choose the seat's ticket type.
  2. Tap row 1 seat 14.
  3. Read the legend and the bottom bar.
- **Expected result:** Seat 14 shows as "Booked" (named in the legend) and tapping it picks nothing; the bar still reads "No seats selected".

### TC-DISC-09b — A held seat of another ticket type can't be paid for at this type's price
- **Traces:** US-DISC-04  ·  **Priority:** Won't  ·  **Type:** Negative
_Withdrawn: booking another ticket type's seat by a direct request is not in the web app; the id stays valid — not run._

---

## US-DISC-05 — Pay by card or PromptPay

### TC-DISC-10 — Card payment completes the registration
- **Traces:** US-DISC-05  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** The organizer accepts Card and PromptPay; the payment provider is in test mode.
- **Test data:** 2 × General for "Bangkok Jazz Night", total ฿1,869; the provider's published test card that approves; buyer anong.p@example.com.
- **Steps:**
  1. At "4 · Payment", check both methods are offered, choose "Card" and then "Continue to pay ฿1,869".
  2. On the provider's page, pay with the approving test card.
  3. Wait on the order page you return to, reloading once.
- **Expected result:** "Card" and "PromptPay" are both offered. After the button, "Almost there — your seats are held" reads "Taking you to our payment provider to pay ฿1,869 securely." and the provider's page opens; Eventa never shows a card field. Back on the order page, once the payment clears it reads "Confirmed" with 2 tickets.

### TC-DISC-11 — A declined card issues no ticket and can be retried
- **Traces:** US-DISC-05  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** As TC-DISC-10.
- **Test data:** The provider's published test card that declines; then the approving test card.
- **Steps:**
  1. Choose "Card", "Continue to pay ฿1,869", and pay with the declining card.
  2. Leave the provider's page and return to the order page.
  3. Choose "Pay ฿1,869 by card" and pay with the approving card.
- **Expected result:** (1) The provider refuses the card. (2) The order reads "Awaiting payment" with a "Seats held for" countdown, "Pay ฿1,869 by card" and "Pay with PromptPay"; no ticket is shown. (3) The order reads "Confirmed" with its tickets, charged once.

### TC-DISC-12 — PromptPay QR for the exact total; an unpaid hold expires
- **Traces:** US-DISC-05  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** The organizer accepts PromptPay; the provider is in test mode; the seat hold is 10 minutes.
- **Test data:** Order A and order B, each 2 seats for ฿1,869.
- **Steps:**
  1. For order A, choose "PromptPay" and "Pay ฿1,869".
  2. Pay the code in test mode and open "View my order".
  3. Place order B the same way but don't pay; open its order page.
  4. Wait until the countdown ends.
  5. Open checkout again and look at order B's seats.
- **Expected result:** (1) A QR appears with "Scan this with your banking app to pay ฿1,869". (2) Order A reads "Confirmed" with 2 tickets, issued once. (3) Order B reads "Awaiting payment" with "Seats held for" counting down. (4) Order B reads "Expired": "The seats were released because the payment was not completed in time. Nothing was charged." No ticket exists. (5) Its seats are "Available" again.

---

## US-DISC-06 — Confirm and receive my QR ticket

### TC-DISC-13 — Confirmation screen, order page, email, receipt and text
- **Traces:** US-DISC-06  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Confirmation and receipt emails and texts are switched on; free event "Community Fun Run" and paid event "Bangkok Jazz Night" (VAT 7% included in prices).
- **Test data:** Buyer "Anong Pattana", anong.p@example.com, +66 81 234 5678; 1 free ticket; then 2 × General paid ฿1,869.
- **Steps:**
  1. Register for the free ticket with "Confirm registration".
  2. Choose "View my tickets".
  3. Check the inbox and the phone.
  4. Buy and pay for the 2 paid tickets; check the inbox again.
- **Expected result:** (1) "You're registered!" shows 1 ticket, the event and the reference. (2) The order page opens without signing in, reads "Confirmed", and shows one QR. (3) "You're going to Community Fun Run" arrives with the reference, date, place, "Free registration" and a link to the tickets; a text gives the reference and the same link. (4) The paid order's page shows 2 QRs and "What you paid" with VAT included; "You're going to Bangkok Jazz Night" (total ฿1,869) and "Your receipt for Bangkok Jazz Night" (VAT 7% included) arrive.

### TC-DISC-14 — Double press places one order; the same seat goes to one buyer
- **Traces:** US-DISC-06  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Reserved event "Bangkok Jazz Night"; two testers in two browsers.
- **Test data:** Tester A double-presses on row 2 seats 1–2; testers A and B both pick row 2 seat 5.
- **Steps:**
  1. Tester A picks row 2 seats 1–2 and double-clicks "Continue to pay ฿1,869".
  2. Check the organizer's registrations for tester A.
  3. Both testers pick row 2 seat 5; A books first, then B.
- **Expected result:** (2) One order exists for tester A and one payment is started. (3) Tester A's booking goes through; tester B sees "Some of those seats were just taken. Please choose different seats." and no order or charge is made for B.

---

## US-DISC-07 — View and download my QR ticket

### TC-DISC-15 — Open and download the ticket from My Events
- **Traces:** US-DISC-07  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as anong.p@example.com, who holds a confirmed order of 2 VIP tickets, row A seats 12 and 13, for "Bangkok Jazz Night" at Riverside Hall, Bangkok.
- **Test data:** The order above.
- **Steps:**
  1. On My Events, choose "Ticket" on the event's card.
  2. Switch to "Ticket 2".
  3. Choose "Download ticket" and open the file.
  4. Open the order page from the confirmation email, signed out, and choose "Download" on a ticket.
- **Expected result:** (1) "Your ticket QR" shows a QR, the ticket reference and "Admission 1 of 2". (2) A different QR with "Admission 2 of 2". (3) A printable pass with the QR, "Bangkok Jazz Night", date and time, "Riverside Hall · Bangkok", VIP, "Seat A-13", the reference and "Admission 2 of 2". (4) The order page shows each QR and code, and "Download" saves that ticket's QR.

### TC-DISC-16 — A void ticket shows no QR and can't be downloaded
- **Traces:** US-DISC-07  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Signed in as anong.p@example.com; she holds an order of 2 tickets for "Bangkok Jazz Night" and the organizer voided ticket 2.
- **Test data:** The order above.
- **Steps:**
  1. On My Events, choose "Ticket" and switch to "Ticket 2".
  2. Look for "Download ticket".
  3. Open the order page from the confirmation email.
- **Expected result:** (1) No QR: "This ticket is no longer valid and will not admit anyone." (2) There is no "Download ticket" button. (3) Ticket 2 reads "No longer valid", with no QR or "Download"; ticket 1 is unchanged.

---

## US-DISC-08 — Sign in to my attendee account

### TC-DISC-17 — Sign-in lands on My Events and keeps guest saves
- **Traces:** US-DISC-08  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Attendee account anong.p@example.com exists; the browser is signed out.
- **Test data:** Her password; guest saves "Bangkok Jazz Night" and "Thai Cooking Workshop".
- **Steps:**
  1. As a guest, heart both events on Discover.
  2. Choose "Sign in" and sign in with her email and password.
  3. Open Discover.
  4. Go to the organizer console in the same browser.
- **Expected result:** (2) She lands on My Events. (3) Both events show filled hearts. (4) She is sent to the organizer sign-in, not into the console.

### TC-DISC-18 — Wrong password, lockout, forgot password and Create one
- **Traces:** US-DISC-08  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Attendee account anong.p@example.com exists with no recent failed sign-ins.
- **Test data:** A wrong password; an unknown email nobody@example.com.
- **Steps:**
  1. Sign in with the right email and a wrong password; then with nobody@example.com.
  2. Fail 4 more times with anong.p@example.com and the wrong password.
  3. Try once more with the right password.
  4. Choose "Forgot password?", then go back and choose "Create one".
- **Expected result:** (1) Both read "Invalid email or password". (3) "Too many sign-in attempts. Try again in 15 minutes." and no sign-in. (4) "Forgot password?" opens the reset flow; "Create one" opens the attendee "Create your account" page.

---

## US-DISC-09 — See my upcoming and past tickets

### TC-DISC-19 — My Events splits upcoming and past
- **Traces:** US-DISC-09  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** anong.p@example.com has confirmed bookings, made as a guest before her account existed, with two organizers: one event in 6 days, one today, one ended at which she was checked in, one ended that she missed. somchai.p@example.com has none.
- **Test data:** The two accounts above.
- **Steps:**
  1. Sign in as Anong and open "My Events".
  2. Read the upcoming cards.
  3. Read the past cards and choose "Leave feedback" on the one she attended.
  4. Sign in as Somchai and open "My Events".
- **Expected result:** (1) "Upcoming · 2" and "Past · 2", from both organizers. (2) Countdowns "6 days left" and "Today", each with ticket type, date and time, venue, "Details" and "Ticket". (3) Only the checked-in event reads "Attended"; both have "Leave feedback", which opens that event's survey. (4) "No tickets yet" with "Discover events".

---

## US-DISC-10 — Review my payment history and receipts

### TC-DISC-20 — Payment history tiles, rows and refunds
- **Traces:** US-DISC-10  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** anong.p@example.com has three paid orders, one since refunded; somchai.p@example.com has none.
- **Test data:** ฿1,869 by Card (invoice INV-2026-0001); ฿300 by PromptPay; ฿500 by Card, refunded.
- **Steps:**
  1. Sign in as Anong and open "Payment history".
  2. Read the three tiles.
  3. Read each row.
  4. Sign in as Somchai and open "Payment history".
- **Expected result:** (2) "Total spent" ฿2,169, "Transactions" 3, "Refunded" ฿500. (3) Each row shows the event, its invoice number (or order reference), date, "Card" or "PromptPay", amount and "Paid" or "Refunded"; the ฿500 is struck through. (4) "No payments yet" with "Discover events".

---

## US-DISC-11 — Manage my profile

### TC-DISC-21 — Save, cancel, and pre-fill at checkout
- **Traces:** US-DISC-11  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as anong.p@example.com on the "Profile" tab, with a confirmed phone +66 81 234 5678.
- **Test data:** Name "Anong Pattana", City "Bangkok", Date of birth 1990-05-01, Bio "Jazz fan"; a later edit to City "Chiang Mai".
- **Steps:**
  1. Enter the test data and choose "Save changes".
  2. Change City to "Chiang Mai" and choose "Cancel".
  3. Reload the tab.
  4. Open checkout for any event.
- **Expected result:** (1) "Profile saved." (2–3) City reads "Bangkok". (4) Full name, Email and Phone are pre-filled with Anong's details and can be edited.

### TC-DISC-22 — New email and phone need confirming; bad photos are refused
- **Traces:** US-DISC-11  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Signed in as anong.p@example.com on the "Profile" tab; SMS sending works in test mode.
- **Test data:** New email anong.new@example.com; phone 081 234 5679; a 7 MB JPG, a GIF, and a 3 MB PNG.
- **Steps:**
  1. Type the new email and choose "Send confirmation"; sign out and sign in with the old email.
  2. Enter the phone, choose "Send code", then type the texted code and choose "Confirm".
  3. Upload the 7 MB JPG, then the GIF, then the PNG.
- **Expected result:** (1) "Check the new address for a link."; the new address shows "Pending" and the old one still signs in. (2) "Code sent. Check your phone.", then "Number confirmed." (3) The JPG and GIF are refused with a reason (for example "That photo is larger than 5 MB…") and the old photo stays; the PNG becomes the photo.

---

## US-DISC-12 — Manage my notifications, display preferences, and security

### TC-DISC-23 — Notification switches and preferences
- **Traces:** US-DISC-12  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in as anong.p@example.com on "Settings", with no phone on file; she holds a ticket for an event whose organizer sends a reminder a day before.
- **Test data:** "Event reminders" off; Language ไทย; Currency USD.
- **Steps:**
  1. Look at "Email notifications" and "SMS alerts".
  2. Switch "Event reminders" off and reload.
  3. Pick Language ไทย and Currency USD.
  4. Wait for the reminder day; then book and pay for another event.
- **Expected result:** (1) "Email notifications" reads "Always on"; "SMS alerts" reads "No phone on file". (2) The switch stays off. (3) "Preferences saved."; prices still show in ฿. (4) No reminder arrives; the new booking's confirmation and receipt still arrive, in Thai.

### TC-DISC-24 — Password change and two-factor
- **Traces:** US-DISC-12  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Signed in as anong.p@example.com on "Settings" in two browsers; an authenticator app is ready.
- **Test data:** Current password; a wrong one; new password `Silom#2027`.
- **Steps:**
  1. Choose "Change" on Password, give the wrong current password and choose "Update password".
  2. Give the right one but two different new passwords.
  3. Give the right one and `Silom#2027` twice.
  4. Turn on two-factor, scan the code and enter the app's 6-digit code.
  5. Sign out and sign in again.
- **Expected result:** (1) "Your current password is incorrect." (2) "Those two do not match. Retype the new password." Nothing changes in either case. (3) "Password changed. Your other devices have been signed out."; the second browser is signed out and an email says the password changed. (4) Recovery codes are shown once and two-factor reads on. (5) After the password, "Enter your code" asks for a code.

---

## US-DISC-13 — Share post-event feedback

### TC-DISC-25 — Answer the survey once
- **Traces:** US-DISC-13  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** anong.p@example.com had a confirmed booking for "Bangkok Jazz Night", which has ended; its survey is open with a star question, a choice question, a 0–10 question and a comment box.
- **Test data:** 4 stars; choice "From a friend"; 9; comment "Great venue".
- **Steps:**
  1. Open the survey from the thank-you email's link and sign in.
  2. Answer every question and choose "Submit feedback".
  3. Open the survey link again.
- **Expected result:** (1) The questions show, with "Your response is anonymous to other attendees." (2) "Thank you!" (3) "You’ve already answered"; the organizer's Feedback still shows her first answer (4 stars), once.

### TC-DISC-26 — A required answer is missing; no survey to answer
- **Traces:** US-DISC-13  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** As TC-DISC-25, unanswered; a second event whose survey is closed; somchai.p@example.com never booked "Bangkok Jazz Night".
- **Test data:** Only the comment box filled.
- **Steps:**
  1. As Anong, fill only the comment box and choose "Submit feedback".
  2. Open the closed event's survey.
  3. As Somchai, open the "Bangkok Jazz Night" survey link.
- **Expected result:** (1) "Please answer “<the star question>”." and nothing is recorded. (2) and (3) "No survey to answer".

---

## US-DISC-14 — Delete my account

### TC-DISC-27 — Delete with DELETE and password, warned about paid tickets
- **Traces:** US-DISC-14  ·  **Priority:** Low  ·  **Type:** Functional
- **Preconditions:** Signed in as anong.p@example.com (two-factor off) holding an upcoming paid order of ฿1,869.
- **Test data:** "DELETE"; her password.
- **Steps:**
  1. In "Settings" → "Danger zone", choose "Delete account".
  2. Read the dialog.
  3. Type DELETE and her password, and choose "Delete account".
  4. Check the screen and her inbox.
- **Expected result:** (2) "Delete your account" lists "Bangkok Jazz Night" and warns the tickets are not refunded (฿1,869 in all); "Delete account" stays disabled until DELETE is typed. (4) She is signed out to the sign-in page and a confirmation email arrives; signing in again fails.

### TC-DISC-28 — Wrong password or code deletes nothing
- **Traces:** US-DISC-14  ·  **Priority:** Low  ·  **Type:** Negative
- **Preconditions:** Signed in as anong.p@example.com with two-factor on.
- **Test data:** "DELETE"; a wrong password; then the right password and a wrong code.
- **Steps:**
  1. Open "Delete account", type DELETE and the wrong password, and submit.
  2. Give the right password and a wrong code, and submit.
  3. Choose "Keep my account" and reload.
- **Expected result:** (1) "That password didn't match — your account is unchanged." (2) "That code didn't match — your account is unchanged." (3) She is still signed in with her bookings, and no deletion email arrives.

## US-DISC-15 — Create my account from my confirmation

### TC-DISC-29 — Create the account from the emailed link
- **Traces:** US-DISC-15  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** A guest with no account has a confirmed order with one ticket; the tester can read the inbox.
- **Test data:** "Anong Pattana", anong.p@example.com; password `Silom#2026`.
- **Steps:**
  1. Open the order page and read the offer below the ticket.
  2. Choose "Create account".
  3. Open the link in the inbox.
  4. Enter `Silom#2026` twice, tick Terms & Privacy, and choose "Create account".
- **Expected result:** (1) "Keep these tickets" with "Create account", "Sign in" and "Not now — your ticket is already in your email"; no password field. (2) No account yet; the card reads "We've sent a link to anong.p@example.com. Open it to choose a password — it proves the address is yours." and one email arrives ("Keep your tickets for <event> in an Eventa account"). (3) "Create your account" shows the name and email, locked. (4) She lands signed in on My Events with the booking listed; no "confirm your email" step follows.

### TC-DISC-29b — The locked email can't be swapped
- **Traces:** US-DISC-15  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** The sign-up page is open from the link emailed to anong.p@example.com.
- **Test data:** Another address, other.person@example.com.
- **Steps:**
  1. Using the browser's developer tools, change the locked email box to other.person@example.com.
  2. Choose a password, tick Terms & Privacy, and submit.
- **Expected result:** The account is created for anong.p@example.com with the booking's name; no account exists for other.person@example.com.

### TC-DISC-29c — Booking with someone else's email can't take their account
- **Traces:** US-DISC-15  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** A free event takes RSVPs; somchai.k@example.com belongs to someone else, has no account and two earlier bookings. The tester can't read that inbox.
- **Test data:** Booking email somchai.k@example.com; password `Guess#2026`.
- **Steps:**
  1. RSVP as a guest with somchai.k@example.com.
  2. Choose "Create account", then reload and look for a password field.
  3. Try to sign in as somchai.k@example.com with `Guess#2026`.
- **Expected result:** (2) Only "We've sent a link to somchai.k@example.com…"; no password field anywhere. (3) Sign-in fails, so none of that address's bookings can be seen. The owner received one link and, while it's ignored, no account exists.

### TC-DISC-29d — An expired, altered or used link is refused
- **Traces:** US-DISC-15  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Three emailed links: one 8 days old; one with a character changed; one already used to create anong.p@example.com's account (password `Silom#2026`).
- **Test data:** The three links.
- **Steps:**
  1. Open the expired link, then the altered one.
  2. Open the used link.
  3. Sign in as anong.p@example.com with `Silom#2026`.
- **Expected result:** (1) "That link didn't work": "This link has expired or is no longer valid. Ask for a new one from your booking confirmation.", with no name, email or password field. (2) "You already have an account" with "Sign in instead" and no password field. (3) Sign-in works with the original password.

### TC-DISC-29e — One link a minute and five a day per address
- **Traces:** US-DISC-15  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Two confirmed guest orders with somchai.k@example.com, with different organizers; no account and no link sent in the last 24 hours. The tester can read the inbox.
- **Test data:** The two orders.
- **Steps:**
  1. On order 1 choose "Create account".
  2. At once, choose it again on order 2.
  3. Ask four more times, a minute apart, alternating orders.
  4. A minute later ask a sixth time, then count the emails.
- **Expected result:** (1) A link is sent. (2) "A confirmation link was just sent. Give it a minute before asking for another." and nothing is sent. (3) Each sends a link. (4) "That is as many links as we can email one address for now…" and nothing is sent; the inbox holds exactly five.

### TC-DISC-29f — No offer for a booking that isn't confirmed
- **Traces:** US-DISC-15  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Guest orders with addresses that have no account: (a) unpaid, hold still running; (b) waiting for organizer approval; (c) on the waitlist; (d) expired unpaid; (e) refunded.
- **Test data:** Orders (a) to (e).
- **Steps:**
  1. Open each order's page.
  2. Pay order (a) and reload its page.
- **Expected result:** (1) None shows "Keep these tickets", "Create account", "Sign in" or "Not now — your ticket is already in your email". (2) Once (a) reads "Confirmed", the offer appears below its ticket.

### TC-DISC-30 — Declining leaves the booking untouched
- **Traces:** US-DISC-15  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** A guest has just confirmed a registration.
- **Test data:** anong.p@example.com, no account.
- **Steps:**
  1. Choose "Not now — your ticket is already in your email".
  2. Check the ticket, QR and confirmation email.
  3. Open the confirmation email's tickets link while signed out.
- **Expected result:** The offer disappears; the ticket, QR and order are unchanged and no account link is emailed. The tickets link opens without signing in.

### TC-DISC-31 — Earlier bookings appear once the account exists
- **Traces:** US-DISC-15  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** anong.p@example.com has two earlier confirmed guest bookings with different organizers, and no account.
- **Test data:** A third confirmed booking with the same email.
- **Steps:**
  1. On the third booking's page choose "Create account".
  2. Open the emailed link, set a password and submit.
  3. Read My Events.
- **Expected result:** All three bookings are listed, including the two made before the account existed.

### TC-DISC-32 — An email with a confirmed account is offered Sign in
- **Traces:** US-DISC-15  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** A confirmed attendee account exists for anong.p@example.com.
- **Test data:** Her password.
- **Steps:**
  1. Signed out, register for an event with anong.p@example.com.
  2. Read the offer on the order page.
  3. Choose "Sign in" and sign in.
- **Expected result:** (2) Only "Sign in" is offered, with "You already have an account for anong.p@example.com…"; no "Create account". (3) She returns to the same order page, which now reads "Every booking in one place" with "See all your bookings"; no duplicate account exists.

### TC-DISC-32b — The booking identifier cannot be guessed to harvest a buyer's email
- **Traces:** US-DISC-15  ·  **Priority:** Won't  ·  **Type:** Negative
_Withdrawn: probing the account offer with guessed booking references by direct request is not in the web app; the id stays valid — not run._

### TC-DISC-32c — A stranger's unconfirmed sign-up neither blocks nor survives the claim
- **Traces:** US-DISC-15  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Someone else started an attendee sign-up with anong.p@example.com, name "Someone Else", password `Stranger#1`, and never confirmed it. The owner then RSVPs as a guest with that email as "Anong Pattana".
- **Test data:** Owner's password `Silom#2026`.
- **Steps:**
  1. Read the offer on the owner's order page.
  2. Choose "Create account" and open the emailed link.
  3. Enter `Silom#2026` twice, tick Terms & Privacy, and submit.
  4. In another browser, sign in with `Stranger#1`.
- **Expected result:** (1) "Create account" is offered, not only "Sign in". (2) The page shows "Anong Pattana" and the email with password fields. (3) She lands on My Events. (4) Refused: the one account for that email is hers, named from the booking, and only `Silom#2026` opens it.

### TC-DISC-33 — The account can also be made from the sign-in page
- **Traces:** US-DISC-15  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** nok.s@example.com has no account and one earlier confirmed guest booking.
- **Test data:** "Nok Saelim", nok.s@example.com, password `Sathorn#2026`.
- **Steps:**
  1. On the attendee sign-in choose "Create one".
  2. Fill the form, tick Terms & Privacy, and choose "Create account".
  3. Try to sign in before opening the email.
  4. Open the confirmation email's link, sign in, and open My Events.
- **Expected result:** (1) The email box is empty and editable. (2) A check-your-email page follows. (3) Refused: "Please confirm your email — we’ve sent you a fresh link." (4) Sign-in works and the earlier booking is listed.


---

<a id="tc-e07"></a>

# Communicate with Attendees — Test Cases

Epic E7 — Communicate with Attendees. Area code: MSG. Locale: Thai market (฿, VAT 7%, PromptPay, Asia/Bangkok, EN/TH).

---

## US-MSG-01 — Attendees automatically get the right message at the right moment

### TC-MSG-01 — Confirmation email arrives when payment succeeds
- **Traces:** US-MSG-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** "Tech Summit 2026" is published with a paid General ticket; "Registration confirmation" reads Active on "Message templates".
- **Test data:** Buyer "Somchai Jaidee", somchai@example.com; 1 × General ฿1,000 paid by PromptPay.
- **Steps:**
  1. As the buyer, register for one General ticket and pay ฿1,000 by PromptPay.
  2. Open somchai@example.com's inbox.
- **Expected result:** "You're going to Tech Summit 2026" has arrived with the booking reference, when and where, "Total paid" ฿1,000.00, the General ticket and a link to open the tickets. No raw {{...}} placeholder appears anywhere.

### TC-MSG-02 — Reminder a day before, only where it is switched on
- **Traces:** US-MSG-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Workspace A has switched "Event reminder" on; workspace B never touched it (its card reads Inactive). Each has an event starting 2026-07-28 18:00 Bangkok with confirmed attendees.
- **Test data:** A: "Bangkok Jazz Night" at Lumpini Park, attendees nok@example.com and ploy@example.com (Ploy switched "Event reminders" off in her attendee account). B: "Chiang Mai Design Week", attendee anan@example.com.
- **Steps:**
  1. Wait until 2026-07-27 18:00 Bangkok has passed.
  2. Open the three inboxes.
- **Expected result:** Nok has "Tomorrow: Bangkok Jazz Night" with the time (Tuesday, 28 July 2026 at 18:00), the place and the link to her tickets. Ploy and Anan have no reminder.

### TC-MSG-03 — Attendee's language, with the event's as fallback; a switched-off message is not sent
- **Traces:** US-MSG-01  ·  **Priority:** High  ·  **Type:** Edge
- **Preconditions:** "Tech Summit 2026" sends in English, the default. The organizer has switched "Cancellation notice" off, confirming "Turn it off".
- **Test data:** Ploy (ploy@example.com) has an attendee account with Language set to ไทย; Anan (anan@example.com) books as a guest with no account.
- **Steps:**
  1. Register and pay for Ploy and for Anan.
  2. Read each confirmation.
  3. As the organizer, cancel the event.
  4. Check both inboxes.
- **Expected result:** (2) Ploy's confirmation is in Thai ("คุณกำลังจะไป Tech Summit 2026"); Anan's is in English. (4) Neither receives a cancellation notice.

---

## US-MSG-02 — Keep automated messages on-brand and in the organizer's control

### TC-MSG-04 — Reword a message and place merge fields at the cursor
- **Traces:** US-MSG-02  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin; "Registration confirmation" has never been reworded. Somchai received a confirmation for "Tech Summit 2026" yesterday.
- **Test data:** English Subject "See you at {{event_name}}"; Opening "Welcome {{first_name}} to {{event_name}}!"; Thai left empty. New attendee "Nok", English.
- **Steps:**
  1. On "Message templates", choose "Reword" on "Registration confirmation".
  2. On the English tab, type the Subject and Opening, adding each merge field by clicking its chip where the cursor is; choose "Save".
  3. Register Nok for Tech Summit 2026.
  4. Open Nok's confirmation and Somchai's earlier one.
- **Expected result:** Each chip lands at the cursor; "Wording saved." shows and the card's button now reads "Edit wording". Nok's email is titled "See you at Tech Summit 2026" and opens "Welcome Nok to Tech Summit 2026!", followed by Eventa's booking reference, tickets and link. Somchai's earlier email is unchanged.

### TC-MSG-05 — SMS live segment count and second-segment cost warning (Thai encoding)
- **Traces:** US-MSG-02  ·  **Priority:** Won't  ·  **Type:** Edge
_Withdrawn: an SMS wording editor with a segment counter is not in the web app; the id stays valid — not run._

### TC-MSG-06 — Switching off an entitled message asks first; bad wording and Staff are refused
- **Traces:** US-MSG-02  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Signed in as Admin on "Message templates"; "Payment receipt" is Active. Staff member "Aran" is on the default Staff role.
- **Test data:** Thai Subject "ใบเสร็จของคุณ" with an empty Thai Opening; English Opening "You earned {{loyalty_points}} points".
- **Steps:**
  1. Switch "Payment receipt" off and choose "Keep it on"; switch it off again and choose "Turn it off".
  2. "Reword" "Registration confirmation": on the ไทย tab enter only the Subject; "Save".
  3. Clear it; on the English tab enter a Subject and the Opening with {{loyalty_points}}; "Save".
  4. As Aran, open "Message templates".
- **Expected result:** (1) "Turn off “Payment receipt”?" appears; "Keep it on" leaves it Active, "Turn it off" makes it Inactive. (2) Refused: "The Thai message is empty. Write one, or clear the Thai subject to use Eventa's own wording." (3) Refused: "This message cannot fill {{loyalty_points}}. Use one of the fields offered, or remove it." Nothing is saved in (2) or (3). (4) "You don't have access to this".

---

## US-MSG-03 — Triage everything happening across my events in one feed

### TC-MSG-07 — Tab counts, older activity, and "Mark all read" that survives a reload
- **Traces:** US-MSG-03  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin. In the last 30 days the workspace has 12 registrations and payments: 9 from today and yesterday, 5 of them after this member last marked the feed read, and 3 more than 10 days old.
- **Test data:** —
- **Steps:**
  1. Open "Notifications" and read the tabs and groups.
  2. Choose "Show older activity".
  3. Choose "Mark all read", then the "Unread" tab.
  4. Reload the page and look at the header bell.
- **Expected result:** (1) "All" reads 12 and "Unread" 5; Today and Yesterday are shown with a dot on unread items; the 3 older items are hidden behind "Show older activity". (2) The older groups appear. (3) "Unread" reads 0 and the tab says "You're all caught up". (4) "Unread" still reads 0 and the bell shows no red dot.

### TC-MSG-08 — Staff's feed leaves out money and alerts
- **Traces:** US-MSG-03  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Staff member "Aran" on the default Staff role (registrations view, no finance access, may not approve).
- **Test data:** In the last 30 days: 3 registrations, 2 payments, 1 payout. Outstanding: 2 registrations awaiting approval and 1 declined payment.
- **Steps:**
  1. As Aran, open "Notifications".
  2. Open the header bell.
- **Expected result:** (1) "All" reads 3: only "New registration" items; no "Payment received" or "Payout completed"; no "Needs action" section. (2) The bell previews the same registrations and "View all notifications".

### TC-MSG-08b — "Needs action" lists every alert the member can act on and drops one once resolved
- **Traces:** US-MSG-03  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Case A: signed in as Admin, with unread activity in the feed. Case B: signed in as an Organizer whose finance access was removed (US-SET-12), who still approves registrations and manages ticket types.
- **Test data:** The outstanding issues of TC-DASH-10: two declined payments for Tech Summit 2026, three Bangkok Jazz Night registrations awaiting approval, a nearly-sold-out Thai Street Food Festival ticket type, and one Tech Summit 2026 registration awaiting approval — four alerts, one more than home's To do shows.
- **Steps:**
  1. Case A: open "Notifications", choose "Mark all read", then choose the Bangkok Jazz Night approvals alert.
  2. Case A: approve or reject the three registrations, then reopen "Notifications".
  3. Case B: open "Notifications".
- **Expected result:** (1) "Needs action" lists all four alerts, including the one home's To do leaves out; "Mark all read" removes none; the alert opens the registrations awaiting approval for Bangkok Jazz Night. (2) That alert is gone; the other three remain. (3) The two approval alerts and the ticket-type alert are listed; the declined-payment alert is not, and nothing stands in for it.

---

## US-MSG-04 — Broadcast a one-off announcement to the right audience

### TC-MSG-09 — Send now reaches each confirmed attendee exactly once
- **Traces:** US-MSG-04  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin. "Tech Summit 2026" has 46 confirmed registrations from 45 people (Nok booked twice), 3 awaiting approval and 2 cancelled.
- **Test data:** Subject "Venue change for Tech Summit 2026"; Message "We have moved to Hall B."
- **Steps:**
  1. On "Announcements", choose "Send announcement" and pick Tech Summit 2026; try "Send" with Subject or Message empty.
  2. Fill both, keep "Send now", choose "Send", then "Send it".
  3. Read the new row; open Nok's inbox and the "Delivery log".
- **Expected result:** (1) "Send" stays disabled. (2) "Send this announcement?" names everyone registered for Tech Summit 2026; "Announcement sent." shows. (3) The row reads "Sent" with "Tech Summit 2026 · 45 attendees". Nok has exactly one email, addressed to her alone; the awaiting-approval and cancelled registrants get none; the log has 45 "Announcement" rows.

### TC-MSG-10 — Schedule for a Bangkok time; too soon and too far are refused
- **Traces:** US-MSG-04  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Signed in as Admin on "Announcements" at 2026-07-31 10:00 Bangkok; the workspace has announcements for Tech Summit 2026 and Bangkok Jazz Night.
- **Test data:** Send times (Bangkok): 2 minutes from now; 2027-09-01 09:00; 2026-08-01 09:00. Late registrant "Ploy".
- **Steps:**
  1. Compose for Tech Summit 2026, choose "Schedule" and leave "Send date & time" empty.
  2. Enter the time 2 minutes away and confirm "Schedule it"; then 2027-09-01 09:00.
  3. Enter 2026-08-01 09:00 and confirm "Schedule it".
  4. Register Ploy before 09:00 on Aug 1 and let the time pass.
  5. Pick Tech Summit 2026 in the event picker above the list.
- **Expected result:** (1) "Schedule" stays disabled. (2) Refused under the field: "Pick a send time at least 5 minutes from now, so there is still time to change it."; then "Pick a send time within the next year — anything later is probably a typo." (3) The row reads "Scheduled" with "Scheduled for Aug 1, 2026 · 09:00". (4) It goes out at 09:00, Ploy included, and the row turns "Sent". (5) Only Tech Summit 2026's announcements are listed.

---

## US-MSG-05 — Change my mind on a scheduled announcement

### TC-MSG-11 — Cancel one scheduled announcement and move another
- **Traces:** US-MSG-05  ·  **Priority:** Low  ·  **Type:** Functional
- **Preconditions:** Two "Scheduled" announcements for Tech Summit 2026, X and Y, both due Aug 5, 2026 · 10:00 Bangkok.
- **Test data:** Y's new time: Aug 6, 2026 15:00 Bangkok. Late registrant "Anan", registering Aug 6 at 12:00.
- **Steps:**
  1. On X choose "Cancel", then "Cancel it".
  2. On Y choose "Reschedule", enter the new time and "Save time".
  3. Register Anan, and let both times pass.
- **Expected result:** (1) "Announcement cancelled."; X reads "Cancelled · was due Aug 5, 2026 · 10:00" with no "Reschedule" or "Cancel", and nobody is emailed on Aug 5. (2) "Announcement rescheduled."; Y reads "Scheduled for Aug 6, 2026 · 15:00". (3) Y goes out at 15:00 on Aug 6, Anan included.

### TC-MSG-12 — An announcement that has started sending cannot be changed
- **Traces:** US-MSG-05  ·  **Priority:** Low  ·  **Type:** Negative
- **Preconditions:** Announcement Z for Tech Summit 2026 is "Scheduled" for 10:00 Bangkok today.
- **Test data:** New time: tomorrow 10:00 Bangkok.
- **Steps:**
  1. At 09:59 open "Reschedule" on Z and leave the dialog open.
  2. Once sending has begun, just after 10:00, enter the new time and "Save time".
  3. Reload the page.
- **Expected result:** (2) The dialog says "This announcement has already started sending and can no longer be changed."; the time is not moved. (3) Z reads "Sent" and offers no "Reschedule" or "Cancel".

---

## US-MSG-06 — Prove messages were delivered and diagnose failures

### TC-MSG-13 — One row per recipient; failures show their reason
- **Traces:** US-MSG-06  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin. Today: Nok's confirmation went by email and SMS; Anan's confirmation SMS was refused by the SMS service; Ploy's "Payment receipt" email was refused by the mail server. Nothing else failed.
- **Test data:** nok@example.com, anan@example.com, ploy@eventa-test.co.th.
- **Steps:**
  1. Open "Delivery log" and find the rows for Nok, Anan and Ploy.
  2. Choose "Needs attention".
  3. Choose "All", search "ploy", then pick the message type "Payment receipt".
- **Expected result:** (1) Each row shows name and email, message type, channel, event, status and the Bangkok send time; "2 of these didn't get through" shows. Nok has two "Registration confirmation" rows, Email and SMS, both "Sent"; Anan's SMS row and Ploy's "Payment receipt" row read "Failed" with the reason. No row reads Delivered or Opened. (2) Only Anan's SMS row and Ploy's "Payment receipt" row. (3) Only Ploy's rows, then only her "Payment receipt".

### TC-MSG-14 — Staff are refused the delivery log
- **Traces:** US-MSG-06  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Staff member "Aran" and an Organizer, both on their default roles.
- **Test data:** —
- **Steps:**
  1. As Aran, open "Delivery log".
  2. As the Organizer, open "Delivery log".
- **Expected result:** (1) "You don't have access to this"; no rows are shown. (2) The log opens.

---

## US-MSG-07 — Export the delivery log for records

### TC-MSG-15 — Export the filtered log, see it audited; empty view and no permission
- **Traces:** US-MSG-07  ·  **Priority:** Low  ·  **Type:** Negative
- **Preconditions:** Signed in as Admin; the log holds 25 "Registration confirmation" rows among others. A member on a custom role with "View registrations & attendees" and "Approve, add & invite registrations" but not "Export attendee data".
- **Test data:** Search term "zzzz" (matches nothing).
- **Steps:**
  1. As Admin, on "Delivery log" pick "Registration confirmation" and choose "Export".
  2. On Settings › Security, choose "View log" beside "Audit log".
  3. Search "zzzz" and choose "Export".
  4. As the custom-role member, open "Delivery log".
- **Expected result:** (1) A CSV downloads with the 25 Registration confirmation rows only. (2) An entry "Exported 25 delivery log rows" by the Admin is listed. (3) "There is nothing to export — no messages match these filters."; no file downloads. (4) The log opens with no "Export" button.

---

## US-MSG-08 — Measure attendee satisfaction across all my events

### TC-MSG-16 — Pooled tiles, "—" for nothing measured, and cards for quiet events
- **Traces:** US-MSG-08  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin. No survey asks "Likelihood to recommend", and the post-event thank-you is switched off.
- **Test data:** Tech Summit 2026: 100 responses, ratings averaging 4.5. Bangkok Jazz Night: 10 responses averaging 3.0. Thai Street Food Festival: 1 live survey, 0 responses. Chiang Mai Design Week: no surveys.
- **Steps:**
  1. Open "Feedback" and read the four tiles.
  2. Read the cards for Thai Street Food Festival and Chiang Mai Design Week.
- **Expected result:** (1) Responses 110; Average rating 4.4 (pooled, not 3.8); NPS "—"; Completion rate "—". (2) Thai Street Food Festival reads "1 survey · 1 live · no responses yet"; Chiang Mai Design Week reads "No surveys yet".

### TC-MSG-17 — Search the event cards and open one event's feedback
- **Traces:** US-MSG-08  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin on "Feedback" with events "Bangkok Jazz Night", "Bangkok Tech Meetup" and "Chiang Mai Design Week".
- **Test data:** Bangkok Tech Meetup has a live survey with star ratings and 12 recommendation answers: 8 at 9–10, 2 at 7–8, 2 at 0–6.
- **Steps:**
  1. Type "Bangkok" in "Search events…".
  2. Type "zzzz", then choose "Clear search".
  3. Open the Bangkok Tech Meetup card.
- **Expected result:** (1) Only the two Bangkok events show. (2) "No events match." then all three cards again. (3) The event's page shows Surveys, "Overall satisfaction" with its average and 5★–1★ bars, "Likelihood to recommend" reading +50 from 12 answers (Promoters 8, Passives 2, Detractors 2), and Responses.

---

## US-MSG-09 — Build and manage surveys for my events

### TC-MSG-18 — A new survey saves as Draft; unanswerable ones are refused
- **Traces:** US-MSG-09  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Signed in as Admin on Tech Summit 2026's feedback page; it has no surveys.
- **Test data:** Title "Post-Event Feedback"; Question 1 Rating "How was the event?"; Question 2 Multiple choice "Which day did you attend?" with only "Saturday"; then Questions 2 and 3 both "Likelihood to recommend", "Would you recommend us?".
- **Steps:**
  1. Choose "New survey", leave the title empty, fill Question 1; "Save".
  2. Enter the title, add Question 2; "Save".
  3. Clear Question 2's text; "Save".
  4. Make Questions 2 and 3 "Likelihood to recommend" questions with text; "Save".
  5. Remove Question 3; "Save".
- **Expected result:** (1) "Give the survey a title." (2) "Question 2 needs at least two options to choose between." (3) "Question 2 has no question in it." (4) "Question 3 is a second recommendation score — a survey asks it once." (5) The panel closes; "Post-Event Feedback" is listed as "Draft" with "2 questions · Created" today's date and a "Make live" button.

### TC-MSG-19 — Duplicate, close, reopen and delete a survey
- **Traces:** US-MSG-09  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** "Post-Event Feedback" is Live on Tech Summit 2026 with 30 responses; attendee Nok holds a confirmed ticket and has not answered.
- **Test data:** —
- **Steps:**
  1. Duplicate "Post-Event Feedback".
  2. Choose "Close"; as Nok, open the survey link.
  3. Choose "Reopen"; as Nok, reload the link.
  4. Choose Delete, then "Keep it"; choose Delete again, then "Delete it".
- **Expected result:** (1) "Post-Event Feedback (copy)" appears as "Draft". (2) The row reads "Closed" with "Reopen"; Nok sees "No survey to answer". (3) The row reads "Live" with "Close"; Nok sees the questions. (4) "Delete “Post-Event Feedback”?" appears; "Keep it" leaves the survey; "Delete it" removes it from the list.

---

## US-MSG-10 — Browse and filter individual feedback responses

### TC-MSG-20 — Read responses, filter by stars, empty score, Staff refused
- **Traces:** US-MSG-10  ·  **Priority:** Low  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin. Tech Summit 2026 has 47 responses to "Post-Event Feedback": 12 at 5★, the rest between 4★ and 2★, none at 1★. Staff member "Aran" on the default Staff role.
- **Test data:** —
- **Steps:**
  1. Open Tech Summit 2026's feedback page and read "Responses".
  2. Click the 5★ bar.
  3. Choose "show all".
  4. Click the 1★ bar.
  5. As Aran, open "Feedback".
- **Expected result:** (1) 47 responses, newest first, each with the attendee's name, stars, "Post-Event Feedback" and date, and the comment. (2) 12 responses, all 5★, under "Showing 5★ only — show all". (3) All 47 again. (4) "No responses at that score." with "Show all". (5) "You don't have access to this".


---

<a id="tc-e08"></a>

# Manage Registrations & Admit Attendees — Test Cases

Area: REG · Epic E8 · Locale: Thai market (฿, VAT 7% inclusive, PromptPay, Asia/Bangkok, EN/TH)

Roles referenced: Organizer, Admin (full), Staff (view + door check-in), Attendee (subject only, never operates screens).

---

## US-REG-01 — Review the registrations queue

### TC-REG-01 — Queue shows tabs, counts, status badges and amounts
- **Traces:** US-REG-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as an Organizer with finance access. Event "Bangkok Tech Meetup" has registrations: one confirmed paid, one confirmed free, one free registration awaiting approval, one waitlisted (next in line), one cancelled.
- **Test data:** Paid total ฿1,070 (VAT included).
- **Steps:**
  1. Open Registrations.
  2. Read the tab counts.
  3. Read each row's status badge, note and amount.
- **Expected result:** Tabs All, Pending, Waitlist and Cancelled each show a count. Rows show attendee, event, ticket, registered date, amount and status. Badges read Confirmed, Pending with "Awaiting approval", Waitlisted with "Next in line", and Cancelled. The paid row shows ฿1,070; the free row a "Free" badge.

### TC-REG-02 — Search and event filter combine; no-match state
- **Traces:** US-REG-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as an Organizer. Events A and B have registrations, more than one page in all. "สมชาย ใจดี" is registered for Event A only.
- **Test data:** Name "สมชาย"; email somchai@example.com; order ID ORD-7K2M9QX4.
- **Steps:**
  1. Go to page 2, then search "สมชาย"; repeat with the email and with the order ID.
  2. Clear the search and choose Event A.
  3. Choose Event B and search "สมชาย".
  4. Choose "Clear filters".
- **Expected result:** Each search shows สมชาย's row on page 1 and the tab counts narrow to match. Event A shows only Event A rows. Event B with "สมชาย" shows "No registrations match"; "Clear filters" brings the whole queue back.

### TC-REG-03 — Staff sees "—" for amounts and no decision buttons
- **Traces:** US-REG-01  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Signed in as Staff (may view registrations and check in; no finance access, may not decide). Paid and free registrations exist, one of them pending.
- **Test data:** A confirmed paid registration of ฿2,140.
- **Steps:**
  1. Open Registrations.
  2. Read the Amount column.
  3. Look at the pending row's Actions column.
- **Expected result:** Every amount reads "—", the free one too; ฿2,140 appears nowhere. No row has Approve, "Offer a seat" or Reject. Status, ticket and date still show.

---

## US-REG-02 — Approve or reject a pending registration

### TC-REG-04 — Approve a pending free registration issues one ticket and confirmation
- **Traces:** US-REG-02  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as an Organizer. An event that requires approval has a free registration Pending with "Awaiting approval". SMS confirmations are on.
- **Test data:** Attendee "พลอย ศรีสุข", ploy@example.com, +66 81 234 5678.
- **Steps:**
  1. In Registrations, double-click Approve on Ploy's row.
  2. Read the row.
  3. Check Ploy's inbox and phone.
  4. Open the ticket link in the email.
- **Expected result:** The row reads Confirmed. Ploy gets exactly one confirmation email and one SMS. The link shows one ticket with a QR code.

### TC-REG-05 — Approve is greyed out until payment completes
- **Traces:** US-REG-02  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Signed in as an Organizer. A pending registration for a paid ticket whose PromptPay payment has not settled.
- **Test data:** Total ฿1,605 (VAT included), payment not complete.
- **Steps:**
  1. Open Registrations and find the unpaid row.
  2. Hover over Approve.
  3. Try to click Approve.
- **Expected result:** Approve is greyed out and its tooltip reads "Payment isn't complete yet, so this registration can't be approved." Clicking does nothing; the row stays Pending and no ticket or confirmation is sent.

### TC-REG-06 — Reject frees the seat; rejected and decided rows can't be decided again
- **Traces:** US-REG-02  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Signed in as an Organizer with refund permission. On one event: a free registration awaiting approval, a confirmed registration, and a registration paid while awaiting approval.
- **Test data:** Awaiting approval: "กมล รักดี", kamon@example.com, 1 seat.
- **Steps:**
  1. Note the ticket's "… sold" figure on Tickets; choose Reject on Kamon's row.
  2. Read Kamon's row and hover over its Approve.
  3. Check the sold figure again and Kamon's inbox.
  4. Hover over Approve and Reject on the confirmed row.
  5. Hover over Reject on the paid-while-awaiting row.
- **Expected result:** Kamon's row turns Rejected at once, with no prompt. Its Approve is greyed out: "This registration was rejected, and a rejection cannot be undone." The sold figure is one lower, and Kamon has a rejection email. The confirmed row's buttons are greyed out: "Only a pending or waitlisted registration can be decided." The paid row's button reads "Reject and refund".

---

## US-REG-03 — Add a registration by hand

### TC-REG-07 — Add free and paid bookings by hand, with and without confirmation email
- **Traces:** US-REG-03  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as an Organizer. An upcoming, published event has seats left on a paid ticket type "General" and a free one "Community".
- **Test data:** "ณัฐยา พรหมมา", nattaya@example.com, General, Quantity 2; "อนุชา แก้วมณี", anucha@example.com, Community, Quantity 1; "ปรีชา ดีงาม", preecha@example.com, Community, Quantity 1.
- **Steps:**
  1. Choose "Add registration", fill in Nattaya's booking and choose "Save registration".
  2. Add Anucha's booking with "Send confirmation email" left on.
  3. Add Preecha's booking with "Send confirmation email" turned off.
  4. Read the three new rows in Registrations.
  5. Check the three inboxes and open Attendees.
- **Expected result:** The panel never asks for a price. Nattaya's row is Pending and shows the 2-seat total in ฿, VAT included. Anucha's and Preecha's rows are Confirmed and read "Free". Only Anucha gets a ticket-and-QR email. All three appear in Attendees.

### TC-REG-08 — Seat limits, closed event and missing fields are refused
- **Traces:** US-REG-03  ·  **Priority:** High  ·  **Type:** Edge
- **Preconditions:** Signed in as an Organizer. Event A has exactly 3 seats left on "General". Draft event D has a ticket type.
- **Test data:** Quantities 9 and 5; attendee "มานี ใจงาม", manee@example.com; invalid email "bad@".
- **Steps:**
  1. In "Add registration" choose Event A, "General", Quantity 9; save.
  2. Change Quantity to 5; save.
  3. Change the event to D; save.
  4. Clear Attendee name and type "bad@" in Email; save.
- **Expected result:** Step 1 shows "Choose between 1 and 8 seats per booking."; step 2 "Only 3 left — please reduce your quantity."; step 3 "That event isn't open for new registrations." Each time the panel stays open with my entries and nothing is reserved (Event A still has 3 left). In step 4 the form won't save until the name and email are fixed.

---

## US-REG-04 — Manage the waitlist & promote attendees

### TC-REG-09 — Offer seats to waitlisted attendees; lapse and payment
- **Traces:** US-REG-04  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in as an Organizer, in a test environment with a 1-hour offer window. Paid ticket "General" (฿1,070) has 1 free seat and two people waitlisted; free ticket "Community" has 1 free seat and one person waitlisted.
- **Test data:** General: "วิภา ทองคำ" (wipa@example.com) first, "ธนา สุขใจ" (thana@example.com) second. Community: "สมพร ดีใจ" (somporn@example.com).
- **Steps:**
  1. Read both General rows; hover over "Offer a seat" on Thana's row.
  2. Choose "Offer a seat" on Wipa's row; read it and Wipa's inbox.
  3. Let Wipa's offer lapse unpaid; read both rows and Wipa's inbox.
  4. Pay Thana's offer from his email link with a test payment.
  5. Choose "Offer a seat" on Somporn's row.
- **Expected result:** Rows read "Next in line" and "#2 in line"; Thana's hint warns 1 person ahead of him will be passed over. Wipa's row turns Pending with "Offer open until …" (Bangkok), and she gets an email with a link to pay. After the lapse, Thana's row shows the offer and Wipa gets an "offer expired" email. Thana's payment makes him Confirmed, with tickets and a confirmation email. Somporn is Confirmed at once.

### TC-REG-10 — Offer refused when no seat is free
- **Traces:** US-REG-04  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Signed in as an Organizer. Ticket "General" is sold out and has at least one person waitlisted.
- **Test data:** First in line: "วิภา ทองคำ", wipa@example.com.
- **Steps:**
  1. Choose "Offer a seat" on Wipa's row.
  2. Read the row and Wipa's inbox.
- **Expected result:** The row shows "No seat is free on this ticket — raise its capacity, or wait for one to free up." It stays Waitlisted and "Next in line"; Wipa gets no email.

---

## US-REG-05 — Browse the attendee directory & profiles

### TC-REG-11 — Directory tabs, counts and sort
- **Traces:** US-REG-05  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as an Organizer. Attendees include one first seen 10 days ago, one first seen 60 days ago, one checked in at an event, one tagged VIP, and two with the same number of events.
- **Test data:** Sorts "Sort: Name (A–Z)", "Sort: Most events", "Sort: Most tickets".
- **Steps:**
  1. Open Attendees; read the tabs, counts and order.
  2. Choose each sort in turn.
  3. Choose the New, Checked in and VIP tabs in turn.
- **Expected result:** Tabs All, New, Checked in and VIP show counts; the list opens on "Sort: Last activity", most recent first. Name sorts A–Z; Most events and Most tickets sort largest first, ties going to the more recent activity. New lists the 10-day attendee but not the 60-day one; Checked in lists only checked-in attendees; VIP only VIP-tagged ones.

### TC-REG-12 — Search, tag filter, badges and profile
- **Traces:** US-REG-05  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as an Organizer. Attendees tagged VIP, Speaker, Sponsor and Student exist, plus untagged ones. "อนันต์ สุขสวัสดิ์" (anan@example.com, +66 81 234 5678) is tagged VIP; this Organizer has corrected his phone number.
- **Test data:** Searches "อนันต์", anan@example.com and "zzz-no-match"; tag "VIP".
- **Steps:**
  1. Search "อนันต์", then anan@example.com.
  2. Keep the search and choose tag "VIP".
  3. Clear the filters and look at the Tags column.
  4. Choose "View profile" on Anan's row.
  5. Search "zzz-no-match".
- **Expected result:** Each search shows Anan; adding VIP still shows Anan (both apply). Tagged rows show coloured badges; untagged rows show "—". "Attendee profile" shows Anan's name, VIP badge, email, phone and Activity with "Updated contact details" and its Bangkok date and time, newest first. The last search shows "No attendees match" with "Clear filters".

---

## US-REG-06 — Invite people to an event

### TC-REG-13 — Send an invitation with a personal note
- **Traces:** US-REG-06  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as an Organizer. Event "Bangkok Tech Meetup" is published.
- **Test data:** Recipient "กัญญา ทองดี", kanya@example.com; Message "Hope you can join us!".
- **Steps:**
  1. On Attendees choose "Invite".
  2. Choose the event and fill Recipient name, Email address and Message.
  3. Choose "Send invitation".
  4. Open Kanya's email and its link.
- **Expected result:** "Invitation sent." shows and the panel closes. Kanya gets one invitation with the note at the top, and its link opens the event's page.

### TC-REG-14 — Invite validation, note limit, repeat and Staff refusal
- **Traces:** US-REG-06  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** An Organizer and a Staff member exist. Event "Bangkok Tech Meetup" exists.
- **Test data:** Invalid email "bad@"; a 260-character note; recipient "กัญญา ทองดี", kanya@example.com.
- **Steps:**
  1. As the Organizer, choose "Send invitation" with Recipient name empty, then with Email address "bad@".
  2. Paste the 260-character note into Message.
  3. Send a valid invite to Kanya, then send the same invite again.
  4. As Staff, send a valid invite.
- **Expected result:** The form won't send with an empty name or "bad@". Message keeps only the first 250 characters. Kanya receives one email, not two. Staff's invite is refused with a message in the panel, and no email goes out.

---

## US-REG-07 — Tag & segment attendees

### TC-REG-15 — Apply a VIP tag and see the segment count update
- **Traces:** US-REG-07  ·  **Priority:** Won't  ·  **Type:** Functional
_Withdrawn: setting an attendee's tag is not in the web app; the id stays valid — not run._

### TC-REG-16 — Single-tag rule, re-apply no-op, and concurrent last-write-wins
- **Traces:** US-REG-07  ·  **Priority:** Won't  ·  **Type:** Edge
_Withdrawn: changing an attendee's tag is not in the web app; the id stays valid — not run._

---

## US-REG-08 — Keep attendee contact details current

### TC-REG-17 — Edit name, email and phone; new routing and Activity entry
- **Traces:** US-REG-08  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in as an Organizer. "สุดา ใจงาม" (suda@example.com, +66 81 111 2222) holds a confirmed registration for an upcoming event.
- **Test data:** New name "สุดา ใจงามดี", email suda.j@example.com, phone +66 89 333 4444.
- **Steps:**
  1. Open Suda's profile and choose "Edit".
  2. Change all three fields and choose "Save changes".
  3. Read the row, then reopen the profile.
  4. Send the event's attendees an announcement from Announcements and check where Suda's copy arrives.
  5. Edit again, empty Phone number and save.
- **Expected result:** "Contact details updated." shows; the row and profile show the new name, email and phone. Activity shows "Updated contact details" with the date, time and who made it. Suda's copy goes to suda.j@example.com. After step 5 the phone reads "—".

### TC-REG-18 — Email already in use is refused; invalid fields shown inline
- **Traces:** US-REG-08  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Signed in as an Organizer. Attendee A "มานพ ศรีทอง" (manop@example.com) and attendee B (b.attendee@example.com) exist.
- **Test data:** Email b.attendee@example.com; blank name; email "no-at-sign".
- **Steps:**
  1. Edit A's Email address to b.attendee@example.com and choose "Save changes".
  2. Clear Full name and save.
  3. Enter "no-at-sign" as the email and save.
- **Expected result:** Step 1 shows "That email address already belongs to another attendee in this workspace. Merge the two records instead of overwriting…" above the form, and nothing is saved. Steps 2 and 3 show an error under that field, and nothing is saved. A's row is unchanged.

---

## US-REG-09 — Email an attendee directly

### TC-REG-19 — Send a one-off email; empty subject/message blocked; failure retryable
- **Traces:** US-REG-09  ·  **Priority:** Won't  ·  **Type:** Functional
_Withdrawn: sending an email to an attendee from Eventa is not in the web app; the id stays valid — not run._

---

## US-REG-10 — Export registration & attendee lists

### TC-REG-20 — Export exactly the filtered set with audit record and correct Thai names
- **Traces:** US-REG-10  ·  **Priority:** Won't  ·  **Type:** Functional
_Withdrawn: exporting Registrations or Attendees is not in the web app; the id stays valid — not run._

### TC-REG-21 — Staff has no Export control; empty filter yields headers-only file
- **Traces:** US-REG-10  ·  **Priority:** Won't  ·  **Type:** Edge
_Withdrawn: exporting Registrations or Attendees is not in the web app; the id stays valid — not run._

---

## US-REG-11 — Run the manual check-in list

### TC-REG-22 — Check-in list, progress bar, search and check in
- **Traces:** US-REG-11  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Staff with check-in permission. Event "Bangkok Tech Meetup" is inside its check-in window with 29 valid tickets, 7 checked in, plus one refunded ticket.
- **Test data:** Not yet in: "ศิริพร ทองดี", siriporn@example.com.
- **Steps:**
  1. Open Check-in and choose the event.
  2. Read the progress strip and tabs.
  3. Search "ศิริพร", then her email.
  4. Choose "Check in" on her row.
- **Expected result:** It reads "7 / 29 attendees checked in", 24%; tabs read All 29, Checked in 7, Not yet 22, and the refunded ticket is not listed. Each search shows her row on page 1. After check-in her row shows the time (HH:MM, Bangkok) and "Checked in"; the strip reads "8 / 29", 28%, and Not yet 21.

### TC-REG-23 — Undo a check-in; scan plus manual counts once
- **Traces:** US-REG-11  ·  **Priority:** High  ·  **Type:** Edge
- **Preconditions:** Signed in as Staff. The event is inside its check-in window. "สมศักดิ์ มีสุข" is checked in at 09:12.
- **Test data:** Somsak's ticket QR.
- **Steps:**
  1. On Check-in choose "Undo" on Somsak's row.
  2. Read his row and the Not yet count.
  3. Choose "Check in" on his row again.
  4. At the station, scan his QR; then reload Check-in.
- **Expected result:** Undo clears his time and Not yet goes up by one. Checking in again shows the new time. The scan reads "Already checked in" with that same time, and the list still counts him once, keeping the earlier time.

### TC-REG-24 — Check-in refused outside the check-in window
- **Traces:** US-REG-11  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Signed in as Staff. Event T starts tomorrow at 10:00 (window opens 08:00 tomorrow). Event P ended 3 hours ago and has a checked-in attendee.
- **Test data:** Any not-yet row on Event T; the checked-in row on Event P.
- **Steps:**
  1. Choose Event T and choose "Check in" on a row.
  2. Choose Event P and choose "Undo" on the checked-in row.
- **Expected result:** Both are refused with "Check-in is not open for this event yet — or has already closed." under the row, and no count changes. Both lists still load.

---

## US-REG-12 — Scan tickets at the live QR check-in station

### TC-REG-25 — A valid scan admits once; switching event reloads the station
- **Traces:** US-REG-12  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Staff on a phone with a camera, in a browser that reads QR codes (e.g. Chrome on Android), over https. Event A is inside its check-in window; Event B exists.
- **Test data:** Unused Event A ticket QR for "นภา วงศ์ใหญ่".
- **Steps:**
  1. On Check-in for Event A choose "Open check-in tool", then "Start camera".
  2. Hold Napa's QR in the frame for about 2 seconds.
  3. Read the result, the count and "Just checked in".
  4. Choose Event B in the station's event picker.
- **Expected result:** A green "Checked in" shows "นภา วงศ์ใหญ่" and "Welcome — let them through."; the count rises by one and Napa tops the feed, once only. The station reloads with Event B's counts and feed.

### TC-REG-26 — Already used, wrong event, refunded and unknown codes
- **Traces:** US-REG-12  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Station on Event A, camera running.
- **Test data:** (a) an Event A ticket checked in at 09:12; (b) a valid Event B ticket; (c) a refunded Event A ticket; (d) a QR of https://example.com.
- **Steps:**
  1. Scan (a).
  2. Scan (b).
  3. Scan (c).
  4. Scan (d).
- **Expected result:** (a) "Already checked in", "Arrived at 09:12"; (b) "Wrong event", "This ticket is for another event."; (c) "Entry denied", "This ticket has been refunded or voided."; (d) "Not a valid ticket", "This code is not one of ours." No one is admitted and the count does not change.

### TC-REG-27 — Camera unavailable points to a fallback
- **Traces:** US-REG-12  ·  **Priority:** High  ·  **Type:** Edge
- **Preconditions:** Station on Event A. The browser blocks the camera for this site.
- **Test data:** A photo of a valid, unused Event A ticket QR.
- **Steps:**
  1. Choose "Start camera".
  2. Read the message.
  3. Choose "Upload QR" and pick the photo.
  4. Open the station from a plain http (not https) address and read the camera area.
- **Expected result:** Step 2 reads "Camera access was blocked. Allow the camera for this site in the browser settings." and then "Upload a photo of the QR code instead, or use “Can’t scan? Find attendee manually” below." The upload admits the ticket ("Checked in"). Over http it reads "This page is not secure (https), so the browser will not scan QR codes here." and points to the manual lookup. The station never breaks.

---

## US-REG-13 — Check attendees in manually when a QR won't scan

### TC-REG-28 — Manual lookup checks someone in like a scan
- **Traces:** US-REG-13  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Staff. Station on Event A, inside its check-in window. "จันทร์เพ็ญ ศรีสวัสดิ์" (chanpen@example.com) is not yet checked in.
- **Test data:** Searches "จันทร์เพ็ญ" and chanpen@example.com.
- **Steps:**
  1. Open "Can’t scan? Find attendee manually".
  2. Search "จันทร์เพ็ญ", then her email.
  3. Choose "Check in" on her row.
  4. Read the count and "Just checked in".
- **Expected result:** Each search lists her. After "Check in" she is admitted, the count rises by one, she tops "Just checked in", and her search row shows "In at HH:MM".

### TC-REG-29 — Already checked in shows the time; no match shows a message
- **Traces:** US-REG-13  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Station on Event A. "สมศักดิ์ มีสุข" checked in at 09:12.
- **Test data:** Searches "สมศักดิ์" and "zzz-no-match".
- **Steps:**
  1. In the manual lookup search "สมศักดิ์".
  2. Search "zzz-no-match".
- **Expected result:** Somsak's row shows "In at 09:12" and no "Check in" button. The second search shows "No attendees match your search."

---

## US-REG-14 — Watch live turnout at the door

### TC-REG-30 — Counts and feed update per check-in and per event
- **Traces:** US-REG-14  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in as an Organizer. Event A is inside its check-in window with 200 valid tickets, 117 checked in. Event B has its own check-ins; Event C has none.
- **Test data:** Three not-yet Event A attendees: one ticket QR, one QR photo, one found by name.
- **Steps:**
  1. Open the station for Event A and read the "Checked in" card.
  2. Check in the three: by camera, by "Upload QR" and by manual lookup.
  3. Read the card and "Just checked in".
  4. Switch to Event B, then to Event C.
- **Expected result:** The card first reads 117 / 200 and 59%, with On-site and Late adding up to 117 and Remaining 83. After the three it reads 120 / 200, 60%, Remaining 80. Each arrival tops the feed with ticket type and time, and the feed shows at most 8. Event B shows its own figures; Event C's feed reads "Nobody has come through yet."


---

<a id="tc-e09"></a>

# Get Paid & Manage Finances — Test Cases

Epic E9 — Get Paid & Manage Finances. Area code: **FIN**. Locale defaults: ฿ (VAT-inclusive), VAT 7%, Asia/Bangkok time, EN/TH, PromptPay / Card / Bank transfer. RBAC baseline: Admin = full control; Organizer = view/invoice/export only; Staff & Attendee = no finance access.

---

## US-FIN-01 — See every payment

### TC-FIN-01 — Tabs, method and event filters, and search on Payments
- **Traces:** US-FIN-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as an Organizer. Payments exist in all four statuses, on Card, PromptPay and Bank transfer, for "Tech Summit 2026" and "Riverside Craft Fair 2026".
- **Test data:** Paid PromptPay payment TXN-90210 by "Somchai Jaidee", ฿1,284, for "Tech Summit 2026".
- **Steps:**
  1. Open Finance → Payments and note each tab's count.
  2. Choose the method "PromptPay" and note the counts again.
  3. Search "Somch", then replace it with "90210".
  4. Choose the Paid tab, then the event "Tech Summit 2026".
- **Expected result:** (1) All equals Paid + Pending + Refunded + Failed. (2) Every count now covers PromptPay payments only. (3) Both searches list TXN-90210 with Somchai Jaidee, Tech Summit 2026, PromptPay, ฿1,284, a Paid badge and its Bangkok date and time. (4) Only Paid PromptPay payments for that event matching the search show; choosing the tab left the counts unchanged.

### TC-FIN-02 — Newest first, and a greyed Refund that says why
- **Traces:** US-FIN-01  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Signed in as an Admin. Payments include a Pending PromptPay attempt, a Failed card attempt, a Refunded payment and Paid payments made at different times.
- **Test data:** —
- **Steps:**
  1. Open Payments and read the order of the rows and their dates.
  2. Point at the Refund button on the Pending row, then the Failed row, then the Refunded row.
- **Expected result:** (1) Paid and Refunded rows run newest first; the Pending and Failed attempts show "—" as their date and sit above them. (2) Refund is greyed on all three, saying "No completed charge yet.", "This attempt failed, so it is not eligible for refund." and "This payment was already refunded." in turn.

### TC-FIN-03 — A PromptPay charge turns Paid on reload
- **Traces:** US-FIN-01  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Signed in as an Organizer. The payment provider is in test mode. A buyer has started a PromptPay payment and not paid yet, so Payments shows it Pending.
- **Test data:** PromptPay charge of ฿980.
- **Steps:**
  1. Note the Pending and Paid tab counts.
  2. Complete the ฿980 PromptPay payment in the provider's test mode.
  3. Reload Payments.
- **Expected result:** The row reads Paid with its Bangkok paid date and time; Pending is one lower and Paid one higher. Nobody edited the row.

---

## US-FIN-02 — Issue a refund

### TC-FIN-04 — Admin refunds a paid card payment in full
- **Traces:** US-FIN-02  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as an Admin. The payment provider is in test mode. "Somchai Jaidee" paid by card for 2 tickets to "Tech Summit 2026" this month (TXN-90210), and invoice INV-2026-0146 was issued for the order. Note the ticket type's sold count and this month's row on Taxes.
- **Test data:** TXN-90210, ฿2,140 (฿2,000 + ฿140 VAT).
- **Steps:**
  1. On Payments, choose the Refund button on TXN-90210 and read the dialog.
  2. Choose "Refund".
  3. Read the payment row, the ticket type's sold count on Tickets, and INV-2026-0146 on Invoices.
  4. Open Taxes and read this month's row.
- **Expected result:** (1) "Refund this payment?" shows "TXN-90210 · Somchai Jaidee · ฿2,140" and "This cannot be undone." (2) "Refund started." shows. (3) The payment reads Refunded, the sold count is 2 lower, and INV-2026-0146 reads Void. (4) This month's Sales (ex-VAT) is ฿2,000 lower and its VAT ฿140 lower.

### TC-FIN-05 — Organizer cannot refund
- **Traces:** US-FIN-02  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Signed in as an Organizer. A Paid payment exists.
- **Test data:** Paid charge ฿2,140.
- **Steps:**
  1. Open Payments.
  2. Point at the Refund button on the Paid row and try to choose it.
- **Expected result:** Refund is greyed with "Only an Admin can issue refunds."; no dialog opens and the payment stays Paid.

### TC-FIN-06 — Pressing Refund twice issues one refund
- **Traces:** US-FIN-02  ·  **Priority:** High  ·  **Type:** Edge
- **Preconditions:** Signed in as an Admin. The payment provider is in test mode. A Paid card payment exists.
- **Test data:** Paid charge ฿5,000.
- **Steps:**
  1. Choose the Refund button on the payment.
  2. Double-click "Refund" in the dialog.
  3. Read the payment row and the refunds for that charge in the provider's test dashboard.
- **Expected result:** The button reads "Refunding…" and is disabled while it works. Exactly one ฿5,000 refund exists for the charge, and the payment reads Refunded.

---

## US-FIN-03 — Track balances and payout history

### TC-FIN-07 — Three balances, and payout history by status
- **Traces:** US-FIN-03  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as an Organizer. A payout account ending 6789 is connected and has payouts in every status.
- **Test data:** PO-2026-0031 Paid ฿30,000; PO-2026-0032 Processing ฿48,000; PO-2026-0033 Scheduled ฿12,000; PO-2026-0030 Failed ฿5,000.
- **Steps:**
  1. Open Finance → Payouts and read the three tiles.
  2. Choose the Scheduled, Processing, Paid and Failed tabs in turn.
- **Expected result:** (1) Available, Pending and Paid out show in ฿; Pending reads ฿60,000 (฿48,000 + ฿12,000) and Paid out ฿30,000. (2) Each tab lists only its payouts, newest first, with reference, period, "•••• 6789", amount, status and Requested date; Completed reads "—" except on PO-2026-0031. No full bank number appears anywhere.

### TC-FIN-08 — No payout account: no ฿0, asked to connect
- **Traces:** US-FIN-03  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Signed in as an Admin. Case A: a new workspace with no payout account and no payouts. Case B: a workspace whose payout account was disconnected after payouts were made.
- **Test data:** —
- **Steps:**
  1. Case A: open Payouts.
  2. Case B: open Payouts.
- **Expected result:** (1) "No payouts yet" offers "Connect a payout account"; no balance shows as ฿0. (2) The three tiles read "—", a banner says "Connect a payout account to receive the money from your ticket sales." with "Connect account", and the history still lists the past payouts.

---

## US-FIN-04 — Follow and recover a payout

### TC-FIN-09 — Admin retries a failed payout
- **Traces:** US-FIN-04  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in as an Admin. A payout account ending 6789 is connected and the provider (test mode) will accept a retry. PO-2026-0030 failed last week with a reason.
- **Test data:** PO-2026-0030, ฿5,000.
- **Steps:**
  1. Open Payouts, choose the Failed tab and read PO-2026-0030's row.
  2. Choose "Retry payout".
  3. Choose the All tab and read the rows and the Pending tile.
  4. Open Settings → Security → Audit log.
- **Expected result:** (1) The reason shows under the Failed badge, with a "Retry payout" button. (3) PO-2026-0030 reads Processing with today as Requested; no second ฿5,000 payout appears; Pending is ฿5,000 higher. (4) "Retried payout PO-2026-0030" is listed.

### TC-FIN-10 — Organizer's retry is refused
- **Traces:** US-FIN-04  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Signed in as an Organizer. PO-2026-0030 is Failed.
- **Test data:** PO-2026-0030, ฿5,000.
- **Steps:**
  1. Open Payouts and choose the Failed tab.
  2. Choose "Retry payout" on PO-2026-0030.
  3. Reload Payouts.
- **Expected result:** PO-2026-0030 still reads Failed with its reason and its old Requested date; no new payout appears.

---

## US-FIN-05 — Manage bank and payout settings

### TC-FIN-11 — Admin opens payout settings at the provider, or connects first
- **Traces:** US-FIN-05  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as an Admin. Case A: a payout account is connected. Case B: none is connected.
- **Test data:** —
- **Steps:**
  1. Case A: open Payouts and choose "Manage payouts".
  2. Case B: open Payouts, look for "Manage payouts", then choose "Connect a payout account" (or "Connect account").
- **Expected result:** (1) The browser leaves Eventa for the payment provider's own dashboard, on its payout settings, where the bank account, payout schedule, tax forms and statements are managed. (2) "Manage payouts" is not offered; the link opens Settings → Payments to connect the provider.

### TC-FIN-12 — Organizer has no "Manage payouts"
- **Traces:** US-FIN-05  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Signed in as an Organizer. A payout account ending 6789 is connected.
- **Test data:** —
- **Steps:**
  1. Open Payouts.
  2. Look in the page header.
- **Expected result:** No "Manage payouts" button. The balances and history still show, with the bank account only as "•••• 6789".

---

## US-FIN-06 — Invoice ledger with ageing

### TC-FIN-13 — Overdue or Issued by the Bangkok day
- **Traces:** US-FIN-06  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as an Organizer. Run between 00:00 and 07:00 Bangkok time, while the UTC date is still yesterday. Five unpaid invoices exist.
- **Test data:** INV-2026-0140 due 3 days ago; INV-2026-0141 due in 9 days; INV-2026-0143 due tomorrow; INV-2026-0144 due today; INV-2026-0145 due yesterday (all Bangkok dates).
- **Steps:**
  1. Open Finance → Invoices.
  2. Read the status and due date of each invoice.
- **Expected result:** INV-2026-0140 reads Overdue with "Overdue by 3 days" in red; INV-2026-0141 Issued, "Due in 9 days"; INV-2026-0143 Issued, "Due tomorrow"; INV-2026-0144 Issued, "Due today"; INV-2026-0145 Overdue, "Overdue by 1 day".

### TC-FIN-14 — Tabs, event filter and search; Paid and Void rows
- **Traces:** US-FIN-06  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as an Organizer. Invoices in all four statuses exist for "Tech Summit 2026" and "Riverside Craft Fair 2026". INV-2026-0142 for "Naruemon Co., Ltd." is Issued and its order is not yet paid.
- **Test data:** "INV-2026-0142", "Naruemon".
- **Steps:**
  1. Open Invoices and note each tab's count.
  2. Choose the Overdue tab, then the event "Tech Summit 2026".
  3. Clear the filters; search "INV-2026-0142", then "Naruemon".
  4. Read the due date of a Paid row and of a Void row.
  5. Have the buyer pay INV-2026-0142's order, then reload Invoices.
- **Expected result:** (1) All equals Issued + Paid + Overdue + Void. (2) Only overdue invoices for that event show; the counts now cover that event only, and choosing the tab left them unchanged. (3) Both searches find INV-2026-0142. (4) They read "Paid" and "Void". (5) INV-2026-0142 reads Paid, with "Paid" under its due date.

---

## US-FIN-07 — Issue a VAT invoice

### TC-FIN-15 — Issue a compliant invoice with sequential number, 14-day term and 7% VAT
- **Traces:** US-FIN-07  ·  **Priority:** Won't  ·  **Type:** Functional
_Withdrawn: issuing an invoice is not in the web app; the id stays valid — not run._

### TC-FIN-16 — Validation and duplicate-issue guard
- **Traces:** US-FIN-07  ·  **Priority:** Won't  ·  **Type:** Negative
_Withdrawn: issuing an invoice is not in the web app; the id stays valid — not run._

---

## US-FIN-08 — View and download a tax invoice

### TC-FIN-17 — The downloaded invoice adds up to its total
- **Traces:** US-FIN-08  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as an Organizer. INV-2026-0142 for "Tech Summit 2026" was paid by card on 2026-07-20.
- **Test data:** INV-2026-0142, ฿32,500; buyer "Naruemon Co., Ltd.", billing@naruemon.example.com.
- **Steps:**
  1. Open Invoices and choose "Invoice" on INV-2026-0142.
  2. Open the downloaded file.
- **Expected result:** The file is named INV-2026-0142 and reads "TAX INVOICE" with that number. It shows the buyer's name and email, the order, the issued and due dates, and Payment "Card · 2026-07-20". The line "Event registration — Tech Summit 2026" reads ฿30,373.83; "Subtotal (excl. VAT)" ฿30,373.83 plus "VAT 7%" ฿2,126.17 equals "Total" ฿32,500.

### TC-FIN-18 — One A4 page with my organization's details; a refunded sale's invoice reads VOID
- **Traces:** US-FIN-08  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as an Admin. The payment provider is in test mode. Before INV-2026-0142 was issued, Settings → Organization held the details below. Its card payment is Paid.
- **Test data:** Organization name "Test Events (Thailand) Co., Ltd."; Address "99 Sukhumvit Road, Bangkok 10110"; Tax ID "0105599999999".
- **Steps:**
  1. On Invoices, choose "Invoice" on INV-2026-0142 and open the file.
  2. On Payments, refund INV-2026-0142's payment.
  3. On Invoices, choose "Invoice" on INV-2026-0142 again and open the file.
- **Expected result:** (1) A single A4 page shows "Test Events (Thailand) Co., Ltd.", its address and "Tax ID 0105599999999" above the buyer. (3) The row reads Void; the document keeps INV-2026-0142 and carries a large "VOID" mark.

---

## US-FIN-09 — Chase unpaid invoices

### TC-FIN-19 — Resend, automatic overdue reminders (EN/TH), stop-on-settle and flood limit
- **Traces:** US-FIN-09  ·  **Priority:** Won't  ·  **Type:** Functional
_Withdrawn: resending invoices and overdue reminders are not in the web app; the id stays valid — not run._

---

## US-FIN-10 — Void an invoice

### TC-FIN-20 — Admin voids an Issued invoice; its number stays and the void is logged
- **Traces:** US-FIN-10  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as an Admin. INV-2026-0150 was raised in error and is Issued, unpaid. Note this month's VAT on Taxes.
- **Test data:** INV-2026-0150, ฿12,000, buyer "Kittisak Wongsawat"; Reason "Duplicate order".
- **Steps:**
  1. On Invoices, choose the Void button on INV-2026-0150 and read the dialog.
  2. Type "Duplicate order" as the Reason and choose "Void invoice".
  3. Choose the Void tab.
  4. Open Settings → Security → Audit log.
  5. Open Taxes and read this month's VAT.
- **Expected result:** (1) "Void this invoice?" shows "INV-2026-0150 · Kittisak Wongsawat · ฿12,000" and "The number stays in the sequence, marked void. It is not deleted." (2) "Invoice voided." shows. (3) INV-2026-0150 is listed with its number, a Void badge and "Void" under its due date. (4) "Voided invoice INV-2026-0150" is listed with "Duplicate order". (5) This month's VAT is unchanged.

### TC-FIN-21 — Void refused for Paid and Void invoices and for an Organizer
- **Traces:** US-FIN-10  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** A Paid invoice, a Void invoice and an Issued invoice exist. Admin and Organizer accounts are available.
- **Test data:** Paid INV-2026-0139 ฿9,000; Void INV-2026-0138; Issued INV-2026-0151.
- **Steps:**
  1. As the Admin, point at the Void button on INV-2026-0139, then on INV-2026-0138.
  2. As the Organizer, choose the Void button on INV-2026-0151, then "Void invoice".
  3. Reload Invoices.
- **Expected result:** (1) Both are greyed, saying "Paid — void it by refunding the payment." and "Already void." (2) The dialog stays open and shows a permission refusal. (3) INV-2026-0151 still reads Issued.

---

## US-FIN-11 — Monthly VAT ledger

### TC-FIN-22 — VAT at 7%, and year headlines that match the rows
- **Traces:** US-FIN-11  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as an Organizer. In 2026, buyers paid ฿3,223,696 in June with no refunds; other months also have sales; May is filed.
- **Test data:** June 2026: Sales (ex-VAT) ฿3,012,800, VAT ฿210,896.
- **Steps:**
  1. Open Finance → Taxes with "Tax year" 2026.
  2. Read June's row.
  3. Total the VAT, Remitted and Withholding columns over the 12 rows and compare with the headlines.
- **Expected result:** (2) June reads Sales (ex-VAT) ฿3,012,800 and VAT ฿210,896. (3) VAT collected equals the VAT total, VAT remitted the Remitted total, Withholding the Withholding total, and VAT payable equals VAT collected minus VAT remitted.

### TC-FIN-23 — Twelve months, due dates, statuses and the tax-year picker
- **Traces:** US-FIN-11  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in as an Organizer. Today is 2026-07-20 (Bangkok). May 2026 was filed with withholding; June has not been filed.
- **Test data:** May 2026 withholding ฿4,500.
- **Steps:**
  1. Open Taxes without choosing a year.
  2. Read the May, June, July and August rows and the Withholding headline.
  3. Open the "Tax year" list, then choose 2025.
- **Expected result:** (1) It opens on 2026 with 12 rows, Jan to Dec. (2) May reads Filed with Withholding ฿4,500; June reads Due (although Jul 15, 2026 has passed); July (due Aug 15, 2026) and August read Upcoming. The Withholding headline includes the ฿4,500. (3) The list runs 2026 back to 2021; 2025 shows its own 12 months and headlines.

---

## US-FIN-12 — File the VAT return each period

### TC-FIN-24 — Admin records the filing for a Due month
- **Traces:** US-FIN-12  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as an Admin. Today is 2026-07-10 (Bangkok). June 2026 is Due and not filed.
- **Test data:** June VAT ฿210,896; Withholding tax 4500.
- **Steps:**
  1. On Taxes, choose "Record filing" on June and read the dialog.
  2. Enter 4500 in "Withholding tax (฿)" and choose "Record filing".
  3. Read June's row and the headlines.
- **Expected result:** (1) "Record this filing?" shows "Jun · VAT ฿210,896 · due Jul 15, 2026" and "This records the VAT above as remitted and marks the period Filed." (2) "VAT period filed." shows. (3) June reads Filed (no "Filed late"), Remitted ฿210,896, Withholding ฿4,500; VAT remitted is ฿210,896 higher, VAT payable ฿210,896 lower and Withholding ฿4,500 higher.

### TC-FIN-25 — Upcoming not offered, late filing marked, later refund lands in the open month, Organizer refused
- **Traces:** US-FIN-12  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Today is 2026-07-20 (Bangkok). April and June 2026 are Due, May is Filed, July is Upcoming. A ฿2,140 card payment was made in May. Admin and Organizer accounts are available.
- **Test data:** —
- **Steps:**
  1. As the Admin, look for "Record filing" on July.
  2. As the Admin, record June's filing.
  3. As the Admin, refund the May payment on Payments, then reopen Taxes.
  4. As the Organizer, choose "Record filing" on April and confirm.
- **Expected result:** (1) Not offered. (2) June reads Filed with "Filed late". (3) May's row is unchanged; July's Sales (ex-VAT) is ฿2,000 lower and its VAT ฿140 lower. (4) The dialog shows a permission refusal and April stays Due.

---

## US-FIN-13 — Export finance ledgers

### TC-FIN-26 — Exports match the screen, Thai names read, an empty set is refused
- **Traces:** US-FIN-13  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in as an Organizer. More Refunded payments exist than fit on one page, one of them by "สมชาย ใจดี". Overdue invoices and 2026 VAT months exist.
- **Test data:** Refunded payment by "สมชาย ใจดี", ฿2,140; search "zzz-no-match".
- **Steps:**
  1. On Payments, choose the Refunded tab, then "Export"; open the file in a spreadsheet.
  2. On Invoices, choose the Overdue tab, then "Export".
  3. On Taxes, with "Tax year" 2026, choose "Export".
  4. On Payments, search "zzz-no-match", then choose "Export".
- **Expected result:** (1) The file holds every Refunded payment, from all pages and no other status, with its amount VAT included; "สมชาย ใจดี" reads correctly. (2) Only Overdue invoices, each with subtotal plus VAT equal to its total. (3) The 12 months of 2026 with sales, VAT, remitted, payable, withholding and status. (4) "There is nothing to export — no payments match these filters." shows under Export and no file downloads.

---

## US-FIN-14 — Keep finance to finance roles

### TC-FIN-27 — Finance access by role
- **Traces:** US-FIN-14  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** One workspace with an Attendee and a Staff, Organizer and Admin member on the default roles. A Paid payment, a Failed payout, an Issued invoice and a Due VAT month exist.
- **Test data:** One user per role.
- **Steps:**
  1. As the Attendee, open the Payments address directly.
  2. As Staff, open Finance → Payments, Payouts, Invoices and Taxes.
  3. As the Organizer, open the same four pages, export Payments, and try Refund, "Retry payout", "Void invoice" and "Record filing".
  4. As the Admin, repeat step 3.
- **Expected result:** (1) They are sent to the organizer sign-in; no Finance data shows. (2) Each page reads "You don't have access to this" with no figures. (3) All four pages open and the export downloads; Refund is greyed with "Only an Admin can issue refunds.", "Manage payouts" is not offered, and the retry, void and filing are refused, changing nothing. (4) Every action works.


---

<a id="tc-e10"></a>

# Build the Event Program — Test Cases

Epic E10 — Build the Event Program. Area code: PROG. Locale context: Asia/Bangkok timezone, EN/TH bilingual content.

---

## US-PROG-01 — See the event's schedule at a glance

### TC-PROG-01 — Agenda shows only the selected event's sessions in correct slots
- **Traces:** US-PROG-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Organizer signed in. "Bangkok Tech Summit" runs 11–13 January 2027 and has sessions; "HR Forum" also has sessions.
- **Test data:** "Keynote: AI in ASEAN", Day 1 09:00–10:00, Keynote, speaker Somsak P.; "Panel: FinTech", Day 2 14:00–15:00, Panel, speaker Anong K.; "Lunch", Day 1 12:00–13:00, Break, room Foyer, no speaker.
- **Steps:**
  1. Open "Agenda" and pick "Bangkok Tech Summit" in the event picker.
  2. Read the day columns and each block.
- **Expected result:** Three columns read Mon 11, Tue 12 and Wed 13. "Keynote: AI in ASEAN" is a blue block on Day 1 at 09:00 reading "09:00 – 10:00" and Somsak P.; "Panel: FinTech" is green on Day 2 at 14:00 with Anong K.; "Lunch" is grey and shows "Foyer". No "HR Forum" session appears.

### TC-PROG-02 — Empty agenda shows a friendly prompt and event switch reloads correctly
- **Traces:** US-PROG-01  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Organizer signed in. "New Meetup 2026" has no sessions; "Bangkok Tech Summit" has sessions.
- **Test data:** —
- **Steps:**
  1. Open "Agenda" and pick "New Meetup 2026".
  2. Pick "Bangkok Tech Summit".
- **Expected result:** (1) "No sessions scheduled yet" shows with "Add the first session" and "Add speakers"; no error. (2) Only Bangkok Tech Summit's sessions show.

---

## US-PROG-02 — Add a session to the schedule

### TC-PROG-03 — Add a valid session and speaker count increments
- **Traces:** US-PROG-02  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Organizer signed in, "Bangkok Tech Summit" (11–13 January 2027) picked on "Agenda". Speaker "Somsak P." on this event shows "2 sessions". Hall A is free on Day 2 from 13:00 to 14:30.
- **Test data:** Title "Cloud Native Workshop"; Day "Day 2 · Tue 12"; Type Workshop; Starts 13:00; Ends 14:30; Room Hall A; Speakers: Somsak P.; Description "Hands-on with containers".
- **Steps:**
  1. Choose "New session" and enter the test data.
  2. Choose "Save session".
  3. Open "Speakers" for the same event.
- **Expected result:** (2) "Session saved." shows and an amber block on Day 2 at 13:00 reads "13:00 – 14:30", "Cloud Native Workshop" and Somsak P. (3) Somsak P.'s card reads "3 sessions".

### TC-PROG-04 — Reject blank title and invalid time ranges
- **Traces:** US-PROG-02  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Organizer signed in, "Bangkok Tech Summit" picked on "Agenda".
- **Test data:** (a) Title blank, Starts 09:00, Ends 10:00; (b) Title "Late Session", Starts 16:00, Ends 15:00; (c) Title "Zero Session", Starts 15:00, Ends 15:00.
- **Steps:**
  1. In "New session", try each of (a)–(c) on its own and choose "Save session".
- **Expected result:** (a) The form asks for a Title and sends nothing. (b) and (c) "A session end time must be after its start time." shows in the panel. No block is added in any case.

---

## US-PROG-03 — Update a session

### TC-PROG-05 — Move a session to a new slot with no clash
- **Traces:** US-PROG-03  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Organizer signed in. "Design Sprint" (no speakers) is a Panel on Day 1 10:00–11:00 in Room B; Room C is free on Day 1 from 15:00 to 16:00.
- **Test data:** Starts 15:00; Ends 16:00; Room C; Type Workshop.
- **Steps:**
  1. On "Agenda", click the "Design Sprint" block.
  2. Enter the test data and choose "Save session".
- **Expected result:** (1) "Edit session" opens with Design Sprint's details filled in. (2) "Session saved." shows; the block now sits on Day 1 at 15:00, reads "15:00 – 16:00" and "Room C", and has turned from green to amber.

### TC-PROG-06 — Concurrent edit is blocked with reload prompt
- **Traces:** US-PROG-03  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Organizers Nattapong and Sirin both have "Opening Keynote" (09:00–10:00, Hall A) open in "Edit session".
- **Test data:** Sirin changes Room to Hall B; Nattapong changes Starts to 09:30.
- **Steps:**
  1. Sirin chooses "Save session".
  2. Nattapong chooses "Save session".
  3. Nattapong reloads "Agenda" and clicks "Opening Keynote".
- **Expected result:** (1) "Session saved." for Sirin. (2) Nattapong sees "This session changed elsewhere. Reload and try again." and his change is not saved. (3) "Edit session" shows Room Hall B and Starts 09:00.

---

## US-PROG-04 — Remove a session

### TC-PROG-07 — Confirmed removal drops speaker count and clears public agenda
- **Traces:** US-PROG-04  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** "Bangkok Tech Summit" is published and its public page lists "Startup Pitches" in the agenda. Speaker "Anong K." of that session shows "3 sessions".
- **Test data:** —
- **Steps:**
  1. On "Agenda", click "Startup Pitches" and choose "Delete".
  2. Read the dialog, then choose "Remove".
  3. Open "Speakers", then the event's public page.
- **Expected result:** (2) "Remove session?" reads "Removing this session won’t notify attendees who bookmarked it, and can’t be undone."; after "Remove", "Session removed." shows and the block is gone. (3) Anong K. is still listed, now with "2 sessions"; the public agenda no longer lists Startup Pitches.

### TC-PROG-08 — Cancelling the confirmation changes nothing
- **Traces:** US-PROG-04  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** "Networking Break" exists with speaker "Wanida C.", who shows "2 sessions".
- **Test data:** —
- **Steps:**
  1. On "Agenda", click "Networking Break" and choose "Delete".
  2. Choose "Cancel".
  3. Check the agenda and Wanida C.'s card on "Speakers".
- **Expected result:** The session is still in its slot and Wanida C. still shows "2 sessions".

---

## US-PROG-05 — Keep the schedule physically feasible

### TC-PROG-09 — Room double-booking is blocked with a clear message
- **Traces:** US-PROG-05  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Organizer signed in. "Opening Keynote" is in Hall A on Day 1 09:00–10:30.
- **Test data:** "Second Talk", Day 1, Starts 10:00, Ends 11:00, Room Hall A.
- **Steps:**
  1. Choose "New session", enter the test data and choose "Save session".
- **Expected result:** The panel shows "Hall A is already booked for "Opening Keynote" from 09:00 to 10:30. Choose another room or time." and no block is added.

### TC-PROG-10 — Speaker overlap warns and needs confirm; parallel tracks allowed; no self-clash
- **Traces:** US-PROG-05  ·  **Priority:** High  ·  **Type:** Edge
- **Preconditions:** "Data Ethics" is on Day 1 11:00–12:00 in Room A with speaker Ploy S. Speaker Wichai T. has no sessions. Rooms B and C are free.
- **Test data:** (a) "Cloud Costs", Day 1 11:00–12:00, Room C, Wichai T.; (b) "Data Lab", Day 1 10:00–11:00, Room A, Wichai T.; (c) "Ethics Q&A", Day 1 11:30–12:30, Room B, Ploy S.
- **Steps:**
  1. Click "Data Ethics" and choose "Save session" without changes.
  2. Add (a) and save.
  3. Add (b) and save.
  4. Add (c) and save; then tick "Book them anyway" and save again.
- **Expected result:** (1)–(3) Each shows "Session saved." with no warning: a session never clashes with itself, parallel rooms are fine, and back-to-back at 11:00 is not a clash. (4) The first save shows "Ploy S. is already speaking in "Data Ethics" at this time. Confirm to assign them anyway." and nothing is added; after ticking, "Session saved." and the block appears.

---

## US-PROG-06 — Find sessions in a busy programme

### TC-PROG-11 — Keyword search filters by title/type (EN & TH) and clears cleanly
- **Traces:** US-PROG-06  ·  **Priority:** Won't  ·  **Type:** Functional
_Withdrawn: agenda search is not in the web app; the id stays valid — not run._

---

## US-PROG-07 — Switch between week and month views

### TC-PROG-12 — Toggle week/month and navigate weeks stay on the same event
- **Traces:** US-PROG-07  ·  **Priority:** Won't  ·  **Type:** Functional
_Withdrawn: week and month views with week navigation are not in the web app; the id stays valid — not run._

---

## US-PROG-08 — Browse and filter the speaker directory

### TC-PROG-13 — Directory shows key details with card/list layouts and event filter
- **Traces:** US-PROG-08  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Organizer signed in. "Bangkok Tech Summit" has 3 speakers and "HR Forum" has 2.
- **Test data:** Speaker "Napat Chaiyo" on Bangkok Tech Summit: title & company "Head of Data · Siam Analytics", napat@eventa-test.co.th, 02 555 0123, in 2 sessions, unrated.
- **Steps:**
  1. Open "Speakers" and pick "Bangkok Tech Summit".
  2. Read Napat's card.
  3. Choose "List view", then "Grid view".
  4. Pick "HR Forum".
- **Expected result:** (1) "3 speakers", in A–Z order. (2) The card shows "NC", Napat Chaiyo, "Head of Data · Siam Analytics", the email, the phone, "2 sessions" and "—". (3) The speakers switch to rows, then back to cards. (4) "2 speakers", and only HR Forum's speakers.

### TC-PROG-14 — Search by name/role/email; no-match shows prompt not error
- **Traces:** US-PROG-08  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** "Bangkok Tech Summit" has 25 speakers (three pages of 10), including Napat Chaiyo ("Head of Data · Siam Analytics", napat@eventa-test.co.th) on page 2.
- **Test data:** Searches "napat", "siam analytics", "napat@eventa-test", "zz-nobody@example.com".
- **Steps:**
  1. On "Speakers", go to page 2.
  2. Search "napat"; then "siam analytics"; then "napat@eventa-test".
  3. Search "zz-nobody@example.com".
  4. Choose "Clear filters".
- **Expected result:** (2) Each search shows only Napat, "1 speaker", on the first page. (3) "No speakers match" with "Clear filters"; no error. (4) The count reads "25 speakers" again, still on Bangkok Tech Summit.

---

## US-PROG-09 — Add a speaker to the line-up

### TC-PROG-15 — Add a valid speaker starting at zero sessions and unrated
- **Traces:** US-PROG-09  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Organizer signed in, "Bangkok Tech Summit" picked on "Speakers".
- **Test data:** Full name "Kanya Wongsa"; Title & company "CTO · Nimble Works"; Email kanya@eventa-test.co.th; Phone 02 555 0107; Talk title "Serverless in Practice"; Bio "Builds cloud platforms for Thai retailers."
- **Steps:**
  1. Choose "Add speaker".
  2. Enter the test data and choose "Save speaker".
- **Expected result:** (1) The "Add speaker" panel names "Bangkok Tech Summit" under its title. (2) "Speaker saved." shows; Kanya's card reads "KW", "CTO · Nimble Works", "0 sessions" and "—", and the count goes up by one.

### TC-PROG-16 — Reject blank name, invalid and duplicate email
- **Traces:** US-PROG-09  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** On "Bangkok Tech Summit", a speaker already uses taken@example.com. "HR Forum" has no speaker with that email.
- **Test data:** (a) Full name blank; (b) "Mali Srisuk", email "not-an-email"; (c) "Mali Srisuk", taken@example.com; (d) on HR Forum, "Mali Srisuk", taken@example.com.
- **Steps:**
  1. On Bangkok Tech Summit's "Add speaker", try each of (a)–(c) on its own and choose "Save speaker".
  2. Pick "HR Forum", choose "Add speaker", enter (d) and choose "Save speaker".
- **Expected result:** (a) and (b) The form asks to fix the field and sends nothing. (c) "Another speaker on this event already uses that email address." and nothing is saved. (d) "Speaker saved." — the same email is fine on another event.

---

## US-PROG-10 — Keep a speaker's profile up to date

### TC-PROG-17 — Edit a speaker's role and see it reflected on the live public page
- **Traces:** US-PROG-10  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** "Bangkok Tech Summit" is published. Speaker "Anon Ratana", title & company "Engineer · OldCo Labs", no talk title, shows in the public page's Speakers section.
- **Test data:** New Title & company "VP Engineering · NewCo Labs".
- **Steps:**
  1. On "Speakers", choose "Edit speaker" on Anon's card.
  2. Enter the new Title & company and choose "Save speaker".
  3. Open the event's public page.
- **Expected result:** (1) "Edit speaker" opens with name, title & company, email and phone filled in. (2) "Speaker saved." shows and the card reads "VP Engineering · NewCo Labs". (3) The Speakers section shows Anon Ratana with "VP Engineering · NewCo Labs".

### TC-PROG-18 — Duplicate email and concurrent edit are both refused
- **Traces:** US-PROG-10  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** On one event, "Somchai D." and "Mali K." (dup@example.com) are speakers. Organizers Nattapong and Sirin are signed in.
- **Test data:** (a) Somchai's email changed to dup@example.com; (b) Sirin changes Somchai's phone, Nattapong his title & company.
- **Steps:**
  1. Nattapong edits Somchai with (a) and chooses "Save speaker".
  2. Both open "Edit speaker" on Somchai; Sirin saves her change first.
  3. Nattapong saves his change.
- **Expected result:** (1) "Another speaker on this event already uses that email address." and nothing is saved. (2) "Speaker saved." for Sirin. (3) Nattapong sees "This speaker changed elsewhere. Reload and try again."; Sirin's phone is kept.

---

## US-PROG-11 — Remove a speaker

### TC-PROG-19 — Confirmed deletion keeps sessions and clears the live public page
- **Traces:** US-PROG-11  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** "Bangkok Tech Summit" is published. Speaker "Preeya M." is in 3 sessions and is the only speaker of "Retail Trends" in Hall B; she shows on the public page.
- **Test data:** —
- **Steps:**
  1. On "Speakers", choose "Remove speaker" on Preeya's card.
  2. Read the dialog, then choose "Remove".
  3. Open "Agenda", then the event's public page.
- **Expected result:** (1) "Remove speaker?" reads "Removing Preeya M. takes them off all of their sessions. This can’t be undone." (2) "Speaker removed." shows and her card is gone. (3) All 3 sessions remain; the "Retail Trends" block shows "Hall B" in place of her name; the public page shows her in neither its Speakers section nor its agenda.

### TC-PROG-20 — Cancelling deletion leaves speaker and assignments intact
- **Traces:** US-PROG-11  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Speaker "Preeya M." is in 3 sessions.
- **Test data:** —
- **Steps:**
  1. On "Speakers", choose "Remove speaker" on Preeya's card.
  2. Choose "Cancel".
  3. Check her card and her sessions on "Agenda".
- **Expected result:** Her card still reads "3 sessions" and all 3 blocks still show her name.

---

## US-PROG-12 — Assign speakers to their sessions and balance the line-up

### TC-PROG-21 — Assigning a second session updates count
- **Traces:** US-PROG-12  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Speaker "Ratcha B." shows "1 session". "Growth Hacks" (Day 2 10:00–11:00, Hall C, no speakers) does not overlap her other session.
- **Test data:** —
- **Steps:**
  1. On "Agenda", click "Growth Hacks", tick Ratcha B. under "Speakers" and choose "Save session".
  2. Open "Speakers" and read Ratcha's card.
  3. Untick her on "Growth Hacks", save, and read her card again.
- **Expected result:** (1) "Session saved." and the block shows "Ratcha B." in place of "Hall C". (2) "2 sessions". (3) "1 session", and the block shows "Hall C" again.

### TC-PROG-22 — Only this event's speakers offered; overlapping assignment warns
- **Traces:** US-PROG-12  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** "Kittisak W." is a speaker on "HR Forum" only. On "Bangkok Tech Summit", "Malee T." speaks in "Morning Panel", Day 1 10:00–11:00.
- **Test data:** "Investor Q&A", Day 1 10:30–11:30, Room D, speaker Malee T.
- **Steps:**
  1. On Bangkok Tech Summit's "Agenda", choose "New session" and read the "Speakers" list.
  2. Enter the test data and choose "Save session".
  3. Tick "Book them anyway" and save again.
- **Expected result:** (1) Only Bangkok Tech Summit's speakers are listed; Kittisak W. is not. (2) "Malee T. is already speaking in "Morning Panel" at this time. Confirm to assign them anyway." and nothing is added. (3) "Session saved." and the block appears.

---

## US-PROG-13 — See how speakers were rated

### TC-PROG-23 — Rated speaker shows aggregate score with reviews and reflects new feedback
- **Traces:** US-PROG-13  ·  **Priority:** Won't  ·  **Type:** Functional
_Withdrawn: speaker ratings from attendee feedback, and their reviews, are not in the web app; the id stays valid — not run._

### TC-PROG-24 — Brand-new speaker shows an unrated indicator, not zero
- **Traces:** US-PROG-13  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Admin signed in. Speaker "Fern S." was just added and has no feedback.
- **Test data:** —
- **Steps:**
  1. Open "Speakers" and find Fern's card.
  2. Read the figure beside the star.
- **Expected result:** It reads "—", never 0 or 0.0.


---

<a id="tc-e11"></a>

# Organizer Home & Dashboard — Test Cases

Area: DASH · Epic E11 — Organizer Home & Dashboard
Locale defaults: Currency ฿ (THB), VAT 7% tracked separately, timezone Asia/Bangkok, languages EN/TH.

---

## US-DASH-01 — Daily operations home

### TC-DASH-01 — Greeting by name, on Bangkok time, in the workspace's language
- **Traces:** US-DASH-01 · **Priority:** High · **Type:** Functional
- **Preconditions:** Organizer "Somchai" in a workspace whose language is Thai. The computer's time zone is Europe/Berlin.
- **Test data:** Bangkok times 08:15 (03:15 in Berlin), 13:00 and 19:00.
- **Steps:**
  1. At 08:15 Bangkok, sign in as Somchai and read the greeting on home.
  2. Repeat at 13:00 and at 19:00 Bangkok.
- **Expected result:** The greeting reads "สวัสดีตอนเช้า คุณSomchai", then "สวัสดีตอนบ่าย คุณSomchai", then "สวัสดีตอนเย็น คุณSomchai": Bangkok's time of day, whatever the computer's zone. In an English workspace the first reads "Good morning, Somchai".

### TC-DASH-02 — "New event" opens event creation; a new workspace gets "Set up your workspace"
- **Traces:** US-DASH-01 · **Priority:** High · **Type:** Functional
- **Preconditions:** Case A: Organizer in a workspace with live events. Case B: Admin of a brand-new workspace whose organization details are not filled in yet.
- **Test data:** —
- **Steps:**
  1. Case A: on home, choose "New event".
  2. Leave without saving and return to home.
  3. Case B: open home.
- **Expected result:** (1) The event creation flow opens. (2) No event was created. (3) Instead of the cards, home shows "Set up your workspace" with five steps: "Set up your organization", "Connect payments", "Create your first event", "Add ticket types" and "Publish your event page"; "Set up your organization" is highlighted as the next step.

### TC-DASH-03 — An attendee is sent to the organizer sign-in
- **Traces:** US-DASH-01 · **Priority:** High · **Type:** Negative
- **Preconditions:** Signed in to the attendee portal as Mali Sri (mali.s@example.com), who has no organizer account.
- **Test data:** The organizer home's address, typed into the browser.
- **Steps:**
  1. Open the organizer home's address.
- **Expected result:** The organizer sign-in page opens. No greeting, card or figure from home is shown.

---

## US-DASH-02 — Today's registrations at a glance

### TC-DASH-04 — Today's count, the 3 newest of today, and "See all"
- **Traces:** US-DASH-02 · **Priority:** High · **Type:** Functional
- **Preconditions:** Signed in as an Organizer.
- **Test data:** Confirmed registrations today at 09:05, 09:47, 10:02 and 10:24 Bangkok; the 10:24 one is Anong Pattana's for Tech Summit 2026, ticket "VIP". One more at 23:50 Bangkok yesterday.
- **Steps:**
  1. Open home and read "Today's Registrations".
  2. Choose "See all".
- **Expected result:** The badge reads 4. Three rows, newest first: 10:24, 10:02, 09:47; the top one reads "Anong Pattana", "Tech Summit 2026 · VIP", "10:24". Neither the 09:05 nor yesterday's registration is listed. "See all" opens the Registrations page.

### TC-DASH-05 — "No registrations yet today."; earlier days never fill the list
- **Traces:** US-DASH-02 · **Priority:** Medium · **Type:** Edge
- **Preconditions:** Signed in as an Organizer. Bangkok Jazz Night is the soonest upcoming event.
- **Test data:** Case A: the last registration was yesterday at 22:00 Bangkok; none today. Case B: one registration today at 08:10 Bangkok and three yesterday.
- **Steps:**
  1. Case A: open home and read the card; choose "Copy registration link".
  2. Case B: open home and read the card.
- **Expected result:** Case A — the card says "No registrations yet today." with no rows; "Copy registration link" copies Bangkok Jazz Night's public page link and the button reads "Link copied". Case B — the badge reads 1 and only the 08:10 registration is listed; none of yesterday's fills the empty places.

### TC-DASH-05b — "Add registration" for members who may add one; not for Staff
- **Traces:** US-DASH-02 · **Priority:** High · **Type:** Functional
- **Preconditions:** Tech Summit 2026 is open for registration. An Organizer and a Staff member, both on their default roles.
- **Test data:** —
- **Steps:**
  1. As the Organizer, open home and choose "Add registration".
  2. Close the form without saving and return to home.
  3. As the Staff member, open home and look for "Add registration".
- **Expected result:** (1) The Registrations page opens with its add-registration form (US-REG-03) open. (2) No registration was created; today's count is unchanged. (3) Staff sees "Today's Registrations" and "See all", but no "Add registration".

### TC-DASH-05c — Only confirmed registrations are counted and listed
- **Traces:** US-DASH-02 · **Priority:** High · **Type:** Edge
- **Preconditions:** Signed in as an Organizer. Tech Summit 2026's "VIP" ticket needs approval.
- **Test data:** Today (Bangkok): confirmed registrations at 09:10 and 09:40; a VIP registration awaiting approval at 10:05; a waitlisted one at 10:20; one rejected at 10:30.
- **Steps:**
  1. Open home and read "Today's Registrations".
  2. Approve the 10:05 VIP registration on the Registrations page, then reload home.
- **Expected result:** (1) The badge reads 2 and only 09:40 and 09:10 are listed; the awaiting-approval, waitlisted and rejected registrations appear nowhere on the card. (2) The badge reads 3 and the 10:05 registration heads the list.

---

## US-DASH-03 — Today's meetings  ·  **Won't**

_Withdrawn with US-DASH-03 on 2026-11-04 (Home redesign); kept so its id stays valid — not run._

### TC-DASH-06 — Today's meetings previewed earliest-first with empty-state and schedule shortcut verified
- **Traces:** US-DASH-03 · **Priority:** Won't · **Type:** Functional
- **Preconditions:** Signed in as organizer.
- **Test data:** Two meetings scheduled today (Bangkok): "Sponsor sync" 15:00–15:30 with counterpart "Nok"; "Vendor call" 10:00–10:45 with counterpart "Arun". No meetings scheduled after these for today.
- **Steps:**
  1. Open home and read the meetings count badge and preview order.
  2. Verify each preview shows title, time range, and counterpart.
  3. Choose "Schedule meeting".
- **Expected result:** Count badge shows 2; meetings are previewed earliest-first ("Vendor call" 10:00–10:45 before "Sponsor sync" 15:00–15:30), each with title, time range (Bangkok), and counterpart. "Schedule meeting" opens the meeting creation flow. (If no meetings existed today, the panel would instead show "no meetings scheduled today".)

---

## US-DASH-04 — Upcoming events progress

### TC-DASH-07 — The 3 soonest events with days left, attendees and progress; card and "See all" links
- **Traces:** US-DASH-04 · **Priority:** Medium · **Type:** Functional
- **Preconditions:** Signed in as an Organizer. Today is 2026-07-19 in Bangkok.
- **Test data:** Four published events: Tech Summit 2026 starts 2026-07-22 (100 seats, 83 sold, newest attendee Anong Pattana); Bangkok Jazz Night 2026-07-31; Thai Street Food Festival 2026-08-03; UX Bangkok Meetup 2026-08-20.
- **Steps:**
  1. Open home and read "Upcoming Events".
  2. Read Tech Summit 2026's card.
  3. Choose that card; go back and choose "See all".
- **Expected result:** Three cards, soonest first: Tech Summit 2026, Bangkok Jazz Night, Thai Street Food Festival; UX Bangkok Meetup is not shown. Tech Summit 2026 shows "3 days left", the initials "AP" and "+83", and "Progress" at 83% with the bar 83% full. The card opens Tech Summit 2026's event page; "See all" opens the Events page.

### TC-DASH-08 — "Today" chip; "No upcoming events."
- **Traces:** US-DASH-04 · **Priority:** Medium · **Type:** Edge
- **Preconditions:** Signed in as an Organizer. Bangkok time is 10:00.
- **Test data:** Case A: Sunrise Yoga Retreat starts today at 18:00 Bangkok. Case B: no upcoming events; past orders exist.
- **Steps:**
  1. Case A: open home and read Sunrise Yoga Retreat's chip.
  2. Case B: open home and read "Upcoming Events".
- **Expected result:** Case A — the chip reads "Today" ("วันนี้" in a Thai workspace). Case B — the card says "No upcoming events." and offers "New event".

---

## US-DASH-05 — Seats sold across active events

### TC-DASH-09 — Each active event's share of the combined seats, "Seats left", legend and count
- **Traces:** US-DASH-05 · **Priority:** Low · **Type:** Functional
- **Preconditions:** Signed in as an Organizer.
- **Test data:** Three active events: Tech Summit 2026 (100 seats, 50 sold), Bangkok Jazz Night (200 seats, 100 sold), Thai Street Food Festival (200 seats, 150 sold). Also a draft event of 300 seats, a completed event and a cancelled event.
- **Steps:**
  1. Open home and look at the "Active events" ring.
  2. Read the legend and the centre; read the ring with a screen reader.
  3. Choose "See all".
- **Expected result:** Three coloured segments, largest first: Thai Street Food Festival 30%, Bangkok Jazz Night 20%, Tech Summit 2026 10%, then a grey "Seats left" track of 40%; the draft, completed and cancelled events add nothing. The legend lists the three events in that order, then "Seats left". The centre shows "3". A screen reader reads "Seats across the 3 active events: 60% sold, 40% left". "See all" opens the Events page.

### TC-DASH-09b — The ring agrees with Capacity filled; unlimited tickets add nothing; "No active events."
- **Traces:** US-DASH-05 · **Priority:** Low · **Type:** Edge
- **Preconditions:** Signed in as Admin.
- **Test data:** Case A: the events of TC-DASH-09 plus Sunrise Yoga Retreat, active, whose only ticket type has no seat limit (40 sold). Case B: no active events (only drafts and completed events); past orders exist.
- **Steps:**
  1. Case A: open home and read the ring, legend and centre.
  2. Case A: open the Dashboard and read Capacity filled.
  3. Case B: open home and read "Active events".
- **Expected result:** Case A — Sunrise Yoga Retreat has no segment and is not in the legend; the segments stay 30%, 20% and 10% with 40% left, and the centre shows "4". Capacity filled reads 60%, the sold segments together. Case B — the card says "No active events." and offers "View events".

### TC-DASH-09c — More than 6 events fold into "Other events"; no seat limits
- **Traces:** US-DASH-05 · **Priority:** Low · **Type:** Edge
- **Preconditions:** Signed in as an Organizer.
- **Test data:** Case A: eight active events of 100 seats each, which sold 80, 70, 60, 50, 40, 30, 20 and 10 (800 seats in all). Case B: two active events whose tickets have no seat limit (25 and 40 sold).
- **Steps:**
  1. Case A: open home and read the ring, legend and centre.
  2. Case B: open home and read "Active events".
- **Expected result:** Case A — six coloured segments, largest first: the five largest events at 10%, 8.75%, 7.5%, 6.25% and 5%, then "Other events" at 7.5% (the other three's 60 seats); "Seats left" is 55%. The legend lists those five, "Other events", then "Seats left"; the centre shows "8". Case B — the centre shows "2" and "No seat limits set" stands in place of segments; no 0% appears.

---

## US-DASH-06 — To do: the most urgent operational alerts

### TC-DASH-10 — The 3 most urgent alerts, each linking out; a resolved alert clears and the next moves up
- **Traces:** US-DASH-06 · **Priority:** High · **Type:** Functional
- **Preconditions:** Signed in as Admin. Today is 2026-07-19 in Bangkok.
- **Test data:** Two declined payments for Tech Summit 2026 (2026-07-17 and 2026-07-18); three Bangkok Jazz Night registrations awaiting approval (since 2026-07-15, 07-16 and 07-18); a Thai Street Food Festival ticket type on sale, 80% sold since 2026-07-16; one Tech Summit 2026 registration awaiting approval since 2026-07-18.
- **Steps:**
  1. Open home and read "To do": each alert's text, badge, date and event, and their order.
  2. Choose the Bangkok Jazz Night alert.
  3. Approve or reject its three registrations, return to home and wait 30 seconds.
- **Expected result:** (1) Exactly three alerts, in this order: "2 payments declined" — High, Jul 17, 2026 · Tech Summit 2026; "3 registrations to approve" — Medium, Jul 15, 2026 · Bangkok Jazz Night; "1 ticket type nearly sold out" — Medium, Jul 16, 2026 · Thai Street Food Festival. The Tech Summit 2026 approval is not shown. (2) The Registrations page opens on its Pending tab for Bangkok Jazz Night; home offered no way to dismiss or edit the alert. (3) The Jazz Night alert is gone and "1 registration to approve" (Medium, Jul 18, 2026 · Tech Summit 2026) is now third.

### TC-DASH-11 — Alerts a member could not resolve are never shown; "You're all caught up"
- **Traces:** US-DASH-06 · **Priority:** High · **Type:** Negative
- **Preconditions:** Case A and C: an Organizer whose finance access was removed (US-SET-12). Case B and C: Admin. Case D: Staff on the default role.
- **Test data:** Case A and D: the alerts of TC-DASH-10. Case B: nothing outstanding. Case C: only one declined payment, for Bangkok Jazz Night.
- **Steps:**
  1. Case A: open home and read "To do"; choose "View all" and read "Needs action".
  2. Case B: as Admin, open home and read "To do".
  3. Case C: as the Organizer, then as Admin, open home and read "To do".
  4. Case D: open home and read "To do".
- **Expected result:** Case A — no declined-payment alert on the card or under "Needs action", and nothing stands in for it; the card lists the Jazz Night approvals (Jul 15, 2026), the Street Food ticket type (Jul 16, 2026) and the Tech Summit approval (Jul 18, 2026). Case B — "You're all caught up". Case C — the Organizer sees "You're all caught up"; Admin sees "1 payment declined" (High). Case D — "You're all caught up": Staff can resolve none of the four.

### TC-DASH-11b — A High alert outranks an older Medium one; priority can't be changed
- **Traces:** US-DASH-06 · **Priority:** High · **Type:** Functional
- **Preconditions:** Signed in as Admin. Today is 2026-07-19 in Bangkok.
- **Test data:** One Bangkok Jazz Night registration awaiting approval since 2026-07-10; a Thai Street Food Festival ticket type nearly sold out since 2026-07-12; one declined payment for Tech Summit 2026 on 2026-07-18.
- **Steps:**
  1. Open home and read the badges and order on "To do".
  2. Look on the card for a way to change an alert's priority or dismiss it.
- **Expected result:** (1) "1 payment declined" is first with **High**, though raised last; then the approval (**Medium**, Jul 10, 2026), then the ticket type (**Medium**, Jul 12, 2026). (2) There is none: each row only links to where the alert is resolved.

### TC-DASH-11c — "View all" opens Notifications with every alert under "Needs action"
- **Traces:** US-DASH-06 · **Priority:** High · **Type:** Functional
- **Preconditions:** Signed in as Admin, with the four alerts of TC-DASH-10.
- **Test data:** —
- **Steps:**
  1. On home, choose "View all" on "To do".
  2. Read "Needs action".
- **Expected result:** The Notifications page opens. "Needs action" lists all four alerts in To do's order, including the Tech Summit 2026 approval that home leaves out.

### TC-DASH-11d — Dates raised are Bangkok dates in the workspace's language; nearly sold out starts at 80%
- **Traces:** US-DASH-06 · **Priority:** High · **Type:** Edge
- **Preconditions:** Signed in as Admin. Bangkok time is 2026-07-19 10:00.
- **Test data:** Two declined payments for Tech Summit 2026, at 2026-07-17 23:30 UTC (2026-07-18 06:30 Bangkok) and 2026-07-19 09:00 Bangkok. On sale: Thai Street Food Festival "Early Bird", 100 seats, 80th sold 2026-07-15 18:30 UTC (2026-07-16 01:30 Bangkok); Bangkok Jazz Night "Standard", 100 seats, 79 sold; Tech Summit 2026 "Day Pass", no seat limit, 500 sold.
- **Steps:**
  1. Open home and read the declined-payments alert.
  2. Read "To do" for nearly-sold-out alerts.
  3. With the workspace's language set to Thai, open home again.
  4. Open the Dashboard and read "Tickets Selling Fast".
- **Expected result:** (1) "2 payments declined", Tech Summit 2026, dated Jul 18, 2026: the older decline's Bangkok date, not Jul 17 (UTC) nor Jul 19. (2) One "1 ticket type nearly sold out" (Medium) for Thai Street Food Festival, dated Jul 16, 2026; Standard (79%) and Day Pass (no seat limit) raise none. (3) "การชำระเงิน 2 รายการถูกปฏิเสธ" dated "18 ก.ค. 2569" and "บัตร 1 ประเภทใกล้จะขายหมด" dated "16 ก.ค. 2569". (4) Only Early Bird is listed.

---

## US-DASH-07 — Website template shortcuts  ·  **Won't**

_Withdrawn with US-DASH-07 on 2026-11-04 (Home redesign); kept so its id stays valid — not run._

### TC-DASH-12 — Template shortcut opens landing-pages area; shortcuts reflect current template set on reload
- **Traces:** US-DASH-07 · **Priority:** Won't · **Type:** Functional
- **Preconditions:** Signed in as organizer; templates panel is shown with the current set of landing-page templates.
- **Test data:** Available templates set changes (one added/removed) between the two loads.
- **Steps:**
  1. Choose a template shortcut from the panel.
  2. Change the available template set, then reload home.
  3. Re-inspect the templates panel.
- **Expected result:** Choosing a template navigates to the landing-pages area to work with it. After reload, the shortcuts reflect the current set (added template appears, removed template disappears).

---

## US-DASH-08 — Performance KPIs at a glance

### TC-DASH-13 — Five KPI cards with their change
- **Traces:** US-DASH-08 · **Priority:** High · **Type:** Functional
- **Preconditions:** Signed in as Admin. The Dashboard on Year, its default.
- **Test data:** Last 365 days against the 365 before: 1,340 registrations against 1,000; net ticket revenue ฿128,500; check-in rate 82% against 90%. 4 upcoming events; capacity filled 64%.
- **Steps:**
  1. Open the Dashboard.
  2. Read each card's figure and change.
- **Expected result:** Total registrations "1,340" with a green up arrow and "34%". Ticket revenue "฿128.50k" with its change. Upcoming events "4" and Capacity filled "64%", each with "—" for change. Check-in rate "82%" with a red down arrow and "-8.9%".

### TC-DASH-14 — No Ticket revenue card without finance access
- **Traces:** US-DASH-08 · **Priority:** High · **Type:** Negative
- **Preconditions:** Signed in as Staff on the default role (no finance access). Revenue exists in the period.
- **Test data:** —
- **Steps:**
  1. Open the Dashboard and read the cards.
- **Expected result:** Four cards: Total registrations, Upcoming events, Check-in rate and Capacity filled. There is no Ticket revenue card: not ฿0, not a blank tile.

### TC-DASH-15 — "—" for no data or no baseline; "No activity yet" in a new workspace
- **Traces:** US-DASH-08 · **Priority:** Medium · **Type:** Edge
- **Preconditions:** Case A: Admin; no event started in the last 14 days; 5 registrations in the last 7 days and none in the 7 before. Case B: Admin of a brand-new workspace.
- **Test data:** —
- **Steps:**
  1. Case A: open the Dashboard, choose Week, and read Check-in rate and Total registrations.
  2. Case B: open the Dashboard.
- **Expected result:** Case A — Check-in rate reads "—" with no arrow, never 0% or 100%; Total registrations reads "5" with "—" for change, not a percentage. Case B — instead of the cards, "No activity yet" with "Create your first event" and "Manage events".

---

## US-DASH-09 — Revenue trend with time-range toggle

### TC-DASH-16 — Revenue Overview opens on Year; Week and Month switch the page's range
- **Traces:** US-DASH-09 · **Priority:** High · **Type:** Functional
- **Preconditions:** Signed in as Admin. Revenue across the last year.
- **Test data:** Net revenue: last 365 days ฿1,240,000; last 30 days ฿96,000; last 7 days ฿21,500.
- **Steps:**
  1. Open the Dashboard and read "Revenue Overview" and the Ticket revenue card.
  2. Choose Month and read both again.
  3. Choose Week and read both again.
- **Expected result:** Year is selected; the total reads "฿1240k" with its change "vs previous period", the same as the Ticket revenue card. Month shows "฿96k" and Week "฿21.50k", in both places each time; the chart and the other cards change with the range, and the page does not fully reload.

### TC-DASH-17 — No Revenue Overview without finance access; a range with no revenue
- **Traces:** US-DASH-09 · **Priority:** High · **Type:** Negative
- **Preconditions:** Case A: Staff on the default role (no finance access). Case B: Admin; no sales in the last 14 days.
- **Test data:** —
- **Steps:**
  1. Case A: open the Dashboard and look for "Revenue Overview".
  2. Case B: open the Dashboard and choose Week.
- **Expected result:** Case A — no Revenue Overview at all (not zeroed, not locked); "Registrations by Ticket Type" takes the full row. Case B — the total reads "฿0" with "—" for change, "No revenue in this period." replaces the chart, and the Week / Month / Year toggle stays.

---

## US-DASH-10 — Registrations by ticket type

### TC-DASH-18 — Split by ticket type, largest first, equal to the total; recounted by range
- **Traces:** US-DASH-10 · **Priority:** Medium · **Type:** Functional
- **Preconditions:** Signed in as an Organizer. The Dashboard on Year.
- **Test data:** Last 365 days: 200 registrations — General 120, VIP 50, Early Bird 30. Last 7 days: General 12, VIP 3.
- **Steps:**
  1. Open the Dashboard and read "Registrations by Ticket Type".
  2. Compare its centre with Total registrations.
  3. Choose Week and read both again.
- **Expected result:** The centre reads "200", as does Total registrations. Rows, largest first: General "120 registrations" 60%, VIP "50 registrations" 25%, Early Bird "30 registrations" 15%. On Week the centre and Total registrations read "15": General 80%, VIP 20%.

### TC-DASH-19 — "No registrations yet."
- **Traces:** US-DASH-10 · **Priority:** Medium · **Type:** Edge
- **Preconditions:** Signed in as an Organizer. A published upcoming event with no registrations.
- **Test data:** —
- **Steps:**
  1. Open the Dashboard and read "Registrations by Ticket Type".
- **Expected result:** In place of the donut, the panel says "No registrations yet." and "Once tickets sell, the split by type shows here."

---

## US-DASH-11 — Tickets selling fast

### TC-DASH-20 — Up to 3 scarcest ticket types, coloured by count left; "Manage"
- **Traces:** US-DASH-11 · **Priority:** Medium · **Type:** Functional
- **Preconditions:** Signed in as an Organizer.
- **Test data:** On sale, 100 seats each: VIP (Tech Summit 2026) 98 sold; General (Bangkok Jazz Night) 94 sold; Workshop (UX Bangkok Meetup) 85 sold; Student (Tech Summit 2026) 82 sold. Paused: Staff Pass (Tech Summit 2026) 99 sold.
- **Steps:**
  1. Open the Dashboard and read "Tickets Selling Fast".
  2. Choose "Manage".
- **Expected result:** Three rows, scarcest first: VIP "2 left" in red, General "6 left" in amber, Workshop "15 left" in plain text, each with its event. Student and the paused Staff Pass are not listed. "Manage" opens the Tickets page.

### TC-DASH-21 — Colour boundaries at 3 and 8 left; "No tickets running low."
- **Traces:** US-DASH-11 · **Priority:** Medium · **Type:** Edge
- **Preconditions:** Signed in as an Organizer.
- **Test data:** Case A: three ticket types on sale, 100 seats each, with 97, 92 and 91 sold. Case B: every ticket type on sale is under 80% sold (the fullest 79 of 100).
- **Steps:**
  1. Case A: open the Dashboard and read "Tickets Selling Fast".
  2. Case B: open the Dashboard and read the panel.
- **Expected result:** Case A — "3 left" in red, "8 left" in amber, "9 left" in plain text. Case B — "No tickets running low." with "Manage tickets".

---

## US-DASH-12 — Recent registrations table

### TC-DASH-22 — Newest registrations with attendee, event, amount, status and time; "View all"
- **Traces:** US-DASH-12 · **Priority:** High · **Type:** Functional
- **Preconditions:** Signed in as Admin.
- **Test data:** Today (Bangkok): Anong Pattana, Tech Summit 2026, ฿1,500, paid, 14:20; Mali Sri, UX Bangkok Meetup, free ticket, 13:05; Nutcha Wongsa, Bangkok Jazz Night, ฿800, pending, 11:50; James Wong, Bangkok Jazz Night, ฿480, refunded, 10:30; Kittisak Boon, Tech Summit 2026, ฿1,250, failed, 09:15.
- **Steps:**
  1. Open the Dashboard and read "Recent Registrations".
  2. Choose "View all".
- **Expected result:** Rows newest first, in the order above, with amounts "฿1,500", "Free", "฿800", "฿480", "฿1,250", badges Paid, Paid, Pending, Refunded, Failed, and times 14:20, 13:05, 11:50, 10:30, 09:15. "View all" opens the Registrations page.

### TC-DASH-23 — Amounts read "—" without finance access; no rows without registration access
- **Traces:** US-DASH-12 · **Priority:** High · **Type:** Negative
- **Preconditions:** The registrations of TC-DASH-22. Member A: Staff on the default role (views registrations, no finance access). Member B: a role fine-tuned (US-SET-12) to view payments but not registrations.
- **Test data:** —
- **Steps:**
  1. As A, open the Dashboard and read "Recent Registrations".
  2. As B, open the Dashboard and read "Recent Registrations".
- **Expected result:** A — every amount reads "—", never ฿0; attendee, event, status and time show. B — no one is listed: the table says "No registrations yet."

---

## US-DASH-13 — Trustworthy, always-current overview

### TC-DASH-24 — A new registration reaches home within 30 seconds and the Dashboard on reload
- **Traces:** US-DASH-13 · **Priority:** High · **Type:** Functional
- **Preconditions:** Signed in as Admin, with home open in one tab and the Dashboard in another.
- **Test data:** A new paid "VIP" registration for Tech Summit 2026, ฿1,500, by Pimchanok Saelim (pimchanok.s@example.com).
- **Steps:**
  1. Note home's Today's Registrations count, the top of Recent orders and the insight's registrations today; note the Dashboard's Total registrations, VIP count and top of Recent Registrations.
  2. Complete the registration on the event's public page.
  3. Wait 30 seconds on home without reloading; read the same figures.
  4. Read the Dashboard, then reload it and read again.
- **Expected result:** Home's count rises by 1 with Pimchanok Saelim on top; her ฿1,500 Paid order heads Recent orders; the insight's registrations today rise by 1 and equal the count. The Dashboard is unchanged until reloaded; then Total registrations and VIP rise by 1 and her row heads Recent Registrations.

### TC-DASH-25 — One failing card is isolated; refresh pace; hidden figures stay hidden
- **Traces:** US-DASH-13 · **Priority:** High · **Type:** Edge
- **Preconditions:** Signed in as Staff on the default role (views registrations, no finance access). In the test environment, Upcoming Events' data is made to fail.
- **Test data:** —
- **Steps:**
  1. Open home and look at every card.
  2. Restore Upcoming Events' data and choose "Try again".
  3. Leave home open for 5 minutes, noting when each card refreshes.
  4. Look for any revenue or amount on the page.
- **Expected result:** (1) Only "Upcoming Events" says "This couldn't load just now." with "Try again"; every other card works. (2) The card returns. (3) Today's Registrations, Recent orders, To do and the insight refresh about every 30 seconds; Active events and Upcoming Events about every 5 minutes. (4) There is no Ticket revenue card and no Amount column: nothing reads ฿0. Dates and times are Bangkok's.

---

## US-DASH-14 — Ticket revenue on Home

### TC-DASH-26 — Last 30 days' net revenue, two daily lines and the change pill
- **Traces:** US-DASH-14 · **Priority:** Medium · **Type:** Functional
- **Preconditions:** Signed in as Admin. Today is 2026-07-19 in Bangkok. Paid orders in both 30-day windows, including one refund in the last 30 days.
- **Test data:** Net revenue (after VAT and refunds): 2026-06-20 to 2026-07-19, ฿48,290; 2026-05-21 to 2026-06-19, ฿42,925 (+12.5%).
- **Steps:**
  1. Open home and read "Ticket revenue": figure, caption, lines and pill.
  2. Open the Dashboard, choose Month and read the Ticket revenue card.
- **Expected result:** "฿48,290", "Last 30 days · all events", two lines of 30 daily points (these 30 days and the 30 before), and a green "+12.5%" pill. The refund is netted out and no VAT figure appears. The Dashboard's Month card reads "฿48.29k", the same amount.

### TC-DASH-27 — No Ticket revenue card without finance access
- **Traces:** US-DASH-14 · **Priority:** High · **Type:** Negative
- **Preconditions:** Signed in as Staff on the default role (no finance access). Ticket revenue exists in the last 30 days.
- **Test data:** —
- **Steps:**
  1. Open home and read the first row of cards.
  2. Search the page, including with a screen reader, for any revenue figure.
- **Expected result:** No Ticket revenue card: not ฿0, not a blank tile, not a lock. "Today's Registrations" and "Active events" lay out with no gap, and no revenue figure appears anywhere.

### TC-DASH-28 — Falling, no-baseline and zero revenue
- **Traces:** US-DASH-14 · **Priority:** Medium · **Type:** Edge
- **Preconditions:** Signed in as Admin.
- **Test data:** Case A: last 30 days ฿30,000, the 30 before ฿40,000. Case B: ฿12,000 and ฿0. Case C: ฿0 and ฿0.
- **Steps:**
  1. For each case, open home and read "Ticket revenue".
- **Expected result:** Case A — "฿30,000" with a red "−25%" pill. Case B — "฿12,000" with "—"; no percentage is invented. Case C — the card still shows, reading "฿0" with "—".

---

## US-DASH-15 — Check-in insight on Home

### TC-DASH-29 — Opens on check-in rate; the dots switch metric without a reload
- **Traces:** US-DASH-15 · **Priority:** Medium · **Type:** Functional
- **Preconditions:** Signed in as an Organizer. Today is 2026-07-19 in Bangkok. An event started on each of the last 12 days.
- **Test data:** Daily check-in rates, 2026-07-08 to 2026-07-19: 82, 85, 79, 88, 84, 90, 86, 83, 89, 85, 88, 87 (%).
- **Steps:**
  1. Open home and read the insight's headline, label and candles.
  2. Point at the first and the last candle; repeat with a screen reader.
  3. Choose the second dot, then the third, noting whether the page reloads.
- **Expected result:** Headline "87%" with "Check-in rate · last 12 days" and 12 candles, Jul 8 on the left to Jul 19 on the right. The first reads "Jul 8, 2026: 82%" and the last "Jul 19, 2026: 87%". The Jul 9 candle (82 → 85) is green; Jul 10 (85 → 79) is grey. The dots switch to "Capacity filled · last 12 days" and "Registrations · last 12 days", the highlighted dot following, with no reload.

### TC-DASH-30 — Insight values agree with the Dashboard and Today's Registrations
- **Traces:** US-DASH-15 · **Priority:** Medium · **Type:** Functional
- **Preconditions:** Signed in as Admin. Today is 2026-07-19 in Bangkok.
- **Test data:** 48 registrations today. The Dashboard's Capacity filled reads 78%. One event started on 2026-07-15 with 200 tickets, 170 checked in; no other event started that day.
- **Steps:**
  1. Switch the insight to registrations; compare today's headline with the Today's Registrations badge.
  2. Switch to capacity filled; compare today's headline with the Dashboard's Capacity filled.
  3. Switch to check-in rate; read the Jul 15 candle.
- **Expected result:** Registrations today reads 48, as does the badge. Capacity filled today reads 78%, as on the Dashboard. The Jul 15 candle reads 85%: 170 of 200.

### TC-DASH-31 — Days with nothing to measure have no candle; no registrations is 0
- **Traces:** US-DASH-15 · **Priority:** Medium · **Type:** Edge
- **Preconditions:** Signed in as an Organizer. Today is 2026-07-19 in Bangkok.
- **Test data:** Case A: in the last 12 days, events started only on 2026-07-12 and 2026-07-15, none today; nobody registered on 2026-07-10. Case B: the only active event, Sunrise Yoga Retreat, sells tickets with no seat limit.
- **Steps:**
  1. Case A: view check-in rate, then registrations.
  2. Case B: view capacity filled.
- **Expected result:** Case A — check-in rate has candles only on Jul 12 and Jul 15; other days have none (pointing reads, for example, "Jul 9, 2026: no data") and the headline is "—", never 0%. Registrations shows 0 on Jul 10. Case B — capacity filled has no candles and a "—" headline, never 0%.

### TC-DASH-31b — A past day's capacity filled uses today's active events and capacity
- **Traces:** US-DASH-15 · **Priority:** Medium · **Type:** Functional
- **Preconditions:** Signed in as Admin. Today is 2026-07-19 in Bangkok.
- **Test data:** Active: Tech Summit 2026 (100 seats; 60 sold now, 40 by the end of 2026-07-14) and Bangkok Jazz Night (capacity raised from 150 to 200 on 2026-07-17; 120 sold now, 80 by the end of 2026-07-14). A Tech Summit seat sold on 2026-07-12 was refunded on 2026-07-16. Thai Street Food Festival (100 of 100 sold) completed on 2026-07-16.
- **Steps:**
  1. Switch the insight to capacity filled; read the Jul 14 candle and today's headline.
  2. Open the Dashboard and read Capacity filled.
- **Expected result:** Jul 14 reads 40%: 120 of today's 300 seats. The refunded seat counts on no day, Jazz Night's old 150 is not used and the completed festival adds nothing. Today reads 60% (180 of 300), as does the Dashboard.

---

## US-DASH-16 — Recent orders on Home

### TC-DASH-32 — The three newest orders; "View all" opens Payments
- **Traces:** US-DASH-16 · **Priority:** Medium · **Type:** Functional
- **Preconditions:** Signed in as Admin. Today is 2026-07-19 in Bangkok.
- **Test data:** Anong Pattana (anong.p@example.com), Tech Summit 2026, 2026-07-18, ฿1,250, paid; James Wong (james.w@example.com), Bangkok Jazz Night, 2026-07-17, ฿480, refunded; Nutcha Wongsa (nutcha.w@example.com), Bangkok Jazz Night, 2026-07-15, ฿480, pending; an older order on 2026-07-12 for ฿2,000, paid.
- **Steps:**
  1. Open home and read "Recent orders".
  2. Choose "View all".
- **Expected result:** Three rows, newest first: Anong Pattana "Jul 18", James Wong "Jul 17", Nutcha Wongsa "Jul 15"; the Jul 12 order is not shown. Each row shows buyer name and email, event, date, amount ("฿1,250", "฿480", "฿480") and the badge Paid, Refunded, Pending. "View all" opens the Payments page.

### TC-DASH-33 — Date and Amount sort, reverse on a second click, and bring in no other order
- **Traces:** US-DASH-16 · **Priority:** Medium · **Type:** Functional
- **Preconditions:** As TC-DASH-32.
- **Test data:** The orders of TC-DASH-32.
- **Steps:**
  1. With the card as it opens, choose Date.
  2. Choose Amount.
  3. Choose Amount again.
  4. Choose Date.
  5. Choose Date again.
- **Expected result:** (1) Oldest first: Jul 15, Jul 17, Jul 18. (2) Largest first: ฿1,250, then the two ฿480 orders. (3) The two ฿480 orders, then ฿1,250. (4) Newest first. (5) Oldest first. Only these three orders ever show; the ฿2,000 order never appears.

### TC-DASH-34 — Amounts and buyers hidden by permission; "View all" only with finance access
- **Traces:** US-DASH-16 · **Priority:** High · **Type:** Negative
- **Preconditions:** The orders of TC-DASH-32. Member A: Staff on the default role (views registrations, no finance access). Member B: a role fine-tuned (US-SET-12) to view payments but not registrations.
- **Test data:** —
- **Steps:**
  1. As A, open home, read "Recent orders", sort by Date and look for Amount and "View all".
  2. As B, open home, read the card and choose "View all".
- **Expected result:** A — Buyer, Event, Date and Status show; there is no Amount column or sort (not ฿0, not masked), Date sorts, and there is no "View all". B — Event, Date, Amount and Status show, no buyer name or email anywhere; "View all" opens the Payments page.

### TC-DASH-35 — "No orders yet."
- **Traces:** US-DASH-16 · **Priority:** Medium · **Type:** Edge
- **Preconditions:** Signed in as Admin. A published upcoming event and no orders yet.
- **Test data:** —
- **Steps:**
  1. Open home and read "Recent orders".
- **Expected result:** In place of the table, the card says "No orders yet." and "Paid, pending and refunded orders show up here, newest first."

### TC-DASH-35b — Failed attempts and free registrations take no place; a refund keeps its amount
- **Traces:** US-DASH-16 · **Priority:** Medium · **Type:** Edge
- **Preconditions:** Signed in as Admin. Today is 2026-07-19 in Bangkok.
- **Test data:** Newest first: a failed card payment by Kittisak Boon for Tech Summit 2026 (2026-07-19 11:00, ฿1,250); a free registration by Mali Sri for UX Bangkok Meetup (10:30); a waitlisted registration for Bangkok Jazz Night (10:00); then the three orders of TC-DASH-32, James Wong's fully refunded on 2026-07-18.
- **Steps:**
  1. Open home and read "Recent orders".
  2. Choose "View all" and compare the top of the Payments page.
- **Expected result:** (1) The rows are Anong Pattana, James Wong and Nutcha Wongsa; the failed attempt, the free registration and the waitlisted one take no place. James Wong's row shows ฿480 with Refunded. (2) They are the newest Paid, Pending and Refunded payments on the Payments page, which lists the failed attempt separately as Failed.


---

<a id="tc-e12"></a>

# Coordinate Meetings — Test Cases

Area code: MTG · Epic E12 — Coordinate Meetings
Locale assumptions: Asia/Bangkok timezone, EN/TH content, ฿ (VAT not applicable to this epic — no money fields).

---

## US-MTG-01 — See meetings organized by Today / Upcoming / Past

### TC-MTG-01 — Meetings fall into Today / Upcoming / Past with counts, earliest first
- **Traces:** US-MTG-01 · **Priority:** High · **Type:** Functional
- **Preconditions:** Signed in as an Event Organizer. Only the four meetings below exist (D was booked before its date passed).
- **Test data:** A "Venue walkthrough" today 14:00–15:00; B "Sponsor sync" tomorrow 11:00–11:30; C "Speaker brief" tomorrow 09:00–09:30, booked after B; D "Post-event review" 3 days ago 10:00–11:00. All Bangkok time.
- **Steps:**
  1. Open Meetings and read the tab counts.
  2. Open "Today".
  3. Open "Upcoming".
  4. Open "Past".
- **Expected result:** All 4, Today 1, Upcoming 2, Past 1. Today shows A with a green "Today" badge and "Today · 14:00 – 15:00". Upcoming lists C (09:00) before B (11:00), though B was booked first. Past shows D with a grey "Past" badge.

### TC-MTG-02 — The tab follows the Bangkok date, not the viewer's time zone
- **Traces:** US-MTG-01 · **Priority:** Medium · **Type:** Edge
- **Preconditions:** Signed in as an Event Organizer on a computer set to the America/New_York time zone. Run before 11:00 Bangkok time, while it is still yesterday in New York.
- **Test data:** "Late vendor call" today 23:30–23:59 Bangkok time.
- **Steps:**
  1. Open Meetings and open "Today".
  2. Open "Upcoming", then "Past".
- **Expected result:** "Late vendor call" is under "Today" with the "Today" badge and "Today · 23:30 – 23:59", and appears under neither Upcoming nor Past.

### TC-MTG-03 — Each row shows what, who, when and where; an empty workspace offers to schedule
- **Traces:** US-MTG-01 · **Priority:** Medium · **Type:** Functional
- **Preconditions:** Signed in as an Event Organizer. Google Calendar connected. Event "Tech Summit BKK" has venue "BITEC Hall 2". Case B uses a second workspace with no meetings.
- **Test data:** Video call "Sponsor kickoff", type Sponsor, with Somchai Jaidee, role "Partnerships Lead", event Tech Summit BKK, invite sent. In person "Venue walkthrough", type Venue, with Malee Srisuk, no role, event Tech Summit BKK. Phone call "Catering check", type Vendor, with Arun Chaiyo, role "Caterer", Event "All events".
- **Steps:**
  1. Open Meetings and read each row.
  2. Case B: open Meetings in the empty workspace and choose "Schedule a meeting".
- **Expected result:** Each row shows its title, a type icon, its badge ("Video call", "In person", "Phone call") and its time. The who lines read "Partnerships Lead · Somchai Jaidee · Tech Summit BKK", "Venue · Malee Srisuk · Tech Summit BKK" and "Caterer · Arun Chaiyo · All events". The places read: the Meet link (opens in a new tab), "BITEC Hall 2" and "Phone call". Case B: "No meetings scheduled" is shown, and "Schedule a meeting" opens the "Schedule meeting" panel.

---

## US-MTG-02 — Find a specific meeting in a long list

### TC-MTG-04 — Search matches title, person, role, type or event, and returns to page 1
- **Traces:** US-MTG-02 · **Priority:** High · **Type:** Functional
- **Preconditions:** Signed in as an Event Organizer. 25 meetings; Rows per page 10; viewing page 3 of All.
- **Test data:** "Somchai" (the guest on 2 meetings); "Tech Summit" (the event of 3 meetings, none with it in the title); "ผู้สนับสนุน" (in one title, "ประชุมผู้สนับสนุนหลัก").
- **Steps:**
  1. Type "Somchai" in "Search meetings or people…".
  2. Replace it with "Tech Summit".
  3. Replace it with "ผู้สนับสนุน".
- **Expected result:** (1) Only the 2 Somchai meetings, on page 1: "Showing 1–2 of 2 meetings". (2) The 3 Tech Summit BKK meetings. (3) Only "ประชุมผู้สนับสนุนหลัก".

### TC-MTG-05 — Type and event filters combine with the tab
- **Traces:** US-MTG-02 · **Priority:** Medium · **Type:** Functional
- **Preconditions:** Signed in as an Event Organizer. Upcoming and past meetings of every type exist; some Sponsor meetings are for "Tech Summit BKK", others for "Bangkok Design Week".
- **Test data:** Type Sponsor; event "Tech Summit BKK".
- **Steps:**
  1. Open "Upcoming".
  2. Choose Sponsor in "All types".
  3. Choose "Tech Summit BKK" in "All events".
  4. Read the tab counts.
- **Expected result:** Step 2 leaves only upcoming Sponsor meetings; step 3 only those for Tech Summit BKK. Every tab count now counts only Sponsor meetings for Tech Summit BKK.

### TC-MTG-06 — No-match message and paging summary
- **Traces:** US-MTG-02 · **Priority:** Medium · **Type:** Negative
- **Preconditions:** Signed in as an Event Organizer. 23 meetings under All.
- **Test data:** Search "zzzznomatch"; Rows per page 10, then 20.
- **Steps:**
  1. Search "zzzznomatch".
  2. Choose "Clear filters".
  3. Set Rows per page to 10 and choose Next.
  4. Set Rows per page to 20.
- **Expected result:** (1) "No meetings match" with "Clear filters", and no rows. (2) All 23 meetings are back. (3) "Showing 11–20 of 23 meetings". (4) Back on page 1: "Showing 1–20 of 23 meetings".

---

## US-MTG-03 — Schedule a meeting with a speaker, sponsor, venue or vendor

### TC-MTG-07 — Schedule a valid video meeting
- **Traces:** US-MTG-03 · **Priority:** High · **Type:** Functional
- **Preconditions:** Signed in as an Event Organizer. Google Calendar connected. Event "Tech Summit BKK" exists.
- **Test data:** Title "Keynote speaker briefing"; Date tomorrow; Starts 10:00; Ends 10:45; Type Speaker; Mode "Video call (Google Meet)"; Who you are meeting "Anong Phromma"; Their role "Keynote speaker"; Guest email anong@example.com; Event "Tech Summit BKK"; Notes "Confirm slides".
- **Steps:**
  1. Choose "Schedule meeting" and read Type and Mode.
  2. Fill in the test data and choose "Schedule meeting".
  3. Open "Upcoming".
  4. Choose the new meeting's pencil.
- **Expected result:** Type starts on Internal and Mode reads "Choose how you'll meet". "Meeting scheduled." shows. Under Upcoming the meeting reads "Keynote speaker · Anong Phromma · Tech Summit BKK" with a "Video call" badge and "10:00 – 10:45". "Reschedule meeting" opens with every field as entered and "Video call (Google Meet)" chosen.

### TC-MTG-08 — A short title, past date, backwards times or no mode are refused
- **Traces:** US-MTG-03 · **Priority:** High · **Type:** Negative
- **Preconditions:** Signed in as an Event Organizer. The "Schedule meeting" panel is open.
- **Test data:** The valid entries of TC-MTG-07, then one change at a time: (a) Title "ab"; (b) Date yesterday; (c) Starts 14:00, Ends 13:30; (d) Mode left on "Choose how you'll meet".
- **Steps:**
  1. For each of (a)–(d), make only that change and choose "Schedule meeting".
  2. Close the panel and read the list.
- **Expected result:** Each time the panel stays open and says what to fix: (a) the title needs at least 3 characters; (b) "That date has already passed — choose today or later."; (c) "The end time must be after the start time."; (d) the browser asks for a Mode. No meeting was created.

### TC-MTG-09 — The place follows the mode; a double click books once
- **Traces:** US-MTG-03 · **Priority:** Medium · **Type:** Edge
- **Preconditions:** Signed in as an Event Organizer. Event "Tech Summit BKK" has venue "BITEC Hall 2". No meeting is called "Sponsor sync".
- **Test data:** X "Venue walkthrough", In person, Event Tech Summit BKK; Y "Catering check", Phone call; Z "Sponsor sync", any valid entries.
- **Steps:**
  1. Schedule X.
  2. Schedule Y.
  3. Fill in Z and double-click "Schedule meeting".
  4. Search "Sponsor sync".
- **Expected result:** X's card shows "In person" and the place "BITEC Hall 2"; Y's shows "Phone call" as both badge and place. After the first click the button reads "Saving…"; the search finds exactly one "Sponsor sync".

---

## US-MTG-04 — Have invites, video links and reminders handled automatically

### TC-MTG-10 — Video meeting: invite, Meet link and 15-minute reminder
- **Traces:** US-MTG-04 · **Priority:** High · **Type:** Functional
- **Preconditions:** Signed in as an Event Organizer. Google Calendar connected (the Meetings header shows "Google Calendar"). The guest's mailbox can be read.
- **Test data:** "Sponsor kickoff", tomorrow 15:00–15:30, guest partner@example.com.
- **Steps:**
  1. Choose "Schedule meeting", choose "Phone call" and read the note above the button.
  2. Choose "Video call (Google Meet)" and read the hint under Mode and the note.
  3. Fill in the test data and schedule it.
  4. Watch the new card without reloading.
  5. Open the guest's invite and the event on the workspace's Google Calendar.
- **Expected result:** (1) "A Google Calendar invite is emailed to each guest, with a reminder 15 minutes before." (2) "A Meet link is generated automatically." under Mode; the note reads "A Google Meet link and Google Calendar invite are emailed to each guest, with a reminder 15 minutes before." (4) The card first reads "The calendar invite has not been sent yet.", then by itself shows its Meet link and "Join". (5) The invite is for 15:00–15:30 Bangkok time with the Meet link; the workspace calendar's event reminds 15 minutes before.

### TC-MTG-11 — No calendar connected: meeting saved without an invite, next step by role
- **Traces:** US-MTG-04 · **Priority:** High · **Type:** Negative
- **Preconditions:** Workspace Google Calendar not connected. Two members: an Event Organizer without Manage integrations, and an Admin.
- **Test data:** Any valid video meeting, once per member.
- **Steps:**
  1. As the Organizer, choose "Schedule meeting", choose "Video call (Google Meet)" and read the note above the button.
  2. Schedule the meeting and read its card once it settles.
  3. As the Admin, schedule one and read its card.
- **Expected result:** (1) "Google Calendar isn't connected, so this meeting will be saved without an invite or Meet link.", and no Meet-link hint under Mode. (2)–(3) Both meetings are saved and read "No calendar is connected, so no invite was sent.", with no "Join". The Organizer's card adds "Ask an Admin to connect Google Calendar."; the Admin's adds a "Connect Google Calendar" link to Settings → Integrations.

### TC-MTG-12 — Google briefly unavailable: meeting kept, then sent
- **Traces:** US-MTG-04 · **Priority:** Medium · **Type:** Edge
- **Preconditions:** Signed in as an Event Organizer. Google Calendar connected, but made unreachable in the test environment.
- **Test data:** Video meeting "Vendor briefing", tomorrow 13:00–13:30, guest somsak@example.com.
- **Steps:**
  1. Schedule the meeting.
  2. Wait for the card to settle and read it.
  3. Make Google reachable again and choose "Send it now".
- **Expected result:** (2) The meeting is kept; its card reads "The calendar invite couldn't be sent." with "Send it now". (3) The button reads "Sending…", then the note goes, the card shows its Meet link and the guest receives the invite. An automatic retry may send it first, with the same result.

---

## US-MTG-05 — Reschedule or edit a meeting

### TC-MTG-13 — Rescheduling moves the guest's invite
- **Traces:** US-MTG-05 · **Priority:** High · **Type:** Functional
- **Preconditions:** Signed in as an Event Organizer. Google Calendar connected. Video meeting "Sponsor kickoff" tomorrow 15:00–15:30, its invite on partner@example.com's calendar.
- **Test data:** New time 17:00–17:30.
- **Steps:**
  1. Choose the meeting's pencil.
  2. Set Starts 17:00 and Ends 17:30, and choose "Save changes".
  3. Check the guest's calendar and inbox, and the workspace calendar.
- **Expected result:** (1) "Reschedule meeting" opens filled in with "Video call (Google Meet)" chosen. (2) "Meeting saved." and the card reads "17:00 – 17:30". (3) The guest has one invite, now 17:00–17:30, and an email about the change; the workspace calendar's event reminds at 16:45.

### TC-MTG-14 — Video call to In person drops the Meet link and shows the venue
- **Traces:** US-MTG-05 · **Priority:** Medium · **Type:** Functional
- **Preconditions:** Signed in as an Event Organizer. Google Calendar connected. An upcoming video meeting for event "Tech Summit BKK" (venue "BITEC Hall 2") shows its Meet link and "Join".
- **Test data:** Mode "In person".
- **Steps:**
  1. Choose its pencil, set Mode to "In person" and choose "Save changes".
  2. Read the card.
  3. Open the guest's invite.
- **Expected result:** The card shows "In person" and "BITEC Hall 2", with no Meet link and no "Join". The guest's invite now shows the venue and no Meet link.

### TC-MTG-15 — A stale edit is refused; a cancelled meeting cannot be edited
- **Traces:** US-MTG-05 · **Priority:** Medium · **Type:** Negative
- **Preconditions:** Two signed-in sessions have upcoming meeting M "Speaker brief" (10:00–10:30) open in "Reschedule meeting". A cancelled meeting N exists.
- **Test data:** Session 2 renames M "Speaker brief v2"; Session 1 sets Ends to 11:00.
- **Steps:**
  1. In Session 2, choose "Save changes".
  2. In Session 1, choose "Save changes".
  3. Reload, and find N under All.
- **Expected result:** (1) "Meeting saved." (2) Session 1 sees "Someone else changed this meeting — reload it and try again." at the top of the panel. (3) M reads "Speaker brief v2", "10:00 – 10:30". N shows "Cancelled" and has no pencil.

---

## US-MTG-06 — Cancel a meeting

### TC-MTG-16 — Cancel with a reason
- **Traces:** US-MTG-06 · **Priority:** Medium · **Type:** Functional
- **Preconditions:** Signed in as an Event Organizer. Google Calendar connected. Upcoming meeting "Speaker brief" with Anong Phromma, its invite on anong@example.com's calendar.
- **Test data:** Reason "Speaker unavailable".
- **Steps:**
  1. Choose its pencil, then "Cancel meeting".
  2. Enter the Reason and choose "Cancel meeting".
  3. Read the Upcoming count and the card under All.
  4. Check the guest's calendar and inbox.
- **Expected result:** (1) "Cancel this meeting?" shows "Speaker brief · Anong Phromma". (2) "Meeting cancelled." shows. (3) Upcoming drops by 1; under All the card has a red "Cancelled" badge and "Cancelled — Speaker unavailable", with no "Join" or pencil. (4) The meeting is gone from the guest's calendar, and their cancellation email carries "Speaker unavailable".

### TC-MTG-17 — "Keep it" changes nothing
- **Traces:** US-MTG-06 · **Priority:** Low · **Type:** Negative
- **Preconditions:** Signed in as an Event Organizer. An upcoming scheduled meeting with an invite sent.
- **Test data:** —
- **Steps:**
  1. Choose its pencil, then "Cancel meeting".
  2. Choose "Keep it".
  3. Open the dialog again and close it by clicking outside it.
- **Expected result:** The meeting stays scheduled with no "Cancelled" badge, the tab counts are unchanged, and the guest receives nothing.

---

## US-MTG-07 — Join a video meeting in one click

### TC-MTG-18 — Join opens the call; no Join for past or unsent calls
- **Traces:** US-MTG-07 · **Priority:** High · **Type:** Functional
- **Preconditions:** Signed in as an Event Organizer. Google Calendar connected.
- **Test data:** Video meetings: P today, Meet link ready; Q 2 days ago, with a Meet link; R tomorrow, invite couldn't be sent.
- **Steps:**
  1. On P, choose "Join".
  2. Look at Q's row.
  3. Look at R's row.
- **Expected result:** P's call opens in a new tab. Q has a "Past" badge and no "Join". R has no "Join"; its card reads "The calendar invite couldn't be sent." with "Send it now".

---

## US-MTG-08 — Connect and manage the workspace Google Calendar

### TC-MTG-19 — Admin connects Google Calendar; unsent meetings go out
- **Traces:** US-MTG-08 · **Priority:** Medium · **Type:** Functional
- **Preconditions:** Signed in as an Admin on a wide screen. Google Calendar not connected. Upcoming meeting "Vendor follow-up" was saved meanwhile and reads "No calendar is connected, so no invite was sent."
- **Test data:** The workspace's Google test account, calendar@eventa-test.co.th.
- **Steps:**
  1. Open Meetings and choose "Connect Google Calendar" in the header.
  2. On Settings → Integrations, choose "Connect Google Calendar" and allow access in Google.
  3. Read the toast and the Google Calendar card.
  4. Open Meetings; read the header and "Vendor follow-up".
- **Expected result:** (1) Settings → Integrations opens; the card reads "Not connected". (3) "Google Calendar connected."; the card reads "Connected" with calendar@eventa-test.co.th and today's date. (4) The header shows "Google Calendar"; "Vendor follow-up" loses its note once its invite is sent, and the guest receives it.

### TC-MTG-19b — Admin disconnects Google Calendar; invites already sent stay
- **Traces:** US-MTG-08 · **Priority:** Medium · **Type:** Functional
- **Preconditions:** Signed in as an Admin on a wide screen. Google Calendar connected. An upcoming meeting's invite is on its guest's calendar.
- **Test data:** New meeting "Vendor follow-up", tomorrow 11:00–11:30, Phone call.
- **Steps:**
  1. On Settings → Integrations, choose "Disconnect", then "Keep it connected".
  2. Choose "Disconnect" and confirm with "Disconnect".
  3. Open Meetings; read the header and check the earlier guest's calendar.
  4. Schedule "Vendor follow-up".
- **Expected result:** (1) "Disconnect Google Calendar?" says new and changed meetings won't be sent until it's connected again and meetings already on people's calendars stay there; "Keep it connected" changes nothing. (2) "Google Calendar disconnected."; the card reads "Not connected" with no account. (3) The header offers "Connect Google Calendar"; the earlier invite is still on the guest's calendar. (4) "Vendor follow-up" is saved and reads "No calendar is connected, so no invite was sent."

### TC-MTG-20 — Non-Admin sees the status only; an expired connection asks for a reconnect
- **Traces:** US-MTG-08 · **Priority:** Medium · **Type:** Negative
- **Preconditions:** Wide screen. Google Calendar connected. Members: an Event Organizer without Manage integrations, and an Admin.
- **Test data:** Case B: the workspace's access is revoked in its Google account, then the Organizer schedules any valid meeting.
- **Steps:**
  1. As the Organizer, open Settings and look for Integrations; then open the Integrations page by its address.
  2. Case B: as the Organizer, read the Meetings header and the new meeting's card.
  3. Case B: as the Admin, read the Meetings header, then open Settings → Integrations.
- **Expected result:** (1) No Integrations entry; the page shows "Connected" with Account "—", no buttons, and "Only an Admin can connect or disconnect Google Calendar." (2) The header reads "Calendar needs reconnecting"; the card reads "No calendar is connected, so no invite was sent." (3) The header is a "Reconnect Google Calendar" link; the card reads "Reconnect needed", warns "Google stopped accepting this connection. Reconnect to keep sending invites." and offers "Reconnect".

---

## US-MTG-09 — Be warned about overlapping meetings

### TC-MTG-21 — Overlap warning appears but never blocks saving
- **Traces:** US-MTG-09 · **Priority:** Won't · **Type:** Edge
_Withdrawn: the overlap warning is not in the web app; the id stays valid — not run._


---

<a id="tc-e13"></a>

# Measure Performance (Reports) — Test Cases

Area: RPT · Epic E13 — Measure Performance (Reports)
Locale notes applied throughout: currency ฿ (THB), VAT 7%, Asia/Bangkok timezone, EN/TH text, bank accounts masked to last 4 digits.

---

## US-RPT-01 — Workspace health at a glance

### TC-RPT-01 — Overview tiles open on "Year", coloured by what is good for each figure
- **Traces:** US-RPT-01  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin (finance access). Today is 2026-07-19 (Bangkok). Paid sales, refunds and check-ins exist in the last 365 days and in the 365 days before.
- **Test data:** Last 365 days: Revenue ฿120,000, Registrations 400, Attendance rate 80%, Avg ticket ฿300, Refund rate 4%. The 365 days before: Revenue ฿100,000, Refund rate 6%.
- **Steps:**
  1. Open Insights › Overview and note which range tab is selected.
  2. Read the five tiles with their arrows, percentages and colours.
- **Expected result:** "Year" is selected. Revenue reads ฿120,000 with an up arrow and 20% in the positive colour. Registrations 400, Attendance rate 80% and Avg ticket ฿300 each show their change. Refund rate reads 4% with a down arrow and 33.3% in the positive colour: fewer refunds is good.

### TC-RPT-02 — A range tab recomputes the whole screen; a window with no revenue
- **Traces:** US-RPT-01  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Signed in as Admin. Today is 2026-07-19. Revenue exists earlier in the year but none from 2026-07-06 to 2026-07-19 (the last 14 days).
- **Test data:** Range tabs "Year", then "7 days".
- **Steps:**
  1. On the Overview with "Year" selected, note the Revenue and Registrations tiles and the "Revenue trend" total.
  2. Choose "7 days".
  3. Read the tiles, the trend total, its change and the chart area.
- **Expected result:** All tiles and the trend change to the last 7 days, not only the chart. Revenue and the trend total read ฿0; the change reads "—" in a neutral colour (the 7 days before were ฿0 too); "No revenue in this period." replaces the chart. No error.

---

## US-RPT-02 — Filter and search every report

### TC-RPT-03 — One event or a search narrows tiles and table together and returns to page 1
- **Traces:** US-RPT-02  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin. Insights › Registrations lists more than one page of events for the last 365 days.
- **Test data:** "Bangkok Tech Summit 2026" with 400 registrations; search "Jazz" matching "Bangkok Jazz Night" (120) and "Jazz in the Park" (80).
- **Steps:**
  1. Open Insights › Registrations and go to page 2.
  2. Pick "Bangkok Tech Summit 2026" in the event picker.
  3. Set the picker back to "All events" and type "Jazz" in the search box.
- **Expected result:** After step 2 one row remains, Total registrations reads 400 and the table is on page 1. After step 3 only the two Jazz events remain and Total registrations reads 200. The tiles always describe exactly the rows below them.

### TC-RPT-04 — An end date before the start date is refused
- **Traces:** US-RPT-02  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** Signed in as Admin on Insights › Income.
- **Test data:** Start date 2026-07-01, End date 2026-06-01.
- **Steps:**
  1. Enter the two dates.
  2. Choose "Apply".
- **Expected result:** "End date must be on or after the start date" shows under the dates. The tiles and table still show the previous window; nothing breaks.

### TC-RPT-05 — A span over 24 months is trimmed; no match offers "Clear filters"
- **Traces:** US-RPT-02  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Signed in as Admin on Insights › Registrations.
- **Test data:** Start date 2023-01-01, End date 2026-06-30 (42 months); then search "zzzzz".
- **Steps:**
  1. Enter the dates and choose "Apply".
  2. Read the date boxes and the note below them.
  3. Type "zzzzz" in the search box.
  4. Choose "Clear filters".
- **Expected result:** The most recent 24 months are used: the Start date becomes 2024-06-30 and the note reads "Trimmed to the most recent 24 months — a report covers at most 24 months at a time." With "zzzzz" the table says "No registrations match." with "Clear filters", and the tiles read 0. "Clear filters" brings back the last 365 days with no search.

---

## US-RPT-03 — Registration mix and sales-channel breakdown

### TC-RPT-06 — Ticket-type and top-event breakdowns add up and follow the range tab
- **Traces:** US-RPT-03  ·  **Priority:** Low  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin (finance access). Registrations in the last 365 days across three ticket types and eight events.
- **Test data:** Year: General 200, VIP 120, Student 80 (400 in all); busiest event "Bangkok Tech Summit 2026" with 150. Last 30 days: General 30, VIP 10.
- **Steps:**
  1. Open Insights › Overview ("Year").
  2. Read "Registrations by Ticket Type" and compare its centre with the Registrations tile.
  3. Read "Registrations by event".
  4. Choose "30 days" and read both panels again.
- **Expected result:** Ticket types run largest first: General 50% (200 registrations), VIP 30% (120), Student 20% (80); the centre reads 400, as does the Registrations tile. "Registrations by event" lists 6 events, "Bangkok Tech Summit 2026" first with a full bar and the others' bars sized against 150. After "30 days" the donut shows General 75% and VIP 25% with 40 in the centre, and the bars show that window's events.

---

## US-RPT-04 — Rank and explore event performance

### TC-RPT-07 — Events ranked by registrations with revenue, attendance and status badge
- **Traces:** US-RPT-04  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin. Today is 2026-07-19, 10:00 Bangkok.
- **Test data:** "Bangkok Tech Summit 2026" (ended 2026-07-10): 400 registrations, Net ฿92,000, 320 of 400 tickets checked in. "Bangkok Jazz Night" (running today): 250. "Chiang Mai Design Week" (starts 2026-08-15): 90. "Phuket Food Fair" (cancelled, was 2026-06-20): 40.
- **Steps:**
  1. Open Insights › Event performance.
  2. Set the End date to 2026-08-31 and choose "Apply".
  3. Read the row order, the columns and the badges.
- **Expected result:** The page is titled "All events". Rows run 400, 250, 90, 40. "Bangkok Tech Summit 2026" shows Revenue ฿92,000 and Attendance 80%. Badges read Completed, Live, Upcoming and Cancelled.

### TC-RPT-08 — Status picker, "—" for an event not yet started, and drill-in links
- **Traces:** US-RPT-04  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** As TC-RPT-07, with more than one page of events and the End date 2026-08-31 applied.
- **Test data:** Status "Live"; "Chiang Mai Design Week" (Upcoming).
- **Steps:**
  1. Go to page 2, then pick "Live" in the status picker.
  2. Pick "All statuses", find "Chiang Mai Design Week" and read its Attendance %.
  3. Choose its name.
  4. Open Insights › Overview and choose "View all" beside "Top events".
- **Expected result:** (1) Only Live events remain, still ranked by registrations, on page 1. (2) Attendance % reads "—", not 0%. (3) That event's page opens. (4) The "All events" list opens.

---

## US-RPT-05 — Income and reconciliation report

### TC-RPT-09 — Net = Gross − Refunds − Fees, equals Overview Revenue; VAT is in the export
- **Traces:** US-RPT-05  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin. Only one event took money in the last 365 days.
- **Test data:** "Bangkok Tech Summit 2026": gross ฿100,000 (VAT included), completed refunds ฿5,000, processing fees ฿3,000.
- **Steps:**
  1. Open Insights › Income; read the event row and the four tiles.
  2. Open Insights › Overview ("Year") and read Revenue.
  3. Back on Income, choose "Export" › Excel and open the data sheet.
- **Expected result:** The row reads Gross ฿100,000, Refunds ฿5,000, Fees ฿3,000, Net ฿92,000, and the tiles read the same. Overview Revenue reads ฿92,000. The workbook row also carries VAT ≈ ฿6,542.06 and Settled ≈ ฿85,457.94; neither shows on screen.

### TC-RPT-10 — A free event has no Income row but shows ฿0 in Event performance
- **Traces:** US-RPT-05  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Signed in as Admin. In the last 365 days one paid event and one free event took place.
- **Test data:** Paid: "Bangkok Tech Summit 2026" (Net ฿92,000). Free: "Community Meetup Bangkok" (฿0 tickets, 60 registrations, held 2026-07-10).
- **Steps:**
  1. Open Insights › Income and read the rows.
  2. Open Insights › Event performance and find "Community Meetup Bangkok".
- **Expected result:** Income lists "Bangkok Tech Summit 2026" only; the free event has no row and nothing errors. Event performance lists "Community Meetup Bangkok" with Revenue ฿0.

### TC-RPT-11 — No income for the selection: "No events match." and ฿0 tiles
- **Traces:** US-RPT-05  ·  **Priority:** Medium  ·  **Type:** Negative
- **Preconditions:** Signed in as Admin on Insights › Income.
- **Test data:** Start date 2025-02-01, End date 2025-02-28: a month with no paid sales.
- **Steps:**
  1. Enter the dates and choose "Apply".
  2. Choose "Clear filters".
- **Expected result:** The table says "No events match." with "Clear filters", and all four tiles read ฿0; no error. "Clear filters" brings back the last 365 days.

---

## US-RPT-06 — Transaction ledger for investigations

### TC-RPT-12 — A refund reads negative; tiles; a reference opens Payments
- **Traces:** US-RPT-06  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin. Only these entries exist in the last 365 days.
- **Test data:** Payment "TXN-8842" by Anong Pattana, ฿1,250, refunded in full on 2026-07-12; three other succeeded payments of ฿1,000.
- **Steps:**
  1. Open Insights › Transactions.
  2. Find the refund row; read its reference, amount, colour and Type.
  3. Read the four tiles.
  4. Choose the reference "TXN-8842".
- **Expected result:** The refund row reads "TXN-8842-R1", "-฿1,250" in red, Type "Refund"; payment rows are positive with Type "Payment". Tiles: Transactions 5, Payments ฿4,250, Refunds 1, Success rate 100%. Choosing the reference opens Payments with its search set to "TXN-8842"; nothing in the ledger changed.

### TC-RPT-13 — A failed charge counts against Success rate but not Payments; the refunded payment keeps its amount
- **Traces:** US-RPT-06  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Signed in as Admin. Only these entries exist in the last 365 days.
- **Test data:** Nine charges of ฿1,000 that didn't fail, one of them (by Somchai Jaidee) later refunded in full; one Failed charge of ฿800.
- **Steps:**
  1. Open Insights › Transactions and find the ฿800 row.
  2. Read the Payments and Success rate tiles.
  3. Find Somchai Jaidee's payment and its refund.
- **Expected result:** The ฿800 row is badged "Failed". Payments reads ฿9,000 (the ฿800 is left out) and Success rate 90% (9 of 10). Somchai Jaidee's payment still reads ฿1,000 with Status "Refunded"; its refund is a separate "-฿1,000" row with Type "Refund".

---

## US-RPT-07 — Payouts and settlement visibility

### TC-RPT-14 — Payout rows and tiles, with the bank masked
- **Traces:** US-RPT-07  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin. Only these payouts exist in the last 365 days.
- **Test data:** PO-2026-014: Paid, ฿85,000, bank account ending 6789, covers "Bangkok Tech Summit 2026". PO-2026-015: In transit, ฿40,000, covers three events. PO-2026-016: Pending, ฿15,000.
- **Steps:**
  1. Open Insights › Payouts.
  2. Read the PO-2026-014 and PO-2026-015 rows.
  3. Read the four tiles.
- **Expected result:** PO-2026-014 shows its date, Bank "•••• 6789", Events covered "Bangkok Tech Summit 2026", ฿85,000 and Paid. PO-2026-015 shows Events covered "3 events" and In transit. Tiles: Paid out ฿85,000, Pending ฿15,000, In transit ฿40,000, Avg payout ฿46,667. No full account number appears anywhere.

### TC-RPT-15 — A multi-event payout appears under each of its events; an empty selection
- **Traces:** US-RPT-07  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Signed in as Admin on Insights › Payouts.
- **Test data:** PO-2026-015, ฿40,000, covers "Bangkok Tech Summit 2026", "Bangkok Jazz Night" and "Chiang Mai Design Week"; search "PO-9999" matches nothing.
- **Steps:**
  1. Pick "Bangkok Jazz Night" in the event picker.
  2. Pick "Chiang Mai Design Week".
  3. Pick "All events" and type "PO-9999" in the search box.
- **Expected result:** PO-2026-015 appears for both events, each time at its full ฿40,000. With "PO-9999" the table says "No payouts for this selection" with "Clear filters"; no error.

---

## US-RPT-08 — Registrations report by event and status

### TC-RPT-16 — Status counts add up to the row total, which matches the Overview
- **Traces:** US-RPT-08  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin. Only one event took registrations in the last 365 days.
- **Test data:** "Bangkok Tech Summit 2026": Confirmed 300, Pending 40, Waitlist 30, Cancelled 30 seats, plus 5 rejected seats.
- **Steps:**
  1. Open Insights › Registrations and read the event row.
  2. Read the four tiles.
  3. Open Insights › Overview ("Year") and read Registrations.
- **Expected result:** 300 + 40 + 30 + 30 = 400, the row's Registrations; the rejected seats are not counted. Tiles: Total registrations 400, Confirmed 300, Pending 40, Cancelled 30. Overview Registrations reads 400.

### TC-RPT-17 — Staff can open the Registrations report; an empty selection
- **Traces:** US-RPT-08  ·  **Priority:** Medium  ·  **Type:** Functional
- **Preconditions:** Signed in as a Staff member on the default Staff role. Registrations exist in the last 365 days.
- **Test data:** Search "zzzzz".
- **Steps:**
  1. In Insights, open Registrations.
  2. Type "zzzzz" in the search box.
- **Expected result:** The report opens in full: tiles, table and event picker. With "zzzzz" the table says "No registrations match." with "Clear filters", and the tiles read 0.

---

## US-RPT-09 — Attendance and no-show report

### TC-RPT-18 — 400 registered and 320 checked in read 80 no-shows and 80%; the rate matches the Overview
- **Traces:** US-RPT-09  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin. Only one event started in the last 365 days.
- **Test data:** "Bangkok Tech Summit 2026" (ended 2026-07-10): 400 tickets, 320 checked in, 288 of them before the start.
- **Steps:**
  1. Open Insights › Attendance and read the event row.
  2. Read the four tiles.
  3. Open Insights › Overview ("Year") and read Attendance rate.
- **Expected result:** Registered 400, Checked-in 320, No-shows 80, Attendance % 80%. Tiles: Checked in 320, Attendance rate 80%, No-shows 80, On-time 90% (288 of 320). Overview Attendance rate reads 80%.

### TC-RPT-19 — An event not yet started reads "—" and stays out of the rate; an empty selection
- **Traces:** US-RPT-09  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** As TC-RPT-18. Today is 2026-07-19.
- **Test data:** "Chiang Mai Design Week" starts 2026-08-15: 150 tickets, none checked in. End date 2026-08-31; then search "zzzzz".
- **Steps:**
  1. Set the End date to 2026-08-31 and choose "Apply".
  2. Read the "Chiang Mai Design Week" row and the Attendance rate tile.
  3. Type "zzzzz" in the search box.
- **Expected result:** "Chiang Mai Design Week" shows Registered 150, Checked-in 0, No-shows "—" and Attendance % "—". The Attendance rate tile still reads 80%, not 58%. With "zzzzz" the table says "No events match." with "Clear filters".

---

## US-RPT-10 — Promotion and discount payback

### TC-RPT-20 — Codes show terms, figures, return and status; a scheduled code reads zero; an expired one still counts
- **Traces:** US-RPT-10  ·  **Priority:** Low  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin. Today is 2026-07-19. Only these codes exist.
- **Test data:** SUMMER25: 25% off, active, 40 redemptions, ฿12,000 discount, ฿90,000 revenue. EARLYBIRD: ฿200 off, valid from 2026-09-01. LAUNCH200: ฿200 off, expired 2026-03-31, 10 redemptions, ฿2,000 discount, ฿18,000 revenue.
- **Steps:**
  1. Open Insights › Discounts, set the End date to 2026-09-30 and choose "Apply".
  2. Read the SUMMER25, EARLYBIRD and LAUNCH200 rows.
  3. Read the four tiles.
- **Expected result:** SUMMER25: "25% off", 40, ฿12,000, ฿90,000, Return ×7.5, Active. EARLYBIRD: "฿200 off", 0, ฿0, ฿0, Return "—", Scheduled. LAUNCH200: "฿200 off", 10, ฿2,000, ฿18,000, Return ×9.0, Expired. Tiles: Active codes 1, Redemptions 50, Discount given ฿14,000, Revenue influenced ฿108,000.

---

## US-RPT-11 — Export the current view

### TC-RPT-21 — Excel holds every filtered row plus a Summary sheet
- **Traces:** US-RPT-11  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** Signed in as Admin. Other events also have transactions in the last 365 days.
- **Test data:** Transactions for "Bangkok Tech Summit 2026": 45 entries, including refunds (5 pages of 10 on screen).
- **Steps:**
  1. Open Insights › Transactions, pick "Bangkok Tech Summit 2026" and note the tiles.
  2. Choose "Export" › Excel.
  3. Open the saved file.
- **Expected result:** The Summary sheet shows "Transactions", a "Generated … (Asia/Bangkok)" line, the filters (the period and Event: Bangkok Tech Summit 2026) and key figures equal to the tiles. The data sheet holds all 45 entries of that event and no other: amounts in ฿ with refunds negative, dates as dates.

### TC-RPT-22 — CSV opens Thai correctly with plain values; PDF is branded with filters and time
- **Traces:** US-RPT-11  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Signed in as Admin on Insights › Income.
- **Test data:** Event "มหกรรมเทคโนโลยีกรุงเทพ", starts 2026-07-18, gross ฿12,500; search "มหกรรม".
- **Steps:**
  1. Type "มหกรรม" in the search box.
  2. Choose "Export" › CSV and open it in a spreadsheet.
  3. Choose "Export" › PDF and open it.
- **Expected result:** CSV: the Thai name reads correctly, Starts reads 2026-07-18 and Gross 12500.00, and the first line is the column headings (no summary or timestamp). PDF: the Eventa wordmark, title "Income", a "Generated … (Asia/Bangkok)" line, a filters line (period, Event: All events, Search: มหกรรม), the four key figures and the table, with the Thai name readable.

### TC-RPT-23 — Over 5,000 rows are cut with a note; "Preparing…" blocks a second click
- **Traces:** US-RPT-11  ·  **Priority:** Medium  ·  **Type:** Edge
- **Preconditions:** Signed in as Admin. 6,200 transactions in the last 365 days.
- **Test data:** Insights › Transactions with no filters.
- **Steps:**
  1. Choose "Export" › Excel and watch the button.
  2. While it reads "Preparing…", try to open Export again.
  3. Open the workbook.
  4. Choose "Export" › PDF and open it.
- **Expected result:** While the file is prepared the button reads "Preparing…" and can't be used; one file is saved. The workbook and the PDF each hold 5,000 rows and say "Showing the first 5,000 of 6,200 matching rows."

---

## US-RPT-12 — Role-appropriate access and safe finance sharing

### TC-RPT-24 — Staff sees no finance reports or money; a member without "Export attendee data" can't export Transactions
- **Traces:** US-RPT-12  ·  **Priority:** High  ·  **Type:** Negative
- **Preconditions:** A Staff member on the default Staff role. A second member on a custom role "Finance viewer" with "View payments & payouts" and "View registrations & attendees" but not "Export attendee data".
- **Test data:** The Income report's address.
- **Steps:**
  1. As Staff, open Insights and read its menu.
  2. As Staff, open the Income report's address directly.
  3. As Staff, open the Overview.
  4. As the Finance viewer, open Transactions and choose "Export" › CSV.
- **Expected result:** (1) Only Overview, Registrations and Attendance are offered. (2) "You don't have access to this" shows, with no figures. (3) Registrations and Attendance rate show; Revenue, Avg ticket and Refund rate read "—"; the trend reads "Revenue is only shown to members with finance access."; there is no Registrations by event or Top events. (4) "You don't have permission to export attendee data." shows under Export and no file is saved.

### TC-RPT-25 — Attendee refused; exports land in the Audit log; the reports agree
- **Traces:** US-RPT-12  ·  **Priority:** High  ·  **Type:** Functional
- **Preconditions:** An attendee account (somsri.k@example.com). An Admin account. Registrations and paid sales exist in the last 365 days.
- **Test data:** Income export to Excel with "Bangkok Tech Summit 2026" picked (it took money in the period).
- **Steps:**
  1. As the attendee, open the Insights overview address.
  2. As Admin, on Income, pick "Bangkok Tech Summit 2026" and choose "Export" › Excel.
  3. Open Settings › Security and choose "View log" on Audit log.
  4. Compare Overview ("Year") Registrations and Revenue with the Registrations report's Total registrations and Income's Net revenue.
- **Expected result:** (1) The attendee lands on the organizer sign-in and sees no figures. (2–3) A file is saved, and the Audit log's newest entry reads "Exported the Income report (XLSX, 1 row)" with the period, Event: Bangkok Tech Summit 2026, the Admin's name and the Bangkok time. (4) Both pairs are equal, and no event, registration or payment changed.


---

