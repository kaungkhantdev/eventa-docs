# Non-Functional (Quality) Requirements — Product Owner View

| | |
|---|---|
| **Author** | Product Owner |
| **Format** | Quality requirements — story-style, MoSCoW, measurable targets |
| **Version** | 1.0 |
| **Date** | 2026-07-23 |
| **Scope** | 46 quality requirements |

These requirements define the quality bar Eventa must clear to earn the trust of Thai event organizers and their attendees. Functional features win a customer; these qualities keep them. On our platform, real money changes hands, real seats sell out in minutes, and real personal data is entrusted to us under Thai law — so a page that feels slow, a checkout that double-charges, an on-sale that oversells, or a privacy slip does direct, measurable business damage. Every item below is written as an outcome the business can observe and hold us to, with a MoSCoW priority reflecting how much revenue, reputation, or compliance rides on it. "Must" items are non-negotiable for launch; "Should" items strongly shape competitiveness; "Could" items are valued polish.

---

## Performance & Responsiveness

### NFR-PERF-01 — Pages feel instant  ·  **Must**
**As an** attendee, **I need** event and discovery pages to feel instant, **so that** I don't abandon before I ever reach registration.
**Target:** Public and portal pages become usable within **2.5 seconds for 95%** of real visits, and noticeably faster on repeat visits; nothing feels janky as it loads.

### NFR-PERF-02 — Checkout is quick and confident  ·  **Must**
**As an** attendee, **I need** payment to confirm quickly, **so that** I trust the transaction and complete my purchase instead of second-guessing it.
**Target:** After I pay, the PromptPay QR or card confirmation comes back within **2 seconds for 95%** of checkouts (excluding time I spend on a bank/3-D Secure challenge).

### NFR-PERF-03 — Organizer lists and searches respond fast  ·  **Must**
**As an** organizer, **I need** attendee, registration, and payment lists to open and search fast, **so that** I can run a busy event desk without waiting on the tool.
**Target:** Any list page or directory search returns its first page of results within about **half a second for 95%** of uses, even on lists in the tens of thousands.

### NFR-PERF-04 — Reports load without a wait  ·  **Should**
**As an** organizer, **I need** dashboards and charts to appear promptly, **so that** I can make on-the-day decisions on ticket sales and attendance.
**Target:** Overview, income, registration, and attendance charts render within about **1.5 seconds for 95%** of views.

### NFR-PERF-05 — Exports don't hold me hostage  ·  **Should**
**As an** organizer, **I need** exports to be quick or to run in the background, **so that** pulling a guest list or finance report never freezes my work.
**Target:** Exports up to **5,000 rows finish within 10 seconds**; larger exports are handed off and delivered via a ready-to-download link within **2 minutes for 95%** of jobs.

### NFR-PERF-06 — Check-in keeps the door moving  ·  **Must**
**As** event check-in staff, **I need** each scan to confirm almost instantly, **so that** the entry queue keeps flowing during the arrival rush.
**Target:** Each valid scan shows an on-screen confirmation within **half a second**, and a single station reliably handles **30 or more scans per minute**.

---

## Reliability & Availability

### NFR-REL-01 — The revenue path is always open  ·  **Must**
**As an** organizer, **I need** registration and checkout to be available around the clock, **so that** I never lose sales to downtime during an on-sale window.
**Target:** Registration and checkout are up **at least 99.9% of every month** (under ~43 minutes of downtime); the admin console at least 99.5%.

### NFR-REL-02 — Selling and check-in survive partial outages  ·  **Must**
**As an** organizer, **I need** ticket-selling and check-in to keep working even when reporting or messaging is temporarily down, **so that** a minor fault never blocks revenue or entry.
**Target:** When non-critical features fail, registration, payment, and check-in stay fully operational and the affected feature shows a clear notice — never a hard error on the money path.

### NFR-REL-03 — Check-in works through spotty venue Wi-Fi  ·  **Must**
**As** check-in staff, **I need** scanning to keep working through brief network drops, **so that** a weak venue connection never stalls the door.
**Target:** The scanner keeps validating and queuing check-ins offline and reconciles automatically on reconnect, with **zero duplicate or missed entries** and repeat scans rejected.

### NFR-REL-04 — No double charges, no duplicate registrations  ·  **Must**
**As an** attendee, **I need** a retried or flaky payment to never charge me twice or duplicate my order, **so that** I trust the platform with my money.
**Target:** **Zero** double-charges, double-refunds, or duplicate registrations across payment retries and network glitches.

