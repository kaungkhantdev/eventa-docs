# Eventa — Functional Requirements (Product Backlog)

| | |
|---|---|
| **Product** | Eventa — Event Registration & Management Platform |
| **Author** | Product Owner |
| **Format** | Product backlog — epics → user stories, with MoSCoW priority & acceptance criteria |
| **Version** | 1.0 |
| **Date** | 2026-07-23 |
| **Scope** | 13 epics · 165 user stories |

> **How to read this.** This is the functional backlog from the **Product Owner's** point of view — what the product must do and *why it matters to the business*, expressed as user stories (`As a <role>, I want <goal>, so that <benefit>`) with business-observable acceptance criteria. Each story carries a **MoSCoW** priority. The technical detail (data model & ERD) lives in [Stage 4 — Architecture](../04-architecture/); quality bars live in [non-functional-requirements.md](non-functional-requirements.md).

## Prioritisation & MVP

The backlog holds **165 user stories**: **90 Must**, **59 Should**, **14 Could** and **2 Won't**. The **Must** stories define the MVP — the core money path *(discover → register → pay → get a QR ticket → check in)* plus creating and publishing an event, sign-in, and getting paid. Should/Could stories deepen finance, insights, engagement, meetings and workspace administration in later releases.

**Won't** marks a story withdrawn from the product: it keeps its id, because code and test cases cite it, and a one-line note under its title says what replaced it. A Won't story is not (or no longer) part of the product: anything already built for it is removed, and its test cases are not run.

Each story describes what the web app does. A story noted **_Not in the web app yet._** is planned: its feature has no screen today, so it lists only the core rules, and test cases for the missing parts are withdrawn until it ships.

## Epics

