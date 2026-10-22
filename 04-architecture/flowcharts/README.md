# Eventa Flowcharts

Step-by-step flowcharts for the Eventa attendee portal, public landing pages and organizer admin console. They use the same tables, columns and statuses as the [ERD](../erd.md).

Each one draws what the code does today, and was checked claim by claim against `eventa-api`, `eventa-web` and `eventa-worker`. Where a journey stops short of what its story asks, the chart says so in a node that begins **Not built:** — or **API-only:** where the endpoint exists but no screen calls it — rather than drawing it as working.

| # | Flow | Screens |
|---|---|---|
| 01 | [Discover and register](01-discover-and-register.md) | Portal Discover, Event landing page, Checkout |
| 02 | [Checkout and payment](02-checkout-and-payment.md) | Checkout, Stripe Checkout / PromptPay, My Tickets |
| 03 | [Free registration and confirmation](03-free-registration-and-confirmation.md) | Checkout, My Tickets, email and SMS |
| 04 | [Waitlist: join, offer and pass-on](04-waitlist.md) | Checkout, Admin Registrations, email |
| 05 | [Approval on a require-approval event](05-approval.md) | Checkout, Admin Registrations |
| 06 | [Check-in by QR and by hand](06-check-in.md) | Check-in tool, Check-in list |
| 07 | [Refund and cancellation](07-refund-and-cancellation.md) | Admin Payments, Event detail, email |
| 08 | [Invoice, ageing and VAT filing](08-invoice-and-vat.md) | Admin Invoices, Taxes |
| 09 | [Payouts and balances](09-payouts.md) | Admin Payouts, Payment settings |
| 10 | [Announcements, now and scheduled](10-announcements.md) | Admin Announcements, email |
| 11 | [Surveys, feedback and NPS](11-surveys-and-feedback.md) | Admin Feedback, Portal survey, email |
| 12 | [Sign-in, roles and permissions](12-sign-in-roles-and-permissions.md) | Sign-in, Settings Roles, Settings Users, Accept invitation, email |
| 13 | [Create and publish an event](13-create-and-publish-an-event.md) | Create event wizard, Landing pages, Tickets |
| 14 | [Sign-up, confirmation and password reset](14-sign-up-confirm-and-reset-password.md) | Organizer sign-up, Portal sign-up, Check your email, Verify email, Guest order page, Forgot password, Reset password |
| 15 | [Account security: password, two-factor and sessions](15-account-security.md) | Settings Security, Portal My events · Settings tab, email |
| 16 | [Profile, preferences and deleting an account](16-profile-preferences-and-account-deletion.md) | Settings Profile, Settings Notifications, Portal My events · Profile and Settings tabs, email |
| 17 | [Set up the workspace](17-workspace-setup.md) | Home, Settings Organization, Settings Payments |
| 18 | [Agenda, speakers and session changes](18-event-programme.md) | Admin Agenda, Admin Speakers, Event detail, Public event page, email |
| 19 | [Discount codes: create, stop and redeem](19-discount-codes.md) | Admin Discounts, Checkout, Discounts report |
| 20 | [Meetings with speakers, sponsors, venues and vendors](20-meetings.md) | Admin Meetings, Home, Settings Integrations |
| 21 | [Attendees, invitations and manual registration](21-attendees-invitations-and-manual-registration.md) | Admin Attendees, Attendee profile panel, Admin Registrations, email |
| 22 | [Automated messages: templates, reminders and the delivery log](22-automated-messages.md) | Message templates, Delivery log, email |
| 23 | [My tickets and payments, after purchase](23-my-tickets.md) | Guest order page, Portal sign-in, My Account (My Events, Payment history), Ticket QR modal, Portal Discover |
| 24 | [Reports, filters and exports](24-reports-and-exports.md) | Insights Overview, Income, Transactions, Payouts, Registrations, Attendance, Discounts, Event performance |
| 25 | [Categories, duplicating and deleting events](25-manage-events.md) | Event categories, Events list, Create event wizard, Portal Discover |
| 26 | [Notifications: the feed and the bell](26-notifications.md) | Admin Notifications, header notification bell |

## Journeys not drawn, because they are not built

A flowchart documents built behaviour, so these have no chart of their own. Smaller gaps inside a drawn journey are marked in that chart instead.

| Journey | Story | What exists today | Where it would sit |
|---|---|---|---|
| Recording payouts | US-FIN-03, US-RPT-07 | Stripe's `payout.*` and `transfer.*` webhooks are ignored and no code writes `payouts` or `payout_items`, so the paid-out balance, the payouts report and payout notifications are always empty | [09](09-payouts.md), [24](24-reports-and-exports.md), [26](26-notifications.md) |
| Cancelling an event from the console | US-EVT-08 | `POST /events/:id/cancel` is API-only; the console has no Cancel control | [07](07-refund-and-cancellation.md), [25](25-manage-events.md) |
| Signing in with Google, Apple or LinkedIn | US-ACC-06 | API endpoints only; no screen completes the journey | [14](14-sign-up-confirm-and-reset-password.md) |
| Confirming a changed email address | US-SET-01, US-DISC-11 | the confirmation link lands on a route that does not exist, so the new address is never confirmed | [16](16-profile-preferences-and-account-deletion.md) |
| Verifying a phone number | US-SET-01, US-DISC-11 | the code is issued, but no worker handler sends the text | [16](16-profile-preferences-and-account-deletion.md) |
| Notification preferences taking effect | US-SET-06 | the switches are stored; nothing reads them | [16](16-profile-preferences-and-account-deletion.md), [22](22-automated-messages.md) |
| Emailing one attendee | US-REG-09 | "Send email" opens a `mailto:` link; nothing is queued or logged | [21](21-attendees-invitations-and-manual-registration.md) |
| Chasing unpaid invoices | US-FIN-09 | no resend and no automatic reminder | [08](08-invoice-and-vat.md) |
| Speaker ratings from feedback | US-PROG-13 | only a rating badge that is always empty | [18](18-event-programme.md) |