### NFR-REL-05 — Every paid ticket is accounted for  ·  **Must**
**As an** organizer, **I need** every confirmed payment to reliably produce exactly one ticket and one financial record, **so that** my seat counts and revenue always reconcile.
**Target:** One confirmed payment always yields exactly one registration, one ledger entry, and (where applicable) one invoice; any mismatch is detected and flagged **daily**.

### NFR-REL-06 — Fast recovery from a major outage  ·  **Must**
**As an** organizer, **I need** the platform to recover quickly and lose almost no data after a serious incident, **so that** a disaster doesn't cost me my event.
**Target:** Service is restored within **1 hour**, with **no more than 5 minutes** of transactional data at risk.

### NFR-REL-07 — Updates ship without a maintenance window  ·  **Should**
**As an** organizer, **I need** product updates to roll out without taking selling offline, **so that** improvements never interrupt an active on-sale.
**Target:** Releases cause **zero downtime** on the registration and checkout path.

---

## Security & Trust

### NFR-SEC-01 — Attendee and organizer worlds stay separate  ·  **Must**
**As an** organizer, **I need** attendee logins and organizer logins to be completely separate, **so that** no attendee can ever reach admin tools or another organizer's data.
**Target:** The two account types share no login or session; access across the boundary is **impossible** and verified by test.

### NFR-SEC-02 — People only see and do what their role allows  ·  **Must**
**As an** organizer, **I need** staff and teammates limited to exactly their permissions, **so that** sensitive finance and settings stay protected from accidental or malicious access.
**Target:** Role permissions are enforced on **every request** (not just hidden in the UI), deny-by-default, with **zero** unauthorized-access findings in the access-control review.

### NFR-SEC-03 — We never hold card or bank numbers  ·  **Must**
**As an** attendee, **I need** my card and bank details handled only by the certified payment provider, **so that** my financial data is never exposed by Eventa.
**Target:** No card, CVV, or full bank number ever reaches Eventa's systems or logs — confirmed by scan; only safe references (token, last-4, brand) are retained.

### NFR-SEC-04 — Accounts are hard to take over  ·  **Must**
**As an** organizer, **I need** strong account protection and the ability to sign out any device, **so that** my event, attendees, and payouts stay safe if a password leaks.
**Target:** Strong-password policy enforced, optional two-factor authentication available and policy-enforceable for organizer accounts, and any active session **revocable within 60 seconds**.

### NFR-SEC-05 — Abuse and brute-force attempts are blocked  ·  **Should**
**As an** organizer, **I need** login, checkout, and export endpoints protected from abuse, **so that** bots and attackers can't disrupt sales or scrape attendee data.
**Target:** Repeated failed sign-ins and abusive request bursts are throttled and lockout-protected, and logged for review.

### NFR-SEC-06 — Attendee data is protected everywhere it lives  ·  **Must**
**As an** attendee, **I need** my personal data protected in transit and in storage, **so that** I can register without worrying it will be intercepted or leaked.
**Target:** All connections are secured (modern TLS, HTTPS-only), and all stored data and backups are encrypted — verified by independent scan (e.g. SSL Labs "A").

### NFR-SEC-07 — Security is independently proven  ·  **Should**
**As an** organizer, **I need** the platform's security independently tested, **so that** I can trust it with my attendees and my brand.
**Target:** **No known Critical/High vulnerabilities** are shipped, and an independent penetration test is run at least **quarterly** with findings tracked to closure.

---

## Privacy & Compliance (Thailand PDPA)

### NFR-PDPA-01 — Clear, bilingual consent and privacy notice  ·  **Must**
**As an** attendee, **I need** a plain, bilingual privacy notice and genuine opt-in choices at sign-up and checkout, **so that** I know how my data is used and consent on my own terms.
**Target:** A **bilingual EN/TH** notice appears at every collection point; marketing consent is separate, explicit opt-in, timestamped, and withdrawable — applied equally to guest (no-account) registrants.

### NFR-PDPA-02 — Data-subject rights honored on time  ·  **Must**
**As an** attendee, **I need** to access, correct, export, or object to the use of my data on request, **so that** I stay in control of my personal information under Thai PDPA.
**Target:** Verified requests (access, rectification, portability, objection, consent withdrawal) are fulfilled within **30 days**.

