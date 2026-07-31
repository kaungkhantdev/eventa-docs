# Eventa — Functional Requirements (Product Backlog)

| | |
|---|---|
| **Product** | Eventa — Event Registration & Management Platform |
| **Author** | Product Owner |
| **Format** | Product backlog — epics → user stories, with MoSCoW priority & acceptance criteria |
| **Version** | 1.0 |
| **Date** | 2026-07-23 |
| **Scope** | 13 epics · 161 user stories |

> **How to read this.** This is the functional backlog from the **Product Owner's** point of view — what the product must do and *why it matters to the business*, expressed as user stories (`As a <role>, I want <goal>, so that <benefit>`) with business-observable acceptance criteria. Each story carries a **MoSCoW** priority. The technical detail (data model & ERD) lives in [Stage 4 — Architecture](../04-architecture/); quality bars live in [non-functional-requirements.md](non-functional-requirements.md).

## Prioritisation & MVP

The backlog holds **161 user stories**: **89 Must**, **57 Should**, **15 Could**. The **Must** stories define the MVP — the core money path *(discover → register → pay → get a QR ticket → check in)* plus creating and publishing an event, sign-in, and getting paid. Should/Could stories deepen finance, insights, engagement, meetings and workspace administration in later releases.

## Epics

| # | Epic | Stories | Must | Should | Could |
|---|------|:-------:|:----:|:------:|:-----:|
| 1 | [Accounts & Sign-in](#epic-e01) | 12 | 9 | 3 | 0 |
| 2 | [Configure the Workspace & Team](#epic-e02) | 13 | 8 | 4 | 1 |
| 3 | [Create & Manage Events](#epic-e03) | 15 | 11 | 4 | 0 |
| 4 | [Public Event Pages](#epic-e04) | 10 | 6 | 3 | 1 |
| 5 | [Sell Tickets & Run Promotions](#epic-e05) | 12 | 3 | 7 | 2 |
| 6 | [Discover & Register for Events (Attendee)](#epic-e06) | 14 | 10 | 3 | 1 |
| 7 | [Communicate with Attendees](#epic-e07) | 10 | 1 | 6 | 3 |
| 8 | [Manage Registrations & Admit Attendees](#epic-e08) | 14 | 8 | 6 | 0 |
| 9 | [Get Paid & Manage Finances](#epic-e09) | 14 | 11 | 3 | 0 |
| 10 | [Build the Event Program](#epic-e10) | 13 | 9 | 3 | 1 |
| 11 | [Organizer Home & Dashboard](#epic-e11) | 13 | 7 | 4 | 2 |
| 12 | [Coordinate Meetings](#epic-e12) | 9 | 6 | 2 | 1 |
| 13 | [Measure Performance (Reports)](#epic-e13) | 12 | 0 | 9 | 3 |
| | **Total** | **161** | **89** | **57** | **15** |

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
- **E6 Discover & Register is sixth, not first.** Eleven of its fourteen stories stand on data other
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

**Goal (business value):** Give organizers and attendees a fast, trustworthy, bilingual (EN/TH) way to create accounts, sign in, and stay signed in safely — with the two audiences kept cleanly apart — so that access friction never blocks the core money path (discover → register → pay → ticket → check-in) and accounts stay protected from takeover.

**Primary users:** Event Organizer (including the workspace owner/Admin), Staff / Team member, Registered Attendee, Guest Attendee (no account).

**Success measures:**
- Sign-in success rate (and a low share of failed sign-ins).
- Account activation rate — sign-ups that confirm their email and reach first use.
- Self-service password-reset success rate (support tickets avoided).
- Share of organizer accounts with two-factor turned on.
- Account-takeover / fraudulent sign-in incidents kept at or near zero.
- Time-to-first-console-access for a new organizer.

## User stories

### US-ACC-01 — Create an organizer account · **Must**
**As an** Event Organizer, **I want** to create a workspace account with my name, email and password and confirm that I own that email, **so that** I can start setting up events, selling tickets and managing registrations.
**Acceptance criteria**
- Given I complete the sign-up form with a valid email, a strong-enough password, and I accept the Terms & Privacy, when I submit, then my account is created, I'm told to check my inbox, and I am not signed in until I confirm my email.
- Given I open the confirmation link in my email while it is still valid, when it loads, then my account becomes active and I can sign in.
- Given I try to sign up with an email that is already registered, when I submit, then I see the same "check your inbox" confirmation (my existing account is never revealed) and no duplicate account is created.
- Given I have not accepted the Terms & Privacy, when I try to submit, then I am asked to accept them before I can continue.
- Given my password is too weak, when I type it, then I see live guidance on making it stronger and I cannot submit until it meets the minimum.
_(Notes: The first person to register becomes the workspace owner; teammates join an existing workspace by invitation, which is covered by the Access & Team epic. The same password-strength guidance and minimum standard apply wherever a password is set — sign-up, reset and change.)_

### US-ACC-02 — Organizer sign-in · **Must**
**As an** Event Organizer or Staff member, **I want** to sign in with my email and password, **so that** I can reach my organizer console and manage my events.
**Acceptance criteria**
- Given my account is active and email-confirmed, when I enter the correct email and password, then I am signed in and taken to my console home.
- Given I enter a wrong email or password, when I submit, then I see a single neutral message that does not reveal which was wrong or whether the email exists.
- Given my email is not yet confirmed, when I try to sign in, then I am asked to confirm it first and a fresh confirmation email is sent.
- Given my access has been suspended, when I sign in with correct details, then I am refused.
_(Notes: When a sign-in comes from an unrecognized device, the account owner is emailed a "new device" heads-up for reassurance.)_

### US-ACC-03 — Attendee sign-in without forcing accounts · **Must**
**As a** Registered Attendee, **I want** to sign in to the portal, **so that** I can see my tickets and registration history — while still being able to browse events and register without any account.
**Acceptance criteria**
- Given I have a confirmed attendee account, when I sign in with correct details, then I land on my "My tickets / My events" page and never inside the organizer console.
- Given I am a Guest Attendee with no account, when I browse events and register for one, then I can complete registration and receive my ticket without ever creating an account.
- Given I use an organizer-only email at the attendee sign-in, when I submit, then I see the neutral failure and no attendee session is created.

### US-ACC-04 — Reset a forgotten password · **Must**
**As an** Attendee or Organizer who forgot my password, **I want** to request a reset link by email and set a new password, **so that** I can regain access on my own without contacting support.
**Acceptance criteria**
- Given I enter my email on the forgot-password screen, when I submit, then I always see the same neutral "if an account matches, a link is on its way" message — whether or not the email is registered.
- Given my email is registered, when I request a reset, then I receive a single-use link that lets me set a new password.
- Given I open a valid reset link and set a new, strong-enough password, when I confirm, then my password is changed, every device is signed out, and I must sign in again.
- Given my reset link has expired or was already used, when I open it, then I am told it is no longer valid and can request a new one.
- Given I try to set my previous password again, when I submit, then it is rejected.
_(Notes: If two-factor is turned on, the next sign-in still asks for the second step — a reset does not skip it.)_

### US-ACC-05 — Change my password while signed in · **Must**
**As a** signed-in Organizer or Attendee, **I want** to change my password after confirming my current one, **so that** I can keep my account secure without going through email.
**Acceptance criteria**
- Given I enter my correct current password and a new, strong-enough one, when I save, then my password changes and my other devices are signed out while this one stays signed in.
- Given I enter the wrong current password, when I save, then the change is refused and nothing is signed out.
- Given my new password is the same as my current one, when I save, then it is rejected.

_Note: the same capability as **US-SET-02** (E2 — Configure the Workspace & Team), which is its Settings surface. **Build it once, here in E1**; E2 only links to it. Both are Must — do not implement twice._

### US-ACC-06 — Sign in with Google, Apple or LinkedIn · **Should**
**As an** Attendee or Organizer, **I want** to sign in or sign up using an account I already have, **so that** I can get started in one tap without creating another password.
**Acceptance criteria**
- Given I am new and choose a social provider, when it confirms my email, then an account is created for me already marked as confirmed and I am taken to the right home for my audience.
- Given I already have a password account with the same email, when I use a social provider, then it links to my existing account and no duplicate is created.
- Given I cancel the provider's consent screen, when I return, then I am told sign-in was cancelled and I am not signed in.
_(Notes: Available providers differ by audience (organizer: Google/LinkedIn; attendee: Google/Apple). A social sign-in started from an organizer page only ever creates or links an organizer account; from the attendee portal, only an attendee account — never crossing between the two.)_

### US-ACC-07 — Extra security with two-factor sign-in · **Should**
**As a** security-conscious Organizer or Attendee, **I want** to add a second step at sign-in — a code from my authenticator app — **so that** a stolen password alone cannot get into my account.
**Acceptance criteria**
- Given I have turned on two-factor, when I sign in with the right password, then I am asked for a 6-digit code before I am let in.
- Given I enter a valid current code, when I submit, then I am signed in.
- Given I have lost my authenticator, when I enter one of my saved single-use recovery codes, then I am let in and that recovery code can never be used again.
- Given I choose "trust this device," when I sign in again from that device within 30 days, then I am not asked for a code.
_(Notes: Turning two-factor on and off and managing recovery codes lives in the security-settings surface; this story covers only the challenge shown at sign-in time.)_

### US-ACC-08 — Stay signed in on my device · **Must**
**As an** Organizer or Attendee, **I want** to choose to be remembered on my device, **so that** I can return without re-entering my details every time — while shared or public devices do not stay signed in.
**Acceptance criteria**
- Given I sign in without ticking "Remember me," when I close and reopen my browser, then I am signed out and must sign in again.
- Given I sign in with "Remember me," when I come back within the remembered period, then I am still signed in.
- Given I stay away for a long time, when the remembered period lapses, then I am asked to sign in again.

### US-ACC-09 — See and sign out my other devices · **Should**
**As an** Organizer or Attendee, **I want** to see where my account is currently signed in and sign out any device other than the one I am using, **so that** I can cut off access from a lost or shared device.
**Acceptance criteria**
- Given I open my active-sessions list, when it loads, then I see each session's device, rough location and how recently it was active, with my current device clearly marked.
- Given I sign out another device, when I confirm, then that device loses access the next time it is used and disappears from my list.
- Given I view my current device in the list, when I look at it, then it is marked "This device" and offers no sign-out control there (I use normal sign-out for that).

_Note: the same capability as **US-SET-04** (E2 — Configure the Workspace & Team), which is its Settings surface, plus a bulk 'sign out all other sessions' action. **Build it once, here in E1.**_

### US-ACC-10 — Sign out · **Must**
**As a** signed-in Organizer or Attendee, **I want** to sign out, **so that** I can end my session and protect my account on this device.
**Acceptance criteria**
- Given I am signed in, when I choose Sign out, then my session ends and I am returned to the sign-in screen, and protected pages require signing in again.
- Given I am signed in as both an organizer and an attendee in the same browser, when I sign out of one, then the other stays signed in.

### US-ACC-11 — Keep organizer and attendee accounts cleanly separated · **Must**
**As the** business, **I want** organizer accounts and attendee accounts kept completely separate — even for the same email — **so that** each audience only ever reaches its own area and the two can never be confused or crossed.
**Acceptance criteria**
- Given I am signed in as an attendee, when I try to open an organizer console page, then I am sent to the organizer sign-in instead.
- Given I am signed in as an organizer, when I try to open a personal attendee page, then I am sent to the attendee sign-in instead.
- Given the same email is registered as both an organizer and an attendee, when I change or reset the password on one, then the other is unaffected, and I can be signed into both at once in the same browser.
- Given I am not signed in, when I try to open a protected page, then I am taken to the right sign-in and returned to where I was headed once I sign in.
_(Notes: Event browsing and event registration stay public and never require an account. Detailed team roles and what each organizer teammate may do are covered by the Access & Team / Settings epic; this epic covers only audience-level separation.)_

### US-ACC-12 — Protect accounts from password-guessing · **Must**
**As an** Attendee or Organizer, **I want** repeated wrong sign-in attempts to be slowed and temporarily blocked, **so that** no one can guess their way into my account.
**Acceptance criteria**
- Given several wrong password attempts in a row, when the limit is reached, then further attempts are refused for a cool-off period and I am told to try again later or reset my password.
- Given a lockout is in force, when I enter the correct password after the cool-off, then I can sign in again.
- Given the account does not exist, when the attempt is blocked, then the message is identical to any other failure and never reveals whether the account exists.


---

<a id="epic-e02"></a>

# Epic E2 — Configure the Workspace & Team

**Goal (business value):** Give every organizer a single place to set up who they are, keep their account secure, present a correct legal and branded identity on tax documents, turn on the ways attendees can pay, and bring the right teammates in with exactly the access their job needs — so the organization can start selling tickets, get paid, and run events safely without waiting on support.

**Primary users:** Admin (owner of workspace, payments, team and roles), Event Organizer, Staff / Team member (self-service on their own account), and indirectly the Attendee (who sees correct receipts, branding and payment options).

**Success measures:**
- Time from account creation to "ready to sell" (organization details complete + a payment method live).
- Share of workspaces that successfully accept a first real payment.
- Team-onboarding speed: time from inviting a teammate to them completing their first task (e.g. a check-in).
- Account-security adoption (share of Admins with two-factor turned on) and zero unauthorized-access incidents.
- Invoice / receipt accuracy (correct seller name, Thai tax ID, VAT 7%) — measured by billing disputes and reissued documents.

## User stories

### US-SET-01 — My profile & preferences  ·  **Must**
**As a** team member, **I want** to keep my name, contact details, timezone, language and photo up to date, **so that** colleagues can reach me and the console shows dates and copy in the way I read them.
**Acceptance criteria**
- Given I open my profile, when the page loads, then it shows my current name, email, phone, timezone, language and photo, ready to edit.
- Given I edit my name or phone, when I save, then the change is kept and I see a confirmation.
- Given I change my email, when I save, then my current sign-in email keeps working, my email shows as "unverified", and a confirmation link is sent to the new address so I can prove it's mine.
- Given I edit fields but change my mind, when I choose Cancel, then everything returns to its last-saved values.
- Given I set my timezone to Bangkok and language to Thai, when I view any date or menu, then times and copy appear in that timezone and language.
_Notes: Profile photo (upload/remove, falls back to my initials) is a nice-to-have refinement within this story and can trail the identity fields if needed. A member edits only their own profile here._

### US-SET-02 — Change my password  ·  **Must**
**As a** team member, **I want** to change my own password by confirming my current one, **so that** I can keep my account secure if my password is ever guessed or shared.
**Acceptance criteria**
- Given I enter my current password correctly and a strong new one (at least 8 characters with a number and a symbol), when I submit, then my password is updated and I see a confirmation.
- Given my new password is weak or doesn't match its confirmation, when I submit, then I'm told why and nothing changes.
- Given I enter the wrong current password, when I submit, then the change is refused and my password stays as it was.
- Given my password changes successfully, when it's done, then I'm signed out everywhere else and receive an email letting me know my password was changed.

_Note: the Settings surface for **US-ACC-05** (E1 — Accounts & Sign-in), which owns the capability. Built in E1; this story is the entry point, not a second implementation._

### US-SET-03 — Two-factor authentication & recovery codes  ·  **Should**
**As a** security-conscious member, **I want** to protect my account with an authenticator app and keep backup recovery codes, **so that** a stolen password alone can't get anyone into my account.
**Acceptance criteria**
- Given I start setup, when I scan the code with my authenticator app and enter the 6-digit code it shows, then two-factor turns on and I'm given 8 one-time recovery codes to save.
- Given I enter a wrong or expired code, when I try to finish setup, then two-factor stays off and I'm asked to try again.
- Given two-factor is on, when I later choose to turn it off, then I must first re-confirm it's me, and I'm emailed that two-factor was disabled.
- Given I run low on recovery codes, when I regenerate them, then I receive a fresh set of 8 and the old codes stop working.
_Notes: If the workspace requires two-factor for everyone, an individual member can't switch it off. Recovery codes are shown once and can be downloaded._

### US-SET-04 — Review & sign out my active sessions  ·  **Should**
**As a** team member, **I want** to see everywhere my account is signed in and sign a lost or unfamiliar device out, **so that** I stay in control of where my account is active.
**Acceptance criteria**
- Given I open my active sessions, when the list loads, then I see each device with its rough location and how recently it was used, and my current device is clearly marked.
- Given a device I don't recognize, when I sign it out, then that device is logged out on its next attempt and the entry disappears.
- Given my current device, when I view the list, then it has no "sign out" control (I sign out normally instead).
- Given I choose "sign out all other sessions", when I confirm, then every device except the one I'm using is signed out.

_Note: the Settings surface for **US-ACC-09** (E1 — Accounts & Sign-in), which owns the capability; this story adds only the bulk 'sign out all other sessions' action._

### US-SET-05 — Security & access audit log  ·  **Should**
**As an** Admin, **I want** a tamper-proof, time-ordered record of sign-ins, role changes, exports and key security events that I can download, **so that** sensitive actions are traceable for compliance and investigations.
**Acceptance criteria**
- Given a teammate's role was changed, when I open the audit log, then I see an entry showing who changed it, from which role to which, and when.
- Given I open the log, when it loads, then entries appear newest-first and cannot be edited or deleted from anywhere in the product.
- Given I export the log for a date range, when the download completes, then I get a file for that range and the export itself is recorded as a new audit entry.
- Given an entry involves a secret (like a saved payment key), when I view it, then only a masked hint is shown, never the full value.
_Notes: A member always sees their own security events; workspace-wide events (others' role changes, exports, payment-key changes) are visible to Admins._

### US-SET-06 — Notification preferences  ·  **Should**
**As a** team member, **I want** to choose per topic whether I'm alerted by email and/or SMS, **so that** I hear about what matters to me without noise.
**Acceptance criteria**
- Given the topics registrations, payments, reminders and product updates, when I view preferences, then I can switch email and SMS on or off for each independently.
- Given I turn a channel off for a topic, when a matching event happens, then I'm not alerted on that channel for that topic.
- Given I have no phone number on file, when I view preferences, then the SMS switches are unavailable with a prompt to add a phone.
- Given I've turned off payment alerts, when a payment succeeds, then I still receive the receipt and other required legal/transactional messages, which always send.

### US-SET-07 — Organization profile, tax details & branding  ·  **Must**
**As an** Admin, **I want** to maintain our legal organization name, address, website, currency, Thai tax ID and logo, **so that** every invoice, receipt and public event page shows the correct, VAT-compliant seller identity.
**Acceptance criteria**
- Given I fill in a valid organization name, address and 13-digit Thai tax ID, when I save, then the details are kept and appear on documents issued from then on.
- Given a document was already issued, when I later change the organization details, then that past document is unchanged and only future documents use the new details.
- Given a valid tax ID is on file, when a receipt is issued, then VAT 7% is itemized on it.
- Given an invalid tax ID or website, when I save, then I'm told what's wrong and nothing is saved.
- Given I'm an Organizer or Staff, when I open this page, then it's read-only or hidden and I can't change organization settings.
_Notes: Uploading/removing the organization logo (shown on branded surfaces, applying to future documents only) is included here and is a lower-priority refinement than the legal details._

### US-SET-08 — Connect our payment account  ·  **Must**
**As an** Admin, **I want** to connect our payment account in test mode first and then go live, and disconnect it if needed, **so that** we can safely prove payments work before accepting real money from attendees.
**Acceptance criteria**
- Given I connect a valid live payment account, when I save, then the workspace shows "Connected" and we can accept real payments.
- Given I'm still in test mode, when I view the payments page, then a banner reminds me no real charges happen until we go live.
- Given saved payment credentials, when I choose "Test connection", then I'm told whether they work — without any money moving — or given a clear reason they failed.
- Given payments are connected, when I disconnect and confirm, then paid checkout is switched off, free events keep working, and past orders and payouts are untouched.
_Notes: Sensitive saved payment details are never shown back in full after saving and never exposed to attendees._

### US-SET-09 — Choose payment methods at checkout  ·  **Must**
**As an** Admin, **I want** to turn each payment method (cards, PromptPay, Apple Pay, Google Pay, bank transfer) on or off, **so that** attendees are offered exactly the ways to pay we support.
**Acceptance criteria**
- Given PromptPay is enabled, when an attendee reaches checkout, then PromptPay is offered as a way to pay.
- Given I disable a method, when new orders are placed, then that method no longer appears at checkout.
- Given we still sell paid tickets, when I try to switch off the last remaining method, then I'm blocked and asked to keep at least one enabled.
- Given a wallet method needs extra setup first, when its prerequisites aren't met, then the switch stays off with guidance on what to complete.

### US-SET-10 — Checkout & receipt preferences  ·  **Must**
**As an** Admin, **I want** to set our default charge currency, the short label attendees see on their card statement, and whether we save cards and email receipts, **so that** attendees recognize the charge and get a proper receipt every time.
**Acceptance criteria**
- Given a statement label of 22 characters or fewer, when I save, then it's kept and appears on attendees' card statements for later charges.
- Given a label that's too long or uses disallowed characters, when I save, then I'm asked to shorten or fix it before it's accepted.
- Given "email receipts" is on, when a payment succeeds, then the attendee receives a receipt with VAT 7% itemized.
- Given my default currency differs from the organization currency, when I save, then I'm warned but not blocked.

### US-SET-11 — Invite & manage teammates  ·  **Must**
**As an** Admin, **I want** to find, invite, suspend, reactivate and remove teammates, **so that** the right people can work in our workspace and access ends the moment someone should no longer have it.
**Acceptance criteria**
- Given the team list, when I search or filter by status (all, active, invited, suspended) or role, then I see matching members with their role, status and last-active, and the counts update.
- Given a new person's name, email and role, when I send an invite, then they appear as "Invited" and receive a join link by email.
- Given an email that's already invited, when I invite it again, then the invitation is simply re-sent rather than duplicated; an already-active member is rejected as already in the workspace.
- Given an active member who should pause access, when I suspend them, then they can no longer sign in and are signed out, while their role is preserved so reactivating restores it.
- Given a member who should leave, when I confirm removal, then their access ends immediately, and their past work (events, exports) is kept for the record; they can be re-invited later.
- Given the last remaining Admin, when I try to remove, suspend or demote them (or myself), then the action is blocked so the workspace is never left without an Admin.

### US-SET-12 — Assign roles & fine-tune permissions  ·  **Must**
**As an** Admin, **I want** to give each member one of the standard roles (Admin, Organizer, Staff, Attendee) and fine-tune what they can do across events, registrations, finance and settings, **so that** everyone can do exactly their job and nothing more.
**Acceptance criteria**
- Given a Staff member, when I change them to Organizer, then they gain the Organizer capabilities and the change is recorded with before/after and who made it.
- Given a role change, when the member next uses the console, then their access reflects the new permissions immediately, without them signing out and in.
- Given the standard roles, when I review them, then Admin can do everything, Organizer can run events but not issue refunds or manage users/settings, Staff can only view registrations and check attendees in, and Attendee has no console access.
- Given I try to grant myself more access than I currently have, when I save, then it's blocked.
- Given sensitive capabilities (like exporting attendee data or issuing refunds) are granted, when saved, then the grant is clearly flagged in the record.

### US-SET-13 — Create custom roles  ·  **Could**
**As an** Admin, **I want** to see all our roles at a glance and create a custom role by picking exactly which capabilities it has, **so that** I can match access to real-world jobs (e.g. "Volunteer") that the standard roles don't fit.
**Acceptance criteria**
- Given the roles overview, when it loads, then I see each role with its member count, description and headline capabilities, and I can search them.
- Given a unique role name and a chosen set of capabilities, when I create the role, then it appears as a new role I can assign to teammates.
- Given a name that matches an existing role, when I try to create it, then it's rejected and I'm asked for a unique name.
- Given I edit any role's capabilities, when I save, then everyone holding that role picks up the change on their next use, and the workspace is never left without anyone able to manage users and roles.


---
<a id="epic-e03"></a>

# Epic E3 — Create & Manage Events

**Goal (business value):** Give event organizers a fast, guided way to create, publish, and run events end to end — from first draft to a live public page that sells tickets — so that a great-looking, correctly-priced event can go on sale in minutes and be managed safely afterwards without ever losing money or attendee trust to accidental changes.

**Primary users:** Event Organizer (author), Admin (publish, cancel, refunds, categories), Staff/Team member (read-only registration context), Attendee (consumes the public page produced here).

**Success measures:**
- Time to create and publish a first event (target: minutes, not hours).
- Draft-to-published conversion (share of started events that go live).
- Registration conversion on the published landing page.
- Revenue collected per published event; correct VAT-inclusive pricing.
- Zero accidental destruction of events that hold paid registrations (data-safety incidents = 0).
- Share-driven registrations (traffic/registrations attributable to the share/flyer tools).

---

## User stories

### US-EVT-01 — Find and manage my events  ·  **Must**
**As an** Event Organizer, **I want** to browse all my events in one searchable, filterable, sortable list split into Active and Completed, **so that** I can quickly find any event and act on it.
**Acceptance criteria**
- Given I have many events, when I open my events list, then I see the Active events first, newest activity surfaced, with a clear "Showing X–Y of N events" summary and page controls.
- Given I search by name, pick an event type, change the sort (by registrations, name, or date), or switch between Active and Completed, then the list narrows to matching events and returns me to the first page of results.
- Given a filter matches nothing, when the list refreshes, then I see a friendly "No events match" message rather than a blank screen.
- Given each event row, when it renders, then I see its registrations-vs-capacity progress and a percentage, so I can judge how well it is filling.
- Given I open a row's action menu, then I can View details, Edit, Duplicate, or Delete that event.
- Given I delete or duplicate an event, when the list updates, then the Active and Completed counts immediately reflect the change without a page reload.
_(Notes: This is an organizer/admin surface; Staff and Attendees do not see it. A team member without event-management access is told they don't have permission.)_

### US-EVT-02 — Create an event with a guided wizard and save drafts  ·  **Must**
**As an** Event Organizer, **I want** to build an event through a step-by-step wizard (Basics, Date & location, Seating, Tickets, Review & publish) and save a draft at any point, **so that** I capture everything needed to go live and never lose work if I stop partway.
**Acceptance criteria**
- Given I start a new event, when the wizard opens, then I see a progress indicator and a step rail, plus a live summary of what I've entered so far (name, ticket types, capacity, seating).
- Given I'm partway through, when I choose "Save as draft," then the event is saved as a Draft and appears in my events list so I can resume later.
- Given I move between steps or pause, then my work is preserved automatically so an interrupted session is recoverable.
- Given I try to leave with unsaved changes, then I'm asked whether to save the event as a draft before leaving.
- Given a teammate changed the same event since I opened it, when I save, then I'm told it changed elsewhere and prompted to reload the latest version, so no one's edits are silently overwritten.
- Given I edit an already-published event, when I save valid changes, then they apply without taking the event offline.

### US-EVT-03 — Describe the event (Basics)  ·  **Must**
**As an** Event Organizer, **I want** to enter the event's title, a short rich description, category, tags, cover image, and highlight badges, **so that** the event has a clear, attractive identity on its public page.
**Acceptance criteria**
- Given the Basics step, when I enter a title and a formatted description, then I see a live character count and the description is capped at 250 characters; pasting more keeps only the first 250 and shows the counter in red.
- Given I pick a category from my workspace's categories and add tags, then they are saved and later help attendees discover the event.
- Given I upload a cover image that is the wrong type or larger than 5 MB, then it is rejected with a clear message and nothing is stored.
- Given I add highlight badges (icon + short label), then they appear on the public page in the order I entered them, and empty ones are dropped on save.
- Given I leave the title blank, when I try to publish, then publishing is blocked with "Add an event title before publishing."

### US-EVT-04 — Set the date, time and location  ·  **Must**
**As an** Event Organizer, **I want** to set the start/end date-time and timezone and choose whether the event is in-person or online, **so that** attendees know exactly when and where to show up (or how to join).
**Acceptance criteria**
- Given I set a start and end, when the end is not after the start, then I'm blocked with "End time must be after the start time."
- Given I choose In-person, then I must provide a venue name and address before publishing.
- Given I choose Online, then venue and address are not required, and I provide a meeting link that stays private — it is sent only to people after they register, never shown publicly.
- Given all dates, when they display anywhere in the product, then they are shown in Bangkok time.
_(Notes: Switching to Online removes the seating requirement; switching back to In-person re-enables seating.)_

### US-EVT-05 — Choose seating (general admission or reserved)  ·  **Must**
**As an** Event Organizer, **I want** in-person events to use either general admission or a reserved seat map I lay out by rows and seats-per-row, **so that** I can sell the right kind of ticket and, for reserved events, hold a specific seat for each attendee.
**Acceptance criteria**
- Given an in-person event, when I choose Reserved and set rows and seats-per-row, then I see a live seat-map preview and a total-seat count (rows × seats/row).
- Given I choose General admission, then I simply set a headcount with no specific seat assignment.
- Given an online event, when I view the Seating step, then no seating options are offered and I'm told a join link is emailed after registration.
- Given my seat map holds fewer seats than the total ticket quantities I've set, when I go to publish, then I'm warned that the seat map is smaller than my ticket quantities.
- Given a published reserved event, when I reduce rows or seats, then seats already sold are never removed.
_(Notes: The per-booking seat limit for attendees is enforced during registration, not here — see the Attendee epic.)_

### US-EVT-06 — Set up tickets, capacity and registration rules  ·  **Must**
**As an** Event Organizer, **I want** to define ticket types (name, price, quantity), overall capacity, the registration open/close window, and approval and waitlist options, **so that** the right tickets go on sale at the right time and price.
**Acceptance criteria**
- Given I add ticket rows, then each has a name (unique within the event), a price in ฿ (0 allowed for free tickets), and a quantity; at least one ticket type must exist to publish and the last row can't be removed.
- Given a paid ticket priced at ฿890, when it's saved, then it is treated as VAT-inclusive at 7% (finance records ฿831.78 net + ฿58.22 VAT), so pricing is correct for receipts and reporting.
- Given a published event with tickets already sold, when I try to set a ticket quantity or event capacity below what's already sold/registered, then the change is rejected with a message showing the already-committed count.
- Given I set a registration window, when the current time is outside it, then public registration is closed even while the event is published; the window is judged in Bangkok time.
- Given I turn on Require approval, then each registration is held for my review before it's confirmed.
- Given I turn on the waitlist, then attendees can join a waitlist once tickets run out; with it off, the event simply shows "sold out."

### US-EVT-07 — Pick a public page template, set visibility, and publish  ·  **Must**
**As an** Event Organizer, **I want** to choose a landing-page template, set who can see the event, review a readiness checklist, preview the page with my real content, and publish, **so that** the public page looks right and only goes live when it's complete.
**Acceptance criteria**
- Given the template gallery (Classic, Spotlight, Minimal, Vibrant), when I select one, then my event's own title, date, venue, agenda, speakers, highlights and tickets fill the page automatically — I never author page content separately.
- Given any template, when I choose Preview, then the public page opens in a new tab reflecting the details I've entered, including unsaved ones.
- Given I set visibility, then Public lists the event on the public Discover page, Unlisted is reachable only by direct link, and Private is admin-only.
- Given my event is missing a required item (title, description, date/time, location, or at least one ticket type), when I click Publish, then publishing is blocked and the missing items are flagged; a missing cover image is only a warning, not a blocker.
- Given a complete event, when I publish, then it goes live at its public link, becomes discoverable if Public, and my team gets an "Event published" notice.
- Given the event's start date is already in the past, when I publish, then I'm asked to confirm before it goes live.
_(Notes: Publishing requires publish rights; an organizer without them sees Publish disabled and is told they don't have permission.)_

### US-EVT-08 — Cancel or delete an event without losing paid registrations  ·  **Must**
**As an** Admin, **I want** deletion to always require confirmation and to be blocked for any event that holds registrations or payment history — routing me to Cancel (which refunds and notifies) instead — **so that** I can never accidentally destroy an event people have paid for.
**Acceptance criteria**
- Given a draft with zero registrations and no payment history, when I confirm Delete, then the event is permanently removed.
- Given a published event or any event with registrations or payment history, when I choose Delete, then hard deletion is blocked and I'm offered Cancel: "This event has registrations and can't be deleted. Cancel it instead to refund and notify attendees."
- Given I cancel such an event, then I must enter a reason and confirm; the event stops accepting registrations, the waitlist is voided, issued tickets are invalidated, refunds are queued for paid attendees, and affected attendees receive a cancellation message in their language.
- Given a cancel completes but some refunds are still processing, then the cancellation still succeeds and I'm told refunds are still being handled.
- Given the event is cancelled, when I return to the list, then it moves out of the Active bucket but is retained for reporting.
_(Notes: Cancelling requires publish rights; posting refunds is an Admin finance action. No confirmation-free deletion is ever allowed.)_

### US-EVT-09 — Manage the event program (agenda and speakers)  ·  **Must**
**As an** Event Organizer, **I want** to add and remove agenda sessions per day and manage the speaker line-up directly from the event workspace, **so that** I can keep the published program accurate as plans change.
**Acceptance criteria**
- Given the Agenda tab, when I add a session (title, day, start time, duration, type, speaker, room), then it appears on the chosen day sorted by start time; a title is required.
- Given two sessions in the same room overlap in time, when I add one, then I'm warned but not blocked, so multi-track events across rooms still work.
- Given I remove a session, then it disappears immediately and the public page's agenda updates.
- Given the Speakers tab, when I add a speaker (name, and optionally role, talk, tag, photo), then they appear as a card and show on the public page.
_(Notes: Agenda editing is the core (Must) capability; speaker management is a lighter, supporting part of the same program workspace.)_

### US-EVT-10 — Adjust ticket inventory after publishing  ·  **Must**
**As an** Event Organizer, **I want** to add new ticket types and remove ones with no sales directly from the event workspace, **so that** I can respond to demand without rebuilding the event.
**Acceptance criteria**
- Given the Tickets tab, when I add a ticket type (name, price, quantity, optional description, optional "Popular" flag), then it appears immediately starting at 0 sold and ฿0 revenue, and the public ticket list and capacity update.
- Given I add a ticket whose name already exists on the event, then it is rejected as a duplicate.
- Given a ticket type that has sales, when I try to remove it, then removal is blocked with "This ticket type has sales and can't be removed. Close it instead."
- Given a valid new ticket at ฿1,250 × 200, when I save, then it shows 0/200 sold and ฿0 revenue.

### US-EVT-11 — Organize events with categories  ·  **Must**
**As an** Admin, **I want** to create, edit, and safely delete event categories (name, description, icon, colour), **so that** events are grouped consistently for attendee discovery.
**Acceptance criteria**
- Given the categories view, when I browse, then I can search and sort them and each shows its icon, colour, description, and live count of events using it.
- Given I create or edit a category, then I see a live preview of its icon, colour, and name; the name must be unique in my workspace and duplicates are rejected.
- Given I edit a category's name, icon, or colour, then the change is reflected on every event that uses it.
- Given a category with no events, when I confirm Delete, then it is removed.
- Given a category still used by events, when I try to delete it, then I'm blocked until I move those events to another category (or clear their category): "This category has 12 events. Move them to another category before deleting."

### US-EVT-12 — See my events on a calendar and what's coming up  ·  **Should**
**As an** Event Organizer, **I want** a month calendar of my scheduled events plus a soonest-first view of upcoming events, **so that** I can see my schedule at a glance and jump straight to any event.
**Acceptance criteria**
- Given the calendar, when I open it, then each event appears on its start date, the header reads "{n} events this month," and I can move between months or jump to Today.
- Given a busy day, when its cell renders, then up to three events show with a "+N more" roll-up, and picking a day lists that day's events; an empty day reads "No events scheduled."
- Given the upcoming view, when it loads, then future events appear soonest-first with correct "days left" (in Bangkok time) and how full each is.
- Given I click an event on the calendar or upcoming view, then I go straight to that event's detail workspace.

### US-EVT-13 — Duplicate an event to relaunch quickly  ·  **Should**
**As an** Event Organizer, **I want** to duplicate an existing event as a fresh draft, **so that** I can relaunch a recurring or similar event without re-entering everything.
**Acceptance criteria**
- Given an event with 312 registrations, when I duplicate it, then a new draft named "…(Copy)" is created with all its content (details, schedule, seating, tickets, agenda, speakers, template, visibility) but with 0 registrations, 0 sold, ฿0 revenue, empty waitlist, and no attendees.
- Given the duplicate, when it's created, then it gets its own unique public link and any past registration open/close dates are cleared so I re-set the sales window.
- Given the new draft, when the list refreshes, then it appears next to the original and the Active/Completed counts update.
- Given I accidentally trigger the duplicate twice, then only one copy is created.

### US-EVT-14 — Monitor an event's performance from its workspace  ·  **Should**
**As an** Event Organizer, **I want** a single event workspace showing live headline numbers, its registrations, and its confirmed attendees, **so that** I can track how the event is doing and reach attendees.
**Acceptance criteria**
- Given the Overview, when it loads, then I see live registrations-vs-capacity with a percentage, revenue, tickets sold, and days left, plus a "Copy" button that copies the event's public link.
- Given I lack finance access, when Overview loads, then registration and attendance numbers still show but revenue is hidden.
- Given the Registrations tab, when I filter by status (all, Paid, Pending, Refunded) or page through, then the list and its counts update and show attendee, ticket, amount (in ฿), and when they registered.
- Given the Attendees tab, when it loads, then confirmed attendees are listed with a count badge, and "Email all" opens a message to this event's attendees that requires my explicit confirmation before sending.
_(Notes: Attendee names/emails are shown only to members with registration access; broadcast messaging is governed by the Engagement epic.)_

### US-EVT-15 — Promote an event with share links and a printable flyer  ·  **Should**
**As an** Event Organizer, **I want** to share the event to social channels and email and download a printable flyer with a scannable QR code, **so that** I can drive registrations across channels.
**Acceptance criteria**
- Given the Share dialog, when I click "Copy link," then the event's public link is copied and the button confirms "Copied!"
- Given the Share dialog, when I pick Facebook, X, LINE, WhatsApp, or Email, then a pre-filled share message ("Join me at {title} — {date} at {location}") opens for that channel.
- Given the flyer preview, when I save it, then a poster image downloads that includes a scannable QR code leading to the event's registration page.
- Given the event isn't public, when I share it, then I'm warned the link may not be publicly reachable.


---

<a id="epic-e04"></a>

# Epic E4 — Public Event Pages

**Goal (business value):** Give every event a fast, beautiful, shareable public page that turns a click from a link, a search result, or the Discover listing into a completed registration — without forcing anyone to create an account first. The same page doubles as the organizer's design-and-publish surface, so launching an event is self-service and the shared link never breaks.

**Primary users:** Attendee (guest or registered), Event Organizer, Admin, Staff/Team member.

**Success measures:**
- Registration conversion: share of page visitors who click Register/Get tickets and go on to complete a booking.
- Reach quality: share-to-visit rate and click-through from search/social (rich cards rendering correctly).
- Mobile experience: page loads fast on a phone (target under ~2.5s on 4G); low bounce.
- Time-to-publish: organizer can go from event details to a live public page in one sitting, self-service.
- Preview usage: template previews opened per new event (organizers choosing confidently before publishing).

---

## User stories

### US-PAGE-01 — Open a shareable event page  ·  **Must**
**As a** guest Attendee, **I want** to open an event's page from a shared link and immediately see what it is, when it happens and where, **so that** I can decide to register without creating an account.

**Acceptance criteria**
- Given a published event, when I open its shared link, then I see the event's title, date, time, location and tickets with no prompt to log in or sign up.
- Given a link that points to no live event (or an event that isn't published yet), when I open it, then I see a clear "this event page isn't available" message rather than the wrong event or a broken screen.
- Given a section has no content (no highlights, agenda, speakers, tickets or FAQs), when the page renders, then that section is hidden entirely — never an empty heading.
- Given the event runs in Thai or English, when the page loads, then all text and all dates/times read correctly in that language and in Bangkok time.

_Notes: The public page is anonymous and read-only — it starts registration but never books, holds seats or exposes attendee data._

### US-PAGE-02 — Event identity and one-tap Register  ·  **Must**
**As a** guest Attendee, **I want** the top of the page to show the event identity and key facts with an obvious Register / Get tickets button always in reach, **so that** I can start booking the moment I'm convinced.

**Acceptance criteria**
- Given I'm on the page, when I scroll, then a Register / Get tickets button is available at the top, in the opening hero, on each ticket, and at the foot of the page.
- Given I click any Register button, when it responds, then I land on the registration step for this event in the same tab.
- Given registration isn't open yet for the event, when the page renders, then the Register button is visibly unavailable with a short "registration isn't open yet" hint instead of leading to a dead end.
- Given the event's cover image is missing or fails to load, when the hero renders, then a branded background shows and I never see a broken-image icon.

### US-PAGE-03 — Clear about section and online-event handling  ·  **Must**
**As an** Attendee considering an online event, **I want** the page to state plainly that it's online and how I'll get the join link, **so that** I know I don't need to travel and what to expect after I register.

**Acceptance criteria**
- Given an online event, when I read the details, then it shows "Online event" and a note that the join link is sent after I register — and never shows a physical address.
- Given an in-person event, when I read the details, then I see the venue and, when provided, the address, date, time, category and price.
- Given any online event, when the page renders, then the private join link is never shown publicly — only the promise of when I'll receive it.
- Given a detail (such as address) is blank, when the section renders, then that single row is simply left out and the rest still shows.

### US-PAGE-04 — Highlights, agenda and speakers  ·  **Should**
**As a** guest Attendee, **I want** to see the event's highlights, schedule and line-up of speakers, **so that** I can judge whether it's worth my time and money.

**Acceptance criteria**
- Given the organizer added highlights, agenda items or speakers, when I scroll, then each appears in the order the organizer arranged them.
- Given the schedule and speaker sections have custom titles, when they render, then those titles are used (with sensible defaults otherwise).
- Given any of these lists is empty, when the page renders, then that whole section is absent rather than shown blank.

### US-PAGE-05 — Ticket tiers, pricing and availability  ·  **Must**
**As a** guest Attendee, **I want** each ticket option shown with its all-in price, what's included, and whether it's still available, **so that** I can pick the right ticket with no surprises at checkout.

**Acceptance criteria**
- Given the event has ticket tiers, when the Tickets section renders, then each tier shows its name, price, any "what's included" list and its own Register button.
- Given prices are shown, when I read them, then every Baht price already includes the 7% VAT, and free or RSVP tiers read as "Free" or "RSVP" with no price prefix.
- Given one tier is marked as the recommended option, when the section renders, then it stands out and shows its badge (e.g. "Most popular").
- Given a tier is sold out, when I view it, then its button reads "Sold out" and cannot be clicked through to registration.
- Given very few seats remain, when the tier renders, then an urgency line (e.g. "Going fast — only N left") is shown.
- Given registration for the event has closed, when the tickets render, then all tier buttons are disabled with a "registration is closed" message.

_Notes: Choosing quantity happens later in registration — the page just carries the chosen tier through._

### US-PAGE-06 — Frequently asked questions  ·  **Could**
**As a** guest Attendee, **I want** an FAQ I can expand, **so that** I can clear up doubts about the event before committing to register.

**Acceptance criteria**
- Given the event has FAQs, when the page loads, then the first answer is open and the rest are collapsed, and I can expand any of them.
- Given the event has no FAQs, when the page renders, then the FAQ section is absent.

### US-PAGE-07 — Add the event to my calendar  ·  **Should**
**As a** guest Attendee on my phone, **I want** to add the event to my calendar in one tap, **so that** I don't forget to attend after I've decided to go.

**Acceptance criteria**
- Given the event has a clear start and end time, when I tap Add to calendar, then I can choose Apple, Google or Outlook and get an entry set to Bangkok time with the event title and location.
- Given an online event, when I add it to my calendar, then the entry contains no private join link.
- Given the event's date or time is missing or unclear, when the page renders, then the Add to calendar option is simply hidden rather than producing a broken entry.
- Given I add the same event again later, when it lands in my calendar, then it updates the existing entry rather than creating a duplicate.

### US-PAGE-08 — Rich search results and social sharing  ·  **Should**
**As an** Event Organizer (marketing owner), **I want** the page to produce an accurate, rich preview card wherever it's shared or searched, plus easy share buttons, **so that** shares and search results attract clicks instead of looking broken.

**Acceptance criteria**
- Given a published event, when its link is shared to social or picked up by search, then a rich card shows the correct title, description, image (or a branded default) and event details including Baht pricing and Bangkok time.
- Given the page offers sharing, when I use it, then I can copy the link or share to channels like Facebook, LINE and X, and the shared link is the clean public address with no personal data attached.
- Given a page is only a preview or is unpublished, when a search engine or scraper reaches it, then it is marked "do not index" so it never leaks into public search.

### US-PAGE-09 — Preview any template with unsaved content  ·  **Must**
**As an** Event Organizer, **I want** to preview any of the four page designs using my live, unsaved event details before I publish, **so that** I can confidently pick the design that fits my event without committing anything.

**Acceptance criteria**
- Given I'm editing an event and I've typed a title, a highlight and chosen "online", when I click Preview, then a new tab shows a live page with exactly those values applied.
- Given I open a preview, when it renders, then nothing is saved — no event, page or draft is created — and it is clearly marked as a preview.
- Given a preview page, when it's viewed, then it is never counted in public visitor analytics and never appears in public search.
- Given I try to preview but my browser blocks the new tab, when nothing opens, then I see a clear hint to allow pop-ups.

_Notes: Preview is for Organizers, Admins and (view-only) Staff — not attendees or the public._

### US-PAGE-10 — Choose a design, brand it, and publish under a stable link  ·  **Must**
**As an** Event Organizer, **I want** to bind one of the four designs to my event, set its public web address and accent colour, and publish or unpublish it, **so that** I control how my event looks and the link I share never breaks.

**Acceptance criteria**
- Given four designs (Classic, Spotlight, Minimal, Vibrant), when I choose one and publish, then my event content renders in that design under a stable public address, and I can switch designs later with no loss of content.
- Given I try to publish, when the event is missing a title, a date, or at least one ticket/RSVP option, then publishing is blocked with a message telling me exactly what to add first.
- Given I set an accent colour, when the page renders, then buttons and highlights take that colour; an invalid colour quietly falls back to the brand default.
- Given I set a public web address that's already taken, when I save, then I'm told the address is in use and asked to pick another.
- Given a published page, when I unpublish it, then its public link stops working immediately and drops out of search, while the page is preserved so I can re-publish it later under the same address.

_Notes: The recommended design follows the event category, but any of the four can be chosen for any event._


---

<a id="epic-e05"></a>

# Epic E5 — Sell Tickets & Run Promotions

**Goal (business value):** Give organizers a fast, trustworthy way to define what they sell — the ticket classes, prices, quantities and sales windows — and to run promotions that grow sign-ups without leaking margin, so every event can turn interest into paid, confirmed attendance. Attendees see the right ticket at the right price and can join in a couple of taps.

**Primary users:** Event Organizer, Admin (define tickets & promotions); Attendee — guest or registered (buys tickets, redeems codes); Staff (no access to this surface).

**Success measures:**
- Revenue collected per event and overall (฿).
- Ticket sell-through rate (sold ÷ available) and time-to-sell-out for hot events.
- Registration conversion on shared ticket links/QRs.
- Promotion performance: code redemption rate, incremental sign-ups from promoted codes, and discount cost vs. revenue (margin protected).
- Zero oversells and zero over-redeemed codes (integrity of inventory and promo budgets).

---

## User stories

### US-TKT-01 — Create paid and free ticket types  ·  **Must**
**As an** Event Organizer, **I want** to create ticket types for my event — paid or free, each with a price, quantity, sales start/end dates, a per-order limit and an optional description — **so that** attendees can register for the right class of admission at the right time and I start collecting revenue.

**Acceptance criteria**
- Given I am setting up an event that is not cancelled, when I add a ticket type with a name, a price in baht, a quantity, and a sales window, then it is saved and appears in my ticket list ready to sell.
- Given I mark a ticket type as Free, when I save it, then no price is asked for or charged, and attendees confirm that ticket without any payment step.
- Given I choose a sales start date in the future, when I save, then the ticket shows as Scheduled and is not yet on sale; if the window is already open and quantity is available, it shows as On sale.
- Given I try to reuse a ticket name that already exists on the same event, when I save, then I am told a ticket with that name already exists and it is not created.
- Given I set a per-order limit above the quantity available, or a sales end date before the start, when I save, then I am shown a plain-language error and can correct it before saving.

_Notes: Per-order limit is capped at 8 seats to match the platform booking cap. Prices are whole baht; the attendee-facing total adds 7% VAT at checkout._

---

### US-TKT-02 — Adjust a ticket type safely after it is live  ·  **Must**
**As an** Event Organizer, **I want** to edit a ticket type — raise capacity, extend the sales window, tidy the description or transfer setting — with guardrails once tickets have sold, **so that** I can respond to demand without corrupting orders people already paid for.

**Acceptance criteria**
- Given a ticket type has sold some seats, when I increase the quantity available, then the extra seats go on sale immediately and, if it had sold out, it returns to On sale.
- Given a ticket type has sold 640 seats, when I try to lower the quantity to 500, then the change is refused with a message that quantity cannot go below the seats already sold.
- Given tickets have already sold, when I try to change the price, the paid/free setting, or the event, then I am told these can't change after sales began and am advised to create a new ticket type instead.
- Given no tickets have sold yet, when I change the price or switch paid/free, then the change is accepted.
- Given a colleague edited the same ticket type while I had it open, when I save, then I am told it changed since I opened it and asked to reload, so neither of us silently overwrites the other.

_Notes: Description and the "transferable" setting can be changed at any time. If capacity is raised on a sold-out ticket that has a waitlist, waitlisted attendees are offered the freed seats._

---

### US-TKT-03 — Availability that manages itself (opens, sells out, pause/resume, closes)  ·  **Must**
**As an** Event Organizer, **I want** a ticket type to open, sell out, and close on its own based on its sales window and inventory — and to let me pause and resume it on demand — **so that** I never oversell and never have to police availability by hand.

**Acceptance criteria**
- Given a ticket is Scheduled and its sales start date arrives, when the day begins (Bangkok time), then it automatically becomes On sale.
- Given a ticket has one seat left, when two people try to buy that last seat at the same moment, then exactly one succeeds and the ticket immediately shows Sold out to everyone else.
- Given a ticket's sales end date has passed, when an attendee tries to buy, then the purchase is refused even if the badge hasn't visibly refreshed — no sales happen outside the window.
- Given I choose Pause on a ticket, when it is paused, then it stops being purchasable even inside its sales window, until I Resume it.
- Given a ticket whose sales window has already ended, when I try to Resume it, then I am told to extend the sales end date first.

---

### US-TKT-04 — See all my tickets and inventory at a glance  ·  **Should**
**As an** Event Organizer, **I want** a searchable, filterable list of my ticket types showing price (or Free), how many have sold, and their current status, **so that** I can monitor sales across all my events in one place.

**Acceptance criteria**
- Given I have tickets in mixed states, when I select the "Sold out" filter, then only sold-out tickets are shown and the tab count matches.
- Given I search for "jazz", when I type it, then only tickets whose name or event contains "jazz" remain (case-insensitive).
- Given a ticket type, when I view its card, then I see its sales progress (sold out of total) and whether it is On sale, Scheduled, Paused or Sold out.
- Given my filters match nothing, when the list refreshes, then I see a clear "No matches" empty state.

_Notes: Sold and total counts are read-only here; they reflect confirmed registrations and configured capacity, and are never edited from this view._

---

### US-TKT-05 — Retire a ticket type without harming existing holders  ·  **Should**
**As an** Event Organizer, **I want** to delete a ticket type I created by mistake, or retire one that has already sold, **so that** I can clean up my line-up while everyone who already holds a ticket keeps it and their records stay intact.

**Acceptance criteria**
- Given a ticket type has never sold, when I confirm delete, then it is removed completely.
- Given a ticket type has sold 210 seats, when I confirm delete, then it is retired instead of removed: it disappears from my active list and can't be bought anymore, but existing holders keep valid tickets.
- Given I retire a ticket type, when the action completes, then no attendee is refunded or cancelled automatically — those are handled separately as finance actions.
- Given a ticket type has checkouts in progress, when I try to delete it, then I am told to pause it and try again shortly.

---

### US-TKT-06 — Share a ticket's registration link and QR  ·  **Should**
**As an** Event Organizer, **I want** a shareable registration link and a printable QR code for a specific ticket type, **so that** I can drive sign-ups from posters, social posts and email straight into the right ticket.

**Acceptance criteria**
- Given a ticket on a published event, when I open its share view, then I get a registration link and a matching QR that both open the sign-up page with that ticket already selected.
- Given the share view is open, when I choose "Copy link", then the link is copied and I get a brief "Copied" confirmation.
- Given the share view is open, when I choose "Download QR", then I receive a printable QR image I can put on a poster.
- Given the event is not yet published, when I try to share, then I am prompted to publish the event first so the link actually works.

_Notes: This link opens registration for the ticket type; it is not a personal entry pass. Each buyer receives their own unique entry QR later at check-in._

---

### US-TKT-07 — Create a discount code with a quick generator  ·  **Should**
**As an** Admin, **I want** to create a discount code — a percentage or a fixed-baht amount — scoped to one event or all events, with a total usage limit, a per-person limit, a validity window and an optional minimum order, **so that** I can run controlled promotions that grow sign-ups without giving away more margin than I intend.

**Acceptance criteria**
- Given I am creating a code, when I click Generate, then a memorable, unused code (for example "PROMO42") is proposed, and generating again never suggests one that already exists.
- Given I create a 25%-off code for a specific event with a usage limit of 500, when I save, then it appears in my list with 0 redemptions used and the correct live/scheduled status based on its dates.
- Given I set the code to start in the future, when I save, then it shows as Scheduled and is not yet usable at checkout; given it starts today, it shows as Active.
- Given I set a per-person limit higher than the total usage limit, or an end date before the start, when I save, then I get a plain-language error and can fix it.
- Given I try to save a code whose validity window has already ended, when I save, then it is refused as already expired.

_Notes: Codes apply only to paid tickets and reduce the price before 7% VAT is recalculated; free tickets are never discountable. Scope "All events" applies everywhere; a specific-event code only works for that event._

---

### US-TKT-08 — Adjust a discount code without breaking past redemptions  ·  **Could**
**As an** Admin, **I want** to edit a discount code's limits, dates, scope and minimum order, with the code text and its value locked once it has been used, **so that** I can tune a live promotion while everyone who already redeemed it keeps exactly the deal they were given.

**Acceptance criteria**
- Given a code has been redeemed 342 times, when I try to lower its usage limit to 300, then the change is refused because it can't drop below redemptions already made.
- Given a code has never been redeemed, when I change its discount from 25% to 30%, then the change is accepted.
- Given a code has already been redeemed, when I try to change the code text or its value, then I am told these can't change after use and advised to create a new code instead.
- Given I narrow a code's scope or shorten its window, when I save, then people who already redeemed it are unaffected.

---

### US-TKT-09 — Stop a promotion safely  ·  **Should**
**As an** Admin, **I want** to switch a code off or delete it mid-flight without touching attendees who already redeemed it, **so that** I can end a promotion the moment it has done its job or is being abused, without clawing back deals already given.

**Acceptance criteria**
- Given an active code, when I switch it off, then it is immediately refused at checkout while its redemption history is preserved.
- Given a code that has been redeemed 89 times, when I delete it, then it is retired (not erased): prior orders keep their discount and it no longer works at checkout.
- Given a code that has never been redeemed, when I delete it, then it is removed completely.
- Given a code is in use in an active checkout, when I try to delete it, then I am told to switch it off and try again shortly.

---

### US-TKT-10 — Codes go live and expire on schedule  ·  **Could**
**As an** Admin, **I want** codes to activate on their start date and expire when their window ends or their usage runs out, automatically, **so that** I can schedule promotions in advance and trust that they turn on and off without me policing them.

**Acceptance criteria**
- Given a Scheduled code whose start date arrives, when the day begins, then it automatically becomes Active and usable at checkout.
- Given an active code reaches its usage limit, when the final redemption completes, then it automatically becomes Expired.
- Given an active code passes its end date, when the window closes, then it automatically becomes Expired and no longer applies.
- Given a code has already expired, when I try to switch it back on, then I am told to update its dates or usage limit to revive it.

---

### US-TKT-11 — Apply a discount code at checkout  ·  **Should**
**As an** Attendee (guest or registered), **I want** to enter a discount code during registration and see it applied to my total, **so that** I get the promoted price and know instantly whether my code is valid.

**Acceptance criteria**
- Given a valid 25%-off code and a ฿1,000 order, when I apply it, then I see ฿250 off, a discounted total with 7% VAT recalculated on the reduced amount, and my code marked as applied.
- Given a fixed ฿300-off code on a ฿250 order, when I apply it, then the discount is capped at ฿250 and my total never goes below zero.
- Given a code that is unknown, not yet started, expired, switched off, or for a different event, when I apply it, then I get a clear reason it can't be used.
- Given I have already used a single-use code, when I try it again, then I am told I've already used it; given the code has hit its overall limit, then I am told it has reached its redemption limit.
- Given the order is below the code's minimum, when I apply it, then I am told how much more to add to qualify.

_Notes: Only one code applies per order. If an order is later cancelled or refunded, its redemption is released so the code can be used again._

---

### US-TKT-12 — Track which promotions are working  ·  **Should**
**As an** Event Organizer, **I want** to browse my codes with their type, scope, live redemption counts and status, and copy any code to share it, **so that** I can see at a glance which promotions are landing and act on them.

**Acceptance criteria**
- Given a code scoped to "All events", when I filter by a specific event, then that all-events code still shows because it applies everywhere.
- Given a list of codes, when I filter by status (Active, Scheduled, Expired, Disabled) or search by code or event, then the table narrows accordingly.
- Given any code, when I view its row, then I see how many redemptions it has used out of its limit and its current status.
- Given a code, when I click copy, then the code is on my clipboard with a brief confirmation so I can paste it into a post or email.

_Notes: Redemption counts are live and read-only. Deeper redemption reporting, including attendee-level detail, lives in Insights & Reports and is access-controlled._


---

<a id="epic-e06"></a>

# Epic E6 — Discover & Register for Events (Attendee)

**Goal (business value):** Give event-goers a fast, low-friction path to find events they care about, register and pay in Thai Baht (card or PromptPay), and walk in with a QR ticket — while giving returning attendees a home for their tickets, receipts, profile, and feedback. This is the core revenue path: every ticket sold and every seat filled starts here.

**Primary users:** Attendee (guest or registered). Organizers, Admins, and Staff do not act in this portal — they consume the registrations, payments, and feedback it produces in the admin console.

**Success measures:**
- Registration conversion (browse → completed registration) and checkout completion rate.
- Guest-checkout success rate (no account required).
- Payment success rate by method (card vs PromptPay) and total revenue collected (฿).
- Ticket delivery rate (confirmed registrations that receive a usable QR ticket).
- Repeat-attendee sign-in and return-registration rate.
- Post-event survey response rate and average event rating.

---

## User stories

### US-DISC-01 — Discover and browse what's on  ·  **Must**
**As an** Attendee, **I want** to browse a visual list of upcoming public events with the key details at a glance, **so that** I can quickly spot something worth attending in Bangkok.

**Acceptance criteria**
- Given published, publicly visible events exist, when I open the Discover page, then I see a grid of event cards each showing cover image, category, title, date, venue/city, organizer, how many people are going, a price-from (or "Free"), and an average rating when the event has been reviewed.
- Given an event has sold out, when the grid renders, then that card shows a "Waitlist" badge instead of a buy-now price.
- Given an event is nearly sold out, when the grid renders, then that card shows a "Selling fast" badge.
- Given an event has already started or finished, when the grid renders, then it is not shown in the list.
- Given no events match, when the page renders, then I see a friendly empty state suggesting I try a different search or category.
- Given I tap an event card, then I am taken to that event's public page where I can register.

_Notes: Events are ordered soonest-first by default. Ratings shown come from real attendee feedback (US-DISC-13), not a placeholder._

### US-DISC-02 — Search and filter events  ·  **Must**
**As an** Attendee, **I want** to search by keyword and narrow by category, **so that** I can find the right event without scrolling through everything.

**Acceptance criteria**
- Given the Discover page, when I type a keyword, then the list narrows to events whose title, category, city, or venue matches, and the result count updates.
- Given I type in Thai or English (with or without accents/tone marks), when I search, then matching works consistently for both languages.
- Given I select a category, when the list refreshes, then only events in that category are shown; selecting "All events" clears the category.
- Given both a keyword and a category are active, when the list refreshes, then only events matching both are shown.
- Given a keyword and category together match nothing, when applied, then the empty state is shown.
- Given I leave stray spaces around my search, when applied, then they are ignored.

### US-DISC-03 — Save events for later  ·  **Should**
**As an** Attendee, **I want** to save events I'm interested in, **so that** I can come back and register later.

**Acceptance criteria**
- Given an event card, when I tap the save (heart) control, then the event is marked saved and stays saved when I return.
- Given I am signed in and save an event on one device, when I sign in on another, then the event still shows as saved.
- Given I save events as a guest and then sign in, when I sign in, then my in-session saves are merged into my account with no duplicates.
- Given a save can't be recorded, when I tap it, then the control reverts and I see a brief "couldn't save right now" message.

_Notes: Saved events can trigger a reminder only if the attendee has event reminders turned on (US-DISC-12)._

### US-DISC-04 — Register and choose my tickets (guest or signed in)  ·  **Must**
**As an** Attendee (guest or registered), **I want** to pick a ticket type, choose my seats or quantity, and review a clear running total, **so that** I know exactly what I'm getting and paying before I commit — without being forced to create an account.

**Acceptance criteria**
- Given I click "Register"/"Get tickets" on an event, when checkout opens, then I see the event summary and can choose exactly one ticket type, with prices shown in Baht (or "Free").
- Given the event has reserved seating, when I reach seat selection, then I can pick up to 8 available seats from a seat map, already-taken seats are not selectable, and my chosen seats are held for me while I check out.
- Given the event is general-admission or online, when I reach quantity, then I choose a quantity between 1 and 8; online events tell me a join link will be emailed, general-admission events note seating is first-come.
- Given I make any selection, when it changes, then the order summary updates live with subtotal, service fee, and total, mirrored in a sticky action bar.
- Given I am signed in, when checkout opens, then my contact details are pre-filled from my profile and remain editable for this order; given I am a guest, then I enter name, email, and (when needed for PromptPay) a mobile number.
- Given I complete a guest checkout without opting in, when the order is placed, then no account is silently created for me.
- Given a free event, when I register, then all tiers show "Free", no fees are added, and the payment step is skipped.
- Given my seat hold expires or a seat is taken before I confirm, when I try to continue, then I'm told and asked to re-select, and I am not charged.

_Notes: Registration requires no account. Prices, seats, and totals shown are always the event's real, current values._

### US-DISC-05 — Pay by card or PromptPay  ·  **Must**
**As an** Attendee, **I want** to pay the total securely by card or Thai PromptPay, **so that** I can complete my registration the way that suits me.

**Acceptance criteria**
- Given a paid order, when I reach payment, then I can choose card or PromptPay.
- Given I choose card and my payment is approved, when it succeeds, then I move to confirmation and my registration is completed.
- Given my card is declined, when I submit, then I see a clear decline message, no ticket is issued, and I can try another card or switch to PromptPay.
- Given I choose PromptPay, when I continue, then I see a QR code for the exact total to scan in my banking app, valid for a limited time.
- Given my PromptPay payment settles, when confirmation arrives, then my registration completes and my ticket is issued exactly once.
- Given the PromptPay code expires before I pay, when it lapses, then no ticket is issued, my held seats are released, and I can generate a new code.
- Given any payment hiccup, when it happens, then I am reassured whether or not I was charged and never charged twice for the same order.

_Notes: Card details are entered by the attendee into the secure payment provider; Eventa never stores card numbers. All charges settle in THB._

### US-DISC-06 — Confirm and receive my QR ticket  ·  **Must**
**As an** Attendee, **I want** a confirmation and a scannable QR ticket the moment my registration is placed, **so that** I know I'm in and can get through the door.

**Acceptance criteria**
- Given a valid selection and (for paid events) an approved payment, when I confirm, then my registration is placed once, I get one QR ticket per seat/ticket, and I see a "You're registered!" recap with links to view my tickets.
- Given my registration is confirmed, when it completes, then I receive a confirmation email containing my QR ticket(s), an order summary, a VAT receipt for paid orders, and a calendar invite; online events also include a join link.
- Given I provided a mobile number and SMS is applicable, when I register, then I also get a confirmation text.
- Given I double-tap confirm or retry, when the requests arrive, then only one registration is created and I'm charged only once.
- Given two people try to take the same seat, when both confirm, then only the first succeeds and the other is not charged (or is refunded) and asked to pick again.

### US-DISC-07 — View and download my QR ticket  ·  **Must**
**As a** registered Attendee, **I want** to open and download my QR ticket, **so that** I always have my entry pass on hand, printed or on my phone.

**Acceptance criteria**
- Given I own a valid ticket, when I open it, then I see a scannable QR and its ticket reference.
- Given I download a ticket, then I get a printable ticket image showing the QR, event name, date, venue, doors-open time, ticket class (VIP/General), seat/row/gate where applicable, and admission number.
- Given a ticket has been refunded or voided, when I open it, then it shows as no longer valid and cannot be downloaded as a valid pass.
- Given I try to open a ticket I don't own, then I am refused access.

### US-DISC-08 — Sign in to my attendee account  ·  **Must**
**As a** registered Attendee, **I want** to sign in with email/password or Google/Apple, **so that** I can reach my tickets, receipts, and profile — and only my attendee area, never the organizer console.

**Acceptance criteria**
- Given valid credentials, when I sign in, then I land on my account (My Events) with attendee access only and no organizer/admin access.
- Given I sign in after saving events as a guest, when I sign in, then those saves are attached to my account.
- Given wrong credentials, when I submit, then I see a single "email or password is incorrect" message without revealing which was wrong.
- Given repeated failed attempts, when the limit is passed, then further attempts are temporarily blocked and I'm pointed to reset my password.
- Given I forgot my password, when I choose "Forgot password?", then I'm taken to a reset flow.

### US-DISC-09 — See my upcoming and past tickets  ·  **Must**
**As a** registered Attendee, **I want** my registrations split into upcoming and past, **so that** I can grab my ticket for what's next and revisit what I've attended.

**Acceptance criteria**
- Given I'm signed in, when I open My Events, then I see an Upcoming section and a Past section, each with a count, showing only my registrations.
- Given an upcoming event, when it's listed, then I see a countdown (e.g. "6 days left", "Today"), my ticket type, date and venue, a link to the event page, and a "Ticket" action to open my QR.
- Given a past event I attended, when it's listed, then I see an "Attended" badge and a "Leave feedback" action while feedback is open.
- Given I have no registrations, when the tab loads, then I see an empty state inviting me to discover events.

### US-DISC-10 — Review my payment history and receipts  ·  **Must**
**As a** registered Attendee, **I want** a history of my payments with downloadable receipts and an export, **so that** I can track and expense my spending.

**Acceptance criteria**
- Given I'm signed in, when I open Payment history, then I see summary tiles for total spent, number of transactions, and total refunded, plus a paged list of my transactions.
- Given a transaction row, when I view it, then I see the event, invoice number, date, payment method (masked card or PromptPay), amount, and status (Paid/Refunded), with refunded amounts struck through and excluded from total spent.
- Given a transaction, when I download its receipt, then I get a VAT receipt showing the 7% VAT breakdown in Baht.
- Given I export my history, then I receive my full transaction list.
- Given I have no transactions, when the tab loads, then tiles show zero and I see an empty state.

### US-DISC-11 — Manage my profile  ·  **Must**
**As a** registered Attendee, **I want** to keep my personal details and photo up to date, **so that** organizers can reach me and my checkout details stay correct.

**Acceptance criteria**
- Given the Profile tab, when I edit and save name, email, phone, city, date of birth, bio, or photo, then valid changes are saved and confirmed, and "Cancel" discards unsaved edits.
- Given I change my email, when I save, then the new email is marked unverified and I'm sent a verification link while my old email keeps working until confirmed.
- Given I change my phone, when I save, then I must confirm it by code before it's used for texts.
- Given I upload a photo over 5 MB or not a JPG/PNG, when I upload, then it's rejected with guidance and my current avatar is unchanged.
- Given I update my profile, when I next check out, then my details pre-fill from the saved profile.

### US-DISC-12 — Manage my notifications, display preferences, and security  ·  **Should**
**As a** registered Attendee, **I want** to control how Eventa contacts me, how things are displayed, and my account security, **so that** the experience fits me and my account stays protected.

**Acceptance criteria**
- Given the Settings tab, when I toggle email notifications, event reminders, SMS alerts, or marketing/promotions, then my choice is saved and applied to future messages.
- Given marketing is off, when a promotion goes out, then I'm excluded from it but still receive order confirmations and receipts.
- Given I set language, timezone, or currency, then the interface reflects my choice, while all charges still settle in Baht and event times stay anchored to each event's own timezone.
- Given I change my password with the correct current password, when I save, then it's updated, I'm sent a confirmation email, and I can sign out of other sessions.
- Given I enable two-factor authentication, when I complete verification, then it's turned on and required at my next sign-in, with recovery codes provided.
- Given a wrong current password or mismatched new passwords, when I submit, then I see a clear error and nothing changes.

_Notes: Marketing opt-in/out is recorded with a timestamp for PDPA compliance; transactional emails are always sent regardless of marketing choice._

### US-DISC-13 — Share post-event feedback  ·  **Should**
**As an** Attendee who attended an event, **I want** to leave a star rating and a few optional comments, **so that** organizers can improve future events and other attendees can judge quality.

**Acceptance criteria**
- Given I attended an event and feedback is open, when I open the survey, then I can give a 1–5 star rating and optionally say what I enjoyed, how I heard about it, and whether I'd recommend Eventa events.
- Given I don't pick a rating, when I submit, then I'm prompted to choose one and nothing is recorded until I do.
- Given I submit feedback, then I see a thank-you confirmation and my rating contributes to the event's average shown on Discover.
- Given I submit feedback again within the window, when I resubmit, then my earlier response is updated rather than duplicated.
- Given I'm not eligible or the window is closed, when I open the survey, then I'm told feedback isn't open.

_Notes: Feedback is anonymous to other attendees; organizers see aggregate ratings and comments._

### US-DISC-14 — Delete my account  ·  **Could**
**As a** registered Attendee, **I want** to permanently delete my account and personal data, **so that** I can leave the platform on my own terms.

**Acceptance criteria**
- Given the Settings danger zone, when I choose to delete my account, then I must explicitly confirm and re-verify my identity before it proceeds.
- Given I have upcoming paid tickets, when I try to delete, then I'm warned about any non-refundable tickets before continuing.
- Given I confirm and re-verify, when deletion runs, then my personal data is scheduled for removal, I'm signed out, and I receive a confirmation email.
- Given I can't re-verify my identity, when I try, then deletion is aborted and my account is unchanged.

_Notes: Deletion is irreversible. Financial/tax records are retained in anonymized form for the legal retention period, disassociated from the personal profile._


---

<a id="epic-e07"></a>

# Epic E7 — Communicate with Attendees

**Goal (business value):** Give organizers one place to keep every attendee informed — automatic transactional messages that fire on their own, one-off broadcasts to the right audience, a feed of what's happening across the event, proof that messages landed, and a way to measure how attendees felt. Reliable, on-brand communication protects revenue (people get their tickets and receipts), reduces support load, and turns each event into insight for the next one.

**Primary users:** Event Organizer, Admin (author and manage), Staff/Team member (personal notification feed only), Attendee (receives messages; answers surveys in the attendee portal).

**Success measures:**
- Message delivery rate (share of sent messages the provider confirms delivered).
- Email open rate on transactional and announcement messages.
- Announcement reach (recipients actually addressed vs. audience size).
- Survey response rate and completion rate per event.
- Attendee satisfaction — average rating and NPS across the portfolio.
- Triage speed — time for organizers to clear the unread notification backlog.

## User stories

### US-MSG-01 — Attendees automatically get the right message at the right moment  ·  **Must**
**As an** Attendee, **I want** to automatically receive accurate confirmation, receipt, reminder, waitlist, cancellation, and thank-you messages, **so that** I always have my ticket and the information I need without chasing the organizer.

**Acceptance criteria**
- Given I complete registration and payment, when the payment succeeds, then I receive a confirmation message with my ticket and order details.
- Given my event is a day away, when the reminder time arrives, then I receive a reminder with the event time and place.
- Given the organizer has turned a given message off, when its trigger occurs, then no message of that kind is sent to me.
- Given I have a language preference, when a message is sent, then I receive it in English or Thai accordingly, falling back to the event's default language.

_Notes: Message content is personalized with the attendee's and event's details; a message is never sent with a personalization field left unfilled. Ticket delivery on successful payment is an expected communication and is not subject to marketing opt-out._

### US-MSG-02 — Keep automated messages on-brand and in the organizer's control  ·  **Should**
**As an** Event Organizer, **I want** to edit the wording of each automated email/SMS, insert personalization fields, and turn each message on or off, **so that** every attendee gets accurate, on-brand communication without me sending anything by hand.

**Acceptance criteria**
- Given the registration-confirmation message, when I edit its wording and save, then messages sent afterwards use the new wording while messages already sent are unaffected.
- Given I am editing a message, when I place a personalization field (e.g. first name, event name) into the text, then it appears where my cursor was and shows correctly filled in for each recipient.
- Given I write an SMS that crosses into a second message segment, when I type, then I see a live character/segment count and a warning, so that I understand the added cost.
- Given a message that attendees legally expect, such as a payment receipt or cancellation notice, when I try to turn it off, then I am warned attendees won't receive it and must confirm before it is disabled.
- Given I try to save an active message with its subject or body empty, or referencing an unsupported personalization field, then the save is blocked with a clear reason.

_Notes: Each message exists in English and Thai; an active channel cannot be saved with its language version blank. SMS cost is driven by encoding — Thai text uses shorter segments — and the count reflects the actual text._

### US-MSG-03 — Triage everything happening across my events in one feed  ·  **Should**
**As an** Event Organizer, **I want** a single notification feed grouped by recency with an unread filter and a one-click "mark all read", **so that** I can triage new registrations, payments, feedback, and alerts without opening every module.

**Acceptance criteria**
- Given I have unread notifications, when I open the feed, then I see All and Unread counts and the most recent activity expanded, with older activity available on demand.
- Given I switch to the Unread filter with nothing unread, when the feed renders, then I see a "you're all caught up" message.
- Given unread notifications exist, when I click "mark all read", then the unread count drops to zero and stays cleared after I reload.
- Given I am a Staff/Team member, when I open the feed, then I see only my own personal activity and cannot compose messages, edit templates, broadcast, view the delivery log, or manage feedback.

_Notes: A member only sees notifications they are entitled to — for example, payment and payout items appear only for members with finance access._

### US-MSG-04 — Broadcast a one-off announcement to the right audience  ·  **Should**
**As an** Event Organizer, **I want** to send a one-off announcement to a chosen audience of an event over email and/or SMS, either immediately or scheduled, **so that** I can reach the right attendees at the right time (e.g. a venue change or last-minute update).

**Acceptance criteria**
- Given I choose the "checked-in attendees" audience for an event with email and SMS, when I send now, then only checked-in attendees with a valid contact receive it and the recorded recipient count matches who was addressed.
- Given I choose to schedule for a future Bangkok date and time, when I send, then the announcement is recorded as Scheduled and goes out at that time.
- Given I have not selected any channel, or left the message (or an email subject) empty, when I click send, then I am blocked with a clear reason.
- Given the chosen audience resolves to nobody, when I send, then nothing is sent and I am told no recipients matched.
- Given I broadcast a marketing-class announcement, when it is delivered, then it includes an unsubscribe option and anyone who has opted out is excluded and counted as skipped.

_Notes: Audience options are all registrants, checked-in attendees, or waitlist. A double-click or retry never double-sends to the same recipient._

### US-MSG-05 — Change my mind on a scheduled announcement  ·  **Could**
**As an** Event Organizer, **I want** to cancel or reschedule an announcement that hasn't gone out yet, **so that** I can correct a mistake or shift timing without messaging attendees wrongly.

**Acceptance criteria**
- Given a scheduled announcement still in the future, when I cancel it, then no messages are sent and it no longer shows as Scheduled (but remains in history).
- Given a scheduled announcement, when I reschedule it to a later valid time, then it goes out at the new time to a freshly resolved audience.
- Given an announcement has already started sending, when I try to change it, then I am told it can no longer be changed.

### US-MSG-06 — Prove messages were delivered and diagnose failures  ·  **Should**
**As an** Admin, **I want** a delivery log showing each message's recipient, type, channel, and status (sent, delivered, opened, failed), **so that** I can prove a message went out and spot and fix failures.

**Acceptance criteria**
- Given messages have gone out, when I open the log, then I see one entry per recipient with the recipient, message type, channel, status, and time.
- Given an SMS the provider reported as undelivered, when the log renders, then that entry shows the SMS channel and a Failed status.
- Given an email the recipient opened, when that is confirmed, then the entry advances to an Opened status.
- Given a critical message (a ticket or receipt) failed, when I review the log, then failures are surfaced so I can follow up.

_Notes: Statuses reflect only what the delivery provider has actually confirmed — the log never claims a delivery the provider hasn't reported. The log exposes attendee contact details, so it is limited to organizers/admins with attendee-view access; Staff are denied._

### US-MSG-07 — Export the delivery log for records  ·  **Could**
**As an** Event Organizer, **I want** to export the delivery log to a file, **so that** I have a record for compliance, reconciliation, or diagnostics.

**Acceptance criteria**
- Given I have permission to export attendee data, when I export, then a file of the current log view downloads and the export is recorded for audit.
- Given I am a Staff member or lack export permission, when I view the log, then the export option is unavailable.
- Given the current view has nothing to export, when I export, then I am told there is nothing to export.

_Notes: The file contains attendee contact data in bulk, so export requires the stronger attendee-export permission, not just view access._

### US-MSG-08 — Measure attendee satisfaction across all my events  ·  **Should**
**As an** Event Organizer, **I want** portfolio-wide feedback KPIs and a per-event drill-down with a rating breakdown, **so that** I can see how attendees felt across all my events and dig into any one of them.

**Acceptance criteria**
- Given several events have collected feedback, when I open the feedback overview, then I see portfolio KPIs — total responses, average rating, NPS, and completion rate — and a card per event.
- Given I search by event name, when I type, then the event grid filters to matching events, with a clear empty state when none match.
- Given I open an event's detail, then I see its KPIs, its rating distribution, and its surveys; and clicking a rating bar filters the responses to that rating.
- Given an event has collected no responses yet, when its card or detail renders, then it clearly shows "no responses yet" rather than misleading numbers.

_Notes: Portfolio figures are weighted by each event's number of responses; events with no responses don't distort the averages._

### US-MSG-09 — Build and manage surveys for my events  ·  **Should**
**As an** Event Organizer, **I want** to build surveys with rating, text, and multiple-choice questions and then duplicate, close, reopen, or delete them, **so that** I can collect the feedback I need and reuse and lifecycle-manage my forms.

**Acceptance criteria**
- Given I create a survey with a title and at least one question against an event, when I save, then it is created as a Draft under that event and appears in its surveys list, collecting nothing until I make it live.
- Given a multiple-choice question with fewer than two options, or a question with blank text, when I save, then I am blocked with a clear reason.
- Given an existing survey, when I duplicate it, then a fresh Draft copy with zero responses appears next to it.
- Given a live survey, when I close it, then it stops accepting responses; and when I reopen it, then it accepts responses again.
- Given a survey with responses, when I delete it, then I must confirm before it is removed.

_Notes: Attendees answer surveys in the attendee portal, not here — this story is only about authoring and managing them. The post-event thank-you message is what distributes the survey link to attendees._

### US-MSG-10 — Browse and filter individual feedback responses  ·  **Could**
**As an** Event Organizer, **I want** to browse individual responses and filter them by survey and by rating, **so that** I can read what specific attendees said, not just the averages.

**Acceptance criteria**
- Given an event has responses, when I filter by a rating or a specific survey, then the list narrows to matching responses and paginates, showing respondent, survey, rating, comment, and date.
- Given a page size and multiple pages, when I move between pages, then the "showing X–Y of N" range updates accordingly.
- Given filters that match no responses, when applied, then I see a clear "no responses match these filters" message.

_Notes: Responses carry attendee identity, so browsing is limited to organizers/admins with attendee-view access; Staff are denied._


---

<a id="epic-e08"></a>

# Epic E8 — Manage Registrations & Admit Attendees

**Goal (business value):** Give organizers one dependable place to turn sign-ups into confirmed, ticketed attendees — reviewing and approving registrations, filling seats fairly from the waitlist, keeping a clean attendee directory, inviting the right people, and getting everyone through the door fast on event day by QR scan or by hand. The outcome is more paid seats filled, less door queue, and no fraudulent or double entry.

**Primary users:** Event Organizer, Admin (full registration + attendee management), Staff / Team member (view + door check-in), Attendee (the subject of these records — never operates these screens).

**Success measures:**
- Registration approval turnaround (time from sign-up to confirmed ticket).
- Seat fill rate and waitlist conversion (share of freed seats re-sold before the event).
- No-show rate and overall check-in rate on event day.
- Average time-to-check-in per attendee / door throughput at peak.
- Share of admissions handled cleanly by QR vs. manual fallback, and rejected/duplicate scans caught.
- Invite-to-registration conversion.
- Data cleanliness handed to CRM (tagged VIPs/speakers/sponsors, exportable segments).

## User stories

### US-REG-01 — Review the registrations queue · **Must**
**As an** Event Organizer, **I want** to see every registration for my events in one filterable list with pending, waitlist and cancelled views, **so that** I can quickly find who needs a decision and keep only legitimate attendees moving toward a ticket.

**Acceptance criteria**
- Given registrations exist across events, when I open the registrations workbench, then I see a list with tabs for All, Pending, Waitlist and Cancelled, each showing a live count that updates as statuses change.
- Given I want a specific person, when I search by attendee name, email or booking reference, then the list narrows to matches and returns me to the first page.
- Given many events, when I filter by a chosen event and/or ticket type, then the list shows only matching registrations and the filters combine (all conditions must hold).
- Given any registration, when it is listed, then its status shows as a clear badge — Confirmed, Pending, Waitlisted or Cancelled — and its amount shows as a VAT-inclusive Baht figure or a "Free" badge.
- Given a pending registration, when I look at its row, then Approve and Reject actions are offered; on non-pending rows only View and an overflow menu appear.
- Given no registrations match my filters, when the list refreshes, then I see a clear "No matches" empty state.
- Given I open a single registration's detail, when the panel appears, then I can read its current status, payment state, issued ticket(s) and a dated history (created / confirmed / checked-in / cancelled) in Bangkok time.
- Given a Staff member without finance access, when they open a registration, then monetary amounts are masked.

_Notes: Read-only browsing is open to Organizer, Admin and Staff; the underlying detail view is lower priority (Could) but folded here as the natural "open a row" action._

### US-REG-02 — Approve or reject a pending registration · **Must**
**As an** Event Organizer, **I want** to approve a pending sign-up so a ticket and QR are issued, or reject it so the seat is released, **so that** only legitimate attendees are confirmed and freed seats can go to someone else.

**Acceptance criteria**
- Given a pending registration for a free ticket with seats available, when I approve it, then it becomes Confirmed, the attendee is seated, a ticket with QR code is issued, and a confirmation is sent by email and SMS.
- Given a pending registration for a paid ticket that isn't paid yet, when I try to approve it, then approval is blocked with "Payment isn't complete yet, so this registration can't be approved."
- Given the ticket sold out between opening the list and approving, when I approve, then the registration stays pending and I'm told the ticket is now sold out and offered the waitlist instead.
- Given I approve the same registration twice (e.g. a retry), when the action repeats, then only one ticket is issued and only one confirmation is sent.
- Given a pending or waitlisted registration with no funds captured, when I reject it and confirm the prompt, then it becomes Rejected, the held seat is released, and a rejection notice is emailed to the attendee.
- Given a rejected registration, when I view it later, then it cannot be re-approved and no Approve action is shown.
- Given a confirmed registration where money was captured, when I look for a reject action, then it is not offered and I'm directed to cancel and refund instead.

_Notes: Reject requires a confirmation step so a single misclick can't terminate a registration. Rejecting may free capacity and trigger waitlist promotion. Approve/reject are Organizer/Admin only — Staff cannot decide._

### US-REG-03 — Add a registration by hand · **Must**
**As an** Event Organizer, **I want** to add a registration for a walk-up or phone booking myself, **so that** off-platform attendees still get a proper ticket and QR and are counted against capacity.

**Acceptance criteria**
- Given an event open for registration, when I fill in the attendee's name, email, event, ticket type and a seat quantity, then a registration is created, the seats are reserved against capacity, and the attendee is matched to an existing directory record or added as a new one.
- Given the "Send confirmation" toggle is on, when I save, then the attendee receives their ticket and QR by email and SMS; when it's off, the ticket is created quietly for me to deliver later.
- Given I request more than the allowed maximum of 8 seats in one booking, when I try to save, then I'm asked to choose between 1 and 8 seats and nothing is reserved.
- Given I request more seats than remain, when I save, then I'm told exactly how many seats are left and no partial booking is made.
- Given the chosen event is a draft, completed or cancelled event, when I try to add a registration, then I'm told the event isn't open for new registrations.
- Given a free ticket, when I save, then the amount shows as "Free" and the registration is confirmed immediately; for a paid ticket the amount is calculated automatically (VAT-inclusive) and I never type it.
- Given required fields are missing or the email is invalid, when I try to save, then I see inline errors and the panel keeps my entries.

### US-REG-04 — Manage the waitlist & promote attendees · **Should**
**As an** Event Organizer, **I want** to promote waitlisted attendees in order when a seat frees, **so that** I fill my event to capacity fairly and don't lose demand.

**Acceptance criteria**
- Given a free seat and a waitlist, when I promote the person at the front of the queue, then they receive a time-limited offer (24 hours by default) to complete payment and are notified; free tickets confirm immediately.
- Given no seat is free, when I try to promote someone, then promotion is refused and I'm told to free capacity first.
- Given the default order is first-come-first-served, when I deliberately promote someone out of order, then the out-of-order choice is recorded.
- Given a promotion offer that isn't taken up in time, when it lapses, then the seat passes to the next person in line automatically and the attendee is told their offer expired.
- Given a promoted attendee completes payment within the offer window, when it settles, then their registration confirms with ticket, QR and confirmation just like a normal approval.

### US-REG-05 — Browse the attendee directory & profiles · **Must**
**As an** Event Organizer, **I want** to search, segment and sort my whole attendee directory and open a full profile for anyone, **so that** I can understand and act on who is coming across all my events.

**Acceptance criteria**
- Given attendees exist, when I open the directory, then I see a list with segments for All, New, Checked-in and VIP, each with a live count, sorted by most recent activity by default.
- Given I want to reorder, when I sort by name, most events or most tickets, then the list re-orders accordingly (ties broken by most recent activity).
- Given I search by name or email, or filter by tag (VIP, Speaker, Sponsor, Student), then the list narrows to matches and the conditions combine.
- Given a tagged attendee, when they appear in the list, then their tag shows as a coloured badge, and untagged attendees show a dash.
- Given I open an attendee's profile, when the panel appears, then I see their contact details, tags, every event they've registered for with its status (Checked-in, Registered or Past), their tickets (VAT-inclusive amount or "Free"), and a dated activity timeline, newest first.
- Given no attendees match, when the list refreshes, then I see a "No matches" empty state.

### US-REG-06 — Invite people to an event · **Must**
**As an** Event Organizer, **I want** to email an invitation for a specific event to a named person with an optional personal note, **so that** I can grow attendance beyond public discovery.

**Acceptance criteria**
- Given I enter a recipient name, email and exactly one event, when I send the invite, then a tracked invite record is created and one invitation email is queued to that person with a link into the event's registration flow.
- Given I add a personal message, when the invite is sent, then that note appears at the top of the invitation email.
- Given I try to send with a missing name, invalid email or no event chosen, when I submit, then I see inline errors and nothing is sent.
- Given my note is longer than 250 characters, when I submit, then I'm asked to shorten it.
- Given I send a second identical invite to the same person and event within a short window, when it's submitted, then no duplicate email is dispatched.

_Notes: An invite is a promise to attend, not a booking — it does not reserve a seat until the invitee actually registers. Inviting is Organizer/Admin only; Staff cannot invite._

### US-REG-07 — Tag & segment attendees · **Should**
**As an** Event Organizer, **I want** to tag an attendee as VIP, Speaker, Sponsor or Student, **so that** I can segment my audience and hand clean, labelled data to my CRM.

**Acceptance criteria**
- Given an untagged attendee, when I set a tag, then the tag badge appears on their row and the matching segment count (e.g. VIPs) increases by one.
- Given a tagged attendee, when I clear or change the tag, then the badge updates and the affected segment counts adjust live (an attendee carries at most one tag).
- Given I re-apply the tag an attendee already has, when I confirm, then nothing changes.
- Given another team member tags the same attendee at the same moment, when my change lands, then the most recent change wins and I'm told the attendee was just updated.

_Notes: Staff can view tags but not change them._

### US-REG-08 — Keep attendee contact details current · **Should**
**As an** Event Organizer, **I want** to correct an attendee's name, email and phone, **so that** confirmations and SMS reach the right person and my directory stays accurate.

**Acceptance criteria**
- Given valid changes, when I save, then the directory row and profile show the new details and the change is recorded for audit.
- Given I change the phone number, when I save, then future confirmations and reminders go to the new number.
- Given the new email already belongs to another attendee, when I save, then the change is blocked and I'm prompted to merge the two records rather than overwrite.
- Given an invalid name or email, when I try to save, then I see inline errors and the change is not applied.
- Given a successful edit, when I reopen the profile, then an "Updated contact details" entry appears in the activity timeline.

### US-REG-09 — Email an attendee directly · **Should**
**As an** Event Organizer, **I want** to send a one-off email to an attendee from their profile, **so that** I can answer a question or share event-specific information without leaving the platform.

**Acceptance criteria**
- Given a valid subject and message, when I send, then one email is queued to the attendee and the send is logged on their activity timeline.
- Given an empty subject or message, when I try to send, then I'm asked to add a subject and a message before sending.
- Given a send fails, when it errors, then I'm told it couldn't be sent and can retry.

_Notes: Staff cannot send attendee emails._

### US-REG-10 — Export registration & attendee lists · **Should**
**As an** Event Organizer, **I want** to export exactly the registrations or attendees I've filtered to, **so that** I can share clean data with my CRM, finance or sponsors.

**Acceptance criteria**
- Given a filter is active (tab, event, ticket, tag, search), when I export, then the file contains exactly that filtered set — not just the current page — with attendee, event, ticket, date, amount and status details, and Thai names render correctly.
- Given I export, when the file is produced, then the export is recorded (who exported, how many rows, which filters) because it discloses attendee personal data.
- Given a Staff member, when they view the list, then the Export control is not available to them.
- Given no rows match, when I export, then I get an empty file with headers only rather than an error.

_Notes: Export is limited to members with export permission (Organizer/Admin); Staff and attendees are excluded._

### US-REG-11 — Run the manual check-in list · **Must**
**As** Staff at the door, **I want** a searchable roster for one event with a live checked-in progress bar, **so that** I can admit people quickly and mark or undo a check-in even when a QR can't be scanned.

**Acceptance criteria**
- Given I select an event, when the roster loads, then I see its confirmed attendees, a live progress readout (e.g. "7 / 29" with a matching percentage bar), and segments for All, Checked-in and Not-yet.
- Given I search by name, email or ticket, or filter by ticket type, then the roster narrows to matches and returns to the first page.
- Given a not-yet attendee, when I check them in, then their row shows a check-in time (HH:MM, Bangkok), the checked-in count goes up by one, and the progress bar advances immediately.
- Given a checked-in attendee, when I undo the check-in, then the time clears and the not-yet count goes up by one.
- Given an attendee whose registration isn't valid for entry (e.g. cancelled), when I try to check them in, then I'm told the registration isn't valid for check-in.
- Given the event's check-in window isn't open, when I try to check someone in, then I'm told check-in isn't open for this event.
- Given an already-checked-in attendee, when they are checked in again, then nothing changes and their original arrival time is kept.

_Notes: Anyone with view access can watch the list; only members with check-in permission (Organizer/Admin/Staff) can mark or undo. Manual and QR check-ins stay consistent — the same person can't be double-counted, and the earliest arrival time is kept._

### US-REG-12 — Scan tickets at the live QR check-in station · **Must**
**As** Staff running a check-in station, **I want** to bind the station to one event and scan ticket QR codes with my phone camera, getting an instant valid / duplicate / invalid / wrong-event / cancelled result, **so that** entry is fast and fraud is caught at the door.

**Acceptance criteria**
- Given I bind the station to an event, when I switch it to a different event, then the scanner, stats and feed re-point to the new event and start fresh.
- Given the station is bound to an event, when I start the camera and hold a valid, unused ticket for this event in view, then a green "Checked in" result shows, the attendee is admitted, and the live count and feed update.
- Given the same ticket is held or presented again, when it's read again, then the result is "Already checked in" showing the original arrival time, and the count does not change.
- Given a QR that isn't a recognisable ticket, when it's read, then the result is "Invalid ticket" and no one is admitted.
- Given a valid ticket for a different event, when it's scanned at this station, then the result is "Wrong event" and no one is admitted.
- Given a cancelled or refunded ticket, when it's scanned, then the result is "Ticket cancelled — entry denied" and no one is admitted.
- Given the camera can't be opened (permission denied, no camera, insecure page, or camera in use), when I try to start it, then I see a plain-language reason and am pointed to upload an image or search manually — the station never crashes.
- Given a valid code is held steady in view, when it's read, then exactly one check-in happens and immediate re-reads of the same code are ignored.

_Notes: Torch, camera flip, decoding an uploaded QR image, and a simulate control (a demo/testing aid, not a real entry method) are available at the station to support real door conditions. Requires check-in permission._

### US-REG-13 — Check attendees in manually when a QR won't scan · **Must**
**As** Staff at the station, **I want** to look someone up by name, email or ticket and check them in without a scan, **so that** a damaged, missing or unscannable QR never stops a legitimate attendee from getting in.

**Acceptance criteria**
- Given I open the "Can't scan? Find attendee manually" lookup and search, when I find a not-yet-checked-in attendee for this event and check them in, then they are admitted, the live count goes up, and they appear at the top of the feed — exactly as a scan would.
- Given an already-checked-in attendee, when I find them, then their row shows a "Checked in" badge and no check-in action, so they can't be admitted twice.
- Given no one matches my search, when the list refreshes, then I see a "No attendees match your search" message.

### US-REG-14 — Watch live turnout at the door · **Should**
**As an** Event Organizer, **I want** a live count and feed of who just arrived, **so that** I can monitor turnout in real time and react to a slow or crowded door.

**Acceptance criteria**
- Given check-ins are happening, when one succeeds, then the counter (checked / total, percentage, on-site and remaining) updates immediately and never counts past the event's total capacity.
- Given a successful check-in from any method (scan, uploaded image, simulate or manual), when it lands, then the attendee appears at the top of a "Just checked in" feed marked "just now", with earlier arrivals ageing below.
- Given I switch the station to another event, when the view refreshes, then the counter and feed show that event's figures.


---

<a id="epic-e09"></a>

# Epic E9 — Get Paid & Manage Finances

**Goal (business value):** Give event organizers one trusted money workspace where they can see every payment, refund a cancellation cleanly, receive settled funds in their bank, hand buyers compliant VAT invoices, and stay on top of monthly VAT filing — so revenue is always reconciled and Thai tax obligations are met on time.

**Primary users:** Event Organizer (view, invoice, export), Admin (full control — refunds, payout money-movement, invoice void, VAT filing), Staff and Attendee (no access to finance).

**Success measures:**
- Revenue collected and fully reconciled against the ledger (share of payments matched, no unexplained gaps).
- Refund turnaround time (request to settled, with ticket released).
- Payout settlement time to the organizer's bank and failed-payout recovery rate.
- Invoice on-time payment rate and average days-to-pay against the 14-day term.
- VAT returns filed by the statutory due date (share of periods filed on time).

---

## User stories

### US-FIN-01 — See every payment · **Must**
**As an** Event Organizer, **I want** a searchable, filterable ledger of every charge, refund, and failed attempt — with the attendee, event, payment method, amount, and status — that stays in step with what actually cleared, **so that** I can reconcile revenue and answer buyer questions without leaving the console.
**Acceptance criteria**
- Given payments exist across Paid, Pending, Refunded, and Failed, when I filter by status and by method (Card, PromptPay, or Bank transfer) and type part of a name or transaction reference, then only matching payments show and each status tab shows a live count for the whole ledger.
- Given the ledger, when it loads, then payments are listed newest first with VAT-inclusive Baht amounts.
- Given a Pending or Failed payment, when I look at its row, then "View invoice" and "Refund" are unavailable, each explaining why (no completed charge / not eligible for refund).
- Given a PromptPay charge that later settles, when I refresh, then the row shows as Paid without any manual editing.
_Notes: Payment methods in scope are Card, PromptPay, and Bank transfer; all amounts are VAT-inclusive Thai Baht._

### US-FIN-02 — Issue a refund · **Must**
**As an** Admin, **I want** to issue a full refund of a paid charge back to the buyer's original payment method in one guarded, confirmed action, **so that** a cancellation is settled fairly, the ticket seat is freed, and everything stays reconciled and auditable.
**Acceptance criteria**
- Given a Paid payment, when I open the refund dialog, then it restates the exact amount and warns that the refund is final before I confirm.
- Given I confirm the refund, when it succeeds, then the payment shows as Refunded, the attendee's ticket is cancelled and its seat returned to availability, and the buyer receives a refund confirmation (email, plus SMS if a mobile is on file).
- Given the refund succeeds, when I look at the related invoice, then it is marked Void and no longer counts toward outstanding receivables, and the VAT on that sale is backed out of the current tax period.
- Given an Organizer (not an Admin), when they view a paid payment, then the Refund action is not available to them.
- Given the same refund is submitted twice (e.g. a double click or a retry), when both arrive, then only one refund is ever issued.
_Notes: Baseline is full refunds only; partial refunds are out of scope this release. Only Paid payments are refundable, and only an Admin can refund._

### US-FIN-03 — Track balances and payout history · **Must**
**As an** Event Organizer, **I want** to see my available, pending, and all-time paid-out balances alongside a history of every payout to my bank, **so that** I always know how much I've earned, how much is on the way, and what has already landed.
**Acceptance criteria**
- Given settled sales and past payouts, when I open Payouts, then I see three headline balances — available, pending, and paid out to date — in Baht.
- Given one payout is currently in progress, when the page loads, then the pending balance equals that payout's amount and is labelled accordingly.
- Given a history of payouts, when I filter by status (Paid, Processing, Scheduled, Failed), then matching payouts show with amount, a masked destination bank account, and their requested and completed dates.
- Given no bank account is connected yet, when I open Payouts, then balances read as unavailable and I'm prompted to connect payouts.
_Notes: Only a masked last-4 of the bank account is ever shown; Eventa never stores full bank or card numbers._

### US-FIN-04 — Follow and recover a payout · **Should**
**As an** Admin, **I want** to open a payout to see its progress timeline and, when one fails, retry it to my connected bank, **so that** I can understand where my money is and unblock settlement myself.
**Acceptance criteria**
- Given any payout, when I open it, then I see its amount, status, masked destination, dates, and a plain-language timeline (requested → processing → paid) with a note on what to expect.
- Given a payout that is in progress, when I view it, then the note explains funds typically arrive within 1–3 business days.
- Given a failed payout, when I retry it as an Admin and it is accepted, then the same payout moves to processing (no duplicate payout is created) and the retry is recorded for audit.
- Given a completed payout, when I choose to download its receipt, then I get a record showing the payout reference, amount, and completion date.
- Given an Organizer (not an Admin), when they view a failed payout, then the retry action is not available to them.
_Notes: The downloadable payout receipt is a record of transfer, not a tax document — priority Could within this story._

### US-FIN-05 — Manage bank and payout settings · **Must**
**As an** Admin, **I want** to manage my bank account, payout schedule, and tax forms through the secure payments provider, **so that** settlement details stay accurate and safe without Eventa ever holding my bank or card numbers.
**Acceptance criteria**
- Given I am an Admin, when I choose to manage payouts, then I'm taken securely to the payments provider to update bank details, set my payout schedule, and access tax forms and statements.
- Given no account is connected, when I start, then I'm guided through connecting my payout account first.
- Given an Organizer (not an Admin), when they view Payouts, then bank and payout management is not available to them.
_Notes: Bank/schedule/tax-form data lives with the payments provider, not in Eventa. This is Admin-only account control._

### US-FIN-06 — Invoice ledger with ageing · **Must**
**As an** Event Organizer, **I want** an invoice ledger that automatically ages unpaid bills to Overdue once the 14-day term passes, with search and per-event filtering, **so that** I can see at a glance who still owes and how late they are.
**Acceptance criteria**
- Given an unpaid invoice whose due date passed 3 days ago, when the ledger renders, then it shows as Overdue with "3 days overdue" highlighted.
- Given an unpaid invoice due in 9 days, when it renders, then it shows as Issued with "in 9 days".
- Given the ledger, when I filter by status (Paid, Issued, Overdue, Void), by event, or by invoice number or buyer name, then only matching invoices show and tab counts reflect the whole ledger.
- Given a paid invoice, when I view its row, then it shows how and when it was paid; a voided invoice shows "Voided".
_Notes: Invoice terms are fixed at 14 days; "today" is judged on Bangkok time._

### US-FIN-07 — Issue a VAT invoice · **Must**
**As an** Event Organizer, **I want** to issue a compliant tax invoice for an order — with a proper sequential number, 14-day terms, and a 7% VAT breakdown — **so that** business buyers get the document they need and my books stay in order.
**Acceptance criteria**
- Given a billable order for ฿48,000, when I issue an invoice today, then it receives the next invoice number in an unbroken sequence, a due date 14 days out, a subtotal and 7% VAT that add back exactly to ฿48,000, and a status of Issued.
- Given the order is already paid, when the invoice is issued, then it appears as Paid with how and when it was paid.
- Given the same order, when issuing is triggered twice, then only one invoice exists for it.
- Given a missing buyer email or a zero/negative amount, when I try to issue, then I'm asked to provide a valid buyer email and amount and no invoice is created.
_Notes: Invoice numbers are sequential and never reused (Thai tax requirement); once issued, the amount, buyer, and number can't be edited — corrections are made by voiding and re-issuing._

### US-FIN-08 — View and download a tax invoice · **Must**
**As an** Event Organizer, **I want** to open any invoice and download it as a clean, single-page tax-invoice PDF carrying Eventa's legal identity and the 7% VAT breakdown, **so that** I can hand a buyer a compliant document on demand — whether I start from the invoice ledger or from the payment behind it.
**Acceptance criteria**
- Given a ฿32,500 paid invoice, when I open its detail, then it shows the buyer, order reference, event line item, issued and due dates, a subtotal and 7% VAT that reconcile to ฿32,500, and how it was paid.
- Given a completed payment, when I open the invoice behind it, then I see the same reconciling breakdown, and a refunded payment's invoice is marked Void.
- Given an invoice, when I download the PDF, then it is a single A4 page showing Eventa's legal name, address, and tax ID, the buyer, the event service line, and the reconciling VAT breakdown.
- Given a due-but-unpaid invoice, when I view it, then the due date is shown with its ageing (e.g. "in 9 days" or "3 days overdue").

### US-FIN-09 — Chase unpaid invoices · **Should**
**As an** Event Organizer, **I want** to resend an invoice to its buyer and have overdue bills reminded automatically, **so that** I get paid faster without manually tracking every due date.
**Acceptance criteria**
- Given an invoice with a valid buyer email, when I resend it, then the buyer receives it again and the invoice's number, amount, and status are unchanged.
- Given an unpaid invoice that reaches its due date, when the daily reminder runs, then the buyer is sent a due/overdue reminder and the invoice shows as Overdue thereafter.
- Given an invoice becomes Paid or Void, when the next reminder cycle runs, then no further reminders are sent for it.
- Given repeated resend attempts, when they exceed a sensible limit, then further sends are held back so the buyer isn't flooded.
_Notes: Every send is recorded in the messaging delivery log; reminders render in the buyer's language (EN/TH)._

### US-FIN-10 — Void an invoice · **Must**
**As an** Admin, **I want** to void an issued or overdue invoice that was raised in error, with an optional reason, **so that** I can correct receivables and VAT while keeping the invoice sequence intact and auditable.
**Acceptance criteria**
- Given an Issued or Overdue invoice, when I confirm the void as an Admin, then it becomes Void, drops out of outstanding receivables, its VAT is reversed from the current open tax period, and the action is recorded for audit.
- Given a voided invoice, when I look at the sequence, then its number is preserved and never reused — a correction is a brand-new invoice.
- Given an already-paid or already-void invoice, when I try to void it, then I'm told it can't be voided (a paid invoice is only voided by refunding its payment).
- Given an Organizer (not an Admin), when they attempt to void, then the action is denied.

### US-FIN-11 — Monthly VAT ledger · **Must**
**As an** Event Organizer, **I want** a monthly VAT ledger with clear headlines — VAT collected, remitted, and still payable, plus withholding tax — scoped by year, **so that** I always know my current tax position and can prepare the return.
**Acceptance criteria**
- Given a period with taxable sales of ฿3,012,800, when its row renders, then VAT collected shows ฿210,896 (7% of sales).
- Given a chosen year, when the headlines recompute, then VAT payable always equals VAT collected minus VAT remitted for that year, and the headlines never disagree with the rows beneath them.
- Given periods across a year, when I filter by filing status (Filed, Due, Upcoming), then matching periods show with their due dates (the 15th of the following month) and status.
- Given periods with withholding-tax figures, when I view the chosen year, then the withholding headline sums those figures and is tracked separately from VAT payable.
_Notes: VAT rate is fixed at 7%; withholding tracking (Thai PND) is reconciled separately from VAT and is Should-level depth within this ledger._

### US-FIN-12 — File the VAT return each period · **Must**
**As an** Admin, **I want** each monthly VAT period to move from Upcoming to Due to Filed, and to record the PP30 filing when I remit, **so that** I file on time and my remitted and payable figures stay accurate.
**Acceptance criteria**
- Given a Due period with VAT of ฿210,896, when I record its filing as an Admin, then the period becomes Filed, its remitted amount shows ฿210,896, and VAT payable drops by that amount.
- Given a period I try to mark filed that isn't yet Due, when I attempt it, then I'm told only a due period can be marked filed.
- Given a filing recorded after its due date, when it is saved, then it is flagged as late for the record.
- Given an already-filed period, when a later refund or void affects it, then the adjustment carries into the next open period rather than changing the filed return.
- Given an Organizer (not an Admin), when they attempt to record a filing, then the action is denied.

### US-FIN-13 — Export finance ledgers · **Should**
**As an** Event Organizer, **I want** to export the payments, invoices, and VAT ledgers exactly as I've filtered them on screen, **so that** I can reconcile offline and hand a clean file to my accountant.
**Acceptance criteria**
- Given the payments ledger filtered to Refunded, when I export, then the file contains exactly those refunded payments with a VAT-inclusive amount.
- Given the invoice ledger filtered to Overdue, when I export, then the file contains only overdue invoices with reconciling subtotal and VAT columns.
- Given the VAT ledger scoped to a year and status, when I export, then the file lists those periods with reconciling VAT and remitted figures.
- Given an export containing Thai names, when I open it in a spreadsheet, then the names render correctly.
- Given an empty filtered set, when I export, then I'm told there's nothing to export.
_Notes: Payments export is Should; invoice and tax exports are Could this release._

### US-FIN-14 — Keep finance to finance roles · **Must**
**As an** Admin, **I want** the entire Finance area to be invisible to Staff and Attendees, **so that** revenue, payout, and tax data stay restricted to the people accountable for money.
**Acceptance criteria**
- Given a Staff member or Attendee, when they use the console, then the Finance area is not visible or reachable for them.
- Given an Organizer, when they use Finance, then they can view, invoice, and export, but cannot issue refunds, move payout money, void invoices, or record VAT filings.
- Given an Admin, when they use Finance, then all finance actions are available.


---

<a id="epic-e10"></a>

# Epic E10 — Build the Event Program

**Goal (business value):** Give organizers a simple way to lay out their event's day-by-day schedule of sessions and to manage the speaker line-up behind it, so that attendees see a clear, trustworthy programme and decide the event is worth attending. A well-built program lifts registration confidence, drives "add to my schedule" engagement, and gives organizers the evidence they need to re-invite the best speakers.

**Primary users:** Event Organizer, Admin (author and maintain the program); Staff/Team member (read-only, on-site support). Attendees consume the published programme on the event page (covered in the Event Landing epic).

**Success measures:**
- Programme completeness — share of events published with a full agenda and speaker line-up before doors open.
- Scheduling reliability — near-zero room double-bookings or speaker clashes reaching attendees.
- Speaker profile quality — share of speakers with photo, bio and contact complete (drives a credible public page).
- Attendee engagement — "add to my schedule" adoption against published sessions.
- Re-invite insight — speakers carrying a usable satisfaction rating for future planning.

## User stories

### US-PROG-01 — See the event's schedule at a glance  ·  **Must**
**As an** Event Organizer, **I want** to see all of my selected event's sessions laid out on a week-at-a-glance calendar by day and time, **so that** I can understand my programme and spot gaps before the event goes live.

**Acceptance criteria**
- Given my event has sessions across several days, when I open the agenda for that event, then every session appears in the correct day and time position, and none from any other event is shown.
- Given a session, when I look at its block, then I can see its title, its type (shown by a consistent colour), and who is speaking.
- Given an event that has no sessions yet, when I open its agenda, then I see a friendly "no agenda yet" prompt inviting me to add the first session — not an error.
- Given I switch to a different event, when the agenda reloads, then it shows only that event's schedule.

_Notes: Staff/Team members can open this view read-only to support attendees on-site; they cannot change the programme._

### US-PROG-02 — Add a session to the schedule  ·  **Must**
**As an** Event Organizer, **I want** to add a session with a title, day, start and end time, type, room, speaker(s) and an optional description, **so that** attendees can see a structured, informative schedule.

**Acceptance criteria**
- Given I am adding to a chosen event, when I fill in a valid session and save, then it immediately appears on the calendar in the right day and time, coloured by its type, showing the assigned speaker.
- Given I leave the title blank, when I try to save, then I am asked to enter a session title and nothing is saved.
- Given I set an end time that is not after the start, or a time that falls outside the event day's schedulable hours, when I save, then I am told the time is invalid and the session is not saved.
- Given I assign a speaker, when the session saves, then that speaker's session count goes up by one.

_Notes: A session may have more than one speaker; the schedule supports parallel tracks in different rooms at the same time._

### US-PROG-03 — Update a session  ·  **Must**
**As an** Event Organizer, **I want** to open an existing session and change any of its details, **so that** I can react to line-up or timing changes right up to and during the event.

**Acceptance criteria**
- Given a scheduled session, when I move it to a new day, time or room and save with no clash, then the block relocates on the calendar and the change is recorded.
- Given I change a session's type, when I save, then its colour updates to match the new type.
- Given a colleague changed the same session after I opened it, when I try to save, then I am told it changed and asked to reload, so their work is never silently overwritten.
- Given a material change (day, time, room) to a session on a live event, when I save, then I may optionally choose to notify attendees who added it to their schedule; by default no message is sent.

### US-PROG-04 — Remove a session  ·  **Must**
**As an** Event Organizer, **I want** to remove a session with a clear confirmation step, **so that** I can drop cancelled sessions while keeping my speakers in the directory.

**Acceptance criteria**
- Given I choose to remove a session, when the confirmation appears, then it warns me the removal won't automatically notify attendees who bookmarked it, and only removes on my explicit confirm.
- Given a removed session had speakers, when it is deleted, then those speakers stay in the directory and each affected speaker's session count drops by one.
- Given the event is live, when I remove a session, then it stops appearing on the public agenda.
- Given I cancel the confirmation, when I return, then nothing has changed.

### US-PROG-05 — Keep the schedule physically feasible  ·  **Must**
**As an** Event Organizer, **I want** the system to stop me booking the same room for two overlapping sessions, and to warn me when I put the same speaker in two overlapping sessions, **so that** my programme is realistic and never embarrasses us on the day.

**Acceptance criteria**
- Given a room is already booked for an overlapping time that day, when I try to save another session in that room, then I am blocked and told which room and time clashes.
- Given a speaker is already assigned to an overlapping session, when I assign them again, then I am warned and must confirm before it saves ("assign anyway?").
- Given two sessions run at the same time in different rooms with different speakers, when I save, then both are allowed as parallel tracks.
- Given I am editing an existing session, when conflicts are checked, then the session is never flagged as clashing with itself.

### US-PROG-06 — Find sessions in a busy programme  ·  **Should**
**As an** Event Organizer, **I want** to search my agenda by keyword, **so that** I can quickly locate a session across a packed schedule.

**Acceptance criteria**
- Given sessions with different titles and types, when I type a keyword, then only sessions matching that keyword by title or type stay visible.
- Given I clear the search, when the box is empty, then all sessions are shown again.
- Given no sessions match, when I search, then the calendar still shows its layout with nothing highlighted — not an error.

_Notes: Search is scoped to the currently selected event and works for both English and Thai text._

### US-PROG-07 — Switch between week and month views  ·  **Could**
**As an** Event Organizer, **I want** to toggle between a detailed week view and a lighter month overview, and move between weeks, **so that** I can both fine-tune time slots and see the bigger picture of a multi-week programme.

**Acceptance criteria**
- Given the week view, when I switch to month, then I see a day-by-day overview I can drill back into; switching to week returns me to the time-slot layout.
- Given any week, when I move to the previous or next week, then the dates and month label shift accordingly while staying on the same event.
- Given I navigate beyond the event's scheduled weeks, when the view loads, then I see an empty week with the correct dates — never sessions from another event.

### US-PROG-08 — Browse and filter the speaker directory  ·  **Must**
**As an** Event Organizer, **I want** a searchable, event-filterable directory of speakers showing their key details at a glance, **so that** I can manage my line-up and check who is speaking where.

**Acceptance criteria**
- Given a set of speakers, when I open the directory, then I see each speaker's name, role, contact, session count and rating, and can switch between a card and a list layout.
- Given I filter by a specific event, when the list refreshes, then only that event's speakers appear and the count reflects the filtered total.
- Given I search by name, role or email, when I type, then only matching speakers show and the list returns to the first page.
- Given no speakers match, when the list refreshes, then I see a "no speakers found" prompt, not an error.

_Notes: Staff/Team members can browse the directory read-only; add/edit/delete controls appear only for organizers and admins._

### US-PROG-09 — Add a speaker to the line-up  ·  **Must**
**As an** Event Organizer, **I want** to add a speaker with their photo, name, role, contact details, bio, topic, event and social links, **so that** they are represented accurately on the public event page.

**Acceptance criteria**
- Given a new speaker's details, when I save, then a speaker profile is created, assigned to the chosen event, starting at zero sessions and shown as unrated until feedback exists.
- Given I leave the name or email blank, or enter an invalid email, when I save, then I am asked to correct it and nothing is saved.
- Given the email already belongs to another speaker, when I save, then I am told a speaker with that email already exists and nothing is saved.
- Given I upload a photo that is the wrong type or too large, when I save, then I am told the accepted format and size limit and it is not accepted.
- Given a badly formed website or social link, when I save, then I am asked to enter a valid link rather than have it silently dropped.

### US-PROG-10 — Keep a speaker's profile up to date  ·  **Must**
**As an** Event Organizer, **I want** to edit a speaker's details, **so that** the public page always reflects their current title, company, bio and links.

**Acceptance criteria**
- Given a speaker whose company changed, when I update and save, then their card shows the new role and the change is recorded.
- Given I change the email to one already used by another speaker, when I save, then I see the duplicate-email message and nothing is saved.
- Given a colleague changed the same speaker after I opened it, when I save, then I am told it changed and asked to reload, so no work is silently lost.
- Given the speaker appears on a live event, when I save an update, then the public speakers section reflects it.

### US-PROG-11 — Remove a speaker  ·  **Must**
**As an** Event Organizer, **I want** to delete a speaker with a clear, confirmed warning, **so that** I can remove someone who dropped out without losing the sessions they were attached to.

**Acceptance criteria**
- Given a speaker assigned to several sessions, when I confirm deletion, then the speaker disappears from the directory and those sessions remain but no longer list them.
- Given the confirmation, when it appears, then it warns me the speaker will be removed from all their sessions and that the action cannot be undone.
- Given the speaker was shown on a live event, when they are deleted, then they no longer appear in the public agenda or speakers section.
- Given I cancel the confirmation, when I return, then the speaker and all their assignments are unchanged.

### US-PROG-12 — Assign speakers to their sessions and balance the line-up  ·  **Should**
**As an** Event Organizer, **I want** to assign speakers to sessions and see how many sessions each speaker carries, **so that** I can balance the programme and avoid overloading anyone.

**Acceptance criteria**
- Given a speaker with one session, when I add them to a second session, then their session count reads two and the session shows their initials.
- Given a session, when I assign one or more speakers, then the assignments are saved and the session's displayed speaker updates.
- Given a speaker who belongs to a different event, when I try to assign them, then I am told they belong to another event and the assignment is refused.
- Given I assign a speaker to a session overlapping one they already have, when I save, then I get the double-booking warning and must confirm.

### US-PROG-13 — See how speakers were rated  ·  **Should**
**As an** Admin, **I want** each speaker to show an aggregate rating drawn from post-session attendee feedback, **so that** I can decide who to re-invite to future events.

**Acceptance criteria**
- Given a speaker whose feedback averages 4.9, when I view their card, then it shows "4.9".
- Given a brand-new speaker with no feedback, when I view their card, then it shows an unrated indicator, not a zero score.
- Given a speaker with ratings, when I open their reviews, then I can see the underlying feedback that produced the score.
- Given new feedback arrives, when I next view the speaker, then their rating reflects the latest results.

_Notes: Ratings are read-only here and come from the Feedback/Insights module; they are never hand-edited in the program._


---

<a id="epic-e11"></a>

# Epic E11 — Organizer Home & Dashboard

**Goal (business value):** Give organizers a single daily landing place that tells them what needs attention right now (today's sign-ups, meetings, and alerts) and a clear analytics view of how their events are performing (revenue, registrations, ticket mix, and events at risk of selling out) — so they can act fast without digging through every module and never lose money to a missed alert or an overlooked sell-out.

**Primary users:** Event Organizer, Admin, Staff/Team member (Attendees have no access).

**Success measures:**
- Time-to-triage: organizers act on today's alerts and sign-ups without opening multiple screens.
- Fewer at-risk events reaching their date behind on sales (early "selling fast" and progress signals acted on).
- Revenue visibility: organizers can read short-term momentum and long-term trend at a glance.
- Check-in rate and capacity-fill awareness improves per-event operational decisions.
- Fewer sell-outs missed and fewer declined payments left unresolved.

## User stories

### US-DASH-01 — Daily operations home  ·  **Must**
**As an** Event Organizer, **I want** a personalized home screen that greets me and gathers today's most important activity in one place, with a quick way to start a new event, **so that** I can size up my day and act without hunting through separate modules.
**Acceptance criteria**
- Given I sign in, when I land on the home screen, then I see a time-of-day greeting addressed to me by name in my chosen language (English or Thai).
- Given the local time in Bangkok is morning, afternoon, or evening, when I open home, then the greeting reflects Bangkok time regardless of where my device is.
- Given I am on the home screen, when I choose "New event", then I am taken to the event creation flow.
- Given I am an Attendee (not a team member), when I try to open the admin home, then I am refused access.
_Notes: The home screen only summarizes and links out — it never changes any records itself._

### US-DASH-02 — Today's registrations at a glance  ·  **Must**
**As an** Event Organizer, **I want** to see how many people registered today and a preview of the most recent sign-ups, **so that** I can confirm registrations are flowing and jump straight to the full list when something looks off.
**Acceptance criteria**
- Given people registered today (Bangkok calendar day), when I view home, then a count badge shows today's total and the newest sign-ups are previewed with attendee name, their event and ticket type, and the time.
- Given no one has registered yet today, when I view home, then the panel clearly shows "no registrations yet today".
- Given I want the full picture, when I choose "View all registrations", then I am taken to the registrations list.
_Notes: Attendee names are personal data — this panel is only shown to team members allowed to view registrations._

### US-DASH-03 — Today's meetings  ·  **Should**
**As an** Event Organizer, **I want** to see the meetings scheduled for today with their times and who I am meeting, **so that** I stay ahead of my day and can jump to scheduling when needed.
**Acceptance criteria**
- Given meetings are scheduled for today, when I view home, then a count badge shows how many and the soonest meetings are previewed earliest-first with title, time range, and the counterpart.
- Given no meetings are scheduled today, when I view home, then the panel shows "no meetings scheduled today".
- Given I want to arrange one, when I choose "Schedule meeting", then I am taken to the meeting creation flow.

### US-DASH-04 — Upcoming events progress  ·  **Should**
**As an** Event Organizer, **I want** my next events shown with how many days remain and how full they are, **so that** I can spot an event falling behind on sales before it runs out of runway.
**Acceptance criteria**
- Given an event is coming up, when I view home, then its card shows a days-remaining chip, a preview of who's attending, and a progress bar for how full it is.
- Given an event is filled to a given share of its capacity, when I view its card, then the progress reflects that share (for example, 83 of 100 seats reads as 83%).
- Given an event starts today, when I view its card, then the chip reads "Today".
- Given I have no upcoming events, when I view home, then the panel shows "no upcoming events".
_Notes: Shows the soonest-starting events first, a small number at a time, with a link to the full events list._

### US-DASH-05 — Active-events activity share  ·  **Could**
**As an** Event Organizer, **I want** a visual breakdown of how my currently active events compare on activity, **so that** I can see at a glance which events are driving the most sign-ups.
**Acceptance criteria**
- Given I have active events, when I view home, then a ring shows each event's share of activity with a color-keyed legend of event names and the total number of active events in the center.
- Given I have no active events, when I view home, then the panel shows "no active events".
- Given I want more detail, when I choose "See all", then I am taken to the events list.

### US-DASH-06 — Operational alerts  ·  **Must**
**As an** Event Organizer, **I want** a running list of things that need action — approvals, declined payments, unconfirmed speakers, pending replies — each linking straight to where I can fix it, **so that** nothing that costs me money or attendees slips through.
**Acceptance criteria**
- Given there are unresolved issues I can act on, when I view home, then each appears with a clear severity indicator and text, with the most urgent shown first, and a link to the right place to resolve it.
- Given I click an alert, when I resolve the issue in that module, then the alert is gone the next time home refreshes.
- Given an alert concerns finance (for example a declined payment), when I do not have finance access, then that alert is not shown to me.
- Given there is nothing outstanding, when I view home, then the panel shows "you're all caught up".
_Notes: Alerts only point me to the right module — resolving always happens there, never on the home screen._

### US-DASH-07 — Website template shortcuts  ·  **Could**
**As an** Event Organizer, **I want** quick shortcuts to the available event landing-page templates, **so that** I can move quickly from my overview into building an event's public page.
**Acceptance criteria**
- Given the templates panel is shown, when I choose a template, then I am taken to the landing-pages area to work with it.
- Given the available templates change, when I reload home, then the shortcuts reflect the current set.

### US-DASH-08 — Performance KPIs at a glance  ·  **Must**
**As an** Admin, **I want** headline numbers for registrations, ticket revenue, upcoming events, check-in rate, and capacity filled — each with how it moved versus the previous period — **so that** I can judge the health of my events in seconds.
**Acceptance criteria**
- Given I open the dashboard, when the KPI cards load, then I see total registrations, ticket revenue (in ฿), upcoming events, check-in rate, and capacity filled, each with a change versus the previous comparable period.
- Given a metric improved, when I read its card, then the change shows an upward, positive-colored indicator; given it worsened (for example, check-in rate falling), then it shows a downward, warning-colored indicator.
- Given I do not have finance access, when I view the dashboard, then the ticket revenue figure is not disclosed to me while the other cards render normally.
- Given a metric has no data for the period, when I view its card, then it shows a neutral, empty state rather than a misleading value.
_Notes: Revenue is shown net of VAT and refunds; the 7% VAT is tracked separately and not surfaced here._

### US-DASH-09 — Revenue trend with time-range toggle  ·  **Must**
**As an** Event Organizer, **I want** a revenue chart I can switch between Week, Month, and Year, with the period total and how it compares to the previous period, **so that** I can read both short-term momentum and long-term trend from one view.
**Acceptance criteria**
- Given I open the dashboard, when the revenue view loads, then it defaults to the yearly view showing the period total and its change versus the previous period.
- Given I switch to Week or Month, when I make the selection, then the chart and its headline total and change update to that range without reloading the page.
- Given there is no revenue in the selected period, when I view the chart, then it shows an empty (zero) result with a neutral change.
- Given I do not have finance access, when I view the dashboard, then the revenue section is not shown to me at all.
_Notes: For the yearly view, the revenue total agrees with the ticket-revenue KPI for the same scope and period._

### US-DASH-10 — Registrations by ticket type  ·  **Should**
**As an** Event Organizer, **I want** to see how my total registrations split across ticket types, **so that** I understand which tiers are selling and can adjust pricing or promotion.
**Acceptance criteria**
- Given registrations exist, when I view the dashboard, then a breakdown shows each ticket type's share with its count and percentage, and the total in the center, largest share first.
- Given the total shown equals the total registrations KPI, when I compare the two, then they match for the same scope.
- Given the ticket mix changes, when the dashboard refreshes, then the breakdown updates to match.
- Given there are no registrations, when I view the panel, then it shows "no registrations yet".

### US-DASH-11 — Tickets selling fast  ·  **Should**
**As an** Event Organizer, **I want** a list of ticket types running low on inventory, with how many are left and how urgent it is, **so that** I can add capacity or promote before a sell-out.
**Acceptance criteria**
- Given ticket types are low on remaining inventory, when I view the dashboard, then they are listed scarcest-first with event context and a remaining count colored by urgency (critical, warning, or normal).
- Given a ticket type is critically low, when I view it, then its remaining count is highlighted in the most urgent color.
- Given I want to act, when I choose "Manage", then I am taken to ticket management for that inventory.
- Given nothing is close to selling out, when I view the panel, then it shows "no tickets running low".
_Notes: Because one booking can hold up to 8 seats, anything at or below 8 remaining is flagged as a possible sell-out within a single booking._

### US-DASH-12 — Recent registrations table  ·  **Must**
**As an** Event Organizer, **I want** a table of the latest registrations showing attendee, event, amount, payment status, and time, **so that** I can confirm money and sign-ups are flowing without leaving the overview.
**Acceptance criteria**
- Given recent registrations exist, when I view the dashboard, then they are listed newest-first with attendee, event, amount (฿), a Paid / Pending / Refunded status badge, and the time.
- Given a walk-in registration was taken on-site, when it appears in the table, then it follows the same amount and status rules (and may show as Pending until settled).
- Given I do not have finance access, when I view the table, then the amount is hidden while attendee, event, status, and time still show.
- Given I want the full list, when I choose "View all", then I am taken to the registrations list.
_Notes: Attendee names are only shown to team members allowed to view registrations._

### US-DASH-13 — Trustworthy, always-current overview  ·  **Must**
**As an** Event Organizer, **I want** every figure on home and the dashboard to stay current, stay consistent with each other, and only show what I'm allowed to see, **so that** I can trust the numbers and act on them with confidence.
**Acceptance criteria**
- Given I am viewing home or the dashboard, when a new registration is confirmed, then within a short refresh window the registration total, ticket-type breakdown, and recent-registrations table all update and agree with each other.
- Given operational feeds (today's sign-ups, alerts, selling-fast) versus longer-term analytics, when I view them, then the operational feeds refresh near-real-time while analytics refresh on a slightly longer cadence, and I can force a manual refresh.
- Given a single panel can't load, when the page renders, then only that panel shows an "unavailable / retry" state while every other panel works normally.
- Given a figure I'm not permitted to see (finance or attendee personal data), when I view the surface, then it is hidden — never shown as zero or a placeholder value.
- Given all dates, "today", days-remaining, and times, when I read them, then they are in Bangkok time and formatted in my language (English or Thai).
_Notes: Home and the dashboard never change data — every actionable item is a link into the module that owns it, which enforces its own permissions and records the change._


---

<a id="epic-e12"></a>

# Epic E12 — Coordinate Meetings

**Goal (business value):** Give organizers one place to schedule and keep track of every operational conversation around an event — with speakers, sponsors, venues, vendors and internal teams — so that meetings are set up in seconds, everyone gets an invite (and a video link when needed), and nothing falls through the cracks. Less time juggling calendars means events get organized faster and more reliably.

**Primary users:** Event Organizer, Admin. (Staff and Attendees have no access to Meetings.)

**Success measures:**
- Meetings scheduled per active event (adoption of the console vs. organizing calls elsewhere).
- Share of meetings that reach the guest as a delivered invite.
- Video-meeting join rate / meeting no-show rate.
- Median time to schedule a meeting.
- Share of meetings that stay in sync with the organizer's calendar (fewer stale or missing invites).

---

## User stories

### US-MTG-01 — See meetings organized by Today / Upcoming / Past  ·  **Must**
**As an** Event Organizer, **I want** all my event meetings laid out in Today, Upcoming and Past groups with a live count on each, **so that** I can instantly see what needs my attention today and what is already behind me.
**Acceptance criteria**
- Given a meeting scheduled for today (Bangkok time), when I open Meetings, then it appears under **Today**, marked as today, with its time shown as "Today · start – end".
- Given several meetings on the same future day, when I look at **Upcoming**, then they are listed earliest-start first.
- Given I switch between the All / Today / Upcoming / Past tabs, then each tab shows how many meetings fall in that group.
- Given each meeting row, then I can see at a glance its title, who it is with and their role, the related event, the meeting type, whether it is video / in person / phone, and where it takes place (join link, venue, or "Phone call").

_Notes: Which group a meeting falls into is always worked out from its date in Bangkok time, so it is correct no matter where the viewer is._

### US-MTG-02 — Find a specific meeting in a long list  ·  **Must**
**As an** Event Organizer, **I want** to search my meetings by title, person, role, event or type and filter by meeting type, with the list paged into manageable chunks, **so that** I can find one conversation quickly even when I have many booked.
**Acceptance criteria**
- Given I type a search term, when it is applied, then only meetings whose title, person, role, event or type match remain, and the list returns to the first page.
- Given I choose a type (Venue, Sponsor, Vendor, Speaker or Internal), then only meetings of that type are shown, combined with whichever time tab I am on.
- Given search and filters return nothing, then I see a clear "No meetings match your filters" message.
- Given a long list, when I choose how many rows to show per page and move between pages, then I always see a "showing X–Y of N meetings" summary.

_Notes: Search works equally for Thai and English content._

### US-MTG-03 — Schedule a meeting with a speaker, sponsor, venue or vendor  ·  **Must**
**As an** Event Organizer, **I want** to book a meeting in a few simple fields — title, date, start/end time, type, mode, who it is with, their email, the related event and any notes — **so that** I can set up calls and walkthroughs without leaving the console.
**Acceptance criteria**
- Given I open the schedule panel and fill in the required details, when I save, then a new meeting is created with status Scheduled and appears in the correct time group.
- Given I leave the title blank or too short, choose a date in the past, or set an end time that is not after the start, then I am told exactly what to fix before I can save.
- Given I set the mode to In person, then the meeting shows the related event's venue as its location; given Phone, then it shows "Phone call".
- Given I double-submit the form by accident, then only one meeting is created, not two.

_Notes: A meeting can be tied to a specific event or to "General (all events)". A guest email is needed so the invite can be sent._

### US-MTG-04 — Have invites, video links and reminders handled automatically  ·  **Must**
**As an** Event Organizer, **I want** scheduling a meeting to automatically send the guest a calendar invite, generate a Google Meet link for video meetings, and set a 15-minute reminder, **so that** I never create or paste join links by hand and the guest always has what they need.
**Acceptance criteria**
- Given I schedule a video meeting, when I save, then a Google Meet link is created for me, added to the meeting, and included in the invite sent to the guest.
- Given any meeting I schedule, then the guest receives a calendar invite for the correct Bangkok time and both of us get a 15-minute reminder before it starts.
- Given my calendar is not connected, when I save a meeting, then the meeting is still kept and I am prompted to connect my calendar to send the invite and link — nothing is lost.
- Given the calendar service is briefly unavailable, when I save a meeting, then the meeting is kept and marked as not yet synced, with a way to retry, and the invite goes out once the service is reachable.

_Notes: The resilience/retry behaviour is the safety net; the automatic invite and Meet link are the core value of scheduling from the console._

### US-MTG-05 — Reschedule or edit a meeting  ·  **Must**
**As an** Event Organizer, **I want** to change a meeting's time or details and have the guest's invite updated in place, **so that** the invitation on everyone's calendar is never stale.
**Acceptance criteria**
- Given I move a video meeting to a new time, when I save, then the guest's existing invite is updated (not duplicated), the reminder shifts to the new start, and the guest is re-notified of the new time.
- Given I change the mode from video to in person, when I save, then the Meet link is removed and the location shows the event's venue instead.
- Given someone else has changed the same meeting since I opened it, when I try to save, then I am told the meeting changed and asked to reload, so no one's edit is silently overwritten.
- Given a meeting is already cancelled, then it cannot be edited.

### US-MTG-06 — Cancel a meeting  ·  **Should**
**As an** Event Organizer, **I want** to cancel a scheduled meeting, optionally with a reason, and have the guest told it is off, **so that** no one shows up to a meeting that is no longer happening.
**Acceptance criteria**
- Given a scheduled meeting, when I cancel and confirm, then it is marked Cancelled, removed from the guest's calendar, and the guest receives a cancellation notice.
- Given I add a reason, then it is included in the cancellation the guest sees.
- Given a meeting is cancelled, when the list loads, then it no longer counts toward Today or Upcoming and offers no Join or Edit.
- Given I open the cancel confirmation but dismiss it, then nothing changes.

_Notes: Cancelling keeps the meeting on record (as Cancelled) rather than erasing it; cancel is only offered for scheduled meetings that have not yet happened._

### US-MTG-07 — Join a video meeting in one click  ·  **Must**
**As an** Event Organizer running between tasks, **I want** a Join button on today's and upcoming video meetings, **so that** I can jump straight into the call from the console.
**Acceptance criteria**
- Given a today or upcoming video meeting with a ready link, when I click Join, then the meeting opens in a new tab.
- Given a past video meeting, then no Join button is shown.
- Given a video meeting whose link is not ready yet, then Join is unavailable with a hint to retry the calendar sync.

### US-MTG-08 — Connect and manage the workspace Google Calendar  ·  **Should**
**As an** Admin, **I want** to connect, reconnect or disconnect the workspace's Google Calendar, **so that** meeting sync, invites and reminders are governed centrally and I control access.
**Acceptance criteria**
- Given I complete the Google connection, when it succeeds, then the workspace is marked connected and the Meetings header shows a connected status.
- Given the workspace is disconnected, when organizers schedule meetings, then those meetings are saved but flagged as not synced until the calendar is reconnected.
- Given the connection later expires or is revoked, then I am prompted to reconnect rather than sync failing silently.
- Given a non-Admin opens Meetings, then no connect/disconnect control is available to them.

_Notes: Disconnecting stops future sync but does not remove meetings already on people's calendars._

### US-MTG-09 — Be warned about overlapping meetings  ·  **Could**
**As an** Event Organizer, **I want** a gentle warning when a new meeting overlaps one I already have, **so that** I can avoid accidental double-booking while still deciding for myself.
**Acceptance criteria**
- Given I schedule a meeting that overlaps another of mine at the same time, when I save, then I see a "this overlaps another meeting — schedule anyway?" warning.
- Given I choose to continue, then the meeting is saved normally.

_Notes: The warning never blocks saving — the organizer stays in control._


---

<a id="epic-e13"></a>

# Epic E13 — Measure Performance (Reports)

**Goal (business value):** Give organizers and admins a single, trustworthy place to see how their events perform — money collected, money settled, sign-ups, show-ups, and promotion payback — across all events or one, for any period, and to take those numbers offline to share with finance and stakeholders.

**Primary users:** Event Organizer, Admin, Staff/Team member (registration and attendance figures only).

**Success measures:**
- Organizers can answer "how are we doing?" without opening each event one by one (time-to-insight).
- Reported revenue reconciles with money that actually settles to the bank (finance trust / zero reconciliation disputes).
- Reports are used to steer decisions: no-show rate, refund rate, and promotion payback are actively tracked period-over-period.
- Numbers can be exported and shared in the format finance needs (self-serve export adoption).

---

## User stories

### US-RPT-01 — Workspace health at a glance  ·  **Should**
**As an** Event Organizer, **I want** a single overview showing revenue, registrations, attendance rate, average ticket price, and refund rate for a period I choose — with a revenue trend I can switch between 7 days, 30 days, 90 days, and the year — **so that** I can read the health of all my events in one screen and see whether things are improving or slipping.
**Acceptance criteria**
- Given I open the overview, when it loads, then I see the current revenue, registration count, attendance rate, average ticket price, and refund rate for the default period, each with an up-or-down change versus the previous equal period.
- Given I switch the revenue trend from "Year" to "7 days", when the view updates, then the chart, its headline total, and the "vs previous period" change all recompute for the shorter window.
- Given a favorable movement (revenue up, or refund rate down), when the change is shown, then it appears in the positive colour; an unfavorable movement appears in the warning colour — direction of "good" fits each metric, not just the sign of the number.
- Given there was no revenue in the chosen window, when the panel renders, then it shows a zero total and a neutral "—" change instead of an error.
_Notes: "Revenue" here means money collected minus refunds within the period (VAT-inclusive), and it should match the Income report's Net for the same scope._

### US-RPT-02 — Filter and search every report  ·  **Should**
**As an** Admin, **I want** to narrow any report by date range, by a specific event or all events, by status, and by a free-text search, **so that** I can focus on exactly the slice I need without wading through everything.
**Acceptance criteria**
- Given any report, when I pick one event and type a search term, then the totals, charts, and table all recompute to show only matching rows for that event — not just the table.
- Given I set an end date earlier than the start date, when I apply it, then I'm told "End date must be on or after the start date" and no broken result is shown.
- Given I choose a date span longer than 24 months, when I apply it, then it is trimmed to 24 months with a note explaining the limit.
- Given a filter combination that matches nothing, when it applies, then I see a friendly empty message and zeroed figures, not an error.
- Given I change any filter, when the results refresh, then I'm returned to the first page of results.

### US-RPT-03 — Registration mix and sales-channel breakdown  ·  **Could**
**As an** Event Organizer, **I want** to see how registrations split by ticket type, which events draw the most sign-ups, and which channels (website, email, social, partner) bring people in, **so that** I understand where my audience and demand actually come from.
**Acceptance criteria**
- Given the overview, when the ticket-type breakdown renders, then each ticket type shows its share of total registrations, the shares add up to the whole, and the centre shows the same registration total as the headline registrations figure.
- Given the "registrations by event" list, when it renders, then it shows my top events by sign-ups with a visual bar for relative size.
- Given the "sales by channel" breakdown, when it renders, then each channel shows its share of registrations and the shares add up to the whole.
- Given I filter to a single event, when the breakdowns refresh, then they reflect only that event.

### US-RPT-04 — Rank and explore event performance  ·  **Should**
**As an** Event Organizer, **I want** a ranked list of my events by registrations — with revenue, attendance rate, and status — that I can search, filter by status, and click into an event's detail, **so that** I can quickly spot my best and worst performers and dig deeper.
**Acceptance criteria**
- Given the full event-performance report, when it loads, then events are listed best-first by registrations, showing registrations, revenue, attendance rate, and a status badge (Upcoming, Live, or Completed).
- Given I filter to status "Live", when it applies, then only live events remain, still ranked by registrations, starting from the first page.
- Given an event that hasn't happened yet, when its row renders, then attendance shows "—" rather than a misleading zero.
- Given I click an event's name, when the link opens, then I land on that event's detail page.
- Given the overview's "top events" summary, when I click "View all", then I reach this full ranked report.

### US-RPT-05 — Income and reconciliation report  ·  **Should**
**As an** Admin, **I want** an income report that separates gross revenue, refunds, processing fees, and net per event, **so that** I can reconcile what was collected against what actually settles.
**Acceptance criteria**
- Given any event row, when it renders, then Net equals Gross minus Refunds minus Fees for that row.
- Given the summary tiles, when they render, then the Net tile equals Gross minus Refunds minus Fees overall, and it matches the overview's revenue figure for the same period and events.
- Given prices include 7% VAT, when I need it for tax, then the report lets me see the VAT portion of gross for reconciliation.
- Given a free event with no revenue, when its row renders, then all money columns show ฿0 rather than being hidden or erroring.
- Given no income matches my filters, when the report renders, then I see "No income for this selection" and zeroed tiles.

### US-RPT-06 — Transaction ledger for investigations  ·  **Could**
**As an** Admin, **I want** a searchable ledger of every payment and refund — with its reference, date, attendee, event, method, amount, and status — **so that** I can investigate a specific charge, refund, or dispute.
**Acceptance criteria**
- Given a refund of ฿1,250, when its row renders, then the amount shows as "-฿1,250" in the negative colour with a "Refund" label, while payments show as positive.
- Given the summary tiles, when they render, then I see the number of transactions, total successful payments, number of refunds, and the payment success rate.
- Given a failed charge, when it appears, then it is clearly marked as failed and is excluded from the payments total but still counted when calculating success rate.
- Given a refund is issued, when the ledger updates, then the refund appears as a new entry and the original payment is marked "Refunded" — the original amount is never rewritten.
- Given I click a transaction reference, when it opens, then I reach that payment's detail where refund actions live (this report itself never changes money).

### US-RPT-07 — Payouts and settlement visibility  ·  **Should**
**As an** Event Organizer, **I want** a payouts report showing what has been paid to our bank, what is in transit, and what is still pending, **so that** I know exactly how much money has actually reached the organization's account.
**Acceptance criteria**
- Given the payouts report, when it loads, then each payout shows its reference, date, bank, the events it covers, amount, and status (Paid, In transit, or Pending).
- Given the summary tiles, when they render, then I see total paid out, total pending, total in transit, and the average payout.
- Given any payout row, when the bank is shown, then only the masked last four digits of the account appear — never the full account number.
- Given a payout that covers several events, when I filter to any one of those events, then that payout still appears.
- Given no payouts match my filters, when the report renders, then I see "No payouts for this selection".

### US-RPT-08 — Registrations report by event and status  ·  **Should**
**As a** Staff/Team member, **I want** a per-event breakdown of registrations into confirmed, pending, waitlist, and cancelled, **so that** I can judge sign-up quality and see the true picture for each event.
**Acceptance criteria**
- Given any event row, when it renders, then confirmed, pending, waitlist, and cancelled add up exactly to that event's total registrations.
- Given the summary tiles, when they render, then the total matches the overview's registrations figure for the same period and events.
- Given I am a Staff member, when I open this report, then I can view it fully (registration data is available to my role).
- Given no registrations match my filters, when the report renders, then I see "No registrations for this selection".

### US-RPT-09 — Attendance and no-show report  ·  **Should**
**As an** Event Organizer, **I want** a per-event view of registered vs checked-in, no-shows, attendance rate, and on-time rate, **so that** I can measure show-up quality and reduce no-shows over time.
**Acceptance criteria**
- Given an event with 400 registered and 320 checked in, when its row renders, then no-shows show as 80 and attendance rate as 80%.
- Given an upcoming event with no check-ins yet, when its row renders, then attendance shows "—" and that event is left out of the overall attendance rate.
- Given the summary tiles, when they render, then I see total checked in, overall attendance rate, total no-shows, and the on-time share — and the attendance rate matches the overview's attendance figure for the same scope.
- Given no attendance matches my filters, when the report renders, then I see "No attendance for this selection".

### US-RPT-10 — Promotion and discount payback  ·  **Could**
**As an** Event Organizer, **I want** a discounts report showing, per code, how many times it was redeemed, how much discount was given, and the revenue it influenced, **so that** I can judge which promotions actually drive sales.
**Acceptance criteria**
- Given a discount code, when its row renders, then I see its type (e.g. "25% off" or "฿200 off"), redemptions, total discount given, revenue influenced, and status (Active, Scheduled, or Expired).
- Given the summary tiles, when they render, then I see the count of active codes, total redemptions, total discount given, and total revenue influenced.
- Given a scheduled code that hasn't started, when its row renders, then redemptions, discount, and revenue all read zero and it is not counted among active codes.
- Given an expired code, when it renders, then it is marked expired but its historical redemptions, discount, and revenue still count in the totals.

### US-RPT-11 — Export the current view  ·  **Should**
**As an** Admin, **I want** to export the report exactly as I've filtered it to CSV, Excel, or PDF, **so that** I can share the numbers with finance and stakeholders offline.
**Acceptance criteria**
- Given I've filtered a report to one event, when I export, then the file contains only that event's rows plus the summary figures I see on screen — nothing more.
- Given I choose Excel, when the file is produced, then it contains a summary of the key figures plus the detailed rows, with money and percentages properly formatted.
- Given I choose CSV, when the file is produced, then Thai text reads correctly and dates and raw numbers are in a machine-friendly form.
- Given I choose PDF, when the file is produced, then it is a tidy, branded, readable document showing the key figures, the table, the applied filters, and the generation time.
- Given a very large export, when I request it, then I'm told it's being prepared and I'm notified with a secure, time-limited download link when it's ready.
- Given I click the same export twice within a few seconds, when the second click is handled, then I get the same file rather than a duplicate.
_Notes: Every export reflects the exact figures on screen at the moment of request; the file records that timestamp._

### US-RPT-12 — Role-appropriate access and safe finance sharing  ·  **Should**
**As an** Admin, **I want** finance reports limited to finance-permitted roles, registration and attendance reports available to staff, and every export recorded, **so that** sensitive money and attendee data is only seen and taken by the right people.
**Acceptance criteria**
- Given a Staff member with registration access only, when they open the registrations or attendance report, then it works; but when they try a finance report (income, transactions, payouts, discounts, event performance), then they are not shown it and the option is hidden or disabled.
- Given a Staff member on a report containing attendee names, when they try to export, then they are told they don't have permission to export attendee data and no file is produced.
- Given anyone exports any report, when the file is produced, then a record of who exported what, when, and for which scope is kept.
- Given an attendee (a non-admin) tries to reach any report, when they attempt it, then they are refused access to the admin console entirely.
- Given the same period and events, when I compare screens, then the totals agree across reports (overview registrations equals the registrations report total; overview revenue matches income net) so I can trust the numbers.
_Notes: Viewing a report never changes any event, registration, payment, payout, or discount — reports are read-only; bank account numbers are always masked to the last four digits._


---