| # | Epic | Stories | Must | Should | Could | Won't |
|---|------|:-------:|:----:|:------:|:-----:|:-----:|
| 1 | [Accounts & Sign-in](#epic-e01) | 12 | 9 | 3 | 0 | 0 |
| 2 | [Configure the Workspace & Team](#epic-e02) | 13 | 8 | 4 | 1 | 0 |
| 3 | [Create & Manage Events](#epic-e03) | 15 | 11 | 4 | 0 | 0 |
| 4 | [Public Event Pages](#epic-e04) | 10 | 6 | 3 | 1 | 0 |
| 5 | [Sell Tickets & Run Promotions](#epic-e05) | 12 | 3 | 7 | 2 | 0 |
| 6 | [Discover & Register for Events (Attendee)](#epic-e06) | 15 | 11 | 3 | 1 | 0 |
| 7 | [Communicate with Attendees](#epic-e07) | 10 | 1 | 6 | 3 | 0 |
| 8 | [Manage Registrations & Admit Attendees](#epic-e08) | 14 | 8 | 6 | 0 | 0 |
| 9 | [Get Paid & Manage Finances](#epic-e09) | 14 | 11 | 3 | 0 | 0 |
| 10 | [Build the Event Program](#epic-e10) | 13 | 9 | 3 | 1 | 0 |
| 11 | [Organizer Home & Dashboard](#epic-e11) | 16 | 7 | 6 | 1 | 2 |
| 12 | [Coordinate Meetings](#epic-e12) | 9 | 6 | 2 | 1 | 0 |
| 13 | [Measure Performance (Reports)](#epic-e13) | 12 | 0 | 9 | 3 | 0 |
| | **Total** | **165** | **90** | **59** | **14** | **2** |

### Why the epics are in this order

**The epics are sequenced by build dependency, so you can develop them top to bottom.** An epic only
appears once everything its stories stand on already exists. The chain:

```
E1 Accounts ──┬─→ E3 Create Events ──→ E4 Public Pages ──→ E5 Tickets ──→ E6 Discover & Register
              │                                                                │
              └─→ E2 Workspace & Team ─────────────────────────────────────────┤
                                                                               ├─→ E7 Messaging
                                                                               ├─→ E8 Registrations & Check-in
                                                                               └─→ E9 Finance ──→ E13 Reports
                        E10 Program · E11 Dashboard · E12 Meetings ────────────┘
```

- **E1** is first because it is the only epic that needs nothing — `US-ACC-01` mints an identity from
  a blank slate, and every later epic scopes to the workspace owner it creates.
- **E2** is second because an event cannot be sold before the organization has tax details and a
  connected payment account (`US-SET-07`, `US-SET-08`).
- **E6 Discover & Register is sixth, not first.** Eleven of its fifteen stories stand on data other
  epics create: `US-DISC-01` browses *published* events (E3 + E4), `US-DISC-04` needs a priced ticket
  type (E5), and `US-DISC-06` issues a VAT receipt (E9). Built first, it is a browse screen with no
  rows and a Register button with nowhere to land.
- **E13 Reports is last** because every metric it defines reads records the other twelve produce.
- **E10, E11 and E12 float** — their only hard dependency is E3, so they can be pulled forward by a
  spare team at any point after it; they are placed late only because they earn nothing until events
  and attendees are real.

Two epics contain a cycle that has to be sliced, not sequenced: **E3 ↔ E4** (`US-EVT-07` publishes via
E4's renderer, but `US-PAGE-10` refuses to publish an event with no title, date or ticket) — build the
wizard to *draft* only, then E4, then wire up publish. Likewise **E6 ↔ E8**: build E6's money path, then
all of E8, then return for `US-DISC-13` and the "Attended" badge.

### Renumbering map (2026-07-30)

The epics were renumbered once, from feature-area order into the build order above. **Story ids never
changed** — `US-DISC-04` is still `US-DISC-04`. Only the `E<n>` labels moved:

| Now | Was | Epic |
|:---:|:---:|------|
| **E1** | E3 | Accounts & Sign-in |
| **E2** | E13 | Configure the Workspace & Team |
| **E3** | E5 | Create & Manage Events |
| **E4** | E2 | Public Event Pages |
| **E5** | E6 | Sell Tickets & Run Promotions |
| **E6** | E1 | Discover & Register for Events (Attendee) |
| **E7** | E12 | Communicate with Attendees |
| **E8** | E8 | Manage Registrations & Admit Attendees *(unchanged)* |
| **E9** | E10 | Get Paid & Manage Finances |
| **E10** | E7 | Build the Event Program |
| **E11** | E4 | Organizer Home & Dashboard |
| **E12** | E9 | Coordinate Meetings |
| **E13** | E11 | Measure Performance (Reports) |

Resolve an epic by its **AREA code** (`ACC`, `SET`, `EVT`, …), never by assuming `E<n>` matches the nth
area — `E1` holds `US-ACC-*`, `E6` holds `US-DISC-*`.

---

<a id="epic-e01"></a>

# Epic E1 — Accounts & Sign-in

**Goal (business value):** Give organizers and attendees a fast, bilingual (EN/TH) way to sign up, sign in and stay signed in, with the two audiences kept apart, so access never blocks the path from discovery to check-in and accounts resist takeover.

**Primary users:** Event Organizers (the workspace owner signs up as Admin), Staff and other teammates, Registered Attendees, and Guest Attendees without an account.

**Success measures:**
- Sign-in success rate, with few failed sign-ins.
- Sign-ups that confirm and reach first use; new organizers' time to the console.
- Password resets completed self-service, avoiding support tickets.
- Account takeovers near zero; share of organizers with two-factor on.

## User stories

### US-ACC-01 — Create an organizer account · **Must**
**As an** Event Organizer, **I want** to sign up with a workspace name, my name, email and password, **so that** I can start running events.
**Acceptance criteria**
- Given I sign up, then "Check your email" opens and I'm not signed in; an existing organizer's email gets the same page, no new account.
- Given I open the emailed link within 24 hours, then "Email confirmed" shows and "Sign in" takes me to sign-in; an expired link is refused.
- Given I type a password, then a meter asks for 8+ characters with a number and a symbol, then rates it Weak, Fair or Strong.
- Given the password fails that rule, the passwords differ, Terms is unticked or the workspace name is taken, then sign-up stops and I'm told why.
- Given I choose "Send it again" on "Check your email", then a fresh link is sent and the button waits 60 seconds.
_Notes: Whoever signs up owns the new workspace; teammates join by invitation (US-SET-11)._

### US-ACC-02 — Organizer sign-in · **Must**
**As an** Event Organizer or Staff member, **I want** to sign in with my email and password, **so that** I reach my organizer console.
**Acceptance criteria**
- Given my account is active and confirmed, when I sign in on "Welcome back", then I land on Home.
- Given a wrong email or password, then I'm told my email or password is invalid, the same whether or not the email has an account.
- Given my email is unconfirmed, when I sign in with the right password, then I'm refused, asked to confirm, and sent a fresh link.
- Given my access is suspended, when I sign in with the right password, then I'm refused and told my access is suspended.
_Notes: If one email and password open several workspaces, I pick the workspace before signing in._

### US-ACC-03 — Attendee sign-in without forcing accounts · **Must**
**As a** Registered Attendee, **I want** to sign in to see my tickets, **so that** my bookings are in one place.
**Acceptance criteria**
- Given a confirmed attendee account, when I sign in, then I land on the "My Events" tab of "My Account", never the organizer console.
- Given I'm a guest, then I can browse "What's on in Bangkok" and register without an account; the confirmation email links to my ticket.
- Given my email has only an organizer account, when I use it at the attendee sign-in, then I'm refused as for a wrong password.

### US-ACC-04 — Reset a forgotten password · **Must**
**As an** Organizer or Attendee who forgot my password, **I want** a reset link by email, **so that** I can set a new one myself.
**Acceptance criteria**
- Given I have an account in that audience, when I use "Forgot password?", then I'm told to check my inbox; the link lasts 1 hour.
- Given no such account exists, or mine is unconfirmed, still invited or suspended, then nothing is sent and I'm told why and what to do.
- Given 5 misses for one address within 15 minutes, then further requests for it are refused for 15 minutes.
- Given a valid link, when I choose "Set new password", then "Password reset" shows, every device is signed out and the new password works.
- Given an expired or used link, or a new password matching my current one, then it's refused; a dead link offers "Request a new link".
_Notes: An organizer in two workspaces gets a separate email naming each, and a reset leaves two-factor on._

### US-ACC-05 — Change my password while signed in · **Must**
**As a** signed-in Organizer or Attendee, **I want** to change my password by confirming my current one, **so that** I stay secure without email.
**Acceptance criteria**
- Given I choose "Update password" with my current password and the new one twice, then it changes and only my other devices are signed out.
- Given my password changed, then I get an email, in English or Thai, saying when (Bangkok time) and how many other sign-ins were signed out.
- Given my current password is wrong, then I'm told so and nothing changes or signs out.
- Given the new password matches my current one, or the two copies differ, then nothing changes and I'm told why.
- Given the new password lacks 8+ characters, a number or a symbol, then it's refused, as at sign-up.
_Notes: Organizers find it in Settings → "Security" (US-SET-02), attendees in "My Account" → "Settings"._

### US-ACC-06 — Sign in with Google, Apple or LinkedIn · **Should**
**As an** Attendee or Organizer, **I want** to sign up or in with an account I already have, **so that** I skip another password.
**Acceptance criteria**
- Given the provider confirms my email, then it links to my account with that email, or creates a confirmed one; never a duplicate.
- Given I finish the provider's consent screen, then I land on my audience's home; if I cancel it, I'm told so and not signed in.
- Given an organizer page, then Google or LinkedIn create or link only organizer accounts; on the attendee portal, Google or Apple only attendee ones.
_Notes: Organizer sign-in and sign-up show a Google button that only says it isn't connected yet._
_Not in the web app yet._

### US-ACC-07 — Extra security with two-factor sign-in · **Should**
**As an** Organizer or Attendee, **I want** sign-in to need my authenticator app's code, **so that** a stolen password alone can't get in.
**Acceptance criteria**
- Given two-factor is on, when my password is right, then "Enter your code" asks for a 6-digit code before letting me in.
- Given I enter my app's current code or a recovery code and choose "Verify", then I'm signed in; each recovery code works once.
- Given a wrong or already-used code, then I'm told it didn't work and to try the next one or a recovery code.
- Given more than 5 minutes pass after my password, when I enter a code, then I'm told the sign-in expired and must sign in again.
_Notes: Two-factor and recovery codes are managed in Settings → "Security" (US-SET-03) or the attendee "Settings" tab._

### US-ACC-08 — Stay signed in on my device · **Must**
**As an** Organizer or Attendee, **I want** "Remember me" at sign-in, **so that** I stay signed in for a week, not a day.
**Acceptance criteria**
- Given I sign in without "Remember me", then I stay signed in on that browser, even after closing it, for up to 1 day.
- Given I sign in with "Remember me", then I stay signed in for up to 7 days.
- Given that period has passed since sign-in (not my last visit), when I open a page, then I must sign in again.
_Notes: On a shared computer, use "Sign out" (US-ACC-10)._

### US-ACC-09 — See and sign out my other devices · **Should**
**As an** Organizer or Attendee, **I want** to see and sign out my other devices, **so that** a lost or shared one loses access.
**Acceptance criteria**
- Given I'm an organizer, then Settings → "Security" → "Active sessions" lists each sign-in's device, IP address and sign-in date and time (Bangkok).
- Given my current sign-in, then it's marked "This device" with no "Revoke" button.
- Given I choose "Revoke" or "Sign out everywhere else", then those sessions leave the list and are signed out at their next action.
- Given I'm an attendee, then "My Account" → "Settings" → "Other devices" counts my other sign-ins and "Sign out" ends them all.
_Notes: The organizer list is the same screen as US-SET-04._

### US-ACC-10 — Sign out · **Must**
**As a** signed-in Organizer or Attendee, **I want** to sign out, **so that** my session ends on this device.
**Acceptance criteria**
- Given I'm an organizer, when I choose "Sign out" in my user menu, then the organizer sign-in opens and console pages need sign-in again.
- Given I'm an attendee, when I choose "Sign out" on "My Account", then my session ends and the attendee sign-in opens.
- Given I'm an organizer, when I sign in as an attendee in the same browser, then my console sign-in ends: one sign-in per browser.

### US-ACC-11 — Keep organizer and attendee accounts cleanly separated · **Must**
**As the** business, **I want** organizer and attendee accounts kept separate, even for the same email, **so that** each audience only reaches its own area.
**Acceptance criteria**
- Given I'm an attendee opening an organizer page, or an organizer opening "My Account", then I'm sent to that page's sign-in.
- Given the same email has an organizer and an attendee account, then each has its own password: changing or resetting one leaves the other unchanged.
- Given I'm signed out, when I open a protected page, then its audience's sign-in opens and brings me back to that page afterwards.
_Notes: Browsing and registering never need an account, and team roles are in US-SET-12._

### US-ACC-12 — Protect accounts from password-guessing · **Must**
**As an** Attendee or Organizer, **I want** repeated wrong passwords to lock sign-in for a while, **so that** no one can guess my password.
**Acceptance criteria**
- Given 5 wrong passwords for one email within 15 minutes, then sign-in for that email is locked for 15 minutes, even with the right password.
- Given the lock is on, when I try to sign in, then I'm told there were too many attempts and how many minutes remain.
- Given the cool-off has passed, when I enter the correct password, then I am signed in.
- Given the email has no account, then the failures and the lock read exactly as for a real account, never revealing whether it exists.
_Notes: Wrong two-factor codes lock the same way._


---

<a id="epic-e02"></a>

# Epic E2 — Configure the Workspace & Team

**Goal (business value):** Let an organization set itself up without waiting on support: secure accounts, the right seller details on invoices and receipts, working payments, and teammates with just the access their job needs. Then it can sell tickets and get paid.

**Primary users:** Admin (workspace, payments, team and roles); Organizer and Staff (their own profile, security and notifications; Organization read-only); attendees indirectly (receipts, ways to pay).

**Success measures:**
- Time to ready to sell, and share of workspaces taking a first real payment.
- Time from invite to a teammate's first task, such as a check-in.
- Admins with two-factor on; zero unauthorized-access incidents.
- Receipts with the right seller, Tax ID and VAT 7%: few disputes or reissues.

## User stories

### US-SET-01 — My profile & preferences  ·  **Must**
**As a** team member, **I want** to keep my contact details and language current, **so that** colleagues reach me and emails use my language.
**Acceptance criteria**
- Given I open Settings → "Profile", then I see my photo or initials and whether my email is verified.
- Given I change my name, city, timezone, language or bio and choose "Save changes", then they are kept and I'm told so.
- Given a new phone number, then it's saved only after I enter the 6-digit code texted to it ("Send code", then "Confirm").
- Given I change my email, then the new address must confirm by link; the old gets a heads-up and keeps signing me in until then.
- Given I choose ไทย as my language, then my account emails come in Thai; the console always shows Bangkok time.
_Notes: Members edit only their own profile; photos can't be uploaded here._

### US-SET-02 — Change my password  ·  **Must**
**As a** team member, **I want** to change my password after confirming the current one, **so that** my account stays secure.
**Acceptance criteria**
- Given I enter my current password and a new one twice on Settings → "Security" and choose "Update password", then I'm told it's changed.
- Given the new password isn't 8+ characters with a number and a symbol, or the two entries differ, then I'm told why and nothing changes.
- Given my current password is wrong, then I'm told so and my password stays as it was.
- Given the change succeeds, then my other devices are signed out, this one stays signed in, and I'm emailed about it.

_Notes: The Settings side of **US-ACC-05** (E1), which owns this rule._

### US-SET-03 — Two-factor authentication & recovery codes  ·  **Should**
**As a** security-conscious member, **I want** two-factor sign-in with an authenticator app and backup codes, **so that** a stolen password alone can't get anyone in.
**Acceptance criteria**
- Given I switch two-factor on and choose "Start setup", then I see a QR code to scan or a key to type.
- Given I enter the app's 6-digit code ("Enable 2FA"), then it's on and I get 8 one-time recovery codes, shown once to copy or save.
- Given a wrong or expired code, then I'm told so and two-factor stays off.
- Given two-factor is on, when I enter a current 6-digit code and choose "Turn off", then it's off and I'm emailed about it.
_Notes: The code asked for at sign-in is US-ACC-07._

### US-SET-04 — Review & sign out my active sessions  ·  **Should**
**As a** team member, **I want** to see where I'm signed in and sign out lost or unknown devices, **so that** I control my account.
**Acceptance criteria**
- Given "Active sessions" on Settings → "Security", then each session shows its device (like Chrome on macOS), IP and sign-in time, in Bangkok time.
- Given the session I'm using, then it's marked as this device and can't be revoked.
- Given another session, when I choose "Revoke", then it leaves the list and that device loses access on its very next action.
- Given more than one session, when I choose "Sign out everywhere else", then every session but this one is signed out.

_Notes: The Settings side of **US-ACC-09** (E1); this story adds "Sign out everywhere else"._

### US-SET-05 — Security & access audit log  ·  **Should**
**As an** Admin, **I want** a read-only, downloadable log of sign-ins, role and permission changes and exports, **so that** sensitive actions are traceable.
**Acceptance criteria**
- Given I choose "View log" on Settings → "Security", then the latest 20 entries show newest first: what, who, IP and Bangkok time.
- Given a teammate's role changed, then an entry names them, the old and new role, who changed it and when.
- Given I export a date range ("Export log"), then a CSV of those days downloads and the export is logged; a reversed range is refused.
- Given any entry, then nothing in the product lets anyone edit or delete it.
- Given I can't manage users, then I see only my own entries; an Admin sees the whole workspace's.

### US-SET-06 — Notification preferences  ·  **Should**
**As a** team member, **I want** to pick email and SMS alerts per topic, **so that** I hear what matters without noise.
**Acceptance criteria**
- Given Settings → "Notification preferences", then Registrations, Payments, Sales milestones, Feedback, Payouts, Operational alerts and Tasks each have Email and SMS switches.
- Given I flip one switch, then only that topic's channel changes, saved at once and kept after a reload.
- Given a topic I never changed, then its Email is on and its SMS is off.
- Given I have no confirmed phone (US-SET-01), then the SMS column shows a dash, not switches.
_Notes: Eventa saves these choices but sends no topic alerts yet; receipts and security emails aren't controlled here._

### US-SET-07 — Organization profile, tax details & branding  ·  **Must**
**As an** Admin, **I want** to keep our organization's name, address, Tax ID and logo correct, **so that** invoices and receipts name the right seller.
**Acceptance criteria**
- Given I save our details on Settings → "Organization", then I'm told so and new invoices and receipts show our name, address and Tax ID.
- Given a document was already issued, when our details change, then it keeps the details it was issued with.
- Given a Tax ID that isn't 13 digits or a website not starting with http:// or https://, then the field says why; nothing is saved.
- Given I upload a JPEG, PNG or WebP logo up to 5 MB, then it replaces our side-rail initials; I can "Replace" or "Remove" it.
- Given I'm an Organizer or Staff, then the page is read-only and says only an Admin can change it.
_Notes: Currency, country and VAT rate are fixed; changing one goes through support._

### US-SET-08 — Connect our payment account  ·  **Must**
**As an** Admin, **I want** to connect Stripe with test keys, then live ones, **so that** we prove payments work before taking real money.
**Acceptance criteria**
- Given I save our publishable key, secret key and webhook secret under "Test" or "Live", then Stripe checks them first; refused keys aren't saved.
- Given saved test keys, then the status reads connected (test) and a banner says no real charges happen; live keys read connected (live), no banner.
- Given saved keys, when I choose "Test connection", then I'm told it works or Stripe's reason it failed; no money moves.
- Given a saved secret, then it's never shown again, only a masked tail.
- Given I choose "Disconnect" and confirm, then paid checkout stops and keys are removed; free events, past orders and payouts are unaffected.
_Notes: Live keys are accepted only on the production service._

### US-SET-09 — Choose payment methods at checkout  ·  **Must**
**As an** Admin, **I want** to turn each payment method on or off, **so that** attendees are offered exactly the ways to pay we support.
**Acceptance criteria**
- Given "Payment methods" on Settings → "Payments", then Card, PromptPay, Bank transfer, Apple Pay and Google Pay have switches; Card and PromptPay start on.
- Given Card or PromptPay is on, then paid checkouts offer it; once switched off, it no longer appears.
- Given we sell paid tickets, when I switch off the last method that's on, then I'm told to keep one and it stays on.
- Given no connected payment account, when I switch on Apple Pay or Google Pay, then I'm told to connect one first; it stays off.
_Notes: Checkout offers only Card and PromptPay today; the other switches are saved but not offered yet._

### US-SET-10 — Checkout & receipt preferences  ·  **Must**
**As an** Admin, **I want** to set our card-statement label and whether receipts are emailed, **so that** attendees recognize the charge and get a receipt.
**Acceptance criteria**
- Given I save a statement label, then I'm told it's saved and new card charges carry it; the box stops at 22 characters.
- Given a card charge, then the statement shows our Stripe prefix plus the label's first 12 letters, numbers and spaces (EVENTA TICKETS → EVENTA TICKE).
- Given a label with characters other than letters, numbers, spaces or . , ' -, then I'm told which are allowed and it isn't saved.
- Given email receipts are on, when a payment succeeds, then the buyer is emailed a receipt in English or Thai itemizing the included VAT 7%.
- Given email receipts are off, when a payment succeeds, then no receipt email is sent.
_Notes: This label overrides the one on Settings → "Organization"; PromptPay payments carry none._

### US-SET-11 — Invite & manage teammates  ·  **Must**
**As an** Admin, **I want** to find, invite, suspend, reactivate and remove teammates, **so that** only the right people are in our workspace.
**Acceptance criteria**
- Given Settings → "Users", then I search by name or email, filter by role, or pick a status tab (All, Active, Invited, Suspended) with counts.
- Given I invite a name, email and role, then they're Invited, emailed a link, and Active once they set a password; re-inviting resends, never duplicates.
- Given I suspend a member, then they're signed out at once and told they're suspended at sign-in; reactivating restores the same role.
- Given I remove a member, then access ends at once, past work stays, and the address can be re-invited; an Active member's address can't.
- Given the last Admin, when anyone tries to suspend, remove or change their role, then it's refused and I'm told to promote someone else first.

### US-SET-12 — Assign roles & fine-tune permissions  ·  **Must**
**As an** Admin, **I want** to give each member a role and fine-tune what roles allow, **so that** everyone can do their job, nothing more.
**Acceptance criteria**
- Given I pick a member's new role (it saves at once) or edit permissions, then access follows on their next action, without signing in again.
- Given the built-in roles, then Admin can do everything; Staff views registrations and the agenda, and checks people in.
- Given an Organizer, then they run events, program and registrations but can't refund, void invoices or payouts, or manage users or settings.
- Given I'm not an Admin, when I assign a role or save permissions with access I don't hold, then it's refused and I'm told why.
- Given a role or permission change, then it's logged with who, what and when, flagged sensitive if it grants exporting attendee data or issuing refunds.
_Notes: Attendees have no console role; they use the attendee portal._

### US-SET-13 — Create custom roles  ·  **Could**
**As an** Admin, **I want** to create roles with exactly the permissions a job needs, **so that** jobs like Volunteer fit where built-in roles don't.
**Acceptance criteria**
- Given Settings → "Roles", then each role shows member count, description, whether it's built-in, and its first three permissions plus a count of the rest.
- Given I search roles, then they filter by name or description.
- Given I create a role with a name, description and permissions, then it appears and can be assigned to members; a taken name is refused.
- Given I choose "Edit role" and save new permissions, then everyone holding it gets them on their next action; the name stays fixed.
- Given the only role that can manage users and roles, when I switch that off, then it's refused until another role has it.


---
<a id="epic-e03"></a>

# Epic E3 — Create & Manage Events

**Goal (business value):** Let organizers create, publish and run an event in minutes, from first draft to a live page selling correctly priced tickets, without ever destroying an event people have paid for.

**Primary users:** Event Organizer (creates, publishes, cancels and runs events), Admin (all of that, plus event categories), Attendee (sees the public page made here).

**Success measures:**
- Minutes, not hours, to create and publish a first event.
- Share of started drafts that go live.
- Registrations and revenue per published event, including from share links and flyers.
- Zero paid events lost to accidental changes; VAT-inclusive prices always correct.

---

## User stories

### US-EVT-01 — Find and manage my events  ·  **Must**
**As an** Event Organizer, **I want** one searchable, sortable list of my events, **so that** I can find any event and act on it.
**Acceptance criteria**
- Given Events, then "Active" shows my events 10 a page, with a total, "Rows per page" and "Prev"/"Next"; "Completed" holds finished and cancelled ones.
- Given each row, then I see its name, type, date, status and registrations against capacity with a fill bar (45/100 reads 45%).
- Given I search, pick a type, sort by registrations, name or date, or switch tab, then the list narrows and returns to page 1.
- Given nothing matches, then I'm told so and offered "Clear filters", never a blank table.
- Given a row's "More actions", then I can view, edit, duplicate or delete it, and tab counts update after a duplicate or delete without reloading.
_Notes: "Duplicate" needs create rights and "Delete" publish rights, or they're hidden; members without event access are refused._

### US-EVT-02 — Create an event with a guided wizard and save drafts  ·  **Must**
**As an** Event Organizer, **I want** a step-by-step wizard that saves as I go, **so that** I capture every detail without losing work.
**Acceptance criteria**
- Given any event, then the rail always lists Basics, Date & location, Seating, Tickets, Assign seats, Review & publish, and progress reads 50% at Seating.
- Given a new event, then "Save as draft" or Date & location's "Next" saves a Draft, which needs a title and start date.
- Given a saved event, then "Next" saves silently and "Save changes" confirms; if a teammate saved meanwhile, I'm refused until I reload; their save stands.
- Given an online or general-admission event, then Assign seats has no seats to assign; general admission also offers "Go to Seating" to choose Reserved seating.
- Given a published event, when I press the last step's "Save changes", then my edits apply, it stays published and I return to its workspace.
_Notes: On a wide screen a Summary panel shows the title, ticket types, capacity and seating as I type._

### US-EVT-03 — Describe the event (Basics)  ·  **Must**
**As an** Event Organizer, **I want** to give the event a title, description, type, category, cover and highlights, **so that** its public page stands out.
**Acceptance criteria**
- Given the Basics step, then I enter an "Event title", a formatted "Description", a "Type" and one of my workspace's categories or "No category".
- Given the description, then a counter shows characters out of 5,000; anything typed or pasted past 5,000 is cut off and the counter turns red.
- Given a cover image that isn't PNG or JPG, or is over 5MB, then it's refused, I'm told why, and nothing is kept.
- Given highlights (icon plus label), then they show on the public page in the order entered; one without a label is dropped on save.
- Given the title or description is empty, then the "Publish checklist" flags it and "Publish event" can't be pressed.
_Notes: The "Category" picker shows only to members who may manage categories (US-EVT-11)._

### US-EVT-04 — Set the date, time and location  ·  **Must**
**As an** Event Organizer, **I want** to set the event's dates and whether it's in person or online, **so that** attendees know when and where.
**Acceptance criteria**
- Given the Date & location step, then I set the start and end dates and times in Bangkok time; "Timezone" is fixed at GMT+7.
- Given the end is not after the start, when I save, then I'm refused.
- Given "In-person", then I enter a "Venue name" and "Address" and see the venue on a map; publishing needs the venue name.
- Given "Online", then no venue is needed, and the "Meeting link" is emailed to each attendee once they register.
_Notes: Online events have no seating; switching back to "In-person" brings it back._

### US-EVT-05 — Choose seating (general admission or reserved)  ·  **Must**
**As an** Event Organizer, **I want** general admission or a reserved seat map by ticket type, **so that** each attendee gets the right seat.
**Acceptance criteria**
- Given an in-person event, then "Reserved seating" previews "Rows" × "Seats / row" live, with "Total seats"; "General admission" has no map, only a headcount.
- Given Assign seats, then I set a row's type from its menu or "Paint seats with" one by click or drag; mixed rows read "Mixed".
- Given seats have types, then each takes its type's colour and the legend's count is the type's quantity; the Tickets step has no Quantity column.
- Given a reserved draft with an untyped seat, or a type without seats, then the checklist or a refusal naming the type blocks publishing.
- Given a published reserved event, then sold seats stay, a booked or held seat keeps its type, and no type drops below its sales.
_Notes: Assign seats needs saved ticket types ("Go to Tickets"); on a published map, new seats save only once typed._

### US-EVT-06 — Set up tickets, capacity and registration rules  ·  **Must**
**As an** Event Organizer, **I want** to set ticket types, capacity, approval and waitlist, **so that** the right tickets sell at the right price.
**Acceptance criteria**
- Given the Tickets step, then each type has a name, a ฿ price (0 is free) and, unless reserved, a quantity; "Capacity" sets the headcount.
- Given a ticket name already used on the event, a quantity below tickets already sold, or removing the last saved ticket type, then I'm refused.
- Given a ฿890 ticket, then the price already includes 7% VAT (฿831.78 + ฿58.22), and the receipt shows it within "Total paid", service fee included.
- Given "Require approval" is on, then each new registration awaits my decision (US-REG-02).
- Given "Enable waitlist" is on (general admission only), then a sold-out ticket offers the waitlist at checkout (US-REG-04).
_Notes: Sales windows are set per ticket type on the Tickets screen (US-TKT-03)._

### US-EVT-07 — Pick a public page template, set visibility, and publish  ·  **Must**
**As an** Event Organizer, **I want** to pick a template and visibility, then publish, **so that** the public page goes live only when complete.
**Acceptance criteria**
- Given Review & publish, then the template I pick ("Classic", "Spotlight", "Minimal", or "Vibrant" in "Browse templates") fills only from my event's details.
- Given "Preview" or "Preview landing page", then a new tab shows the page, unsaved edits included; if pop-ups are blocked, I'm told to allow them.
- Given "Visibility", then "Public" is listed on Discover with a live page; "Unlisted" and "Private" have none; only a draft's visibility can change.
- Given the "Publish checklist", then it lists what's missing (title, description, in-person venue, saved ticket type, every reserved seat typed) and blocks "Publish event".
- Given a complete draft, then "Publish event" puts it live at its public link and opens its workspace; without publish rights I'm refused.
_Notes: A cover image is optional._

### US-EVT-08 — Cancel or delete an event without losing paid registrations  ·  **Must**
**As an** Admin, **I want** delete limited to empty drafts and cancel for everything else, **so that** I never destroy an event people paid for.
**Acceptance criteria**
- Given a draft with no registrations or payments, when I confirm "Delete", then it's permanently removed and I'm told so.
- Given a published event, or a draft with registrations or payments, when I confirm "Delete", then I'm refused and offered "Cancel event instead".
- Given "Cancel event", then it needs a "Reason"; once cancelled, registration closes, the event moves to "Completed" and I'm told refunds and notices are underway.
- Given a cancellation, then each confirmed or awaiting-approval registrant is emailed the reason in their language (English or Thai) and promised a refund.
_Notes: Delete and cancel need publish rights; refunds aren't automatic and are issued from Payments (US-FIN-02)._

### US-EVT-09 — Manage the event program (agenda and speakers)  ·  **Must**
**As an** Event Organizer, **I want** to add and remove sessions and see speakers in the event's workspace, **so that** the published program stays accurate.
**Acceptance criteria**
- Given "Add session" on "Agenda", then I set title, day, start time, duration, type and room; it lists under its day in time order.
- Given a session without a title, when I add it, then it's refused.
- Given another session holds that room at that time, then it's refused, naming the clashing session; different rooms may overlap.
- Given I remove a session and confirm, then it leaves the agenda and the public page; attendees who bookmarked it aren't told.
- Given the "Speakers" tab, then each speaker shows as a card (name, role, talk, tag); "Manage speakers" opens this event's Speakers page to add more.

### US-EVT-10 — Adjust ticket inventory after publishing  ·  **Must**
**As an** Event Organizer, **I want** to add ticket types from the event's workspace, **so that** I can respond to demand without rebuilding the event.
**Acceptance criteria**
- Given the "Tickets" tab, when I "Add ticket" (name, ฿ price or free, quantity), then it shows "On sale" at 0 sold and sells publicly.
- Given a name already used on the event, then the new ticket is refused.
- Given a reserved event whose seats have types, then a new type's quantity reads 0 until Assign seats gives it seats (US-EVT-05).
- Given "Manage tickets", then the Tickets screen opens to edit or retire a ticket type (US-TKT-02, US-TKT-05).

### US-EVT-11 — Organize events with categories  ·  **Must**
**As an** Admin, **I want** to create, edit and safely delete event categories, **so that** events are grouped consistently for attendees.
**Acceptance criteria**
- Given "Event categories", then each shows icon, colour, name, description and event count; I can search, sort by name or count, and toggle grid/list.
- Given "New category" or edit, then I set name, description, icon and colour with a live "Preview", and "Save category" confirms it.
- Given a name already used in my workspace, then the category is refused.
- Given "Delete", then a category with no events is removed after I confirm; one with events is refused until I move them to another category.
- Given I rename a category, then every event in it shows the new name on its public page.

### US-EVT-12 — See my events on a calendar and what's coming up  ·  **Should**
**As an** Event Organizer, **I want** a month calendar and a soonest-first list of upcoming events, **so that** I see my schedule at a glance.
**Acceptance criteria**
- Given the "Calendar", then events sit on their Bangkok start dates and the header counts them; I can step months or jump to "Today".
- Given a day with more than three events, then three show plus a count of the others.
- Given I choose a day, then its events are listed by time, or I'm told none are scheduled.
- Given "Upcoming events", then my next events show soonest first, each with days left ("Today" on the day) and how full it is.
- Given I click an event on either view, then its workspace opens.

### US-EVT-13 — Duplicate an event to relaunch quickly  ·  **Should**
**As an** Event Organizer, **I want** to copy an event as a fresh draft, **so that** I can relaunch a similar event without re-entering it.
**Acceptance criteria**
- Given "Duplicate", then a Draft "{name} (Copy)" copies details, date, seating layout, ticket types, agenda, speakers, template and visibility, but no registrations or sales.
- Given the copy, then it has its own public address and its ticket types' sales windows are cleared.
- Given the copy is made, then the list refreshes and the "Active" count goes up by one.
- Given I press "Duplicate", then it reads "Working…" and is locked until the copy exists: one press, one copy.
_Notes: A reserved copy's seats have no ticket types; type them on Assign seats before publishing (US-EVT-05)._

### US-EVT-14 — Monitor an event's performance from its workspace  ·  **Should**
**As an** Event Organizer, **I want** one workspace with an event's headline numbers, registrations and attendees, **so that** I can track how it is doing.
**Acceptance criteria**
- Given "Overview", then I see registrations against capacity (120/300 reads 40%), tickets sold, days left and ฿ revenue, or a dash without finance access.
- Given the public link on "Overview", when I press "Copy", then it's copied and the button reads "Copied".
- Given the "Registrations" tab, then paged rows show attendee, reference, tickets, ฿ amount, registration time and status; filters "All", "Paid", "Pending", "Refunded" show counts.
- Given the "Attendees" tab, then confirmed attendees are listed with name, email, registrations and seats, under a badge counting them.
- Given I lack registration access, then the "Registrations" and "Attendees" tabs are refused to me, since names and emails are personal data.
_Notes: Emailing an event's attendees is done from Announcements (US-MSG-04)._

### US-EVT-15 — Promote an event with share links and a printable flyer  ·  **Should**
**As an** Event Organizer, **I want** share links and a printable flyer for the event, **so that** I drive registrations across channels.
**Acceptance criteria**
- Given "Share" on the event's workspace, when I choose "Copy link", then the public link is copied and the label reads "Copied!"
- Given Facebook, X, LINE, WhatsApp or Email, then it opens with the link; X, WhatsApp and Email add an invite with title, date and place.
- Given the flyer, when I choose "Save", then a PNG poster downloads with the event's title, date, place and link.


---

<a id="epic-e04"></a>

# Epic E4 — Public Event Pages

**Goal (business value):** Give every event a fast, shareable public page that turns a click from a shared link, search or Discover into a registration, with no account needed. Organizers design, preview and publish it themselves under a stable link.

**Primary users:** Attendee (guest or signed in) reads the page and registers; Event Organizer and Admin preview, design and publish it.

**Success measures:**
- Share of page visitors who click Register and complete a booking.
- Visits per shared link and click-through from search and social.
- Page loads under 2.5 seconds on a 4G phone; few visitors bounce.
- Organizers preview designs and publish a live page in one sitting.

---

## User stories

### US-PAGE-01 — Open a shareable event page  ·  **Must**
**As a** guest Attendee, **I want** a shared link to show what, when and where, **so that** I can decide to register without an account.

**Acceptance criteria**
- Given a published public event, when I open its link, then I see title, date, time, place and tickets with no sign-in prompt.
- Given a draft, unlisted, private, cancelled or missing event, then a "Not found" page says it isn't available, with "Back to home", never another event.
- Given any section is empty (highlights, about, agenda, speakers, tickets or FAQs), then it is left out, never an empty heading.
- Given any event, then dates and times read in English and Bangkok time (Aug 15, 2026; 18:00 – 21:00); Thai text shows as typed.

_Notes: The page only starts registration; it never books, holds seats or shows attendee data._

### US-PAGE-02 — Event identity and one-tap Register  ·  **Must**
**As a** guest Attendee, **I want** key facts up top and Register always in reach, **so that** I can start booking once convinced.

**Acceptance criteria**
- Given I open the page, then its top shows the title, date, time, place (or that it's online) and the organizer's cover image.
- Given I scroll, then a Register button stays in reach: sticky top bar, hero, every ticket and the page foot.
- Given I click any Register button ("Register", "Register now", or "Get tickets" in Vibrant), then the event's "Register" page opens in the same tab.
- Given the cover image is missing or fails to load, then a green brand gradient shows, never a broken-image icon.

### US-PAGE-03 — Clear about section and online-event handling  ·  **Must**
**As an** Attendee, **I want** to see where the event is, or how I'll join online, **so that** I know what to expect after registering.

**Acceptance criteria**
- Given an in-person event, then About shows the description, a details list (date, time, venue, address, category, starting price) and a map with "Get directions".
- Given an online event, then the page says it's online and the join link comes after registering, with no address or map.
- Given I register for an online event, then my confirmation email carries the organizer's "Meeting link", which the page never shows.
- Given a detail is blank (say, no address), then only that row is left out.

### US-PAGE-04 — Highlights, agenda and speakers  ·  **Should**
**As a** guest Attendee, **I want** to see the highlights, schedule and speakers, **so that** I can judge if it's worth my time and money.

**Acceptance criteria**
- Given the organizer added "Highlights" in the event wizard, then each shows with its icon and label, in the order entered.
- Given the event has an agenda, then an "Agenda" section lists sessions in agenda order, each with its time range, title, room and speakers.
- Given the event has speakers, then a "Speakers" section shows each one's initials, name and talk title (or job role), in the order added.
- Given any of these is empty, then its section is left out.

### US-PAGE-05 — Ticket tiers, pricing and availability  ·  **Must**
**As a** guest Attendee, **I want** to see each ticket's price and availability, **so that** I can pick the right one before registering.

**Acceptance criteria**
- Given the event has ticket types, then the "Tickets" section shows one card each, cheapest first, with name, price and own Register button.
- Given a price, then it already includes 7% VAT; a free ticket reads Free, and a ฿0 ticket not marked free reads RSVP.
- Given a ticket is sold out or paused, then its card says sold out; with 10 or fewer left, it warns with the real count.
- Given reserved seating, then a "Seat map" under the cards shows the stage and seats in their tickets' "Assign seats" colours, keyed with seat counts.
- Given a seat map, then each card names its seats as the key does (Rows 1–2); general-admission and online events have neither.

_Notes: Ticket, quantity and seats are chosen on the "Register" page, which refuses sold-out tickets; the seat map never shows taken seats._

### US-PAGE-06 — Frequently asked questions  ·  **Could**
**As a** guest Attendee, **I want** an FAQ I can expand, **so that** I can clear up doubts before I register.

**Acceptance criteria**
- Given FAQs, then "Frequently asked" ("Know before the show" in Vibrant) loads with only the first answer open; I can open or close any.
- Given the event has no FAQs, then there is no FAQ section.

_Notes: The web app has no screen for organizers to add FAQs yet._

### US-PAGE-07 — Add the event to my calendar  ·  **Should**
**As a** guest Attendee on my phone, **I want** to add the event to my calendar in one tap, **so that** I don't miss it.

**Acceptance criteria**
- Given an event, when I tap "Add to calendar", then I get an entry with title, start and end in Bangkok time, and place.
- Given an online event, then the entry's place says it's online and it carries no join link.
- Given I add the same event again, then the existing entry is updated, not duplicated.

_Not in the web app yet._

### US-PAGE-08 — Rich search results and social sharing  ·  **Should**
**As an** Event Organizer, **I want** one clean link everywhere I share, and previews hidden from search, **so that** people land on the real page.

**Acceptance criteria**
- Given a published public event, then "View page", the "Landing page" card and every "Share" channel give one clean address: no names, emails or tracking.
- Given the page is open, then the browser tab reads the event title followed by "· Eventa".
- Given a design preview, then search engines are told to skip it; a draft, unlisted, private or cancelled event has no public page.

_Notes: Rich link cards (title, image, price) aren't built yet; sharing itself is US-EVT-15._

### US-PAGE-09 — Preview any template with unsaved content  ·  **Must**
**As an** Event Organizer, **I want** to preview my page in any design with unsaved edits, **so that** I can choose without committing anything.

**Acceptance criteria**
- Given I click the wizard's "Preview" or "Preview landing page", then my design opens in a new tab; if blocked, I'm told to allow pop-ups.
- Given a draft, then the preview shows only my typed title, place and highlights (a blank title gets a placeholder), no tickets, agenda or speakers.
- Given a live event, then the preview is its real page with my typed changes on top.
- Given I preview, then nothing is saved and search engines are told to skip the preview.
- Given a live event under "Preview with" on "Landing Pages", then any design's "Preview" opens it in that design, as saved, in a new tab.

_Notes: Typed changes on a live event show only to signed-in console users; anyone else opening the link sees the saved page._

### US-PAGE-10 — Choose a design, brand it, and publish under a stable link  ·  **Must**
**As an** Event Organizer, **I want** to choose my page's design and address, then publish, **so that** it looks right and my link keeps working.

**Acceptance criteria**
- Given a draft on "Review & publish", when I pick Classic, Spotlight or Minimal and click "Publish event", then it goes live in that design.
- Given I click "Use template" on any design (Vibrant too), then the "Preview with" event switches to it, same address, nothing lost, now "In use".
- Given an unmet "Publish checklist" line (title, description, venue or online link, a ticket type, a type per reserved seat), then "Publish event" is disabled.
- Given I save a "Public address", then it's tidied into link form (bkk-tech-meetup), refused if already taken in my workspace, and the old link retires.
- Given I save an "Accent colour", then a hex colour such as #E91E63 is kept, and anything else becomes the default #2563EB.

_Notes: The designs don't use the accent colour yet; every page shows in Eventa green._


---

<a id="epic-e05"></a>

# Epic E5 — Sell Tickets & Run Promotions

**Goal (business value):** Organizers set what they sell — ticket types, prices, quantities, sales windows — and run discount codes that grow sales without leaking margin. Attendees get the right ticket at the right price in a few taps.

**Primary users:** Event Organizer and Admin (set up tickets and codes); Attendee, guest or registered (buys tickets, applies codes); Staff sees Tickets and Discounts in the menu but can't open them.

**Success measures:**
- Revenue (฿) per event, sell-through (sold ÷ available) and time to sell out.
- Conversion on shared ticket links and QRs.
- Code redemption rate, extra sign-ups, and discount cost vs. revenue.
- Zero oversold tickets and zero over-redeemed codes.

---

## User stories

### US-TKT-01 — Create paid and free ticket types  ·  **Must**
**As an** Event Organizer, **I want** paid or free ticket types on my events, **so that** people buy the right ticket at the right time.

**Acceptance criteria**
- Given I save a "New ticket type", then its card appears on Tickets, reading Scheduled if Sales start is after today, otherwise On sale.
- Given I choose "Free", then the Price box disappears and attendees register for that ticket without paying.
- Given the event already has a ticket of that name, when I save, then I'm told the name is taken and nothing is created.
- Given Sales end before Sales start, or a Per-order limit outside 1–8 or above the quantity, then nothing saves and I'm told what to fix.
- Given a reserved event whose seats have ticket types, then the seat map sets the quantity (US-EVT-05).

_Notes: Prices are whole baht and include VAT, as the attendee pays them._

---

### US-TKT-02 — Adjust a ticket type safely after it is live  ·  **Must**
**As an** Event Organizer, **I want** to edit a live ticket type within guardrails, **so that** I meet demand without changing paid orders.

**Acceptance criteria**
- Given I edit a ticket's quantity, then I can raise it, putting a sold-out ticket back On sale at once, but never below what's sold.
- Given a ticket has sold, then its price and Paid/Free can't change (I'm told to create a new ticket type); before any sale they can.
- Given I edit any ticket, then its Event is locked; a ticket type can't move to another event.
- Given a reserved event whose seats have ticket types, then the seat map sets ticket quantities, which I can't type; other fields stay editable.
- Given a colleague saved the same ticket after I opened it, when I save, then I'm told it changed elsewhere and nothing is overwritten.

_Notes: With the event's waitlist on, raising the quantity offers the new places to people waiting (US-REG-04)._

---

### US-TKT-03 — Availability that manages itself (opens, sells out, pause/resume, closes)  ·  **Must**
**As an** Event Organizer, **I want** sales to open, close and sell out by themselves, and to pause them myself, **so that** I never oversell.

**Acceptance criteria**
- Given a ticket before its Sales start, then checkout shows it as not on sale yet and it can't be bought.
- Given two buyers pay for the last seat at once, then one gets it and the other is refused; once paid, checkout shows Sold out.
- Given a ticket past its Sales end, when a buyer pays, then it's refused because sales have ended.
- Given I choose "Pause sales", then the ticket reads Paused, at checkout too, and can't be bought, even in its sales window, until I resume.
- Given a ticket past its Sales end, when I choose "Resume sales", then it stays Paused and I'm told to extend Sales end first.

---

### US-TKT-04 — See all my tickets and inventory at a glance  ·  **Should**
**As an** Event Organizer, **I want** one list of my ticket types with price, sales and status, **so that** I watch sales across my events.

**Acceptance criteria**
- Given the Tickets page, then each card shows its name, event, price or a Free badge, status, and sold of total with a progress bar.
- Given I pick a status tab (All, On sale, Scheduled, Paused, Sold out), then only those cards show; each tab's count spans all my events.
- Given I search, then only tickets whose name or event contains the text, in any letter case, remain; picking an event narrows to that event.
- Given nothing matches, then I'm told so and "Clear filters" brings the list back.

_Notes: Sold and total counts are read-only here._

---

### US-TKT-05 — Retire a ticket type without harming existing holders  ·  **Should**
**As an** Event Organizer, **I want** to delete ticket types, even sold ones, **so that** I tidy my line-up while holders keep their tickets.

**Acceptance criteria**
- Given I delete a ticket type and confirm, then it leaves the list and can't be bought; holders keep valid tickets and nobody is refunded.
- Given someone is checking out with it, then deleting is refused and I'm told to pause it and try again shortly.
- Given it is the event's last ticket type, then deleting is refused, as every event must keep at least one.
- Given it still has seats on a reserved event, then deleting is refused until I give those seats to another ticket type.

---

### US-TKT-06 — Share a ticket's registration link and QR  ·  **Should**
**As an** Event Organizer, **I want** a ticket's registration link and printable QR, **so that** I promote it on posters, social posts and email.

**Acceptance criteria**
- Given I choose "Share link & QR" on a card, then "Share ticket type" shows its registration link and a QR of it.
- Given the share view, when I choose "Copy link", then the link is copied and the button briefly reads "Copied".
- Given the share view, when I choose "Download QR", then the QR saves as a printable SVG image.
- Given the event is a draft, cancelled or private, then the link and QR still show, with a warning they won't open until it's published.

_Notes: The link is for registering, not entry; each buyer gets their own entry QR with their ticket._

---

### US-TKT-07 — Create a discount code with a quick generator  ·  **Should**
**As an** Event Organizer, **I want** percent or fixed-฿ codes, for one event or all, with limits and dates, **so that** promotions stay in budget.

**Acceptance criteria**
- Given the "New discount code" panel, when I choose "Generate", then the Code box fills with a memorable code unused in my workspace, like PROMO42.
- Given I type a code, then it's saved in capitals and must be 3–24 letters, numbers or hyphens.
- Given I save a 25% code limited to 500, then its row reads 25% off, 0 / 500 and Active, or Scheduled if starting later.
- Given a Per-user limit above the Usage limit, or an End date not after the Start date, then saving is refused and I'm told why.
- Given an End date already past, then saving is refused and I'm told to pick a future one.

_Notes: A Usage limit of 0 means unlimited._

---

### US-TKT-08 — Adjust a discount code without breaking past redemptions  ·  **Could**
**As an** Event Organizer, **I want** to edit a code's limits, dates and scope, **so that** I adjust a promotion without changing past deals.

**Acceptance criteria**
- Given a code redeemed 342 times, then its Usage limit can't be lowered below 342.
- Given a code, then its text and value can change only until its first redemption; after that I'm told to create a new code.

_Not in the web app yet._

---

### US-TKT-09 — Stop a promotion safely  ·  **Should**
**As an** Event Organizer, **I want** to disable or delete a code without touching past redemptions, **so that** I end a promotion once it's done.

**Acceptance criteria**
- Given an Active code, when I choose "Disable code", then it reads Disabled and checkout refuses it at once; its used count stays.
- Given a Disabled code, when I choose "Enable code", then it returns to Active, or Scheduled before its Start date.
- Given I delete a code and confirm, then it leaves the list and stops working at checkout, but past orders keep their discount.

---

### US-TKT-10 — Codes go live and expire on schedule  ·  **Could**
**As an** Event Organizer, **I want** codes to start and stop on their own, by date and usage, **so that** promotions run unattended.

**Acceptance criteria**
- Given a Scheduled code, when its Start date arrives (Bangkok time), then checkout accepts it and it reads Active.
- Given a code reaches its Usage limit or passes its End date, then it reads Expired and checkout refuses it, saying why.
- Given a Disabled code with no time or uses left, then "Enable code" is refused until I update its dates or Usage limit.

---

### US-TKT-11 — Apply a discount code at checkout  ·  **Should**
**As an** Attendee (guest or registered), **I want** to apply a discount code at checkout, **so that** I see at once whether it works.

**Acceptance criteria**
- Given a 25% code on ฿1,000 of tickets, then the Order summary shows Discount ฿250, and the service fee and Total are based on ฿750.
- Given a ฿300-off code on a ฿250 order, then the discount stops at ฿250 and the Total at ฿0, never below.
- Given a code that's unknown, for another event, not started, expired, disabled or at its Usage limit, then a red message says why.
- Given I've used up a code's Per-user limit, when I pay, then I'm told I've already used it.
- Given I chose a free ticket, then a typed code is refused as there's nothing to discount; a free-only event shows no code box.

_Notes: One code per order, entered in the Payment step; the discount comes off the VAT-inclusive price._

---

### US-TKT-12 — Track which promotions are working  ·  **Should**
**As an** Event Organizer, **I want** to see every code's use and status at a glance, **so that** I know which promotions are landing.

**Acceptance criteria**
- Given the Discounts list, then each row shows Code, Type (25% off or ฿200 off), Applies to, Used / Limit, Valid dates and Status.
- Given I choose a status tab (All, Active, Scheduled, Expired, Disabled) or search by code or event name, then the table narrows to match.
- Given a code for All events, when I filter by one event, then it still shows, since it applies there too.
- Given I choose "Copy code" on a row, then the code is copied and its icon briefly shows a tick.

_Notes: Used counts are live and read-only here; the Discounts report goes deeper._


---

<a id="epic-e06"></a>

# Epic E6 — Discover & Register for Events (Attendee)

**Goal (business value):** Let anyone find an event, book without an account, pay in ฿ by card or PromptPay, and walk in with a QR ticket; returning attendees keep tickets, receipts and profile in one account. Every ticket sold starts here.

**Primary users:** Attendees, guest or signed in. Organizers, Admins and Staff don't act here; they work with the bookings, payments and feedback in the organizer console.

**Success measures:**
- Browse-to-booking conversion, checkout completion and guest-checkout success.
- Payment success by method (card, PromptPay) and revenue collected in ฿.
- Share of confirmed bookings that get a usable QR ticket.
- Repeat sign-ins and bookings, survey response rate and average rating.

---

## User stories

### US-DISC-01 — Discover and browse what's on  ·  **Must**
**As an** Attendee, **I want** to browse upcoming events with their key facts, **so that** I quickly spot one worth attending.

**Acceptance criteria**
- Given published public events not yet started, when I open "Discover", then they're listed soonest first, 12 a page; with none, I'm told nothing's on.
- Given a card, then it shows the cover, category, title, date, venue and city or online, organizer, number going, and starting price or free.
- Given every ticket type is sold out, then the card shows a waitlist badge and a dash instead of a price.
- Given 90% or more of an event's limited seats are sold, then its card shows a selling-fast badge.
- Given I tap a card, then the event's own page opens with its "Register" or "Get tickets" button.

### US-DISC-02 — Search and filter events  ·  **Must**
**As an** Attendee, **I want** to search by keyword and narrow by category, **so that** I find the right event without scrolling through everything.

**Acceptance criteria**
- Given I search on "Discover", then only events whose title, category, city or venue contain my words remain, and the event count updates.
- Given accents, Thai tone marks or spaces around the keyword, then search ignores them, so cafe finds Café.
- Given I pick a category chip, then only that category shows; "All events" clears it.
- Given a keyword and a category together, then only events matching both show.
- Given nothing matches, then I'm told to try a different search or category and offered "Clear filters".

### US-DISC-03 — Save events for later  ·  **Should**
**As an** Attendee, **I want** to heart events I'm interested in, **so that** I can come back and register later.

**Acceptance criteria**
- Given an event card, when I tap the heart, then it fills and stays filled when I return; tapping again removes the save.
- Given I'm signed in, then my saves belong to my account and show on any device I sign in on.
- Given I saved events as a guest in this browser, when I next sign in, then they join my account's saves with no duplicates.
- Given a save can't be recorded, then the heart goes back and a note under the card says it couldn't save.

_Notes: There is no separate saved list, only filled hearts on Discover._

### US-DISC-04 — Register and choose my tickets (guest or signed in)  ·  **Must**
**As an** Attendee, guest or signed in, **I want** to pick tickets and seats and see the total, **so that** I know the price first.

**Acceptance criteria**
- Given I choose "Register", then checkout lists the ticket types in ฿ or free to pick one; any I can't buy says why.
- Given reserved seating, then I can pick only available seats of my ticket type, up to 8; switching type clears my picked seats.
- Given general admission or online, then I choose 1 to 8 tickets, noted as first come first served or with a join link emailed.
- Given my pick, then the summary and bottom bar show the total, service fee included; a free event has no payment step, just "Confirm registration".
- Given the details step, then I give name, email and optional phone, pre-filled and editable when signed in; a guest booking creates no account.

_Notes: Seats are held from the moment I book, not while I'm still choosing._

### US-DISC-05 — Pay by card or PromptPay  ·  **Must**
**As an** Attendee, **I want** to pay by card or Thai PromptPay, **so that** I can finish registering the way that suits me.

**Acceptance criteria**
- Given a paid order, then the payment step offers the methods the organizer accepts: "Card", "PromptPay" or both.
- Given Card, when I choose "Continue to pay", then my seats are held and I enter my card only on the provider's secure page.
- Given PromptPay, when I choose "Pay", then a QR for the exact total appears to scan in my banking app.
- Given the payment clears, then my order page confirms it, tickets issued once; while unpaid, it counts down the hold and offers card or PromptPay.
- Given the 10-minute hold ends unpaid, then the order expires: seats go back on sale, nothing is charged and no ticket is issued.

_Notes: Everything is charged in ฿._

### US-DISC-06 — Confirm and receive my QR ticket  ·  **Must**
**As an** Attendee, **I want** a confirmation and QR ticket once I'm booked, **so that** I know I'm in and can get through the door.

**Acceptance criteria**
- Given I book, then a free booking confirms with ticket count, reference and "View my tickets"; a paid one says my seats are held.
- Given my order is confirmed, then its order page, opened without signing in, shows one QR per ticket and what I paid, VAT included.
- Given it's confirmed, then I'm emailed the reference, date, place, total and tickets link, in English or Thai; paid orders add a 7% VAT receipt.
- Given I gave a Thai mobile and the organizer sends texts, then I'm also texted the reference and tickets link.
- Given a double press, then one order is placed and charged once; given someone books my seat first, I pick again and pay nothing.

_Notes: Emails carry a tickets link, not QR codes, and organizers can switch them off._

### US-DISC-07 — View and download my QR ticket  ·  **Must**
**As an** Attendee, **I want** to open and download my QR ticket, **so that** I have my entry pass on my phone or on paper.

**Acceptance criteria**
- Given "My Events", when I choose "Ticket", then I see the QR, reference and admission number; several tickets switch with "Ticket 1", "Ticket 2".
- Given I choose "Download ticket", then its printable pass has QR, event, date, time, venue or online, type, seat or general admission, reference, admission number.
- Given I open my order page from the confirmation, then each ticket shows its QR and code with "Download", no sign-in needed.
- Given a ticket was refunded or voided, then no QR is shown, it says it won't admit anyone, and it can't be downloaded.

### US-DISC-08 — Sign in to my attendee account  ·  **Must**
**As a** registered Attendee, **I want** to sign in with email and password, **so that** I reach my tickets, payments and profile.

**Acceptance criteria**
- Given the right email and password, then I land on "My Events" or back where I was; the organizer console still needs its own sign-in.
- Given I saved events as a guest, when I sign in, then they join my account's saves with no duplicates.
- Given a wrong email or password, then the same message shows either way, never saying which.
- Given 5 failed attempts, then sign-in is locked for 15 minutes.
- Given I choose "Forgot password?", then the reset flow opens; "Create one" opens the attendee sign-up (US-DISC-15).

_Notes: One attendee account holds bookings from every organizer, with no workspace to pick._

### US-DISC-09 — See my upcoming and past tickets  ·  **Must**
**As a** registered Attendee, **I want** my bookings split into upcoming and past, **so that** I find my next ticket fast and revisit past events.

**Acceptance criteria**
- Given "My Events", then "Upcoming" and "Past" tabs, with counts, list every confirmed booking made with my email, from any organizer, even as a guest.
- Given an upcoming event, then its card shows a countdown (days left, tomorrow, today), ticket type, date and time, venue, "Details" and "Ticket".
- Given an event has ended, then it moves to "Past", marked attended only if I was checked in, with "Leave feedback" for its survey.
- Given I have no bookings, then I'm told so and offered "Discover events".

### US-DISC-10 — Review my payment history and receipts  ·  **Must**
**As a** registered Attendee, **I want** a list of my payments with totals, **so that** I can track and expense what I've spent.

**Acceptance criteria**
- Given the "Payment history" tab, then tiles show total spent, number of transactions and amount refunded in ฿, above a paged list of payments.
- Given a row, then it shows the event, invoice number or order reference, date, method (card or PromptPay), amount and a paid or refunded badge.
- Given a refunded payment, then its amount is struck through and left out of total spent.
- Given I have no payments, then I'm told so and offered "Discover events".

_Notes: The VAT receipt for each paid order arrives by email (US-DISC-06)._

### US-DISC-11 — Manage my profile  ·  **Must**
**As a** registered Attendee, **I want** to keep my details and photo current, **so that** organizers can reach me and checkout fills itself in.

**Acceptance criteria**
- Given the "Profile" tab, when I edit name, city, date of birth or bio and choose "Save changes", then they're saved; "Cancel" undoes unsaved edits.
- Given a new email and "Send confirmation", then it's pending and my old email still signs me in until I open the link sent there.
- Given a phone and "Send code", then a 6-digit code is texted; only once I enter it and choose "Confirm" is the number used.
- Given a photo that isn't JPG or PNG, or is over 5 MB, then "Upload" refuses it with the reason and my current photo stays.
- Given saved details, when I next check out signed in, then full name, email and phone are pre-filled.

### US-DISC-12 — Manage my notifications, display preferences, and security  ·  **Should**
**As a** registered Attendee, **I want** to control my notifications, preferences and security, **so that** Eventa suits me and my account stays protected.

**Acceptance criteria**
- Given I switch reminders, SMS alerts (confirmed phone only) or marketing, then it saves at once; turning reminders off stops them, not confirmations or receipts.
- Given I set language (English or Thai), timezone or currency, then emails follow my language; charges stay in ฿, event times in the event's timezone.
- Given my current password and a new one twice, when I choose "Update password", then it changes, other devices sign out and I'm emailed.
- Given a wrong current password or mismatched new ones, then I'm told which and nothing changes.
- Given I turn on two-factor with my authenticator app's 6-digit code, then recovery codes show once and my next sign-in asks for a code.

### US-DISC-13 — Share post-event feedback  ·  **Should**
**As an** Attendee who booked an event, **I want** to answer the organizer's short survey afterwards, **so that** the organizer can make future events better.

**Acceptance criteria**
- Given my confirmed booking and an open survey, when I open it signed in from the thank-you email or "Leave feedback", then its questions show.
- Given the questions, then each is 1 (poor) to 5 (excellent) stars, a choice, a 0 to 10 likelihood scale or a comment box.
- Given I skip a question that isn't a comment box, when I submit, then I'm asked to answer it and nothing is recorded.
- Given I choose "Submit feedback", then I'm thanked; reopening the survey says I've already answered and my first answer stands.
- Given no confirmed booking for the event, or a closed survey, then I'm told there's no survey to answer.

_Notes: Answers are anonymous to other attendees; organizers read them on "Feedback"._

### US-DISC-14 — Delete my account  ·  **Could**
**As a** registered Attendee, **I want** to permanently delete my account and personal data, **so that** I can leave Eventa on my own terms.

**Acceptance criteria**
- Given "Delete account" in "Settings", then I type DELETE and my password, plus a code if two-factor is on; "Keep my account" backs out.
- Given upcoming paid tickets, then the dialog lists them and warns they won't be refunded, with the total in ฿.
- Given I confirm, then my profile, saved events and history are scheduled for removal, I'm signed out to sign-in, and a confirmation email is sent.
- Given a wrong password or code, then I'm told it didn't match and nothing is deleted.

_Notes: Payment and tax records are kept anonymised for the legal retention period._

### US-DISC-15 — Create my account from my confirmation  ·  **Must**
**As an** Attendee who just booked, **I want** to turn my booking into an account, **so that** all my tickets sit in one place.

**Acceptance criteria**
- Given a confirmed, ticketed order, then its page offers "Create account", "Sign in" or "Not now"; other orders get no offer.
- Given I choose "Create account", then nothing is created: a 7-day link is emailed, at most one a minute and five a day per address.
- Given an unexpired, unaltered link, then I set a password, name and email locked, landing on "My Events" with all my bookings; otherwise it fails.
- Given my email has a confirmed account, then the order offers only "Sign in", which brings me back; a used link offers "Sign in instead".
- Given "Create one" on sign-in, then I type my email and must confirm it first; a booking link for that email replaces an unconfirmed sign-up.

_Notes: Declining never affects the ticket, and only the inbox owner can create the account._


---

<a id="epic-e07"></a>

# Epic E7 — Communicate with Attendees

**Goal (business value):** Keep every attendee informed from one place: automatic messages, one-off announcements, an activity feed, a delivery log and surveys. Reliable, on-brand messages get people their tickets, cut support load and make each event teach the next.

**Primary users:** Admin; Event Organizer (everything but "Message templates"); Staff (notification feed, sending announcements); Attendee (receives messages, answers surveys in the attendee portal).

**Success measures:**
- Share of messages marked Failed in the delivery log.
- Survey completion rate per event.
- Average rating and NPS across all events.
- Time for organizers to clear unread notifications.

## User stories

### US-MSG-01 — Attendees automatically get the right message at the right moment  ·  **Must**
**As an** Attendee, **I want** my confirmation, receipt, reminder, waitlist, cancellation and thank-you emails to arrive unasked, **so that** I never chase the organizer.

**Acceptance criteria**
- Given my registration is confirmed (paid or approved), then I'm emailed my booking reference, when and where, total paid and ticket link.
- Given the organizer switched "Event reminder" on (it starts off), then a day before my event I'm emailed its time, place and ticket link.
- Given I switched "Event reminders" off in my account, then I get no reminder; confirmations and receipts can't be switched off there.
- Given the organizer switched a message off on "Message templates", when its trigger fires, then nobody receives it.
- Given my attendee account's Language is ไทย or English, then my messages arrive in it; otherwise (say, as a guest) in English, the default.

_Notes: Eventa always adds ticket and order details below the organizer's wording, fills every merge field, and texts confirmations to Thai mobiles._

### US-MSG-02 — Keep automated messages on-brand and in the organizer's control  ·  **Should**
**As an** Event Organizer, **I want** to reword automated emails with merge fields and switch messages on or off, **so that** attendees get on-brand messages.

**Acceptance criteria**
- Given I "Reword" the subject and opening in English or ไทย and "Save", then only later sends use them; an empty language keeps Eventa's wording.
- Given the "Merge fields" chips, when I click one, then it lands at my cursor and each attendee sees their own details.
- Given a half-written language (subject without opening, or the reverse) or a merge field the message can't fill, then saving is refused, naming it.
- Given a message attendees are entitled to (confirmation, rejection, cancellation, receipt), then switching it off asks first; "Keep it on" leaves it Active.
- Given I lack "Edit workspace settings" (for example Staff or a default Organizer), when I open "Message templates", then I am refused.

_Notes: Only the email is reworded; the confirmation text goes out in Eventa's wording._

### US-MSG-03 — Triage everything happening across my events in one feed  ·  **Should**
**As an** Event Organizer, **I want** one feed of registrations, payments, payouts and alerts that need me, **so that** I triage without opening every module.

**Acceptance criteria**
- Given I open "Notifications", then it shows 30 days with "All" and "Unread" counts; groups older than Earlier this week wait behind "Show older activity".
- Given I "Mark all read", then only my feed is marked and "Unread" stays 0 after reload; an empty "Unread" tab says I'm caught up.
- Given alerts I can act on, then "Needs action" lists all, not just home's 3, each linking to its fix; only resolving one removes it.
- Given I lack finance access (for example Staff), then no payment or payout items show; neither does any alert I couldn't resolve.
- Given unread items, then the header bell shows a red dot; opening it shows my unread count, the 5 newest items and "View all notifications".

### US-MSG-04 — Broadcast a one-off announcement to the right audience  ·  **Should**
**As an** Event Organizer, **I want** to email one event's attendees now or at a Bangkok time, **so that** they hear of changes in time.

**Acceptance criteria**
- Given "Send announcement", then "Send" stays disabled until I pick an event and write a Subject and Message, plus a send time if scheduling.
- Given I confirm "Send now" or a "Schedule" Bangkok time, then the row reads Sent with event and recipient count, or Scheduled for that time.
- Given an announcement goes out, then everyone confirmed by then gets exactly one email, to them alone, even with two bookings or a retry.
- Given a send time under 5 minutes away or over a year ahead, then it is refused under the field.
- Given I pick an event above the list, then "Sent & scheduled" shows only that event's announcements.

_Notes: Email only, always to the event's confirmed attendees._

### US-MSG-05 — Change my mind on a scheduled announcement  ·  **Could**
**As an** Event Organizer, **I want** to cancel or move a scheduled announcement, **so that** I can fix a mistake or shift the timing.

**Acceptance criteria**
- Given a Scheduled row, when I "Cancel" and confirm, then nobody is emailed and it stays in the history as Cancelled.
- Given I "Reschedule" a Scheduled row to a new Bangkok time, then it goes out at that time to whoever is registered then.
- Given a row is sent or cancelled, then it offers no "Reschedule" or "Cancel"; if sending starts mid-change, I'm told it can't be changed.

### US-MSG-06 — Prove messages were delivered and diagnose failures  ·  **Should**
**As an** Admin, **I want** a log of every email and text with failure reasons, **so that** I can help attendees whose ticket never came.

**Acceptance criteria**
- Given sent messages, then "Delivery log" gives each recipient a row: name, email, message type, channel (Email or SMS), event, status, Bangkok send time.
- Given the mail or SMS service refused a message, then its row reads Failed with the reason; otherwise it reads Sent.
- Given failures in view, then I'm told how many didn't get through; the "Needs attention" tab lists only failed rows, or says nothing has failed.
- Given I search a name or address, or pick a message type, then the rows narrow to match.
- Given I lack "Approve, add & invite registrations" (for example Staff), when I open the log, then I am refused.

_Notes: Sent means the mail or SMS service accepted it, not that it arrived or was opened; nothing is re-sent from here._

### US-MSG-07 — Export the delivery log for records  ·  **Could**
**As an** Event Organizer, **I want** to export the delivery log to a file, **so that** I have a record for compliance or diagnosis.

**Acceptance criteria**
- Given I hold "Export attendee data", then "Export" downloads a CSV of the rows matching my tab, search and type; without it, there's no "Export".
- Given I export, then the audit log records it.
- Given nothing matches my filters, when I export, then I'm told there is nothing to export.

_Notes: The file is attendee contact data in bulk, hence its own permission._

### US-MSG-08 — Measure attendee satisfaction across all my events  ·  **Should**
**As an** Event Organizer, **I want** feedback figures across all my events and a page per event, **so that** I see how attendees felt.

**Acceptance criteria**
- Given the "Feedback" tiles, then Responses, Average rating, NPS and Completion rate pool all events: 100 ratings at 4.5 plus 10 at 3.0 read 4.4.
- Given a figure with nothing behind it (no star ratings, no recommendation answers, nobody asked), then it reads "—", never 0.
- Given "Feedback by event", then each event's card shows its date and its survey, live and response counts, or says there are none yet.
- Given I type in "Search events…", then cards narrow by name; with no match, I'm told so and offered "Clear search".
- Given I open a card, then the event's page shows Surveys, "Overall satisfaction" (average, 5★–1★ bars), "Likelihood to recommend" (NPS: promoters, passives, detractors) and Responses.

_Notes: NPS is promoters (9–10) minus detractors (0–6) as a share of answers; Completion is the share of post-event thank-you recipients who answered._

### US-MSG-09 — Build and manage surveys for my events  ·  **Should**
**As an** Event Organizer, **I want** to write and manage surveys for each event, **so that** I collect the feedback I need.

**Acceptance criteria**
- Given I save a "New survey" with a title and questions (Rating, Free text, Multiple choice, Likelihood to recommend), then it's a Draft asking nobody.
- Given no title, a blank question, a choice under two options or two recommendation questions, then saving (new or edited) is refused, naming the question.
- Given a survey, then one button moves it on: "Make live" from Draft, "Close" from Live, "Reopen" from Closed; attendees can't answer a closed one.
- Given I duplicate a survey, then a Draft with its title plus (copy) appears, with no responses.
- Given I delete a survey, then I must confirm first; "Delete it" removes it and its questions.

_Notes: Attendees answer once each in the attendee portal; the post-event thank-you emails them the link the day after the event._

### US-MSG-10 — Browse and filter individual feedback responses  ·  **Could**
**As an** Event Organizer, **I want** to read individual responses, filtered by stars, **so that** I see what attendees said, not just averages.

**Acceptance criteria**
- Given an event has responses, then "Responses" lists them newest first, each with the attendee's name, stars, survey title, date and comment.
- Given I click a star bar under "Overall satisfaction", then only that rating's responses show; clicking it again or "show all" brings all back.
- Given no response has that rating, then I'm told so and offered "Show all".
- Given I lack "Create & edit events" (for example Staff), when I open "Feedback", then I am refused.


---

<a id="epic-e08"></a>

# Epic E8 — Manage Registrations & Admit Attendees

**Goal (business value):** One place to turn sign-ups into confirmed, ticketed attendees and admit them fast, by QR or by hand. More seats filled, shorter door queues, no fake or double entry.

**Primary users:** Event Organizer and Admin (decide, add and invite; only Admin refunds), Staff (view and check in); attendees are the records, never the users.

**Success measures:**
- Time from sign-up to confirmed ticket.
- Seats filled, including freed seats re-sold from the waitlist and invites converted.
- Check-in and no-show rates; door time per attendee at peak.
- Share admitted by QR; bad or duplicate scans caught.

## User stories

### US-REG-01 — Review the registrations queue · **Must**
**As an** Event Organizer, **I want** all registrations in one searchable queue, **so that** I quickly find who needs a decision.

**Acceptance criteria**
- Given I open Registrations, then tabs All, Pending, Waitlist and Cancelled show counts, and rows show attendee, event, ticket, registered date (Bangkok), amount and status.
- Given I search a name, email or order ID, or pick an event, then rows and tab counts narrow to match both, from page 1.
- Given a row, then its badge reads Confirmed, Pending, Waitlisted, Cancelled, Rejected or Expired, noting awaiting approval, place in line or offer deadline where useful.
- Given finance access, then Amount shows a Free badge or the ฿ total, VAT included; without it (e.g. Staff), every amount is a dash.
- Given nothing matches, then I'm told so and offered "Clear filters"; with no registrations at all, I'm told there are none yet.

_Notes: Without permission to decide registrations (e.g. Staff), rows show no Approve or Reject._

### US-REG-02 — Approve or reject a pending registration · **Must**
**As an** Event Organizer, **I want** to approve or reject pending registrations, **so that** only people I accept get tickets.

**Acceptance criteria**
- Given a pending registration, when I approve it, then it's Confirmed, QR tickets are issued and a confirmation emailed, also texted where SMS is on.
- Given Approve is sent twice (a double-click or a retry), then only one set of tickets and one confirmation go out.
- Given payment is incomplete or the registration was rejected, which is final, then Approve is greyed out saying why; once Confirmed, both buttons are.
- Given a pending or waitlisted registration, when rejected, then it's Rejected at once, any held seat returns to sale and a rejection is emailed.
- Given someone paid while awaiting approval, then Reject becomes "Reject and refund", returning the money; without refund permission it's greyed out, saying ask an Admin.

_Notes: Only members who may manage registrations (Organizer, Admin) can decide; a refusal shows under the row._

### US-REG-03 — Add a registration by hand · **Must**
**As an** Event Organizer, **I want** to add a walk-up or phone booking, **so that** off-platform attendees get a proper ticket and count against capacity.

**Acceptance criteria**
- Given I save name, email, event, ticket type and quantity (phone optional), then seats are reserved and the attendee is matched by email or added.
- Given a free ticket, then it's Confirmed at once, shown as Free; a paid one is auto-priced, VAT included, and stays Pending until paid.
- Given a free ticket, then "Send confirmation email" (on by default) emails the QR ticket; off, it's issued quietly for me to hand over.
- Given over 8 seats per booking, more than remain, or a draft, completed or cancelled event, then I'm told why and nothing is reserved.
- Given a missing name or email, or an invalid email, then the form won't save; after any refusal the panel stays open with my entries.

_Notes: "Add registration" is offered only to members who may manage registrations, and is disabled until an event exists._

### US-REG-04 — Manage the waitlist & promote attendees · **Should**
**As an** Event Organizer, **I want** to offer freed seats to the waitlist, **so that** I fill my event fairly and don't lose demand.

**Acceptance criteria**
- Given a waitlisted registration, then its row shows its place in line and "Offer a seat" instead of Approve.
- Given an offered seat on a free ticket, then the attendee is Confirmed at once and gets their confirmation.
- Given an offered seat on a paid ticket, then it's held 24 hours, the row shows the deadline and a pay link is emailed.
- Given no seat is free, then the offer is refused; given people ahead in line, I'm warned how many I'd skip, and skipping is recorded.
- Given an offer lapses, then they're emailed it expired and the seat passes to the next in line; paying in time confirms them, tickets emailed.

### US-REG-05 — Browse the attendee directory & profiles · **Must**
**As an** Event Organizer, **I want** to search, filter and sort everyone who registered and open their profile, **so that** I know who's coming.

**Acceptance criteria**
- Given I open Attendees, then tabs All, New, Checked in and VIP show counts; rows show attendee, phone, events, tickets, tag and last activity.
- Given I search name or email or pick a tag or tab, then rows matching all remain; if none, I'm told so with "Clear filters".
- Given I sort by Name (A–Z), Most events or Most tickets, then the list re-orders; by default, and on ties, latest activity comes first.
- Given an attendee's tag, VIP, Speaker, Sponsor or Student, then it shows as a coloured badge; untagged shows a dash.
- Given I choose "View profile", then "Attendee profile" shows name, tag, email, phone and an Activity timeline, newest first, with Bangkok date and time.

_Notes: New means first seen in the last 30 days; Checked in means checked in at any event._

### US-REG-06 — Invite people to an event · **Must**
**As an** Event Organizer, **I want** to email one person an invitation to one event, **so that** I grow attendance beyond public discovery.

**Acceptance criteria**
- Given I fill event, name and email under "Invite", when I choose "Send invitation", then I'm told it's sent and they're emailed the event's link.
- Given I add a message, then it appears at the top of the invitation, up to 250 characters.
- Given the name or email is missing, or the email is invalid, then the form won't send until I fix it.
- Given I invite the same person to the same event within 24 hours, then no second email is sent.
- Given I may not manage registrations (e.g. Staff), when I send, then the invitation is refused.

_Notes: An invitation holds no seat; "Invite" sits on Attendees and is disabled until an event exists._

### US-REG-07 — Tag & segment attendees · **Should**
**As an** Event Organizer, **I want** to tag an attendee as VIP, Speaker, Sponsor or Student, **so that** I can segment my audience.

**Acceptance criteria**
- Given I set or change an attendee's tag, then their badge and tab counts (e.g. VIP) update; each attendee has at most one tag.
- Given I clear the tag, then the badge shows a dash.
- Given Staff, then they see tags but cannot change them.

_Notes: Tags already show, and filter, in Attendees (US-REG-05); only setting them is missing._

_Not in the web app yet._

### US-REG-08 — Keep attendee contact details current · **Should**
**As an** Event Organizer, **I want** to correct an attendee's name, email and phone, **so that** tickets and messages reach the right person.

**Acceptance criteria**
- Given I edit name, email or phone on a profile and choose "Save changes", then I'm told it's saved and the row and profile update.
- Given a new email or phone, then future confirmations, reminders and SMS go to it; emptying the phone removes it.
- Given the email already belongs to another attendee, then nothing is saved and I'm told to merge the two records instead.
- Given a blank name or an invalid email, then the error shows under that field and nothing is saved.
- Given a saved change, when I reopen the profile, then Activity shows the contact update with date, time and who made it.

_Notes: Only members who may manage registrations see "Edit"; Eventa does not merge records itself._

### US-REG-09 — Email an attendee directly · **Should**
**As an** Event Organizer, **I want** to send one attendee a one-off email from their profile, **so that** I answer questions without leaving Eventa.

**Acceptance criteria**
- Given a subject and message, when I send, then one email goes to the attendee and the send is logged on their Activity.
- Given an empty subject or message, then I'm asked to add both.
- Given a send fails, then I'm told and can retry.

_Notes: The profile's "Send email" only opens my own mail app, addressed to them; Eventa neither sends nor logs it._

_Not in the web app yet._

### US-REG-10 — Export registration & attendee lists · **Should**
**As an** Event Organizer, **I want** to export my filtered registrations or attendees, **so that** I share clean data with CRM, finance or sponsors.

**Acceptance criteria**
- Given I export, then the file holds every filtered row, not just this page, with Thai names intact; no match gives headers only.
- Given an export, then it's recorded: who, how many rows and which filters.
- Given Staff, then no Export is offered.

_Notes: Reports already export their own views (US-RPT-11); Registrations and Attendees have no Export._

_Not in the web app yet._

### US-REG-11 — Run the manual check-in list · **Must**
**As** Staff at the door, **I want** a searchable check-in list per event, **so that** I can check people in by hand and undo mistakes.

**Acceptance criteria**
- Given an event on Check-in, then everyone ticketed is listed with checked-in count, % bar and tabs All, Checked in and Not yet, each counted.
- Given I search by name, email or ticket, then the list narrows and returns to page 1.
- Given I choose "Check in", then the row shows Checked in with the Bangkok time, count and bar rise; "Undo" returns it to Not yet.
- Given it's over 2 hours before the start or over 1 hour after the end, when I check in or undo, then it's refused.
- Given someone is checked in by scan and by hand, then they count once at their earliest time; cancelled or refunded tickets aren't listed.

_Notes: Needs check-in permission (Organizer, Admin, Staff)._

### US-REG-12 — Scan tickets at the live QR check-in station · **Must**
**As** Staff at a check-in station, **I want** to scan tickets with my device's camera, **so that** entry is fast and bad tickets are caught.

**Acceptance criteria**
- Given I pick an event on the "Check-in station", when I switch events, then the scanner, counts and feed reload for it.
- Given I choose "Start camera" and scan a valid, unused ticket for this event, then a green check-in shows their name; count and feed update.
- Given a used ticket, then I see when they arrived and nothing changes; any code held in view is read once, then ignored 2.5 seconds.
- Given an unknown code, another event's ticket, or a refunded or voided one, then the station says why and admits no one.
- Given the camera is blocked, missing, busy or needs https, then I'm told why and offered "Upload QR" or manual lookup; the station still works.

_Notes: Needs check-in permission; "Torch", "Flip" and "Upload QR" help in poor light, swap cameras or read a photo._

### US-REG-13 — Check attendees in manually when a QR won't scan · **Must**
**As** Staff at the station, **I want** to check someone in without a scan, **so that** a broken or missing QR never stops them.

**Acceptance criteria**
- Given a manual search by name, email or ticket ID, when I choose "Check in", then they're admitted, counted and top "Just checked in".
- Given a match is already checked in, then it shows their arrival time instead of "Check in".
- Given no one matches, then I'm told so.

_Notes: The station's "Can’t scan? Find attendee manually" opens this search._

### US-REG-14 — Watch live turnout at the door · **Should**
**As an** Event Organizer, **I want** a live count and feed of arrivals, **so that** I can watch turnout and react to a slow door.

**Acceptance criteria**
- Given the station shows an event, then I see checked in over total tickets with a % bar, plus On-site, Late and Remaining counts.
- Given a check-in succeeds here (camera, "Upload QR" or manual), then counts update and they top "Just checked in" with ticket type and arrival time.
- Given I switch to another event, then counts and feed show its figures; other stations' arrivals appear at my next check-in or reload.
- Given nobody has arrived yet, then the feed says so; otherwise it lists the last 8.

_Notes: On-site means arrived by the start; Late, after it._


---

<a id="epic-e09"></a>

# Epic E9 — Get Paid & Manage Finances

**Goal (business value):** Give organizers one trusted money workspace to see every payment, refund cleanly, receive payouts, hand buyers VAT invoices and file VAT monthly, so revenue reconciles and Thai tax is filed on time.

**Primary users:** Event Organizer (view, download invoices, export), Admin (all of that, plus refunds, payout retries and settings, invoice voids, VAT filing); Staff and Attendees have no finance access.

**Success measures:**
- Share of payments reconciled, with no unexplained gaps.
- Refund turnaround, from request to settled with tickets released.
- Payout settlement time, and failed payouts recovered.
- Invoices paid within 14 days; VAT returns filed by the 15th.

---

## User stories

### US-FIN-01 — See every payment · **Must**
**As an** Event Organizer, **I want** one list of every charge, refund and failed attempt, **so that** I can reconcile sales and answer buyers.
**Acceptance criteria**
- Given I open Payments, then rows run newest first, each showing transaction, attendee, event, method, ฿ amount (VAT included), status and Bangkok date and time.
- Given the All, Paid, Pending, Refunded and Failed tabs, then each counts payments matching my method, event and search; choosing one lists only that status.
- Given I choose a method (Card, PromptPay, Bank transfer) or event, or search part of an attendee name or transaction, then only matches show.
- Given a Pending, Failed or Refunded payment, then "Refund" is greyed and says why: no completed charge, attempt failed, or already refunded.
- Given a Pending PromptPay charge the buyer then pays, when I reload Payments, then it reads Paid without anyone editing it.
_Notes: Never-paid Pending and Failed attempts show a dash for date and sit on top._

### US-FIN-02 — Issue a refund · **Must**
**As an** Admin, **I want** to refund a paid charge in full, **so that** a cancellation is settled and seats go back on sale.
**Acceptance criteria**
- Given a Paid payment, when I choose "Refund", then a dialog restates transaction, buyer and amount and warns it cannot be undone.
- Given I confirm "Refund", then I'm told it started; once the full amount is back on the buyer's original payment method, the payment reads Refunded.
- Given the refund settles, then the order's tickets are void and back on sale, its invoice Void, and Taxes lowers the month it settled in.
- Given I am an Organizer, then "Refund" is greyed, saying only an Admin can issue refunds.
- Given "Refund" is pressed twice (a double click or a retry), then only one refund is issued.
_Notes: A PromptPay refund waits for the buyer's bank details, so the payment stays Paid and tickets valid until then._

### US-FIN-03 — Track balances and payout history · **Must**
**As an** Event Organizer, **I want** my balances and payout history, **so that** I know what I've earned, what's coming and what's landed.
**Acceptance criteria**
- Given a connected payout account, then Payouts shows three ฿ tiles: Available, Pending (Scheduled plus Processing payouts) and Paid out (Paid payouts).
- Given the All, Scheduled, Processing, Paid and Failed tabs, then each lists only its payouts, newest first.
- Given a payout row, then it shows reference, period, bank account (last 4 digits), amount, status, Requested and Completed (a dash until paid).
- Given no payout account is connected, then no balance reads ฿0 (any tile shown reads a dash) and the page offers "Connect a payout account".
_Notes: Only the last 4 digits ever show; Eventa never stores bank or card numbers._

### US-FIN-04 — Follow and recover a payout · **Should**
**As an** Admin, **I want** to see why a payout failed and retry it, **so that** I can unblock my money myself.
**Acceptance criteria**
- Given a Failed payout, then its row shows the reason under the Failed badge and a "Retry payout" button.
- Given I choose "Retry payout" and the provider accepts, then the same payout reads Processing, requested today, and the Audit log records the retry.
- Given the provider refuses the retry, then the payout stays Failed with the new reason.
- Given I am an Organizer, when I choose "Retry payout", then it is refused and the payout stays Failed.

### US-FIN-05 — Manage bank and payout settings · **Must**
**As an** Admin, **I want** to manage my bank and payout settings at the payment provider, **so that** Eventa never holds bank numbers.
**Acceptance criteria**
- Given a connected payout account, then "Manage payouts" on Payouts opens the payment provider's dashboard for bank account, schedule, tax forms and statements.
- Given no payout account is connected, then "Manage payouts" is not offered; "Connect a payout account" (or "Connect account") opens Settings → Payments instead.
- Given I am an Organizer, then "Manage payouts" is not offered.
_Notes: Bank, schedule and tax-form details live with the payment provider, never in Eventa._

### US-FIN-06 — Invoice ledger with ageing · **Must**
**As an** Event Organizer, **I want** each unpaid invoice to show how late it is, **so that** I see who owes and for how long.
**Acceptance criteria**
- Given I open Invoices, then rows run newest issued first, each showing number, buyer name and email, event, VAT, total, status and due date.
- Given an unpaid invoice, then after its due date it reads Overdue, days late in red; until then, Issued, with days left.
- Given the buyer pays the invoice's order, then it reads Paid, as does its due date; a voided invoice reads Void in both.
- Given the All, Issued, Paid, Overdue and Void tabs, then each counts invoices matching my event and search; choosing one lists only that status.
- Given I choose an event, or search part of an invoice number or buyer name, then only matching invoices show.
_Notes: Terms are 14 days from issue; overdue is judged on the Bangkok day, so one due today is still Issued._

### US-FIN-07 — Issue a VAT invoice · **Must**
**As an** Event Organizer, **I want** to issue a VAT invoice for an order, **so that** a business buyer gets a compliant tax document.
**Acceptance criteria**
- Given I invoice an order, then the invoice takes my next yearly number, is due in 14 days and totals the order, 7% VAT included.
- Given the order is already paid, then the invoice is issued as Paid; if already invoiced, issuing again returns that invoice, no new number.
- Given the order has no buyer email or nothing to bill, then no invoice is issued.
_Not in the web app yet._

### US-FIN-08 — View and download a tax invoice · **Must**
**As an** Event Organizer, **I want** to download any invoice as a one-page tax invoice, **so that** I can hand a buyer a compliant document.
**Acceptance criteria**
- Given an invoice row, when I choose "Invoice", then a one-page A4 tax invoice image (not a PDF) downloads, named by its number.
- Given the document, then it shows my organization's name, address and tax ID, and the buyer's name and email.
- Given the document, then it shows the order, issue and due dates, and the payment method and date, or Unpaid.
- Given a ฿32,500 invoice, then it lists an event registration line, subtotal ex-VAT ฿30,373.83 and 7% VAT ฿2,126.17, totalling ฿32,500.
- Given a void invoice (voided, or its payment refunded), then the document keeps its number and carries a large VOID mark.
_Notes: Seller details are those in Settings → Organization when the invoice was issued._

### US-FIN-09 — Chase unpaid invoices · **Should**
**As an** Event Organizer, **I want** to resend an invoice and have overdue ones chased automatically, **so that** I get paid without tracking due dates.
**Acceptance criteria**
- Given an invoice with a buyer email, when I resend it, then the buyer gets it again, unchanged in number, amount and status.
- Given an unpaid invoice passes its due date, then the buyer gets a reminder in their language (English or Thai).
- Given an invoice is Paid or Void, then no more reminders go out for it.
_Not in the web app yet._

### US-FIN-10 — Void an invoice · **Must**
**As an** Admin, **I want** to void an invoice raised in error, **so that** the record is corrected and the number sequence stays whole.
**Acceptance criteria**
- Given an Issued or Overdue invoice, when I choose "Void", then a dialog restates number, buyer and total and says it's kept, marked void.
- Given I confirm "Void invoice" (reason optional), then I'm told it's voided, the row reads Void, and the Audit log records it with the reason.
- Given a voided invoice, then it keeps its number, and no other invoice ever gets it.
- Given a Paid or Void invoice, then "Void" is greyed, saying a Paid one is voided by refunding the payment, or it's already void.
- Given I am an Organizer, when I choose "Void invoice", then it is refused and the invoice is unchanged.
_Notes: Voiding changes no VAT figure: Taxes follows money collected, not invoices._

### US-FIN-11 — Monthly VAT ledger · **Must**
**As an** Event Organizer, **I want** a monthly VAT ledger for each tax year, **so that** I know my tax position and can prepare returns.
**Acceptance criteria**
- Given I open Taxes, then it shows the current Bangkok year, and "Tax year" switches to any of the 5 years before.
- Given a year, then all 12 months list Due (15th of next month), Sales ex-VAT, VAT at 7%, Withholding, Remitted and a status.
- Given a year, then headlines VAT collected, VAT remitted, VAT payable (collected minus remitted) and Withholding each total the 12 months.
- Given an ended month not yet filed, then it reads Due, even after the 15th; a current or future month reads Upcoming.
_Notes: Sales and VAT follow money actually collected, so a refund lowers the month it settles in._

### US-FIN-12 — File the VAT return each period · **Must**
**As an** Admin, **I want** to record each month's VAT filing, **so that** I file on time and remitted and payable figures stay right.
**Acceptance criteria**
- Given a Due month, when I choose "Record filing", then a dialog restates month, VAT and due date, with optional withholding tax in ฿.
- Given I confirm, then I'm told it's filed and the month reads Filed, marked late if recorded after the 15th.
- Given a filed month, then Remitted equals its VAT, VAT payable drops by that, and any withholding adds to the Withholding headline.
- Given a refund settles after a month is filed, then it comes off the month it settles in; the filed month never changes.
- Given an Upcoming month, then "Record filing" is not offered; as an Organizer, recording is refused and the month stays Due.
_Notes: This records the monthly PP30 VAT return; Remitted is never typed._

### US-FIN-13 — Export finance ledgers · **Should**
**As an** Event Organizer, **I want** to export payments, invoices and VAT as I see them, **so that** I can reconcile offline with my accountant.
**Acceptance criteria**
- Given filtered Payments, when I choose "Export", then the file holds every matching payment, not just this page, with amounts VAT included.
- Given filtered Invoices, when I choose "Export", then the file holds only those invoices, with subtotal, VAT and total that add up.
- Given Taxes for a year, when I choose "Export", then the file lists its 12 months with sales, VAT, remitted, payable, withholding and status.
- Given Thai names in an export, when I open it in a spreadsheet, then they read correctly.
- Given filters that match nothing, when I choose "Export", then I'm told there's nothing to export and no file downloads.

### US-FIN-14 — Keep finance to finance roles · **Must**
**As an** Admin, **I want** Finance limited to the people accountable for money, **so that** revenue, payout and tax data stay private.
**Acceptance criteria**
- Given an Attendee, then they cannot open the organizer console, so no Finance page.
- Given a Staff member, when they open Payments, Payouts, Invoices or Taxes, then they're told they lack access and see no figures.
- Given an Organizer, then they can open all four pages and export.
- Given an Organizer, then "Refund" is greyed, "Manage payouts" is not offered, and retrying a payout, voiding an invoice or recording a filing is refused.
- Given an Admin, then every Finance action is available.
_Notes: These are the default roles; Settings → Roles can let another role issue refunds, or void invoices, handle payouts and file VAT._


---

<a id="epic-e10"></a>

# Epic E10 — Build the Event Program

**Goal (business value):** Let organizers lay out each event's sessions day by day and manage the speakers behind them, so attendees see a clear, clash-free programme and trust the event.

**Primary users:** Admin and Event Organizer (build the agenda and speakers); Staff (view only); attendees see the programme on the event page.

**Success measures:**
- Share of events published with a full agenda and speaker line-up.
- Room or speaker clashes reaching attendees: near zero.
- Share of speakers with title, contact details, talk title and bio filled in.
- Share of speakers carrying a rating to guide re-invites.

## User stories

### US-PROG-01 — See the event's schedule at a glance  ·  **Must**
**As an** Event Organizer, **I want** my event's sessions laid out by day and time, **so that** I spot gaps in the programme.

**Acceptance criteria**
- Given I open "Agenda", then the newest event shows, or the one I pick, and only that event's sessions.
- Given the agenda, then each event day is a column headed by weekday and date, with hours from 09:00 down the side.
- Given a session, then its block sits at its day and time, showing time, title and speakers, else its room.
- Given a session's type, then its block is coloured: Keynote blue, Panel green, Workshop amber, Talk rose, Break grey.
- Given no sessions, then I'm told so, with "Add the first session" and "Add speakers", not an error; with no events, "Create your first event".

_Notes: Staff can view the agenda but not change it._

### US-PROG-02 — Add a session to the schedule  ·  **Must**
**As an** Event Organizer, **I want** to add sessions with times, type, room, speakers and description, **so that** attendees see a clear programme.

**Acceptance criteria**
- Given I fill Title, Day, Type, Starts and Ends in "New session" and choose "Save session", then it's confirmed and its block appears.
- Given a saved session, then its block sits at its day and time, coloured by its type.
- Given Title is blank, when I save, then the form asks for it and nothing is saved.
- Given Ends is not after Starts, when I save, then I'm told the end must follow the start and nothing is saved.
- Given I tick a speaker, when the session saves, then their card on "Speakers" counts one session more.

_Notes: Ends may be left blank, and the block then shows only the start time._

### US-PROG-03 — Update a session  ·  **Must**
**As an** Event Organizer, **I want** to change any detail of a session, **so that** the agenda keeps up with line-up and timing changes.

**Acceptance criteria**
- Given I click a session's block, then "Edit session" opens with its details filled in.
- Given I change its day, time or room, when I save, then it's confirmed and the block moves, unless a clash stops it (US-PROG-05).
- Given I change its type from Panel to Workshop, when I save, then the block turns from green to amber.
- Given a colleague saved the session after I opened it, when I save, then I'm asked to reload, and their change is kept.

_Notes: Saving a change sends attendees no message._

### US-PROG-04 — Remove a session  ·  **Must**
**As an** Event Organizer, **I want** to remove a cancelled session after a clear warning, **so that** its speakers stay on my line-up.

**Acceptance criteria**
- Given I choose "Delete" in "Edit session", then I'm warned that attendees who bookmarked it won't be told and it can't be undone.
- Given I choose "Remove", then it's confirmed and the block leaves the agenda and the event's public page.
- Given a removed session, then each of its speakers stays on "Speakers" with one session fewer.
- Given the warning, when I choose "Cancel", then nothing changes; only "Remove" removes the session.

_Notes: The "Remove session" button on the event's "Agenda" tab gives the same warning._

### US-PROG-05 — Keep the schedule physically feasible  ·  **Must**
**As an** Event Organizer, **I want** room double-bookings blocked and speaker double-bookings flagged, **so that** my programme works on the day.

**Acceptance criteria**
- Given Hall A is booked 09:00–10:30, when I save another Hall A session at 10:00 that day, then it's refused, naming the booking.
- Given a speaker is booked at an overlapping time, when I save, then I'm warned; it saves only after I tick "Book them anyway".
- Given two sessions overlap in different rooms with different speakers, or one ends at 10:00 as the next starts, then both save without warning.
- Given I save a session unchanged, then it never clashes with itself.

_Notes: Only sessions with an end time are checked, and rooms only when a room is set._

### US-PROG-06 — Find sessions in a busy programme  ·  **Should**
**As an** Event Organizer, **I want** to search my agenda by keyword, **so that** I find a session fast in a packed programme.

**Acceptance criteria**
- Given I type a keyword in English or Thai, then only sessions whose title or type match stay visible.
- Given I clear the search, then every session shows again.
- Given nothing matches, then the calendar stays, with no sessions, not an error.

_Not in the web app yet._

### US-PROG-07 — Switch between week and month views  ·  **Could**
**As an** Event Organizer, **I want** week and month views with week-by-week navigation, **so that** I can fine-tune slots and see the whole programme.

**Acceptance criteria**
- Given the week view, when I switch to month, then I see a day-by-day overview, and switching back returns the time slots.
- Given any week, when I go to the previous or next week, then the dates and month label move, on the same event.
- Given a week with none of this event's sessions, then it shows empty with the right dates.

_Notes: The agenda already shows every day of the event side by side (US-PROG-01)._

_Not in the web app yet._

### US-PROG-08 — Browse and filter the speaker directory  ·  **Must**
**As an** Event Organizer, **I want** a searchable directory of each event's speakers, **so that** I manage my line-up and see who speaks most.

**Acceptance criteria**
- Given I open "Speakers", then the newest event's speakers show A–Z under a count; picking another event shows only its speakers and count.
- Given a speaker, then their entry shows initials, name, title & company, email, phone, session count and rating.
- Given I type in the search box, then only speakers whose name, title & company or email match stay, from the first page.
- Given I choose "List view" or "Grid view", then the speakers show as rows or as cards.
- Given nothing matches, then I'm told so with "Clear filters"; an event with no speakers says so and offers "Add your first speaker".

_Notes: Ten speakers a page by default; Staff can browse speakers but not add, edit or remove them._

### US-PROG-09 — Add a speaker to the line-up  ·  **Must**
**As an** Event Organizer, **I want** to add a speaker's profile and contact details, **so that** they join my programme and event page.

**Acceptance criteria**
- Given "Add speaker" has at least a full name, when I choose "Save speaker", then it's confirmed and their card appears.
- Given a new speaker, then their card shows 0 sessions and a dash for rating.
- Given a blank full name or an invalid email, when I save, then the form asks me to fix it; nothing is saved.
- Given another speaker on this event uses that email, when I save, then I'm told it's taken and nothing is saved.
- Given the event picked on "Speakers", then the new speaker joins that event, which the panel names under its title.

_Notes: Email is optional and may repeat across events._

### US-PROG-10 — Keep a speaker's profile up to date  ·  **Must**
**As an** Event Organizer, **I want** to edit a speaker's details, **so that** the directory and the event page stay current.

**Acceptance criteria**
- Given I choose "Edit speaker" on a card, then name, title & company, email and phone are filled in; "Save speaker" updates the card.
- Given another speaker on this event uses the new email, when I save, then I'm told it's taken and nothing is saved.
- Given a colleague saved the speaker after I opened it, when I save, then I'm asked to reload, and their change is kept.
- Given the event's public page, then its Speakers section shows the update: the talk title if set, else title & company.

_Notes: Talk title and Bio open blank on edit; leaving them blank keeps what was saved._

### US-PROG-11 — Remove a speaker  ·  **Must**
**As an** Event Organizer, **I want** to remove a speaker who dropped out, after a clear warning, **so that** their sessions stay.

**Acceptance criteria**
- Given I choose "Remove speaker" on a card, then I'm warned it takes them off all their sessions and can't be undone.
- Given I choose "Remove", then it's confirmed, the card leaves, and their sessions stay on the agenda without them.
- Given a session left with no speaker, then its block shows its room instead.
- Given the event's public page, then the speaker no longer appears in its Speakers section or agenda.
- Given the warning, when I choose "Cancel", then the speaker and their sessions are unchanged; only "Remove" removes them.

### US-PROG-12 — Assign speakers to their sessions and balance the line-up  ·  **Should**
**As an** Event Organizer, **I want** to tick each session's speakers and see each one's session count, **so that** I can balance the line-up.

**Acceptance criteria**
- Given "New session" or "Edit session", then I can tick one or more speakers, from this event only.
- Given the event has no speakers, then the speaker list tells me to add one first.
- Given I tick a speaker on a session and save, then its block shows their name.
- Given a 1-session speaker, when I tick them on another and save, then their card shows 2 sessions; unticking returns it to 1.
- Given they already speak at an overlapping time, when I save, then I get the "Book them anyway" warning (US-PROG-05).

### US-PROG-13 — See how speakers were rated  ·  **Should**
**As an** Admin, **I want** each speaker's average rating from attendee feedback, **so that** I can decide whom to invite back.

**Acceptance criteria**
- Given attendees rated a speaker's sessions with an average of 4.9, when I view their card, then it shows 4.9.
- Given nobody has rated a speaker, then the card shows a dash, never 0.
- Given new feedback arrives, then the rating reflects it the next time I look.

_Notes: Every speaker card already shows a dash; nothing turns feedback into a rating yet._

_Not in the web app yet._


---

<a id="epic-e11"></a>

# Epic E11 — Organizer Home & Dashboard

**Goal (business value):** Give organizers one daily home showing what needs them now (today's sign-ups, newest orders, urgent alerts) and a Dashboard of how their events perform, so they act fast and never lose money to a missed alert or sell-out.

**Primary users:** Every organizer-console member (Admin, Organizer, Staff), each seeing only the cards their role allows; attendees have no access.

**Home, top to bottom:** greeting and "New event" (US-DASH-01) · Today's Registrations (02), Ticket revenue (14), Active events (05) · check-in insight (15), Upcoming Events (04) · Recent orders (16), To do (06).

**Success measures:**
- Organizers triage today's sign-ups, orders and alerts from one screen.
- Fewer events reach their date behind on sales or sell out unnoticed.
- Fewer declined payments left unresolved.
- Revenue momentum, check-in rate and capacity fill read at a glance.

## User stories

### US-DASH-01 — Daily operations home  ·  **Must**
**As an** Event Organizer, **I want** a home that greets me and gathers today's activity, **so that** I see my day in one place.
**Acceptance criteria**
- Given home opens, then it greets me by name: morning, afternoon from 12:00, evening from 18:00 Bangkok time, in the workspace's language.
- Given I choose "New event", then the event creation flow opens.
- Given no active event, order, recent sign-up or alert yet, then home shows "Set up your workspace": five steps from organization to publishing, next highlighted.
- Given I'm signed in as an attendee, when I open the organizer home, then I'm sent to the organizer sign-in.
_Notes: Home only summarizes and links out; it never changes data._

### US-DASH-02 — Today's registrations at a glance  ·  **Must**
**As an** Event Organizer, **I want** today's registrations with the three newest and a quick add, **so that** I see sign-ups flowing and record walk-ups.
**Acceptance criteria**
- Given confirmed registrations today (Bangkok day), then "Today's Registrations" counts them and lists the 3 newest, newest first: name, event, ticket type and time.
- Given a registration awaiting approval or payment, waitlisted, rejected or cancelled, then it's neither counted nor listed; earlier days never fill the list.
- Given nobody registered today, then the card says so and offers "Copy registration link" for my soonest upcoming event, or "New event" if none.
- Given I may add registrations, then "Add registration" opens the Registrations page's add form (US-REG-03); Staff aren't offered it.
- Given I choose "See all", then the Registrations page opens.
_Notes: Shown only to members who may view registrations: names are personal data._

### US-DASH-03 — Today's meetings  ·  **Won't**
_Removed from Home on 2026-11-04: today's meetings are under **Today** on the Meetings page (US-MTG-01). The id stays so references keep working._
**As an** Event Organizer, **I want** today's meetings on home, **so that** I stay ahead of my day.

### US-DASH-04 — Upcoming events progress  ·  **Should**
**As an** Event Organizer, **I want** my next three events with days left and seats filled, **so that** I catch one falling behind in time.
**Acceptance criteria**
- Given upcoming events, then "Upcoming Events" shows the 3 soonest first, each with days left, seats sold and a fill bar (sold ÷ seats).
- Given an event starts today, then its chip shows Today instead of days left.
- Given nothing is upcoming, then the card says so and offers "New event".
- Given I choose an event's card, then its event page opens; "See all" opens the Events page.
- Given I may view registrations, then each event also shows its newest attendee's initials; others never see them.
_Notes: Upcoming means active (not draft, completed or cancelled) and not yet started._

### US-DASH-05 — Seats sold across active events  ·  **Could**
**As an** Event Organizer, **I want** a ring of seats sold per active event, **so that** I see which are filling and what's left.
**Acceptance criteria**
- Given active events, then their combined seats split into a coloured sold segment per event plus grey seats left; sold segments total Capacity filled (US-DASH-08).
- Given the ring and its legend, then both run largest first, seats left last; over 6 selling events, the top 5 show, the rest combined.
- Given no active event has a seat limit, then the centre still counts the events and a no-limits note replaces the segments, never 0%.
- Given no active events, then the "Active events" card says so.
- Given I choose "See all", then the Events page opens.
_Notes: Active means not draft, completed or cancelled; the centre counts them, and unlimited tickets count toward neither seats sold nor capacity._

### US-DASH-06 — To do: the most urgent operational alerts  ·  **Must**
**As an** Event Organizer, **I want** my three most urgent alerts, linked to their fixes, **so that** I start on what matters and nothing slips.
**Acceptance criteria**
- Given actionable alerts, then "To do" lists the 3 most urgent, one per kind and event, with count, priority, event, oldest date and fix link.
- Given several alerts, then declined payments (**High**) come before registrations to approve and nearly sold-out ticket types (**Medium**), oldest first within each.
- Given an alert I couldn't resolve (payments without finance access; approvals without approval rights, like Staff; sell-outs without ticket-type rights), then I never see it.
- Given I resolve an alert in its module, then it drops off at the next refresh; with none left, I'm told I'm all caught up.
- Given I choose "View all", then the Notifications page opens with every alert I can act on under "Needs action" (US-MSG-03).
_Notes: Nearly sold out follows Tickets Selling Fast (US-DASH-11), dated when it hit 80%; home can't dismiss, edit or re-rank alerts._

### US-DASH-07 — Website template shortcuts  ·  **Won't**
_Removed from Home on 2026-11-04: templates are chosen when an event's page is set up (US-EVT-07, US-PAGE-10). The id stays so references keep working._
**As an** Event Organizer, **I want** shortcuts to the landing-page templates on home, **so that** I can start an event's public page quickly.

### US-DASH-08 — Performance KPIs at a glance  ·  **Must**
**As an** Admin, **I want** five headline numbers for my events, **so that** I judge their health in seconds.
**Acceptance criteria**
- Given the Dashboard, then five cards show Total registrations, Ticket revenue, Upcoming events, Check-in rate and Capacity filled for the range, Year by default.
- Given the change on the previous range, then it reads green up, red down; for Upcoming events and Capacity filled it always reads a dash.
- Given I have finance access, then Ticket revenue shows in ฿ (฿128,500 as ฿128.50k); without it, there's no such card and the other four show.
- Given no data for a figure (check-in rate with no event held), then it shows a dash; with no previous data, its change does too.
- Given no events or registrations yet, then the Dashboard says there's no activity and offers "Create your first event" instead of cards.
_Notes: Revenue is net of VAT and refunds; check-in rate is checked-in ÷ tickets for events that started in the range._

### US-DASH-09 — Revenue trend with time-range toggle  ·  **Must**
**As an** Event Organizer, **I want** a Week, Month or Year revenue chart with total and change, **so that** I see both momentum and trend.
**Acceptance criteria**
- Given I open the Dashboard, then "Revenue Overview" shows Year: the chart, the total in ฿ and its change against the previous period.
- Given I choose Week or Month, then the chart, total and change switch to that range, and the KPI cards and ticket-type split follow it.
- Given any range, then the total equals the Ticket revenue card; with no revenue it reads ฿0 and the chart says so.
- Given I have no finance access, then Revenue Overview is not shown at all.
_Notes: Week, Month and Year are the last 7, 30 and 365 Bangkok days, today included, each against the same span before._

### US-DASH-10 — Registrations by ticket type  ·  **Should**
**As an** Event Organizer, **I want** my registrations split by ticket type, **so that** I see which tiers sell and adjust pricing or promotion.
**Acceptance criteria**
- Given registrations in the range, then "Registrations by Ticket Type" shows a donut, total in the centre, each type's count and share, largest first.
- Given the centre total, then it equals the Total registrations card.
- Given I switch between Week, Month and Year, then the split is recounted for that range.
- Given no registrations in the range, then the panel says there are none yet.

### US-DASH-11 — Tickets selling fast  ·  **Should**
**As an** Event Organizer, **I want** ticket types close to selling out, with seats left, **so that** I add capacity or promote in time.
**Acceptance criteria**
- Given on-sale, seat-limited ticket types at least 80% sold, then "Tickets Selling Fast" lists up to 3, scarcest first, with event and seats left.
- Given seats left, then 3 or fewer show red, 4 to 8 amber, and more plain.
- Given I choose "Manage", then the Tickets page opens.
- Given no ticket type is 80% sold, then the panel says none are running low and offers "Manage tickets".
_Notes: One booking takes up to 8 seats by default, so amber means a single booking could sell it out._

### US-DASH-12 — Recent registrations table  ·  **Must**
**As an** Event Organizer, **I want** the latest registrations with amount and payment status, **so that** I see money and sign-ups flowing at a glance.
**Acceptance criteria**
- Given registrations, then "Recent Registrations" lists the 10 newest first: attendee, event, amount in ฿ (Free if free), Paid/Pending/Refunded/Failed badge, time.
- Given I have no finance access, then every amount shows a dash; attendee, event, status and time still show.
- Given I may not view registrations, then the table lists no one.
- Given I choose "View all", then the Registrations page opens.

### US-DASH-13 — Trustworthy, always-current overview  ·  **Must**
**As an** Event Organizer, **I want** home and the Dashboard current, consistent and showing only what I may see, **so that** I trust the numbers.
**Acceptance criteria**
- Given home is open, then registrations, orders, To do and check-in insight refresh about every 30 seconds, the rest every 5 minutes; reloading refreshes all.
- Given a new confirmed registration, then home's count and the insight's registrations rise together; the Dashboard's total, split and Recent Registrations show it on reload.
- Given one home card fails to load, then only that card says so and offers "Try again".
- Given money or attendee personal data I may not see, then its card or column is dropped, or its value shows a dash, never ฿0.
- Given any date, today, days left or time, then it's Bangkok's; home writes dates, days left and alerts in the workspace's language (English or Thai).
_Notes: Home and the Dashboard never change data; every action is a link into the module that owns it._

### US-DASH-14 — Ticket revenue on Home  ·  **Should**
**As an** Event Organizer, **I want** my net ticket revenue for the last 30 days against the 30 before, **so that** I see sales momentum.
**Acceptance criteria**
- Given finance access, then "Ticket revenue" shows net ฿ across all events for the last 30 days, today included, with daily lines for both periods.
- Given revenue changed, then a pill shows the percentage change, green when up, red when down.
- Given no revenue in the previous 30 days, then the pill shows a dash; with none in either period, the card reads ฿0.
- Given I have no finance access, then the card is gone, with no ฿0, blank or lock, and home lays out without it.
- Given the same 30 days, then the total equals the Dashboard's Ticket revenue for Month (US-DASH-08).
_Notes: Net of VAT and refunds, as on the Dashboard._

### US-DASH-15 — Check-in insight on Home  ·  **Should**
**As an** Event Organizer, **I want** check-in rate, capacity filled or registrations over 12 days, **so that** I spot a trend without opening the Dashboard.
**Acceptance criteria**
- Given any member views home, then the card opens on check-in rate over 12 days: today's value headlines, one candle per day, today rightmost.
- Given a candle, then it spans the prior day's value to its own: green up or level, grey down; hover or screen reader reads it.
- Given I choose a dot, then the card switches to check-in rate, capacity filled or registrations (none counts as 0) without reloading, highlighting that dot.
- Given a day with nothing to measure (no event started; no seat-limited active event), then no candle and, if today, a dash headline, never 0%.
- Given the Dashboard, then today's capacity filled equals its Capacity filled card, each day's check-in rate follows its rule, and today's registrations match Today's Registrations.
_Notes: A day's capacity filled is seats of today's active events sold by that day's end, refunds never counting, over their capacity today._

### US-DASH-16 — Recent orders on Home  ·  **Should**
**As an** Event Organizer, **I want** my three newest orders, **so that** I see money coming in and catch a pending or refunded order.
**Acceptance criteria**
- Given orders, then "Recent orders" shows the 3 newest, newest first: buyer's name and email, event, date, amount in ฿ and status badge.
- Given I choose Amount or Date, then largest or newest leads; choosing the current sort (Date at first) reverses it; sorting adds no orders.
- Given I lack finance access, then there's no Amount column or amount sort, no "View all" to Payments (US-FIN-01); without registration access, no Buyer column.
- Given a failed payment attempt or free registration, then it's no order here; a refunded order keeps its amount under a Refunded badge.
- Given no orders, then the card says there are none yet.
_Notes: Orders are the Payments page's Paid, Pending and Refunded payments, at what the buyer paid, VAT included, unlike net Ticket revenue._

---

<a id="epic-e12"></a>

# Epic E12 — Coordinate Meetings

**Goal (business value):** Give organizers one place to schedule every conversation around an event — speakers, sponsors, venues, vendors, internal teams — with invites and video links sent for them, so events get organized faster and nothing slips.

**Primary users:** Event Organizers; Admins also connect Google Calendar. Every console member, Staff included, can open Meetings; Attendees can't.

**Success measures:**
- Meetings scheduled per active event.
- Share of guests holding a delivered, up-to-date invite.
- Video-call join rate and meeting no-show rate.
- Median time to schedule a meeting.

---

## User stories

### US-MTG-01 — See meetings organized by Today / Upcoming / Past  ·  **Must**
**As an** Event Organizer, **I want** meetings sorted into Today, Upcoming and Past, **so that** I see at once what needs me today.
**Acceptance criteria**
- Given today's meeting, then it's under "Today" with a green Today badge, its time prefixed Today; past ones show a grey Past badge.
- Given the "All", "Today", "Upcoming" and "Past" tabs, then each shows its count and lists meetings earliest first, by date then start time.
- Given a row, then it shows title, type icon, mode badge, time, role · person · event, and place: Meet link, venue or Phone call.
- Given a meeting has no role, then its row shows the type instead; with no event, it shows "All events".
- Given the workspace has no meetings yet, then the page says none are scheduled and offers "Schedule a meeting".

_Notes: Tabs follow the meeting's Bangkok date, wherever the viewer is._

### US-MTG-02 — Find a specific meeting in a long list  ·  **Must**
**As an** Event Organizer, **I want** to search, filter and page through meetings, **so that** I find one conversation fast.
**Acceptance criteria**
- Given I type in the search box, then only meetings whose title, person, role, type or event match remain, and I'm back on page 1.
- Given I filter by type (Venue, Sponsor, Vendor, Speaker, Internal) or event, then it combines with tab and search, and tab counts follow.
- Given nothing matches, then I'm told no meetings match, and "Clear filters" brings the whole list back.
- Given many meetings, then "Prev" and "Next" page through them, 10, 20, 30 or 50 a page, summarized like 11–20 of 23.

_Notes: Search works for Thai and English text alike; a new page size goes back to page 1._

### US-MTG-03 — Schedule a meeting with a speaker, sponsor, venue or vendor  ·  **Must**
**As an** Event Organizer, **I want** to book meetings from one short panel, **so that** I set up calls and walkthroughs without leaving the console.
**Acceptance criteria**
- Given I fill Title, Date, Starts, Ends, Mode, who I'm meeting and Guest email, then "Schedule meeting" confirms it and lists it in its tab.
- Given a new meeting, then Type starts on Internal and Mode empty; Mode must be "Video call (Google Meet)", "In person" or "Phone call".
- Given In person or Phone call, then the card's place reads the event's venue or Phone call; the badge always repeats the mode.
- Given a title under 3 characters, a past date (Bangkok) or an end not after the start, then nothing saves; I'm told what to fix.
- Given I click "Schedule meeting" twice quickly, then only one meeting is created.

_Notes: Their role, Event and Notes are optional; Event "All events" makes a general meeting._

### US-MTG-04 — Have invites, video links and reminders handled automatically  ·  **Must**
**As an** Event Organizer, **I want** invites, Meet links and reminders handled for me, **so that** I never send them by hand.
**Acceptance criteria**
- Given the "Schedule meeting" panel, then a note promises a Google Calendar invite with a 15-minute reminder, plus a Meet link only for Video call.
- Given a connected calendar, then the guest is emailed the invite at Bangkok time; for video, it carries a Meet link, shown on the card.
- Given the invite is still on its way, then the card says it isn't sent yet and updates by itself once it is.
- Given no connected calendar, then the panel warns, the meeting saves, and its card says no invite was sent: Admins may connect, others ask one.
- Given Google is briefly down, then the meeting stays, its card says the invite failed, and "Send it now" or an automatic retry sends it.

_Notes: The 15-minute reminder sits on the workspace calendar's copy; guests' reminders follow their own settings._

### US-MTG-05 — Reschedule or edit a meeting  ·  **Must**
**As an** Event Organizer, **I want** to change a meeting and have the guest's invite follow, **so that** nobody's calendar is out of date.
**Acceptance criteria**
- Given a scheduled meeting, when I choose its pencil, then "Reschedule meeting" opens filled in, mode included, and "Save changes" saves and confirms it.
- Given I move a meeting, then the guest's existing invite moves too (no second invite), keeps its 15-minute reminder, and the guest is notified.
- Given I change Video call to In person, then the Meet link and "Join" go, and the card shows the event's venue.
- Given someone else saved the meeting after I opened it, when I save, then I'm told to reload and try again, and their change stands.
- Given a cancelled meeting, then it has no pencil and cannot be edited.

### US-MTG-06 — Cancel a meeting  ·  **Should**
**As an** Event Organizer, **I want** to cancel a meeting with an optional reason, **so that** nobody turns up to a meeting that's off.
**Acceptance criteria**
- Given I choose "Cancel meeting" in a meeting's panel and confirm, then it's badged red Cancelled and the guest's invite is withdrawn by email.
- Given I give a Reason, then the card shows it after Cancelled, and the guest's cancellation email carries it.
- Given a cancelled meeting, then it leaves Today and Upcoming and their counts, stays in All and, after its date, Past, without "Join" or pencil.
- Given the "Cancel this meeting?" dialog, when I choose "Keep it" or close it, then nothing changes.

_Notes: Cancelling keeps the meeting on record as Cancelled._

### US-MTG-07 — Join a video meeting in one click  ·  **Must**
**As an** Event Organizer, **I want** a "Join" button on today's and upcoming video calls, **so that** I can jump straight into the call.
**Acceptance criteria**
- Given a today or upcoming video call whose Meet link is ready, when I choose "Join", then the call opens in a new tab.
- Given a past or cancelled video call, then there is no "Join".
- Given a video call with no Meet link yet, then there's no "Join", and its card says why and offers a next step (US-MTG-04).

### US-MTG-08 — Connect and manage the workspace Google Calendar  ·  **Should**
**As an** Admin, **I want** to manage the workspace's Google Calendar in Settings → Integrations, **so that** invites go out from one account I control.
**Acceptance criteria**
- Given I choose "Connect Google Calendar" and allow access, then I'm told it's connected, and the card shows Connected with the account and date.
- Given I connect, then meetings saved while disconnected, dated today or later, go out automatically, and the wide-screen Meetings header's connect link becomes Google Calendar.
- Given I confirm "Disconnect", then the card reads Not connected and new meetings get no invite, but sent invites stay; "Keep it connected" changes nothing.
- Given Google stops accepting the connection, then the card and Meetings header say to reconnect, and Admins get a reconnect button or link.
- Given a member without Manage integrations, then Settings hides Integrations; opened directly, the page shows status only, saying only an Admin can change it.

_Notes: Admins hold Manage integrations by default; the server refuses anyone without it._

### US-MTG-09 — Be warned about overlapping meetings  ·  **Could**
**As an** Event Organizer, **I want** a warning when a new meeting overlaps another, **so that** I avoid double-booking but still decide for myself.
**Acceptance criteria**
- Given a new meeting overlaps one already booked, when I save, then I'm warned and asked whether to schedule anyway.
- Given I choose to continue, then the meeting saves normally; the warning never blocks saving.

_Not in the web app yet._


---

<a id="epic-e13"></a>

# Epic E13 — Measure Performance (Reports)

**Goal (business value):** Give organizers and admins one trusted place to see how their events perform (money collected and settled, sign-ups, show-ups, promotion payback) for one event or all, over any period, and export it for finance.

**Primary users:** Event Organizer and Admin (every report); Staff/Team member (Overview, Registrations and Attendance only, no money figures).

**Success measures:**
- Organizers see how events are doing without opening each one.
- Reported revenue reconciles with money that settles to the bank.
- No-show rate, refund rate and promotion payback are tracked period over period.
- Finance gets self-serve exports in the format it needs.

---

## User stories

### US-RPT-01 — Workspace health at a glance  ·  **Should**
**As an** Event Organizer, **I want** headline figures and a revenue trend for any period, **so that** I see my events' health on one screen.
**Acceptance criteria**
- Given I open Insights › Overview, then Revenue, Registrations, Attendance rate, Avg ticket and Refund rate show for "Year", each with its change.
- Given I pick "7 days", "30 days", "90 days" or "Year" (365 days), then every tile and the whole "Revenue trend" recompute for that window.
- Given a change, then it compares with the previous equal period, positive-coloured when better (revenue up, refund rate down), warning-coloured when worse.
- Given the previous period has nothing to compare (฿0 or no data), then the change shows a dash, never a made-up percentage.
- Given no revenue in the window, then the trend total reads ฿0 and a no-revenue message replaces the chart.
_Notes: Revenue is Income's Net; Avg ticket is gross ÷ paid seats; Refund rate is refunds ÷ gross._

### US-RPT-02 — Filter and search every report  ·  **Should**
**As an** Admin, **I want** to narrow any report by search, event and dates, **so that** I see exactly the slice I need.
**Acceptance criteria**
- Given any report except Overview, then it opens on the last 365 days with search, dates, "Apply" and an event picker (status on Event performance).
- Given I change any filter, then the tiles and table update together, never the table alone, and the table goes back to page 1.
- Given an end date before the start, when I choose "Apply", then I'm told the dates are reversed and the report stays as it was.
- Given a span over 24 months, then only the most recent 24 are used, and a note says the range was trimmed.
- Given nothing matches, then the table says so and offers "Clear filters", and the tiles read 0, ฿0 or a dash.
_Notes: Search matches event names; on Transactions the payer or reference, on Payouts the reference or event, on Discounts the code._

### US-RPT-03 — Registration mix and sales-channel breakdown  ·  **Could**
**As an** Event Organizer, **I want** registrations split by ticket type and by top event, **so that** I know what draws people.
**Acceptance criteria**
- Given the Overview's "Registrations by Ticket Type", then each type shows count and share (totalling 100%), largest first; the centre total matches the Registrations tile.
- Given "Registrations by event", then my top 6 events show sign-ups, busiest first; the busiest fills its bar and the rest scale to it.
- Given I switch the range tab, then both panels follow the new window.
- Given nothing sold in the window, then the ticket-type panel says nothing has sold and the events panel says there are no sign-ups.
_Notes: How people found an event isn't recorded, so there is no sales-channel breakdown._

### US-RPT-04 — Rank and explore event performance  ·  **Should**
**As an** Event Organizer, **I want** my events ranked by registrations, with revenue, attendance and status, **so that** I spot my best and worst events.
**Acceptance criteria**
- Given Insights › Event performance, then events run most registrations first, showing Registrations, Revenue ฿, Attendance % and status: Upcoming, Live, Completed or Cancelled.
- Given I pick a status such as Live, then only those events remain, still ranked, from page 1.
- Given an event that hasn't started, then its Attendance % shows a dash, not 0%.
- Given I choose an event's name, then that event's page opens.
- Given the Overview's "Top events" (the top 6), when I choose "View all", then this full "All events" list opens.
_Notes: It lists non-draft events starting in the period; Registrations are confirmed seats, Revenue the event's Net to date._

### US-RPT-05 — Income and reconciliation report  ·  **Should**
**As an** Admin, **I want** income split into gross, refunds, fees and net per event, **so that** I reconcile what was collected with what settles.
**Acceptance criteria**
- Given Insights › Income, then event rows, highest Net first, show Gross, Refunds, Fees and Net, where Net = Gross − Refunds − Fees.
- Given the tiles Gross revenue, Refunds, Processing fees and Net revenue, then Net follows the same rule and equals Overview Revenue for that period.
- Given I export Income, then each row adds VAT (฿6,542 of a VAT-inclusive ฿100,000) and Settled (Net minus VAT), neither shown on screen.
- Given an event took no money in the period, like a free event, then Income skips it while Event performance shows it at ฿0.
- Given nothing matches, then the table says no events match and offers "Clear filters", and the tiles read ฿0.
_Notes: Income counts payments made in the period, VAT included, minus completed refunds only._

### US-RPT-06 — Transaction ledger for investigations  ·  **Could**
**As an** Admin, **I want** a ledger of every payment and refund, **so that** I can investigate a specific charge, refund or dispute.
**Acceptance criteria**
- Given Insights › Transactions, then entries run newest first with Reference, Date, Payer, Event, Method, Amount, Type (Payment, Refund) and Status (Succeeded, Pending, Refunded, Failed).
- Given a ฿1,250 payment is refunded, then a new entry reads -฿1,250 in red, while the payment still reads ฿1,250 with Status Refunded.
- Given the tiles, then I see Transactions (every entry), Payments (total of every charge that didn't fail), Refunds (a count) and Success rate.
- Given a Failed charge, then it is left out of Payments but counts against Success rate: 9 good charges and 1 failure read 90%.
- Given I choose a reference, then Payments opens searched to it, where refunds are issued; this report changes nothing.
_Notes: A refund gets its own reference, like TXN-8842-R1._

### US-RPT-07 — Payouts and settlement visibility  ·  **Should**
**As an** Event Organizer, **I want** to see payouts paid, in transit and pending, **so that** I know what has actually reached our bank.
**Acceptance criteria**
- Given Insights › Payouts, then each payout shows Reference, Date, Bank, Events covered (a name or count), Amount and Status: Paid, In transit or Pending.
- Given the tiles, then I see Paid out, Pending, In transit and Avg payout (over every listed payout, a dash when none).
- Given any row, then the bank shows only its last four digits (•••• 6789), never the full account number.
- Given a payout covering several events, when I pick any one of them, then it still appears at its full amount.
- Given nothing matches, then the table says there are no payouts and offers "Clear filters".
_Notes: Failed payouts aren't listed here; they sit on Finance › Payouts with their retry._

### US-RPT-08 — Registrations report by event and status  ·  **Should**
**As a** Staff/Team member, **I want** each event's registrations split into confirmed, pending, waitlist and cancelled, **so that** I see the true sign-up picture.
**Acceptance criteria**
- Given Insights › Registrations, then each event row's Confirmed, Pending, Waitlist and Cancelled add up to its Registrations.
- Given the tiles Total registrations, Confirmed, Pending and Cancelled, then Total registrations equals the Overview's Registrations for the same period.
- Given I am Staff, then I can open this report in full, as it holds counts per event, not attendee names.
- Given nothing matches, then the table says no registrations match and offers "Clear filters", and the tiles read 0.
_Notes: It counts seats (a booking for 3 counts 3) by registration date, leaving out rejected and expired bookings._

### US-RPT-09 — Attendance and no-show report  ·  **Should**
**As an** Event Organizer, **I want** each event's registered vs checked-in, no-shows and attendance rate, **so that** I can measure show-up and reduce no-shows.
**Acceptance criteria**
- Given an event on Insights › Attendance with 400 Registered (tickets issued) and 320 checked in, then No-shows read 80 and Attendance % 80%.
- Given an event that hasn't started, then its No-shows and Attendance % show a dash, and it's left out of the overall Attendance rate.
- Given the Attendance rate tile, then it matches the Overview's for the same period: a whole number here, one decimal there.
- Given the tiles Checked in, Attendance rate, No-shows and On-time, then On-time is the share of check-ins made before the event began.
- Given nothing matches, then the table says no events match and offers "Clear filters".
_Notes: An event counts in the period it starts in._

### US-RPT-10 — Promotion and discount payback  ·  **Could**
**As an** Event Organizer, **I want** each discount code's redemptions, discount given and revenue influenced, **so that** I can tell which promotions drive sales.
**Acceptance criteria**
- Given Insights › Discounts, then each code shows Type (25% off or ฿200 off), Redemptions, Discount, Revenue, Return and Status: Active, Scheduled, Expired or Disabled.
- Given a date range, then it only decides which codes are listed (those valid in it); figures are lifetime totals from confirmed orders only.
- Given the tiles, then I see Active codes, Redemptions, Discount given and Revenue influenced (order value after discount, not settled cash).
- Given a Scheduled code that hasn't started, then its figures read 0, its Return shows a dash, and Active codes doesn't count it.
- Given an Expired code, then it is badged Expired but its figures still count in the totals.
_Notes: Return is revenue ÷ discount, so ฿90,000 on ฿12,000 reads ×7.5._

### US-RPT-11 — Export the current view  ·  **Should**
**As an** Admin, **I want** to export a filtered report to CSV, Excel or PDF, **so that** I can share the numbers with finance offline.
**Acceptance criteria**
- Given I choose "Export", then the file holds every row matching my filters, not just this page, up to the first 5,000.
- Given Excel, then a Summary sheet holds key figures, filters, generation time (Bangkok) and, if cut, how many rows matched; ฿ and % are formatted.
- Given PDF, then an Eventa-branded document shows key figures, filters, the table, generation time (Bangkok) and, if cut, how many rows matched.
- Given CSV, then Thai text opens correctly and values are plain (2026-07-18, 1250.00), with no summary above the table.
- Given an export is being prepared, then the button reads "Preparing…" and can't be used again until the file arrives.
_Notes: Every report but the Overview has "Export", and a file shows figures as of the moment it was made._

### US-RPT-12 — Role-appropriate access and safe finance sharing  ·  **Should**
**As an** Admin, **I want** money reports limited to finance access and every export logged, **so that** sensitive data reaches only the right people.
**Acceptance criteria**
- Given Staff (no finance access), then Insights offers only Overview, Registrations and Attendance, and opening another report by its address is refused.
- Given Staff on the Overview, then Revenue, Avg ticket and Refund rate show a dash, the trend says it's finance-only, and per-event panels are hidden.
- Given a member without "Export attendee data" exports Transactions, which names payers, then they're refused and no file is saved.
- Given anyone exports a report, then Settings › Security › Audit log records it with the report, format, row count, filters, who and when.
- Given an attendee opens any report, then they're sent to the organizer sign-in and see no figures.
_Notes: Reports are read-only, and for the same period Overview, Registrations and Income agree._


---