### NFR-PDPA-03 — Right to be forgotten  ·  **Must**
**As an** attendee, **I need** deleting my account to actually remove my personal data, **so that** I can leave the platform with confidence.
**Target:** Account deletion erases or irreversibly anonymizes personal data within **30 days** and revokes access immediately, while retaining only the minimum tax/transaction records the law requires, decoupled from my identity.

### NFR-PDPA-04 — Collect only what's needed  ·  **Should**
**As an** attendee, **I need** Eventa to ask only for data genuinely required, **so that** my exposure is minimized by design.
**Target:** Only fields needed for registration, payment, and check-in are collected; **no** personal data appears in web addresses or analytics.

### NFR-PDPA-05 — Keep required records, purge the rest  ·  **Must**
**As an** organizer, **I need** tax and financial records retained as the law requires while other personal data is not kept indefinitely, **so that** we stay compliant without hoarding data.
**Target:** Financial/tax records (invoices, VAT 7%) retained **at least 5 years** per the Thai Revenue Code; other personal data purged or anonymized on a documented schedule.

### NFR-PDPA-06 — Breaches handled and disclosed promptly  ·  **Must**
**As an** organizer, **I need** any data breach managed and reported without delay, **so that** we meet PDPA obligations and protect our attendees and reputation.
**Target:** A designated data-protection contact is published, and confirmed breaches are notified to the regulator and affected people **without undue delay (target ≤ 72 hours)**.

### NFR-PDPA-07 — Data stays in-region; transfers are disclosed  ·  **Should**
**As an** attendee, **I need** my data kept in the Thailand region by default and any overseas transfer disclosed, **so that** I know where my information goes.
**Target:** Personal data is stored **in-region** by default; any cross-border transfer (e.g. payment or messaging providers) rests on a lawful basis and is disclosed in the privacy notice.

---

## Usability & Accessibility

### NFR-UX-01 — Usable by everyone  ·  **Must**
**As an** attendee, **I need** the platform to be usable regardless of ability, **so that** no one is shut out of registering for an event.
**Target:** Public, portal, and core admin flows meet **WCAG 2.1 Level AA** with **zero** AA violations on audited flows.

### NFR-UX-02 — Works fully on mobile  ·  **Must**
**As an** attendee, **I need** every step to work well on my phone, **so that** I can discover and register on the device I actually use.
**Target:** All attendee flows and the admin shell are fully usable from **320px width up**, with comfortable touch targets (at least 44×44px) and no hover-only actions.

### NFR-UX-03 — Keyboard and screen-reader friendly  ·  **Should**
**As an** attendee **using assistive technology, I need** to operate everything by keyboard and screen reader, **so that** I can complete registration independently.
**Target:** All controls are keyboard-operable with visible focus and logical order, and forms, statuses, and alerts are announced to screen readers.

### NFR-UX-04 — Check-in has an accessible no-camera path  ·  **Must**
**As** check-in staff, **I need** a fully accessible way to check people in without the camera scanner, **so that** entry works for every device and every staff member.
**Target:** A searchable manual lookup and manual/upload check-in path is available and fully keyboard- and screen-reader-operable **without a camera**.

### NFR-UX-05 — Readable in light and dark  ·  **Should**
**As an** attendee, **I need** clear, legible contrast in both light and dark modes, **so that** I can read prices, dates, and statuses in any lighting.
**Target:** Text meets at least **4.5:1** contrast in both themes, and status is never conveyed by color alone (always paired with text or icon).

---

## Localization (EN/TH, ฿, Asia/Bangkok)

### NFR-L10N-01 — Full English/Thai parity  ·  **Must**
**As an** attendee, **I need** the whole experience — screens, emails, SMS, and invoices — in my language, **so that** I can register and get confirmations I fully understand.
**Target:** **Every** user-facing string is available in both **English and Thai** with no hard-coded copy, and my chosen language governs my communications.

### NFR-L10N-02 — Baht and VAT shown correctly  ·  **Must**
**As an** attendee, **I need** prices in Thai Baht with VAT shown clearly, **so that** I know exactly what I'm paying and receive a correct tax receipt.
**Target:** Money renders as **฿ Thai Baht** with correct grouping everywhere (screens, exports, invoices), and **VAT 7%** appears as a distinct line (subtotal + VAT + total).

### NFR-L10N-03 — Times shown in Bangkok time  ·  **Must**
**As an** attendee, **I need** event times in Asia/Bangkok time, **so that** I never miss or mistime an event.
**Target:** All event times display in **Asia/Bangkok (UTC+7)** by default, consistently across the product, emails, and tickets.

### NFR-L10N-04 — Thai text handled correctly  ·  **Should**
**As a** Thai-speaking attendee, **I need** Thai names and text to display, search, and sort correctly, **so that** my details and searches behave as expected.
**Target:** Thai script renders, inputs, searches, and sorts correctly end-to-end, and Thai names fit without truncation or overflow; Buddhist-era year shown where culturally expected on TH receipts.

---

## Scalability (on-sale spikes, large attendee lists)

### NFR-SCALE-01 — Survive the on-sale rush  ·  **Must**
**As an** organizer, **I need** the platform to hold up when a popular ticket opens, **so that** a surge of buyers converts into sales instead of errors.
**Target:** Sustains **at least 500 concurrent checkouts per event and 50+ registrations per second** with checkout speed held and **no** error pages or oversell.

### NFR-SCALE-02 — Never oversell a ticket  ·  **Must**
**As an** organizer, **I need** the platform to never sell more seats than exist, even under a simultaneous buying rush, **so that** I'm never forced to cancel on paying attendees.
**Target:** **Zero oversell** — confirmed seats never exceed a ticket's capacity, proven under heavy simultaneous-purchase testing.

### NFR-SCALE-03 — Handle very large attendee lists  ·  **Must**
**As an** organizer of a large event, **I need** lists, search, export, and check-in to stay fast with huge attendee counts, **so that** big events run as smoothly as small ones.
**Target:** All these functions hold their speed targets at **tens of thousands (up to 50,000) of attendees** per event.

### NFR-SCALE-04 — One busy organizer never slows another  ·  **Should**
**As an** organizer, **I need** another organizer's traffic spike to never slow me down or expose their data, **so that** I can rely on consistent performance and isolation.
**Target:** A heavy load or large data volume from one organizer has **no measurable impact** on another's speed, and no organizer can ever see another's data.

### NFR-SCALE-05 — Headroom to grow  ·  **Should**
**As a** business stakeholder, **I need** the platform to absorb sudden growth beyond recent peaks, **so that** a viral event or fast-growing customer never hits a wall.
**Target:** Capacity sustains **3× the recent peak** before any manual intervention, and scales up automatically within about **2 minutes** of a sustained rise.

---

## Supportability

### NFR-SUP-01 — Problems are caught before customers complain  ·  **Should**
**As an** organizer, **I need** the operations team to detect and respond to issues proactively, **so that** problems are fixed before they hurt my event.
**Target:** Sales, checkout success, check-in, and payment health are monitored with dashboards and on-call alerts that fire on error spikes, slowdowns, or failed-payment surges.

### NFR-SUP-02 — Complete audit trail of security-sensitive actions  ·  **Must**
**As an** organizer, **I need** a trustworthy record of sign-ins, permission changes, exports, and other sensitive actions, **so that** I can investigate incidents and demonstrate accountability.
**Target:** A tamper-evident, append-only log records every security-sensitive action (sign-in, new device, password/2FA change, permission change, data export, failed sign-in, session revoke) with who, when, and from where — admin-viewable and exportable.

### NFR-SUP-03 — Financial audit trail for reconciliation and tax  ·  **Must**
**As an** organizer, **I need** every payment, refund, payout, and invoice permanently recorded, **so that** I can reconcile my books and satisfy a tax audit.
**Target:** Every financial event is immutably recorded with before/after values and the responsible actor, sufficient for reconciliation and Thai tax reporting.

### NFR-SUP-04 — Message delivery is visible  ·  **Should**
**As an** organizer, **I need** to see whether my confirmations and reminders actually reached attendees, **so that** I can trust communications and follow up on failures.
**Target:** Email and SMS outcomes (sent, delivered, bounced, failed) are logged and queryable per message.

### NFR-SUP-05 — Regular backups and tested recovery  ·  **Must**
**As an** organizer, **I need** confidence that my event data is backed up and restorable, **so that** a failure never erases my attendees or finances.
**Target:** Automated encrypted backups run continuously (no more than **5 minutes** of data at risk), are replicated to a second location in-region, and restores are **tested at least quarterly**.
