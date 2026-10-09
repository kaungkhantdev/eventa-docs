# Eventa — Entity Catalog (Data Dictionary)

> **Purpose.** This document is the authoritative relational data dictionary for
> Eventa, the Thai-market multi-tenant **event registration & management** SaaS.
> It is the physical companion to the
> [product backlog](../01-requirements-and-features/functional-requirements.md): where the
> backlog defines *what* the domain entities are and why, this catalog defines *how* they are
> persisted as a normalized, production relational schema.
>
> **Target DBMS.** PostgreSQL 15+. The design is normalized to 3NF, uses native
> `ENUM` types for every status/kind field, surrogate keys on every table,
> explicit foreign keys with chosen `ON DELETE` behavior — with six exceptions the
> schema has not caught up on, noted where they occur — and resolves every
> many-to-many with an associative (junction) table. All monetary values are
> stored as **integer minor units (satang)**, never floating point.
>
> **Schema status.** The **live PostgreSQL schema is the reference**, and this
> catalog describes it. All **57 tables** below exist in `public` today; every
> column, type, key and `ON DELETE` rule is read back from
> `information_schema.columns` and `pg_constraint`, never copied from another
> document. Where the catalog and the database disagree, the database wins and
> this file is what gets corrected.
>
> That reverses the stance this note used to take. It previously called its 53
> tables a *target physical model* and sent the reader to the API's `pgTable`
> definitions for the real inventory, which was honest while the catalog led the
> build. The implementation has since overtaken it **in both directions** — five
> tables shipped with no entry here, and one table that was catalogued
> (`notifications`) was deliberately never built — so a target the reader has to
> reconcile against the database themselves is no longer worth more than a
> description they can trust.
>
> **The forward-looking intent is kept, and labelled.** Design work that the
> database has not taken up is not thrown away: columns and enums specified here
> and never built are recorded in a **Designed, not built** note on the table
> they belong to, saying what the product therefore cannot do, and the
> `notifications` design is recorded under *Engagement & Messaging* with the
> model that replaced it. The rule is that nothing inside a column table is a
> plan — every row there is a column that exists — and anything still ahead of
> the schema says so in prose.
>
> **Provenance.** The **Source** column of *Enumerated types* and the `Foo.bar`
> citations in the notes name where a field came from — the SRS data model and the
> prototype's TypeScript under `src/features/*/data/*.ts`,
> `src/lib/eventCatalog.ts` and `src/lib/format.ts`. They explain *why* a name or
> a token looks the way it does; they no longer decide what it is. Where a
> prototype name and the shipped column differ, the column wins and the citation
> stays for the history. Display-only enum tokens are stored as their lowercase
> wire values; Title-cased variants are UI labels and are not persisted.

---

## Conventions

- **Naming.** `snake_case`, plural table names (`events`, `order_items`).
  Columns `snake_case`. Junction tables are named `<parent>_<child>`
  (`role_permissions`, `session_speakers`, `discount_redemptions`,
  `payout_items`).
- **Surrogate primary key.** Every table has `id` as its primary key —
  `bigint GENERATED ALWAYS AS IDENTITY` for high-volume commerce/ledger tables,
  `uuid DEFAULT gen_random_uuid()` (UUID v7, time-ordered) for aggregate roots.
  Human-facing references (`orders.reference` = `REG-YYYY-NNNNNN`,
  `invoices.number`, `payments.txn`, `payouts.reference`) are separate **UNIQUE**
  natural keys, never the PK.
- **Multi-tenancy.** 50 of the 57 physical tables carry
  `organization_id bigint REFERENCES organizations(id)` (NOT NULL except the
  inbound `webhook_events` log while its tenant is unresolved). Every tenant
  read/write is filtered by the caller's organization (row-level isolation).
  The seven tables without that column are still `organizations`, the global lookups
  `landing_templates` and `permissions`, the transitively-scoped children
  `role_permissions`, `recovery_codes`, and `session_speakers`, and the
  cross-tenant, user-scoped `saved_events` bookmark table.
- **Audit columns.** Mutable domain tables normally carry `created_at timestamptz
  NOT NULL DEFAULT now()` (immutable) and `updated_at timestamptz NOT NULL DEFAULT
  now()` (touched on every mutation, via trigger). Aggregate roots and user-editable entities also
  carry `deleted_at timestamptz NULL` for **soft delete**; append-only ledgers
  (`payments`, `refunds`, `audit_events`, `message_deliveries`, `scan_attempts`,
  `event_invitations`) are **never** soft-deleted and omit the column. `check_ins`
  omits it too but is not a ledger: a row is inserted on arrival and **hard-deleted**
  when an admission is undone, because an admission that was reversed did not happen
  — the scan that produced it survives in `scan_attempts`. `announcements` replaces
  soft delete with an explicit cancellation (`cancelled_at`, `cancelled_by_user_id`),
  and `surveys` carries neither, which is a gap recorded with that table rather than a
  decision. `created_by uuid NULL REFERENCES
  users(id)` records the actor where meaningful (null for guest-originated rows).
  `version integer NOT NULL DEFAULT 1` is the optimistic-concurrency token
  (stale writes are rejected `409 Conflict`). Append-only, lookup, junction, and
  immutable link tables may omit `updated_at` and/or `version`; each exception is
  documented with its table.
- **Money.** All monetary amounts are `bigint` **satang** (1 THB = 100 satang),
  `>= 0` via CHECK, paired with `currency char(3) NOT NULL DEFAULT 'THB'`.
  `bigint` is used (not `integer`) so large payouts/aggregates cannot overflow.
  Never `float`/`numeric` for money. Display strings are derived at render time via
  `format.baht` / `format.bahtCompact`.
- **Percentages / rates.** Stored as `numeric(5,4)` (e.g. VAT `0.0700`, service
  fee `0.0500`) or whole-percent `smallint` for discount percentages (1–100).
- **Timestamps / timezone.** All instants are `timestamptz` stored in UTC and
  rendered `Asia/Bangkok`. Calendar-only fields (invoice/tax due dates) are `date`.
  Wall-clock program/meeting times are `time`. Bucket/derived-time states are
  computed from timestamps at read time, never stored free-hand.
- **Enum handling.** Status/kind/type fields use PostgreSQL native `CREATE TYPE …
  AS ENUM`. Wire values are the lowercase tokens from the source unions
  (`onsale`, `sent`, `email`); enums that the source authors Title-cased
  (`Card`, `Keynote`, `Admin`) keep that exact casing. Every enum used by a column
  is defined in **Enumerated types** below.
- **Referential integrity defaults.** `ON DELETE RESTRICT` by default (protect
  financial/booking history). `ON DELETE CASCADE` only for owned child rows that
  cannot outlive their parent (an event's `ticket_types`, a survey's
  `survey_questions`, an order's `order_items`, a seat_map's `seats`).
  `ON DELETE SET NULL` for optional denormalized/attribution references
  (`events.category_id`, `orders.discount_code_id`, `meetings.event_id`).
  Because most rows are soft-deleted, hard-delete cascades are a backstop.

---

## Enumerated types

One PostgreSQL `ENUM` type per row. Values are listed in wire form (as stored).

| Enum type | Allowed values | Used by | Source |
|---|---|---|---|
| `event_status` | `draft`, `planned`, `upcoming`, `live`, `completed`, `cancelled` | `events.status` | canonical union of `events/types.ts` + `eventCatalog.ts` |
| `event_type` | `Conference`, `Networking`, `Workshop`, `Charity & Gala`, `Sports & Wellness`, `Concert & Festival`, `Exhibition`, `Seminar` | `events.type` | `events/types.ts` |
| `event_bucket` | `active`, `completed` | `events.bucket` (derived) | `events/types.ts` |
| `visibility` | `private`, `unlisted`, `public` | `events.visibility` | production (landing/portal exposure) |
| `seating_mode` | `reserved`, `ga` | `events.seating_mode` | `portal/events.ts`, `landing/types.ts` |
| `ticket_status` | `onsale`, `scheduled`, `paused`, `soldout` | `ticket_types.status` | `ticketing/types.ts`, `tickets.ts` |
| `admission_type` | `general_admission`, `reserved_seat` | `ticket_types.admission_type` | derived from `seating_mode` |
| `discount_type` | `percent`, `fixed` | `discount_codes.type` | `ticketing/types.ts` |
| `discount_status` | `active`, `scheduled`, `expired`, `disabled` | `discount_codes.status` | `ticketing/types.ts`, `discounts.ts`, `reportsDiscounts.ts` |
| `order_status` | `confirmed`, `pending`, `waitlisted`, `cancelled`, `rejected`, `expired` | `orders.status` | SRS `RegistrationStatus` (`registrations.ts`). `rejected` = an organizer turned it down (US-REG-02); `expired` = nobody paid before the seat hold lapsed (US-DISC-05). Both are deliberately distinct from `cancelled`, which is a withdrawal or a refund — a decision somebody made, not the absence of one. |
| `issued_ticket_status` | `issued`, `checked_in`, `void`, `refunded`, `transferred` | `tickets.status` | derived from `RegStatus`/`ScanState` |
| `payment_status` | `paid`, `pending`, `refunded`, `failed` | `payments.status`, `orders.payment_status` | `payments.ts` |
| `payment_method` | `Card`, `PromptPay`, `Bank transfer`, `Apple Pay`, `Google Pay` | `payments.method`, `invoices.paid_via`, `payment_method_settings.method` | `payments.ts` |
| `payment_provider` | `stripe` | `payment_settings.provider` | `payment-settings.ts` |
| `payment_mode` | `test`, `live` | `payment_settings.mode`, `payment_credentials.mode` | `payment-settings.ts` |
| `payment_connection_status` | `disconnected`, `connected` | `payment_settings.status` | `payment-settings.ts` |
| `refund_status` | `pending`, `succeeded`, `failed` | `refunds.status` | derived from `TransactionStatus` (`reportsTransactions.ts`) |
| `payout_status` | `scheduled`, `processing`, `paid`, `failed` | `payouts.status` | `payouts.ts` (reports labels: Paid / In transit / Pending) |
| `invoice_status` | `issued`, `paid`, `overdue`, `void` | `invoices.status` | `invoices.ts` (paid/void terminal; issued/overdue derived) |
| `tax_status` | `upcoming`, `due`, `filed` | `tax_periods.status` | `taxes.ts` |
| `session_type` | `Keynote`, `Talk`, `Workshop`, `Panel`, `Break` | `sessions.type` | `agenda.ts`, `eventDetail.ts` |
| `session_color` | `green`, `amber`, `rose`, `blue`, `slate` | `sessions.color` | `agenda.ts` |
| `speaker_tone` | `green`, `blue`, `purple`, `amber`, `red`, `pink` | `speakers.tone` | `speakers.ts` |
| `meeting_type` | `Venue`, `Sponsor`, `Vendor`, `Speaker`, `Internal` | `meetings.type` | `meetings.ts` |
| `meeting_mode` | `Video`, `In person`, `Phone` | `meetings.mode` | `meetings.ts` |
| `meeting_status` | `scheduled`, `cancelled` | `meetings.status` | production — a meeting is held or called off; the today/upcoming/past bucket it replaced is computed from `meeting_date` |
| `meeting_sync_status` | `pending`, `synced`, `failed`, `not_connected` | `meetings.sync_status` | production (external calendar push) |
| `template_id` | `aurora`, `noir`, `minimal`, `atlas` | `landing_templates.id`, `events.landing_template_id` | `landingTemplates.ts` |
| `message_channel` | `email`, `sms` | `message_templates.channels`, `message_deliveries.channel` | `messagingTemplates.ts`, `deliveryLog.ts` |
| `announcement_status` | `scheduled`, `sent`, `cancelled` | `announcements.status` | `announcements.ts`; `cancelled` added with scheduled sends (migration `0065`) |
| `delivery_status` | `sent`, `failed` | `message_deliveries.status` | `deliveryLog.ts` — the sender knows only whether it handed the message over; `delivered`/`opened` need provider callbacks the log cannot yet receive |
| `notification_kind` | `registration`, `payment`, `sales`, `feedback`, `payout`, `alert`, `task`, `reminder`, `marketing` | `notification_preferences.category` | `notifications.ts` — the category a member opts in or out of, not a column on a stored notification (see the `notifications` note below) |
| `survey_status` | `draft`, `live`, `closed` | `surveys.status` | `feedback.ts` (`FeedbackStatus`) |
| `survey_question_type` | `rating`, `text`, `choice`, `nps` | `survey_questions.type` | `feedback.ts`, stored as lowercase wire tokens; `nps` added by migration `0062` |
| `check_in_method` | `qr`, `manual`, `upload` | `check_ins.method`, `scan_attempts.method` | production — how the door identified the holder: scanned code, staff lookup by name, or bulk upload |
| `scan_outcome` | `admitted`, `already_checked_in`, `invalid`, `wrong_event`, `cancelled` | `scan_attempts.outcome` | production (`0039` created it, `0068` gave it a column) |
| `member_role` | `Admin`, `Organizer`, `Staff`, `Attendee` | *(the type exists but no column uses it — retired from `roles.name`/`memberships.role`, both text since US-SET-13)* | `roles.ts` (`RoleName`) |
| `member_status` | `Active`, `Invited`, `Suspended`, `Unconfirmed` | `memberships.status`, `users.status` | `users.ts` (`UserStatus`); `Unconfirmed` is a self-signup whose email is unverified |
| `user_persona` | `admin`, `attendee` | `users.persona` | SRS §1.23 (personas never share a login) |
| `permission_key` | `evCreate`, `evPublish`, `evSpeakers`, `evProgramView`, `regView`, `regCheckin`, `regExport`, `regManage`, `finView`, `finRefund`, `finDiscount`, `finManage`, `setUsers`, `setSettings`, `setIntegrations` | `permissions.key`, `role_permissions.permission_key` | `roles.ts` (`PermKey`, 15 values) |
| `permission_group` | `Events`, `Registrations`, `Finance`, `Settings` | `permissions.group` | `roles.ts` (`PermGroup`) |
| `audit_type` | `signin`, `newdev`, `pwd`, `twofa`, `perm`, `xport`, `fail`, `apikey`, `revoke`, `invoice`, `payout`, `checkin` | `audit_events.type` | `security.ts` (`AuditType`), extended as finance and door actions became auditable |
| `attendee_tag` | `VIP`, `Speaker`, `Sponsor`, `Student` | `attendees.tag` (nullable) | `attendees.ts` (`TAG_BADGE`) |
| `seat_status` | `available`, `held`, `reserved`, `sold`, `blocked` | `seats.status` | production (reserved-seating model) |
| `category_color` | `pink`, `blue`, `amber`, `brand`, `violet`, `indigo`, `teal`, `red` | `categories.color` | `categories.ts` |
| `locale` | `en`, `th` | `organizations.locale`, `users.locale`, `events.locale` | `format.ts` / SRS §5.5 |
| `social_provider` | `google`, `apple`, `linkedin` | `social_identities.provider` | `identity.ts` |
| `two_factor_method` | `totp` | `two_factors.method` | `security.ts` (authenticator app) |
| `api_key_status` | `active`, `revoked` | `api_keys.status` | production (`setIntegrations` / apikey audit) |
| `hold_status` | `active`, `converted`, `expired`, `released` | `seat_holds.status` | production (checkout seat-hold lifecycle) |
| `webhook_status` | `received`, `processed`, `failed` | `webhook_events.status` | production (inbound webhook processing) |

> **Casing rule.** Lowercase tokens (`onsale`, `sent`, `email`) are wire/DB
> values. Title-cased enum members that the source authored that way
> (`Card`, `PromptPay`, `Keynote`, `Admin`, `VIP`) are stored verbatim. UI badge
> labels like "On sale" / "In transit" are **not** persisted — they are mapped
> from the wire value at render time.

> **Designed, not created.** Four enum types this table used to list do not exist
> in the database, because the column each one was for was never built. They are
> named here so a reader who met them in an older revision can see what became of
> them, and the gap each leaves is described on the table it belongs to:
> `announcement_audience` (audience targeting — see `announcements`); `scan_state`
> (`ok`/`dupe`/`invalid`/`wrong`/`void`, superseded by `scan_outcome` on
> `scan_attempts`); `question_type` (`Rating`/`Text`/`Multiple choice`, superseded
> by the lowercase `survey_question_type`); and `meeting_bucket`
> (`today`/`upcoming`/`past`, superseded by `meeting_status` — the time bucket is
> now derived from `meetings.meeting_date` at read time, which is what the rest of
> this schema does with derived time).
>
> Three further rows were removed because they never described a PostgreSQL type
> at all. `registration_status` was a naming note: `orders.status` realizes it as
> `order_status`. `transaction_type` and `transaction_status` are the shape of the
> insights union that reporting assembles at read time from `payments` and
> `refunds`; no column has either type, and none is intended to.

---

## Entities

Notation for the **Key** column: `PK` primary key · `FK→table.col` foreign key
**that exists in the database** · `UK` part of a unique constraint *or* a standalone
unique index (six uniqueness rules are indexes, not constraints, because they are
partial or functional — they are named in the `UNIQUE` bullet where they apply) ·
`IX` indexed by an index of its own, not merely named in some index's predicate.
**Null** = *is the column nullable?* (`no` = NOT NULL, `yes` = nullable). A column
that holds another table's `id` without a foreign-key constraint behind it carries
no `FK` marker and says so in its Notes — there are six such columns, tabulated in
`erd.md` → [Unenforced references](erd.md#unenforced-references). Every table also
carries the common audit columns from *Conventions* (`created_at`, `updated_at`,
`version`, and `deleted_at`/`created_by` where noted); they are shown per table for
completeness.

### Identity & Access

#### `organizations`
Tenant root / workspace. Not itself tenant-scoped; parent of everything else.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `name` | text | no | UK | | Workspace/company name. Unique among live workspaces, case- and whitespace-insensitive. |
| `slug` | text | no | UK | | Globally unique. `^[a-z0-9-]+$`. |
| `logo_url` | text | yes | | | ≤1MB asset (UI-authoritative). |
| `address` | text | yes | | | Legal address printed on invoices/receipts (US-SET-07). |
| `website` | text | yes | | | Public site, http(s) URL (US-SET-07). |
| `currency` | char(3) | no | | `'THB'` | Fixed THB. |
| `country` | char(2) | no | | `'TH'` | |
| `timezone` | text | no | | `'Asia/Bangkok'` | IANA tz. |
| `locale` | `locale` | no | | `'en'` | Default UI locale. |
| `vat_rate` | numeric(5,4) | no | | `0.0700` | 7% VAT. |
| `service_fee_rate` | numeric(5,4) | no | | `0.0500` | 5% platform service fee. |
| `statement_descriptor` | varchar(22) | yes | | | ≤22 chars (Stripe limit). |
| `tax_id` | text | yes | | | Thai VAT registration number. |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `deleted_at` | timestamptz | yes | | | Soft delete. |
| `version` | integer | no | | `1` | |

- **PRIMARY KEY** (`id`)
- **UNIQUE** (`slug`); partial `uq_organizations_name` (`lower(btrim(name))`) WHERE `deleted_at IS NULL` — one live workspace per name, lowercased and trimmed; a soft-deleted row releases its name
- **INDEX** `ix_organizations_deleted_at` (`deleted_at`)

#### `users`
A login identity. Admin-console and portal personas are distinct rows even at the same email (never share a login).

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | uuid | no | PK | `gen_random_uuid()` | |
| `organization_id` | bigint | no | FK→organizations.id, UK, IX | | Tenant. |
| `name` | text | no | | | |
| `email` | citext | no | UK, IX | | RFC 5322, ≤254. |
| `persona` | `user_persona` | no | UK | `'admin'` | `admin` \| `attendee`. |
| `initials` | text | yes | | | Derivable via `format.initials`. |
| `status` | `member_status` | no | | `'Invited'` | Active/Invited/Suspended. |
| `password_hash` | text | yes | | | Argon2id; never returned/logged. |
| `avatar_url` | text | yes | | | ≤5MB. |
| `phone` | text | yes | | | Contact number; gates the SMS notification toggles (US-SET-01/06). |
| `timezone` | text | yes | | | IANA tz; per-user override of the org timezone (US-SET-01). |
| `city` | text | yes | | | Attendee profile (US-DISC-11). |
| `date_of_birth` | date | yes | | | Attendee profile (US-DISC-11). |
| `bio` | text | yes | | | Attendee profile (US-DISC-11). |
| `display_currency` | char(3) | yes | | | Display-only preference (US-DISC-12); charges settle in THB. |
| `pending_email` | citext | yes | | | Requested new email awaiting confirmation; `email` keeps working until the link is opened (US-SET-01). |
| `two_factor_enabled` | boolean | no | | `false` | |
| `locale` | `locale` | yes | | | User override of org locale. |
| `attendee_id` | bigint | yes | | | 1:1 link for the portal persona. Holds an `attendees.id` but carries **no FK constraint** — see below. |
| `last_active_at` | timestamptz | yes | | | |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `deleted_at` | timestamptz | yes | | | |
| `version` | integer | no | | `1` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEY** `organization_id`→`organizations(id)` **ON DELETE CASCADE**
- **UNIQUE** (`organization_id`, `email`, `persona`) — same email may exist once per persona per org
- **INDEXES** `ix_users_org` (`organization_id`), `ix_users_email` (`email`)
- **`attendee_id` is an unenforced reference.** It holds an `attendees.id` and the application joins on it,
  but no `REFERENCES` clause was ever added, so the portal-persona link can dangle: removing the attendee
  row leaves the pointer behind and nothing nulls it. Listed in `erd.md` →
  [Unenforced references](erd.md#unenforced-references).

#### `roles`
Named preset of the 15 permissions, per organization.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, UK | | |
| `name` | text | no | UK | | Free text so a workspace can add custom roles, e.g. "Volunteer" (US-SET-13). The four seeded presets keep the historical names. |
| `description` | text | no | | | `ROLE_DESC[name]`. |
| `bullets` | jsonb | yes | | | `ROLE_BULLETS` summary bullets. |
| `is_system` | boolean | no | | `true` | `true` for the four seeded presets; `false` for a custom role (US-SET-13). |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `version` | integer | no | | `1` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEY** `organization_id`→`organizations(id)` **ON DELETE CASCADE**
- **UNIQUE** (`organization_id`, `name`)

#### `permissions`
Global fixed lookup of the 15 permission keys and their group/label. Not tenant-scoped.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `key` | `permission_key` | no | PK | | e.g. `finRefund`. |
| `group` | `permission_group` | no | IX | | Events/Registrations/Finance/Settings. |
| `label` | text | no | | | e.g. "Issue refunds". |

- **PRIMARY KEY** (`key`)
- **INDEX** `ix_permissions_group` (`group`)
- Seeded with exactly 15 rows; no `organization_id`.

#### `role_permissions` — JUNCTION (roles ⇄ permissions)
Resolves the M:N role→permission preset matrix (`ROLE_PERMS`).

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `role_id` | bigint | no | FK→roles.id, UK, IX | | |
| `permission_key` | `permission_key` | no | FK→permissions.key, UK | | |
| `granted` | boolean | no | | `false` | Preset value from `ROLE_PERMS`. |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `role_id`→`roles(id)` **ON DELETE CASCADE**; `permission_key`→`permissions(key)` **ON DELETE RESTRICT**
- **UNIQUE** (`role_id`, `permission_key`)
- **INDEX** `ix_role_permissions_role` (`role_id`)

#### `memberships` — JUNCTION (users ⇄ organizations)
Associates a user with an organization and the role they hold there. Carries the assignment lifecycle.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, UK, IX | | |
| `user_id` | uuid | no | FK→users.id, UK, IX | | |
| `role_id` | bigint | no | FK→roles.id, IX | | Preset of 15 permissions. |
| `role` | text | no | | | Denormalized role name; text, so it can hold a custom role (`WorkspaceUser.role`). |
| `status` | `member_status` | no | | `'Invited'` | Active/Invited/Suspended. |
| `invited_at` | timestamptz | yes | | | |
| `joined_at` | timestamptz | yes | | | |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `deleted_at` | timestamptz | yes | | | |
| `version` | integer | no | | `1` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `user_id`→`users(id)` **ON DELETE CASCADE**; `role_id`→`roles(id)` **ON DELETE RESTRICT**
- **UNIQUE** (`organization_id`, `user_id`)
- **INDEXES** `ix_memberships_org` (`organization_id`), `ix_memberships_user` (`user_id`), `ix_memberships_role` (`role_id`)

#### `auth_sessions`
Refresh sessions / devices for JWT auth (ADR-8). One row per sign-in; the JWT **refresh** token carries
this row's `id` as its `sid` claim, and refresh is only honoured while the row is live (not revoked, not
expired) — this is what makes logout/compromise revocable. Access tokens are stateless JWTs and are not
stored. Append-mostly (revoke sets a timestamp).

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | uuid | no | PK | `gen_random_uuid()` | Refresh-session id (the JWT `sid` claim). |
| `organization_id` | bigint | no | FK→organizations.id, IX | | |
| `user_id` | uuid | no | FK→users.id, IX | | Owner. |
| `device` | text | no | | | Device/browser label (`Session.device`). |
| `meta` | text | yes | | | Location/IP/last-seen (`Session.meta`). |
| `ip_address` | inet | yes | | | |
| `is_current` | boolean | no | | `false` | "This device" — no Revoke button. |
| `created_at` | timestamptz | no | | `now()` | Sign-in time. |
| `expires_at` | timestamptz | no | | | Idle/absolute expiry. |
| `revoked_at` | timestamptz | yes | | | Set on revoke; revoking current logs out. |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `user_id`→`users(id)` **ON DELETE CASCADE**
- **INDEXES** `ix_auth_sessions_user` (`user_id`), `ix_auth_sessions_org` (`organization_id`), partial `ix_auth_sessions_active` (`user_id`) WHERE `revoked_at IS NULL`

#### `social_identities`
A provider account linked to a user (US-ACC-06). One row per (provider, subject); a user may link several
providers. Nothing secret is stored — only the provider's opaque subject and the email it asserted at link
time. A social-only account has `users.password_hash = NULL`.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id | | Tenant. |
| `user_id` | uuid | no | FK→users.id, UK, IX | | |
| `provider` | `social_provider` | no | UK | | google/apple/linkedin. |
| `subject` | text | no | UK | | The provider's stable `sub` — never the email, which can change. |
| `email` | citext | yes | | | Asserted at link time, for display only. |
| `linked_at` | timestamptz | no | | `now()` | |
| `last_used_at` | timestamptz | yes | | | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `user_id`→`users(id)` **ON DELETE CASCADE**
- **UNIQUE** `uq_social_identities_provider_subject` (`provider`, `subject`); `uq_social_identities_user_provider` (`user_id`, `provider`)
- **INDEX** `ix_social_identities_user` (`user_id`)
- **RLS** tenant isolation on `organization_id`

#### `two_factors`
Per-user TOTP two-factor enrollment (0..1 per user).

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id | | |
| `user_id` | uuid | no | FK→users.id, UK | | One active enrollment per user. |
| `method` | `two_factor_method` | no | | `'totp'` | Authenticator app. |
| `secret_encrypted` | bytea | no | | | TOTP shared secret, encrypted at rest. |
| `otpauth_uri` | text | yes | | | Provisioning URI (`TWOFA_OTPAUTH`). |
| `confirmed_at` | timestamptz | yes | | | Set once first code verified. |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `user_id`→`users(id)` **ON DELETE CASCADE**
- **UNIQUE** (`user_id`)

#### `recovery_codes`
One-time 2FA backup codes (`RECOVERY_CODES`). One row per code.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `two_factor_id` | bigint | no | FK→two_factors.id, IX | | Owning enrollment. |
| `code_hash` | text | no | UK | | Hashed; plaintext shown once at generation. |
| `used_at` | timestamptz | yes | | | Set when consumed (single-use). |
| `created_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEY** `two_factor_id`→`two_factors(id)` **ON DELETE CASCADE**
- **UNIQUE** (`two_factor_id`, `code_hash`)
- **INDEX** `ix_recovery_codes_2fa` (`two_factor_id`)

### Organization & Settings

#### `api_keys`
Integration/API credentials (`setIntegrations`; emits `apikey` audit events).

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | |
| `name` | text | no | | | Label. |
| `key_prefix` | text | no | UK | | Public prefix for lookup/display. |
| `key_hash` | text | no | | | Hashed secret; full key shown once. |
| `status` | `api_key_status` | no | | `'active'` | |
| `created_by` | uuid | no | FK→users.id | | Requires `setIntegrations`. |
| `last_used_at` | timestamptz | yes | | | |
| `revoked_at` | timestamptz | yes | | | |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `created_by`→`users(id)` **ON DELETE RESTRICT**
- **UNIQUE** (`key_prefix`)
- **INDEX** `ix_api_keys_org` (`organization_id`)

#### `notification_preferences`
Per-user email/SMS toggle per notification category (`NotifCategory`); gates whether a `message_deliveries` row is generated.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id | | |
| `user_id` | uuid | no | FK→users.id, UK, IX | | |
| `category` | `notification_kind` | no | UK | | Category key. |
| `title` | text | yes | | | Display title (`NotifCategory.title`). |
| `description` | text | yes | | | (`NotifCategory.desc`). |
| `email_enabled` | boolean | no | | `true` | |
| `sms_enabled` | boolean | no | | `false` | |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `user_id`→`users(id)` **ON DELETE CASCADE**
- **UNIQUE** (`user_id`, `category`)
- **INDEX** `ix_notif_prefs_user` (`user_id`)

#### `payment_settings`
A workspace's payment connection and checkout preferences (US-SET-08/09/10) — one row per organization.

**PCI SAQ-A:** no card data reaches this schema at all. Card details are entered into the provider's own
hosted fields and never transit the API, so nothing on this row has to be masked or redacted.

`account_id` is a **reference, not a credential**: on its own it authorises nothing. `Payments` reads this
row through `MerchantAccountPort` and `Payouts` through `PayoutAccountPort`; neither reaches into the
table, and the two ports stay distinct because "can this workspace take money" and "can this workspace be
paid" are different answers at the provider (`charges_enabled` against `payouts_enabled`).

A workspace that is not set up to take money **cannot take paid registrations** — checkout refuses rather
than collecting somewhere the organizer cannot reach, which would issue a valid ticket against money they
can never claim. Free events are unaffected.

**This row is the connection; the keys are not here.** It once held the whole story: under a shared-platform-key
model the only facts a charge needed were non-secret — the connected-account reference and the publishable key —
because the platform's own secret signed every request on behalf of `account_id`. A workspace now supplies its
own provider keys, which live in `payment_credentials` (below), one row per mode and encrypted at rest.
`account_id`, the two ports and the refuse-to-sell rule are unaffected by that change; the claim that no
provider secret is stored anywhere is not, and `webhook_token` exists only because of it.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, UK | | One row per org. |
| `provider` | `payment_provider` | no | | `'stripe'` | |
| `mode` | `payment_mode` | no | | `'test'` | Test takes no real money. |
| `status` | `payment_connection_status` | no | | `'disconnected'` | |
| `account_id` | text | yes | | | Provider connected-account ref (e.g. `acct_…`). |
| `publishable_key` | text | yes | | | Non-secret; safe in the browser. |
| `connected_at` | timestamptz | yes | | | |
| `disconnected_at` | timestamptz | yes | | | |
| `default_currency` | char(3) | no | | `'THB'` | May differ from org currency (warn, don't block). |
| `statement_descriptor` | varchar(22) | yes | | | ≤22 chars, shown on card statements. |
| `save_cards` | boolean | no | | `false` | |
| `email_receipts` | boolean | no | | `true` | |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `version` | integer | no | | `1` | |
| `webhook_token` | text | yes | UK | | The unguessable path segment of this workspace's own callback URL. One per workspace, not per mode: the provider's test and live dashboards are separate, so the organizer registers the same URL twice and is handed a different signing secret each time. |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEY** `organization_id`→`organizations(id)` **ON DELETE CASCADE**
- **UNIQUE** `uq_payment_settings_org` (`organization_id`); partial `uq_payment_settings_webhook_token` (`webhook_token`) WHERE `webhook_token IS NOT NULL`
- **RLS** tenant isolation on `organization_id`
- `webhook_token` **is not a credential** and authorises nothing on its own — every callback still has to
  carry a valid provider signature over the raw bytes. It exists because a signature cannot be verified
  until you know *which* secret to verify it with, and one shared endpoint cannot tell one tenant from
  another.

#### `payment_credentials`
A workspace's own provider API keys (US-SET-08) — one row per mode.

**A row per mode rather than more columns on `payment_settings`**, because the Test/Live toggle swaps between
two independent pairs: an organizer keeps test keys while they build the event and adds live keys when they
open sales, and neither should overwrite the other. Mode is data here, not schema.

**The two secrets are write-only.** `secret_key_cipher` and `webhook_secret_cipher` are AES-256-GCM
ciphertext (`iv | authTag | ciphertext`) under `SECRET_ENCRYPTION_KEY`, the same cipher that protects TOTP
seeds in `two_factors.secret_encrypted`. Nothing decrypts them into a response — only into a provider
client — so there is no masked-value endpoint and no "reveal" path to get wrong. The publishable key is
stored in plain text because it is designed to be public.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, UK | | Tenant. |
| `mode` | `payment_mode` | no | UK | | `test` \| `live` — which key pair this row is. |
| `publishable_key` | text | yes | | | `pk_test_…` / `pk_live_…`; safe in the browser. |
| `secret_key_cipher` | bytea | yes | | | `sk_…`, encrypted at rest. Never read back to a caller. |
| `webhook_secret_cipher` | bytea | yes | | | `whsec_…` for this workspace's own endpoint, encrypted at rest. |
| `saved_at` | timestamptz | yes | | | When the organizer last saved this pair; drives the settings screen's "Last saved …" line. |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEY** `organization_id`→`organizations(id)` **ON DELETE CASCADE**
- **UNIQUE** `uq_payment_credentials_org_mode` (`organization_id`, `mode`)
- **RLS** tenant isolation on `organization_id` — defence in depth, as everywhere, but it matters most
  here: one workspace reading another's ciphertext is the worst read in the database.
- Every column but the tenant and the mode is nullable because a row is created when the organizer saves
  *either* half of the pair; a workspace mid-setup has a publishable key and no secret yet.
- No `version`: this is credential storage keyed by (`organization_id`, `mode`), overwritten in place, not
  an audited domain record whose stale writes are worth a `409`.

#### `payment_method_settings`
Per-method on/off for checkout (US-SET-09). An absent row means the method is disabled.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, UK | | |
| `method` | `payment_method` | no | UK | | Card/PromptPay/Bank transfer/Apple Pay/Google Pay. |
| `enabled` | boolean | no | | `false` | |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEY** `organization_id`→`organizations(id)` **ON DELETE CASCADE**
- **UNIQUE** `uq_payment_method_settings_org_method` (`organization_id`, `method`)
- **RLS** tenant isolation on `organization_id`

### Events & Program

#### `categories`
Event categorization for discover/landing. Org-scoped, seeded then customizable.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, UK, IX | | |
| `name` | text | no | UK | | Unique per org. |
| `description` | text | yes | | | `Category.desc`. |
| `icon` | text | no | | | Hugeicons slug (`CATEGORY_ICONS`). |
| `color` | `category_color` | no | | | pink/blue/amber/brand/violet/indigo/teal/red. |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `deleted_at` | timestamptz | yes | | | |
| `version` | integer | no | | `1` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEY** `organization_id`→`organizations(id)` **ON DELETE CASCADE**
- **UNIQUE** (`organization_id`, `name`)
- **INDEXES** `ix_categories_org` (`organization_id`), expression `ix_categories_search_name` (`search_norm(name)`) — `search_norm()` lowercases and strips Latin accents and Thai vowel/tone marks (migration `0023`), so Discover matches a category name however it was typed (US-DISC-02)
- `count` (events in category) is **derived**, not stored.

#### `landing_templates`
Global seed lookup of public landing-page templates. Not tenant-scoped.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | `template_id` | no | PK | | aurora/noir/minimal/atlas. |
| `title` | text | no | | | |
| `badge` | text | no | | | Marketing badge. |
| `description` | text | no | | | |

- **PRIMARY KEY** (`id`)
- The `LandingEvent` render contract is a runtime **projection** of `events` +
  children, not a stored table; only the template *selection* is stored on the event.

#### `events`
Central aggregate: an event owned by an organization.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | uuid | no | PK | `gen_random_uuid()` | |
| `organization_id` | bigint | no | FK→organizations.id, UK, IX | | |
| `slug` | text | no | UK | | Unique per org; `^[a-z0-9-]+$`, 3–80. |
| `name` | text | no | | | 3–120 chars. |
| `description` | text | yes | | | ≤250 chars. |
| `type` | `event_type` | no | IX | | |
| `status` | `event_status` | no | IX | `'draft'` | Lifecycle-controlled. |
| `requires_approval` | boolean | no | | `false` | When true a sign-up lands `pending` and waits for an organizer's decision (US-REG-02); when false checkout confirms it outright. Copied onto the order at placement — see `orders.requires_approval`. |
| `waitlist_enabled` | boolean | no | | `false` | When true attendees may join a waitlist for a general-admission ticket once it sells out (US-REG-04); when false the tier simply shows "sold out". |
| `bucket` | `event_bucket` | no | | | Derived from status + `end_at`. |
| `visibility` | `visibility` | no | | `'private'` | Public exposure gate. |
| `category_id` | bigint | yes | FK→categories.id, IX | | |
| `start_at` | timestamptz | no | IX | | Event start. |
| `end_at` | timestamptz | yes | | | Required to publish; `> start_at`. |
| `timezone` | text | no | | `'Asia/Bangkok'` | |
| `locale` | `locale` | yes | | | Language this event's automated messages default to (US-MSG-01); null → `organizations.locale`. |
| `venue_name` | text | yes | | | Required unless `is_online`. |
| `venue_address` | text | yes | | | |
| `city` | text | yes | | | Filter facet. |
| `is_online` | boolean | no | | `false` | |
| `online_note` | text | yes | | | Shown when online. |
| `seating_mode` | `seating_mode` | no | | `'ga'` | reserved vs general admission. |
| `capacity` | integer | yes | | | ≥1; Σ ticket quantities ≤ capacity. |
| `cover_image` | text | yes | | | |
| `accent_color` | text | yes | | | Landing accent. |
| `organizer_name` | text | no | | | Display organizer. |
| `contact_email` | citext | yes | | | Required to publish. |
| `agenda_title` | text | yes | | | Custom heading for the agenda section; defaults otherwise (US-PAGE-04). |
| `speakers_title` | text | yes | | | Custom heading for the speakers section (US-PAGE-04). |
| `landing_template_id` | `template_id` | yes | FK→landing_templates.id | | |
| `published_at` | timestamptz | yes | | | Set on first public state. |
| `cancelled_at` | timestamptz | yes | | | Triggers refund sweep. |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `deleted_at` | timestamptz | yes | | | |
| `created_by` | uuid | yes | FK→users.id | | Requires `evCreate`. |
| `version` | integer | no | | `1` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `category_id`→`categories(id)` **ON DELETE SET NULL**; `landing_template_id`→`landing_templates(id)` **ON DELETE SET NULL**; `created_by`→`users(id)` **ON DELETE SET NULL**
- **UNIQUE** (`organization_id`, `slug`)
- **INDEXES** `ix_events_org_status` (`organization_id`, `status`), `ix_events_start_at` (`start_at`), `ix_events_type` (`type`), `ix_events_category` (`category_id`); partial `ix_events_discover` (`start_at`) WHERE `deleted_at IS NULL AND visibility = 'public'` (the cross-tenant Discover grid, soonest first); expression `ix_events_search_name` (`search_norm(name)`), `ix_events_search_city` (`search_norm(city)`), `ix_events_search_venue` (`search_norm(venue_name)`) — the three fields Discover searches, diacritic-folded (migration `0023`, US-DISC-02)
- `tone`/`icon`/`tasks_open` are presentation/derived hints; `tasks_open` is a computed rollup.

#### `event_highlights`
A bullet the organizer arranges on the public page (US-PAGE-04). Ordered by `position`; an event with none
simply renders no Highlights section.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id | | Tenant. |
| `event_id` | uuid | no | FK→events.id, IX | | |
| `text` | text | no | | | |
| `icon` | text | yes | | | Optional icon key the template renders. |
| `position` | integer | no | | `0` | Organizer's order. |
| `created_at` / `updated_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `event_id`→`events(id)` **ON DELETE CASCADE**
- **INDEX** `ix_event_highlights_event` (`event_id`, `position`)
- **RLS** tenant isolation on `organization_id`

#### `event_faqs`
A question/answer pair shown on the public page (US-PAGE-06). Same ordering rule as highlights.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id | | Tenant. |
| `event_id` | uuid | no | FK→events.id, IX | | |
| `question` | text | no | | | |
| `answer` | text | no | | | |
| `position` | integer | no | | `0` | |
| `created_at` / `updated_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `event_id`→`events(id)` **ON DELETE CASCADE**
- **INDEX** `ix_event_faqs_event` (`event_id`, `position`)
- **RLS** tenant isolation on `organization_id`

#### `speakers`
A speaker featured at an event.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | uuid | no | PK | `gen_random_uuid()` | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | |
| `event_id` | uuid | no | FK→events.id, IX | | |
| `name` | text | no | | | |
| `role` | text | yes | | | Title/company. |
| `email` | citext | yes | | | Required to send speaker comms. |
| `phone` | text | yes | | | E.164. |
| `talk_title` | text | yes | | | `eventDetail.Speaker.talk`. |
| `tag` | text | yes | | | e.g. "Keynote". |
| `initials` | text | yes | | | `Speaker.ini`. |
| `tone` | `speaker_tone` | yes | | | Avatar colour. |
| `rating` | numeric(3,2) | yes | | | Avg feedback rating (derived). |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `deleted_at` | timestamptz | yes | | | |
| `version` | integer | no | | `1` | |
| `bio` | text | yes | | | The profile copy an organizer keeps current for the public speakers section (US-PROG-09/10). |
| `photo_url` | text | yes | | | Speaker photo; format and size limit are enforced at upload, never silently accepted (US-PROG-09). |
| `website` | text | yes | | | Personal or company link; a malformed link is rejected at the edge rather than silently dropped (US-PROG-09). |
| `social_links` | jsonb | yes | | | `{ twitter, linkedin, … }` — links only, each validated as a URL at the edge (US-PROG-09). |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `event_id`→`events(id)` **ON DELETE CASCADE**
- **UNIQUE** partial `uq_speakers_event_email` (`event_id`, `email`) WHERE `email IS NOT NULL AND deleted_at IS NULL` — one live speaker per email per event, the constraint behind the duplicate-email refusal in US-PROG-09/10. Partial on both counts deliberately: the many speakers with no email on file do not collide, and a soft-deleted row frees its email for re-use.
- **INDEXES** `ix_speakers_event` (`event_id`), `ix_speakers_org` (`organization_id`), `ix_speakers_name` (`event_id`, `name`)
- `sessions_count` is **derived** from `session_speakers`.
- The four profile columns sit after the audit block because they were added later (migration `0037`) and the table is documented in physical column order; they belong with `role`/`talk_title` logically.

#### `sessions`
Agenda/program session (SRS `AgendaSession`). Named `sessions` to pair with the `session_speakers` junction; auth sessions live in `auth_sessions`.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | uuid | no | PK | `gen_random_uuid()` | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | |
| `event_id` | uuid | no | FK→events.id, IX | | |
| `day` | smallint | no | | | 1-based day index. |
| `start_time` | time | no | | | `AgendaSession.s`. |
| `end_time` | time | yes | | | `AgendaSession.e`; `> start_time`. |
| `title` | text | no | | | |
| `type` | `session_type` | no | | | Keynote/Talk/Workshop/Panel/Break. |
| `room` | text | yes | | | `Session.room`. |
| `color` | `session_color` | yes | | | green/amber/rose/blue/slate. |
| `sort_order` | integer | no | | `0` | Ordering within day. |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `deleted_at` | timestamptz | yes | | | |
| `version` | integer | no | | `1` | |
| `description` | text | yes | | | The optional blurb attendees read on the agenda (US-PROG-02). |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `event_id`→`events(id)` **ON DELETE CASCADE**
- **CHECK** `end_time IS NULL OR end_time > start_time` — an open-ended session is allowed; a backwards one is not
- **INDEXES** `ix_sessions_event_day` (`event_id`, `day`), `ix_sessions_org` (`organization_id`)
- `AgendaDay` groupings are derived from the distinct `day` values; not stored.

#### `session_speakers` — JUNCTION (sessions ⇄ speakers)
Resolves the M:N between sessions and speakers (`AgendaSession.who`). `Break` sessions have no rows here.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `session_id` | uuid | no | FK→sessions.id, UK, IX | | |
| `speaker_id` | uuid | no | FK→speakers.id, UK, IX | | |
| `sort_order` | integer | no | | `0` | Speaker order on the session. |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `session_id`→`sessions(id)` **ON DELETE CASCADE**; `speaker_id`→`speakers(id)` **ON DELETE CASCADE**
- **UNIQUE** (`session_id`, `speaker_id`)
- **INDEXES** `ix_session_speakers_session` (`session_id`), `ix_session_speakers_speaker` (`speaker_id`)

### Ticketing & Discounts

#### `ticket_types`
A sellable ticket tier for an event.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | uuid | no | PK | `gen_random_uuid()` | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | |
| `event_id` | uuid | no | FK→events.id, IX | | |
| `name` | text | no | | | 1–60 chars. |
| `is_free` | boolean | no | | `false` | If true, `price_satang` = 0. |
| `price_satang` | bigint | no | | `0` | ≥0. Satang. |
| `currency` | char(3) | no | | `'THB'` | |
| `status` | `ticket_status` | no | | `'scheduled'` | onsale/scheduled/paused/soldout. |
| `admission_type` | `admission_type` | no | | `'general_admission'` | reserved_seat ties to seat_maps. |
| `sold` | integer | no | | `0` | Derived counter; ≥0, ≤`total`. |
| `total` | integer | no | | `0` | Allocation/quota, ≥0. |
| `sales_start_at` | timestamptz | yes | | | scheduled→onsale. |
| `sales_end_at` | timestamptz | yes | | | After → paused. |
| `min_per_order` | integer | no | | `1` | |
| `max_per_order` | integer | no | | `8` | Hard ceiling 8 seats/booking. |
| `includes` | jsonb | yes | | | "What's included" bullets on the public page (US-PAGE-05). |
| `is_recommended` | boolean | no | | `false` | The tier the page highlights (US-PAGE-05). |
| `badge` | text | yes | | | Its badge text, e.g. "Most popular". |
| `icon_class` | text | yes | | | Presentation tint. |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `deleted_at` | timestamptz | yes | | | |
| `version` | integer | no | | `1` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `event_id`→`events(id)` **ON DELETE CASCADE**
- **CHECK** `sold >= 0 AND sold <= total`, `price_satang >= 0`, `NOT is_free OR price_satang = 0`
- **INDEXES** `ix_ticket_types_event` (`event_id`), `ix_ticket_types_org` (`organization_id`)
- `soldout`, `revenue`, `featured` are derived. `sold` is an atomic counter guarded on booking.

#### `discount_codes`
A redeemable discount for an event (or org-wide).

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | uuid | no | PK | `gen_random_uuid()` | |
| `organization_id` | bigint | no | FK→organizations.id, UK, IX | | |
| `event_id` | uuid | yes | FK→events.id, UK, IX | | Null for org-wide codes. |
| `code` | text | no | UK | | 3–24, `^[A-Z0-9-]+$`, uppercase. |
| `type` | `discount_type` | no | | | percent \| fixed. |
| `value` | integer | no | | | percent: 1–100; fixed: satang ≥1. |
| `status` | `discount_status` | no | | `'scheduled'` | active/scheduled/expired/disabled. |
| `used` | integer | no | | `0` | Derived count; ≤`redemption_limit`. |
| `redemption_limit` | integer | no | | `0` | 0 = unlimited (`Discount.limit`). |
| `per_person_limit` | integer | no | | `0` | 0 = unlimited per buyer (US-TKT-07). |
| `min_order_satang` | bigint | no | | `0` | Order must reach this before the code applies (US-TKT-07). |
| `valid_from` | timestamptz | yes | | | scheduled→active. |
| `valid_until` | timestamptz | yes | | | After → expired. |
| `revenue_attributed_satang` | bigint | no | | `0` | Reporting rollup (derived). |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `deleted_at` | timestamptz | yes | | | Retired code — past orders keep their discount (US-TKT-09). |
| `version` | integer | no | | `1` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `event_id`→`events(id)` **ON DELETE CASCADE**
- **UNIQUE** (`organization_id`, `event_id`, `code`) — code unique per event (case-insensitive via uppercase storage). Neither partial nor `NULLS NOT DISTINCT`, so it diverges from the product rule in both directions: two identical org-wide codes (`event_id IS NULL`) do not collide, and `codeExists` in `discounts.repository.ts` is what rejects them; a soft-deleted code meanwhile goes on reserving its text, which that same live-only check does not expect
- **CHECK** `used >= 0`, `redemption_limit >= 0`, `per_person_limit >= 0`, `min_order_satang >= 0`; and the value matches its type — `percent` between 1 and 100, `fixed` at least 1 satang
- **INDEXES** `ix_discount_codes_event` (`event_id`), `ix_discount_codes_org` (`organization_id`)

#### `discount_redemptions` — JUNCTION (discount_codes ⇄ orders)
Records each application of a code to an order; enforces idempotent, once-per-order redemption and drives the `used` counter.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id | | |
| `discount_code_id` | uuid | no | FK→discount_codes.id, UK, IX | | |
| `order_id` | uuid | no | FK→orders.id, UK, IX | | |
| `buyer_email` | citext | no | IX | | Who redeemed it — enforces the per-person limit without joining orders (US-TKT-11). |
| `amount_satang` | bigint | no | | | Discount applied to this order. |
| `redeemed_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `discount_code_id`→`discount_codes(id)` **ON DELETE RESTRICT**; `order_id`→`orders(id)` **ON DELETE CASCADE**
- **UNIQUE** (`discount_code_id`, `order_id`) — re-submitting the same code on the same order is a no-op
- **CHECK** `amount_satang >= 0`
- **INDEXES** `ix_discount_redemptions_code` (`discount_code_id`), `ix_discount_redemptions_order` (`order_id`), `ix_discount_redemptions_buyer` (`discount_code_id`, `buyer_email`)

### Registration & Orders & Seating

#### `saved_events`
A portal user's cross-organizer event bookmarks (US-DISC-03). This table is
deliberately scoped by `user_id`, not `organization_id`: the Discover catalog is
cross-tenant, so one saved list can contain events from several organizers.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `user_id` | uuid | no | FK→users.id, UK, IX | | Bookmark owner. |
| `event_id` | uuid | no | FK→events.id, UK, IX | | Saved event. |
| `saved_at` | timestamptz | no | IX | `now()` | User-visible saved order. |
| `created_at` | timestamptz | no | | `now()` | Immutable creation time. |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `user_id`→`users(id)` **ON DELETE CASCADE**; `event_id`→`events(id)` **ON DELETE CASCADE**
- **UNIQUE** `uq_saved_events_user_event` (`user_id`, `event_id`) — makes repeated saves and guest-save adoption idempotent
- **INDEXES** `ix_saved_events_user` (`user_id`, `saved_at`), `ix_saved_events_event` (`event_id`)
- **Isolation** application-enforced by `user_id`; deliberately no tenant RLS because this is a cross-tenant personal list
- Immutable link table: no `organization_id`, `updated_at`, `deleted_at`, or `version`.

#### `attendees`
A person in the org's attendee CRM. May be a guest (no user) or linked 1:1 to a portal `users` row.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, UK, IX | | |
| `name` | text | no | | | 1–120 chars. |
| `email` | citext | no | UK | | Unique per org. |
| `phone` | text | yes | | | E.164. |
| `company` | text | yes | | | |
| `job_role` | text | yes | | | `eventDetail.Attendee.role`. |
| `tag` | `attendee_tag` | yes | | | VIP/Speaker/Sponsor/Student or null. |
| `initials` | text | yes | | | |
| `first_seen_at` | timestamptz | no | | `now()` | Drives `is_new`. |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `deleted_at` | timestamptz | yes | | | |
| `version` | integer | no | | `1` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEY** `organization_id`→`organizations(id)` **ON DELETE CASCADE**
- **UNIQUE** (`organization_id`, `email`)
- **INDEXES** `ix_attendees_org` (`organization_id`), `ix_attendees_tag` (`organization_id`, `tag`)
- `events_count`, `tickets_count`, `is_new`, `checked_in` are **derived**; per-event check-in is modelled on `tickets`/`check_ins`, not here.

#### `event_invitations`
An invitation to one event (US-REG-06). An invite is a **promise to attend, not a booking**: it reserves no
seat and holds no inventory, which is why this table has no link to `orders`, `order_items` or `tickets`.
An invitee becomes a registration by going through checkout like anybody else, and nothing here is amended
when they do.

There is also no accepted/declined column, because the product never asks the invitee to answer — the
answer is a registration or silence, and both are already readable from `orders`.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, UK, IX | | Tenant. |
| `event_id` | uuid | no | FK→events.id, UK, IX | | The event invited to. |
| `recipient_name` | text | no | | | Addressed name on the email. |
| `recipient_email` | citext | no | UK | | `citext`, and that is load-bearing — see below. |
| `message` | text | yes | | | Optional personal note that heads the email. |
| `invited_by` | uuid | yes | FK→users.id | | The organizer who sent it; null once that account is removed. |
| `sent_at` | timestamptz | no | | `now()` | When the invitation went out. |
| `created_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `event_id`→`events(id)` **ON DELETE CASCADE**; `invited_by`→`users(id)` **ON DELETE SET NULL**
- **UNIQUE** `uq_event_invitations_recipient` (`organization_id`, `event_id`, `recipient_email`) — one invite per person per event; the constraint the dedupe hangs on
- **INDEX** `ix_event_invitations_event` (`organization_id`, `event_id`)
- **RLS** tenant isolation on `organization_id`
- **`recipient_email` is `citext`, and that is what makes the constraint above work.** With plain `text`,
  `Ada@x.com` and `ada@x.com` are two different values, the unique index would not catch the second one,
  and the duplicate invitation US-REG-06 forbids would be sent anyway.
- Append-only link table: it carries no `updated_at`, `deleted_at` or `version`. Withdrawing an invitation
  is deliberately not modelled — the mail has already left, so there is nothing a flag here could undo.

#### `orders`
The booking header — the "registration" as a commerce order. Parent of `order_items` and issued `tickets`.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | uuid | no | PK | `gen_random_uuid()` | |
| `organization_id` | bigint | no | FK→organizations.id, UK, IX | | |
| `reference` | text | no | UK | | `REG-YYYY-NNNNNN`, unique per org. |
| `event_id` | uuid | no | FK→events.id, IX | | |
| `attendee_id` | bigint | yes | FK→attendees.id, IX | | Null for guest bookings. |
| `buyer_name` | text | no | | | |
| `buyer_email` | citext | no | | | Confirmation sent here. |
| `buyer_phone` | text | yes | | | E.164; SMS reminders. |
| `status` | `order_status` | no | IX | `'pending'` | confirmed/pending/waitlisted/cancelled/rejected/expired. **`expired` is written by eventa-worker, not the API** — see the writer note below. |
| `payment_status` | `payment_status` | no | | `'pending'` | Payment sub-state rollup. |
| `seats` | smallint | no | | | 1–8 per booking. |
| `subtotal_satang` | bigint | no | | | Pre-discount, pre-VAT. |
| `discount_code_id` | uuid | yes | FK→discount_codes.id, IX | | Applied code. |
| `discount_amount_satang` | bigint | no | | `0` | |
| `vat_amount_satang` | bigint | no | | `0` | 7% of taxable base; 0 for free. |
| `total_satang` | bigint | no | | | subtotal − discount + VAT. |
| `currency` | char(3) | no | | `'THB'` | |
| `registered_at` | timestamptz | no | | `now()` | |
| `cancelled_at` | timestamptz | yes | | | |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `deleted_at` | timestamptz | yes | | | |
| `created_by` | uuid | yes | FK→users.id | | Null for guest-originated. |
| `version` | integer | no | | `1` | |
| `idempotency_key` | text | yes | UK | | The buyer's key for one checkout attempt (US-DISC-06); unique per workspace, so a double-tapped Confirm or a retried request resolves to the order it already placed and charges once. |
| `confirmed_at` | timestamptz | yes | | | When it actually became confirmed — written on both settlement paths, payment and approval. The confirmation date US-REG-01 renders; `status` alone never carried it. |
| `approved_at` | timestamptz | yes | | | Set only on the approve decision (US-REG-02), never by checkout. |
| `rejected_at` | timestamptz | yes | | | Set on the reject decision (US-REG-02) — the machinery behind `status = 'rejected'` above. |
| `decided_by` | uuid | yes | FK→users.id | | Who approved or rejected (US-REG-02). Null when nobody decided: a free place confirmed off the waitlist by a capacity raise. |
| `rejection_reason` | text | yes | | | The reason given with the rejection; carried into the rejection notice (US-REG-02). |
| `offered_at` | timestamptz | yes | | | A freed seat was offered to this waitlist entry (US-REG-04): it turns `pending` with a seat hold lasting the offer window, so paying for it is the ordinary checkout payment. |
| `offered_by` | uuid | yes | FK→users.id | | The organizer who promoted them. Null when the line offered it by itself — a lapsed offer passed on, or a capacity raise freeing places. |
| `offer_expires_at` | timestamptz | yes | | | Pay-by deadline for the offer (`WAITLIST_OFFER_HOURS`, 24 h by default); the seat hold expires with it. |
| `offer_skipped` | smallint | yes | | | How many were ahead in line when an organizer chose this one — `0` is first-come-first-served, non-zero is the out-of-order promotion US-REG-04 requires be recorded. A count, not a flag, so how far the queue was jumped stays on the row. |
| `requires_approval` | boolean | no | | `false` | The event's "Require approval" rule copied onto the order at placement (US-REG-02), so flipping the event's switch later — or while this buyer's payment is in flight — never changes the deal they were shown. Not backfilled. |
| `approval_requested_at` | timestamptz | yes | | | When the organizer's decision became the only thing left: at placement for a free order, when the money lands for a paid one. `pending` with this set is "awaiting approval" — the worker's expiry sweep skips such an order, since it is on the organizer's clock, not the buyer's. |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `event_id`→`events(id)` **ON DELETE RESTRICT**; `attendee_id`→`attendees(id)` **ON DELETE SET NULL**; `discount_code_id`→`discount_codes(id)` **ON DELETE SET NULL**; `created_by`→`users(id)` **ON DELETE SET NULL**; `decided_by`→`users(id)` **ON DELETE SET NULL**; `offered_by`→`users(id)` **ON DELETE SET NULL**
- **UNIQUE** `uq_orders_org_reference` (`organization_id`, `reference`), `uq_orders_org_idem` (`organization_id`, `idempotency_key`) — a double-submitted checkout resolves to the one order it meant to place rather than charging twice (US-DISC-06)
- **CHECK** `seats BETWEEN 1 AND 8`, all money `>= 0`
- **INDEXES** `ix_orders_event` (`event_id`), `ix_orders_attendee` (`attendee_id`), `ix_orders_status` (`organization_id`, `status`), `ix_orders_discount` (`discount_code_id`)
- `checked_in` is derived from child `tickets`.
- **The lifecycle timestamps are a decision log, not a mirror of `status`.** `confirmed_at` is
  written on both settlement paths; `approved_at`/`rejected_at`/`decided_by`/`rejection_reason`
  only on the decision path (US-REG-02); `offered_at`/`offered_by`/`offer_expires_at`/
  `offer_skipped` only on the waitlist-offer path (US-REG-04). `status` says where a registration
  is now, these say how it got there — and two states the enum does not name are read off them:
  `pending` with `approval_requested_at` set is **awaiting approval**, `pending` with
  `offer_expires_at` set is **holding a waitlist offer**. Both are deliberately on the
  registration row rather than in a separate log, so "who decided, who was offered it, when,
  until when, and who was passed over" is one row and no join.
- **WRITERS — eventa-api owns this table, with one agreed exception.** The
  order-expiry sweep in **eventa-worker** (`modules/order-expiry`, ADR-14) sets
  `status = 'expired'` on `pending`/`pending` orders whose seat holds lapsed, and
  retires those holds in the same transaction. It is the only clock-driven write
  to an API aggregate in the platform, and the only place outside eventa-api that
  writes `orders`. Two consequences of the approval and waitlist columns above:
  the sweep **reads** `approval_requested_at` and excludes an order awaiting an
  organizer's decision, because that order is on the organizer's clock, not the
  buyer's (US-REG-02); and a lapsed offer is passed on in the same sweep — the
  closed order is recognised by `offer_expires_at IS NOT NULL`, and the next
  waitlisted registration for that tier is given the seat, so the worker also
  writes `offered_at`, `offer_expires_at`, `offer_skipped = 0` and `offered_by`
  **null**, null being how the row says the line offered it rather than a person
  (US-REG-04). **Consequence:** the worker's `orders` and `seat_holds` schema
  files are *write* mirrors, not read views — an enum value added here must be
  added there in the same change, because a write fails on a value a read would
  simply never have produced.

#### `order_items`
Line item: a quantity of one ticket type within an order.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id | | |
| `order_id` | uuid | no | FK→orders.id, UK, IX | | |
| `ticket_type_id` | uuid | no | FK→ticket_types.id, UK, IX | | |
| `quantity` | smallint | no | | | ≥1. |
| `unit_price_satang` | bigint | no | | | Captured price at purchase. |
| `line_subtotal_satang` | bigint | no | | | quantity × unit price. |
| `currency` | char(3) | no | | `'THB'` | |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `order_id`→`orders(id)` **ON DELETE CASCADE**; `ticket_type_id`→`ticket_types(id)` **ON DELETE RESTRICT**
- **UNIQUE** (`order_id`, `ticket_type_id`)
- **CHECK** `quantity >= 1` — a line item with nothing on it is not a line item; the order's own money checks live on `orders`
- **INDEXES** `ix_order_items_order` (`order_id`), `ix_order_items_ticket_type` (`ticket_type_id`)

#### `tickets`
One issued admission ticket per admitted person/seat, carrying a signed QR token. Check-in is per-ticket.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | uuid | no | PK | `gen_random_uuid()` | |
| `organization_id` | bigint | no | FK→organizations.id | | |
| `order_id` | uuid | no | FK→orders.id, IX | | |
| `order_item_id` | bigint | no | FK→order_items.id | | Which line issued it. |
| `event_id` | uuid | no | FK→events.id, IX | | Denormalized for check-in scans. |
| `ticket_type_id` | uuid | no | FK→ticket_types.id | | |
| `attendee_id` | bigint | yes | FK→attendees.id, IX | | Named holder if known. |
| `qr_token` | text | no | UK | | Signed, opaque; unique globally. |
| `holder_name` | text | yes | | | Per-ticket name for badges. |
| `ticket_label` | text | yes | | | Ticket-type label at the door. |
| `status` | `issued_ticket_status` | no | | `'issued'` | issued/checked_in/void/refunded/transferred. |
| `checked_in_at` | timestamptz | yes | | | Time of successful scan. |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `deleted_at` | timestamptz | yes | | | |
| `version` | integer | no | | `1` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `order_id`→`orders(id)` **ON DELETE CASCADE**; `order_item_id`→`order_items(id)` **ON DELETE CASCADE**; `event_id`→`events(id)` **ON DELETE RESTRICT**; `ticket_type_id`→`ticket_types(id)` **ON DELETE RESTRICT**; `attendee_id`→`attendees(id)` **ON DELETE SET NULL**
- **UNIQUE** (`qr_token`)
- **INDEXES** `ix_tickets_order` (`order_id`), `ix_tickets_event` (`event_id`), `ix_tickets_attendee` (`attendee_id`), `ix_tickets_status` (`event_id`, `status`)

#### `seat_maps`
A reserved-seating layout for an event (present only when `seating_mode = reserved`).

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | |
| `event_id` | uuid | no | FK→events.id, UK, IX | | One map per event. |
| `name` | text | no | | | e.g. "Grand Hall". |
| `layout` | jsonb | yes | | | Rendering geometry (rows/sections). |
| `total_seats` | integer | no | | `0` | Derived from `seats`. |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `deleted_at` | timestamptz | yes | | | |
| `version` | integer | no | | `1` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `event_id`→`events(id)` **ON DELETE CASCADE**
- **UNIQUE** (`event_id`)
- **INDEX** `ix_seat_maps_org` (`organization_id`)

#### `seats`
An individual addressable seat within a seat map.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id | | |
| `seat_map_id` | bigint | no | FK→seat_maps.id, UK, IX | | |
| `section` | text | yes | | | Section/zone label. |
| `row_label` | text | yes | | | Row. |
| `seat_number` | text | no | UK | | Seat within row. |
| `ticket_type_id` | uuid | yes | FK→ticket_types.id, IX | | Tier this seat belongs to. |
| `status` | `seat_status` | no | | `'available'` | available/held/reserved/sold/blocked. |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `version` | integer | no | | `1` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `seat_map_id`→`seat_maps(id)` **ON DELETE CASCADE**; `ticket_type_id`→`ticket_types(id)` **ON DELETE SET NULL**
- **UNIQUE** (`seat_map_id`, `section`, `row_label`, `seat_number`)
- **INDEXES** `ix_seats_map` (`seat_map_id`), `ix_seats_ticket_type` (`ticket_type_id`)

#### `seat_assignments` — JUNCTION (seats ⇄ tickets)
Binds an issued ticket to a specific seat. Resolves the M:N-over-time relationship (a seat is assignable across events; each assignment is unique per active seat).

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id | | |
| `seat_id` | bigint | no | FK→seats.id, UK, IX | | |
| `ticket_id` | uuid | no | FK→tickets.id, UK, IX | | One seat per ticket. |
| `assigned_at` | timestamptz | no | | `now()` | |
| `released_at` | timestamptz | yes | | | Set when seat released (e.g. refund). |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `seat_id`→`seats(id)` **ON DELETE RESTRICT**; `ticket_id`→`tickets(id)` **ON DELETE CASCADE**
- **UNIQUE** partial `uq_seat_active` (`seat_id`) WHERE `released_at IS NULL` — a seat holds at most one live assignment; **UNIQUE** (`ticket_id`)
- **INDEXES** `ix_seat_assignments_seat` (`seat_id`), `ix_seat_assignments_ticket` (`ticket_id`)

### Attendance & Check-in

#### `check_ins`
**Who is inside: one row per admitted ticket, and at most one.** This is the state of the room, not a log of
the door — refusals and repeat presentations are recorded next door in `scan_attempts`.

`uq_check_ins_ticket`, a `UNIQUE` on `ticket_id` alone, is the entire concurrency story of a busy entrance.
Two staff scanning the same code both insert; one wins, and the loser reads back the winner's arrival time
instead of admitting a second person through the same pass. That single constraint is why the table looks
the way it does, and why `ticket_id` is `NOT NULL`.

Undoing a check-in **deletes** the row, because an admission that was reversed did not happen. The scan
that produced it survives in `scan_attempts`, because the scan did happen.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | Tenant. |
| `event_id` | uuid | no | FK→events.id, IX | | |
| `ticket_id` | uuid | no | FK→tickets.id, UK | | The admitted pass. Unique — see above. |
| `attendee_id` | bigint | yes | FK→attendees.id, IX | | The CRM row admitted, where the ticket names one. |
| `checked_in_at` | timestamptz | no | IX | `now()` | Arrival time; the door feed reads on it. |
| `method` | `check_in_method` | no | | | `qr` \| `manual` \| `upload` — how the holder was identified. |
| `checked_in_by` | uuid | yes | FK→users.id | | Staff member (needs `regCheckin`); null once that account is removed. |
| `station_id` | text | yes | | | Which door or kiosk admitted them. |
| `created_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `event_id`→`events(id)` **ON DELETE RESTRICT**; `ticket_id`→`tickets(id)` **ON DELETE CASCADE**; `attendee_id`→`attendees(id)` **ON DELETE SET NULL**; `checked_in_by`→`users(id)` **ON DELETE SET NULL**
- **UNIQUE** `uq_check_ins_ticket` (`ticket_id`)
- **INDEXES** `ix_check_ins_event` (`organization_id`, `event_id`), `ix_check_ins_feed` (`event_id`, `checked_in_at`), `ix_check_ins_attendee` (`attendee_id`)
- **RLS** tenant isolation on `organization_id`
- `event_id` is `RESTRICT` while `ticket_id` is `CASCADE`: what happened at a door must outlive the event
  row, but an admission is meaningless once the pass it admitted is gone.
- Writing a row authoritatively sets `tickets.status = checked_in` and `tickets.checked_in_at`.
- No `updated_at`, `deleted_at` or `version` — a row is inserted on arrival and deleted on undo; it is
  never edited.

> **Designed, not built.** The catalog originally made this table the scan log itself: one row per *attempt*,
> with `state` (`scan_state`), `scanned_qr` for an unrecognised code, and `other_event_id` for a pass
> belonging elsewhere. **None of those three columns exists** — and the design could not have worked as
> drawn. A refusal is not one row per ticket, since the same damaged pass gets presented five times in a
> minute and an unknown code has no ticket to be unique on at all, so recording refusals here would have
> meant weakening `uq_check_ins_ticket`, the one constraint protecting the admission. The `scan_outcome`
> enum sat unused from migration `0039` until `0068` gave it a table of its own; **until then, failed scans
> were not recorded anywhere** and door-incident reporting was impossible. `scan_attempts` below is where
> that design landed. `device_label` and `scanned_at` were built as `station_id` and `checked_in_at`; those
> are renames and are corrected above.

#### `scan_attempts`
**The history of the door: every scan it made and what it decided**, admissions and refusals alike,
append-only (US-REG-12). `check_ins` above answers *who is inside*; this answers *what happened at the
entrance*, and the two are written in the same transaction on an admission so neither can exist without
the other.

**It records admissions too, `already_checked_in` ones included, and that is the point.** A table of
refusals alone has no denominator: "19 invalid scans" is unreadable without the 1,340 that worked, and a
refusal *rate* is what distinguishes a busy door from a scanner that has stopped working.

**`token_fingerprint` is a SHA-256 digest of the code presented, never the code.** `tickets.qr_token` is a
bearer credential — holding that string *is* the entitlement to walk in — and this table is append-only,
outlives both the event and the ticket, and is readable by everyone who may work or review a door. Raw
tokens in it would turn an incident log into a list of working passes and an export of it into a set of
usable tickets. The digest costs nothing that is needed: a recognised code already carries its `ticket_id`,
so "was this ticket turned away?" never needed the token; grouping on the digest still separates *one
damaged pass presented forty times* from *forty different bad codes*, which is the difference between a
broken ticket and somebody probing the door; and "was **this** code presented?" stays a lookup, because
whoever holds the code can hash it the same way. Storing a credential as a digest is already this schema's
answer in `api_keys.key_hash`.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | Tenant. |
| `event_id` | uuid | no | FK→events.id, IX | | The event the **station** was bound to. |
| `outcome` | `scan_outcome` | no | | | `admitted` \| `already_checked_in` \| `invalid` \| `wrong_event` \| `cancelled`. |
| `method` | `check_in_method` | no | | | `qr` \| `manual` \| `upload`. |
| `ticket_id` | uuid | yes | FK→tickets.id, IX | | Null for an unrecognised code — the commonest refusal, and one with no ticket to point at. |
| `ticket_event_id` | uuid | yes | FK→events.id | | Which event the presented pass actually belonged to. Written **only** when it differs from `event_id`. |
| `token_fingerprint` | text | yes | IX | | SHA-256 of the presented code; null when no code was presented, which is precisely what a manual admission is. |
| `station_id` | text | yes | | | Which door read it, for reconciling a busy entrance. |
| `scanned_by` | uuid | yes | FK→users.id | | Staff member; null once that account is removed. |
| `scanned_at` | timestamptz | no | IX | | When the scan happened, which an offline replay may backdate. |
| `created_at` | timestamptz | no | | `now()` | When the row reached us. |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `event_id`→`events(id)` **ON DELETE RESTRICT**; `ticket_id`→`tickets(id)` **ON DELETE SET NULL**; `ticket_event_id`→`events(id)` **ON DELETE SET NULL**; `scanned_by`→`users(id)` **ON DELETE SET NULL**
- **INDEXES** `ix_scan_attempts_event` (`organization_id`, `event_id`, `scanned_at`) — the incident report reads one event's night newest-first, and the per-outcome counts are filtered off this same index rather than getting one of their own, because a night is a few thousand rows and a second index would be paid on every scan to save a filter over a set that small; `ix_scan_attempts_token` (`organization_id`, `token_fingerprint`) for the one-bad-pass-or-forty question; `ix_scan_attempts_ticket` (`ticket_id`) for settling a single disputed entry
- **RLS** tenant isolation on `organization_id`
- **`ON DELETE` reasoning.** `event_id` is `RESTRICT` for the same reason as `check_ins.event_id` — door
  history must outlive the event row. `ticket_id` is `SET NULL` rather than `CASCADE`, which would let
  erasing a ticket erase the evidence that it was refused, or `RESTRICT`, which would block the hard
  deletes account erasure performs; the outcome, the time, the station and the fingerprint survive either
  way. `ticket_event_id` is `SET NULL` because it is a secondary reference and deleting some *other* event
  should not be blocked by a refusal recorded here.
- **Keeping `ticket_event_id` null unless it differs** is what lets `ticket_event_id IS NOT NULL` read as
  "a pass for somewhere else turned up here" instead of also matching every row where it would merely
  repeat the event we already know.
- **Append-only, and that is what makes it evidence rather than state.** Nothing updates or deletes these
  rows, and an undone check-in writes nothing here — an undo is not a scan, `audit_events` already answers
  who undid what, and a synthetic row would make the outcome counts lie. That append-only property is
  enforced by the single writer having no update or delete path, not by the schema: the app connects as the
  schema owner, so a `REVOKE` would not bind it, exactly as the RLS policy above does not.
- No `updated_at`, `deleted_at` or `version`, for the same reason.

### Payments & Finance

#### `payments`
A captured (or attempted) charge against an order. Append-only ledger.

`gateway_account_id` records **which** provider account took the money, because a refund has to reverse
on that same one and `payment_settings` can change underneath it — a workspace can disconnect, or
reconnect to a different account, between the charge and the reversal. NULL means the platform account,
which is where every payment taken before the column existed genuinely landed, so those rows are correct
as they stand and must not be backfilled.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | uuid | no | PK | `gen_random_uuid()` | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | |
| `txn` | text | no | UK | | `TXN-…` reference, unique per org. |
| `order_id` | uuid | no | FK→orders.id, IX | | |
| `event_id` | uuid | no | FK→events.id, IX | | Denormalized for reporting. |
| `payer_name` | text | no | | | |
| `method` | `payment_method` | no | | | Card/PromptPay/Bank transfer. |
| `amount_satang` | bigint | no | | | Gross charged. |
| `currency` | char(3) | no | | `'THB'` | |
| `status` | `payment_status` | no | IX | `'pending'` | paid/pending/refunded/failed. |
| `paid_at` | timestamptz | yes | | | Set when paid. |
| `gateway_ref` | text | yes | | | PSP charge id. |
| `statement_descriptor` | varchar(22) | yes | | | ≤22 chars. |
| `fee_amount_satang` | bigint | yes | | | Processing fee withheld. |
| `idempotency_key` | text | no | UK | | Exactly-once capture. |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `gateway_account_id` | text | yes | | | Which provider account took the money — see above. Null means the platform account. |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `order_id`→`orders(id)` **ON DELETE RESTRICT**; `event_id`→`events(id)` **ON DELETE RESTRICT**
- **UNIQUE** `uq_payments_org_txn` (`organization_id`, `txn`), `uq_payments_org_idem` (`organization_id`, `idempotency_key`)
- **INDEXES** `ix_payments_order` (`order_id`), `ix_payments_event` (`event_id`), `ix_payments_status` (`organization_id`, `status`)

#### `refunds`
A full/partial reversal of a payment. Append-only.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | uuid | no | PK | `gen_random_uuid()` | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | |
| `payment_id` | uuid | no | FK→payments.id, IX | | Payment reversed. |
| `order_id` | uuid | no | FK→orders.id, IX | | Denormalized. |
| `amount_satang` | bigint | no | | | ≤ payment minus prior refunds. |
| `currency` | char(3) | no | | `'THB'` | |
| `reason` | text | yes | | | Reason code / free text. |
| `status` | `refund_status` | no | | `'pending'` | pending/succeeded/failed. |
| `issued_by` | uuid | no | FK→users.id | | Requires `finRefund`. |
| `issued_at` | timestamptz | no | | `now()` | |
| `gateway_ref` | text | yes | | | PSP refund id. |
| `idempotency_key` | text | no | UK | | Exactly-once. |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `settled_at` | timestamptz | yes | | | When the money actually went back — the VAT month the reversal belongs to (US-FIN-02, US-FIN-11). Null until it succeeds; see below. |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `payment_id`→`payments(id)` **ON DELETE RESTRICT**; `order_id`→`orders(id)` **ON DELETE RESTRICT**; `issued_by`→`users(id)` **ON DELETE RESTRICT**
- **UNIQUE** (`organization_id`, `idempotency_key`)
- **INDEXES** `ix_refunds_payment` (`payment_id`), `ix_refunds_order` (`order_id`)
- **`settled_at` is separate from `issued_at` because the two can be months apart.** A card
  refund settles the moment it is issued; a PromptPay refund settles days later, through the
  provider's webhook, once the buyer has given the provider a bank account. The VAT ledger must
  back the reversal out of the month the money **moved**, because the issue month may already be
  filed and frozen, and a frozen return never takes it (US-FIN-11, and the same rule as
  `tax_periods` "adjustment carries into the next open period"). No backfill was needed: until
  the provider's refund webhook existed a refund only ever settled when it was issued, and the
  ledger reads `coalesce(settled_at, issued_at)`, so no figure already reported moves.

#### `invoices`
A tax invoice for an order.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, UK, IX | | |
| `number` | text | no | UK | | `INV-YYYY-NNNN`, unique per org. |
| `order_id` | uuid | no | FK→orders.id, IX | | Source booking (`Invoice.ref`). |
| `event_id` | uuid | no | FK→events.id, IX | | |
| `buyer_name` | text | no | | | Bill-to. |
| `issued_at` | date | no | | | |
| `due_at` | date | no | | | Terms = 14 days from issue. |
| `subtotal_satang` | bigint | no | | | Pre-VAT. |
| `vat_amount_satang` | bigint | no | | | 7% of subtotal. |
| `amount_satang` | bigint | no | | | Total incl. VAT. |
| `currency` | char(3) | no | | `'THB'` | |
| `status` | `invoice_status` | no | IX | `'issued'` | paid/issued/overdue/void. |
| `paid_via` | `payment_method` | yes | | | Set when paid. |
| `paid_on` | date | yes | | | Set when paid. |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `version` | integer | no | | `1` | |
| `buyer_email` | citext | no | | | Bill-to address, snapshotted at issue rather than read through to the order: a tax invoice must still show who it was billed to after the buyer edits their account. |
| `voided_at` | timestamptz | yes | | | Set when an Admin voids an issued or overdue invoice raised in error (US-FIN-10). |
| `void_reason` | text | yes | | | The optional reason given with the void (US-FIN-10). |
| `voided_by` | uuid | yes | FK→users.id | | The Admin who voided it; null once that account is removed. |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `order_id`→`orders(id)` **ON DELETE RESTRICT**; `event_id`→`events(id)` **ON DELETE RESTRICT**; `voided_by`→`users(id)` **ON DELETE SET NULL**
- **UNIQUE** `uq_invoices_org_number` (`organization_id`, `number`); **UNIQUE** partial `uq_invoices_live_order` (`order_id`) WHERE `status <> 'void'` — issuing twice for one order yields one invoice (US-FIN-07), and the partial clause is what lets a correction be a void plus a brand-new invoice (US-FIN-10) instead of an amendment
- **INDEXES** `ix_invoices_order` (`order_id`), `ix_invoices_event` (`event_id`), `ix_invoices_status` (`organization_id`, `status`), `ix_invoices_due` (`organization_id`, `due_at`) — ageing is judged on `due_at`
- `overdue` is computed (`status = issued AND due_at < today`), never stored free-hand.
- **Voiding never reuses the number.** The three `void_*` columns record the decision on the
  invoice itself and `status` moves to `void`; the sequence keeps the number for audit, which is
  why a correction is a new invoice and not an edit (US-FIN-10). Voiding frees the
  `uq_invoices_live_order` slot above so that new invoice can be issued for the same order.

#### `payouts`
A settlement of net proceeds to the organizer bank account over a period.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, UK, IX | | |
| `reference` | text | no | UK | | `PO-…` human reference. |
| `amount_satang` | bigint | no | | | Net paid out. |
| `currency` | char(3) | no | | `'THB'` | |
| `bank_account` | text | no | | | Masked descriptor. |
| `status` | `payout_status` | no | IX | `'scheduled'` | paid/processing/scheduled/failed. |
| `period_covered` | text | yes | | | Settlement window. |
| `event_id` | uuid | yes | FK→events.id, IX | | Optional attribution. |
| `requested_at` | timestamptz | no | | | |
| `completed_at` | timestamptz | yes | | | Set when paid/failed. |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `version` | integer | no | | `1` | |
| `failure_reason` | text | yes | | | Why the bank rejected the transfer — what the organizer must act on before retrying (US-FIN-04). |
| `gateway_ref` | text | yes | | | The provider's reference for the transfer, replaced on a retry. |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `event_id`→`events(id)` **ON DELETE SET NULL**
- **UNIQUE** (`organization_id`, `reference`)
- **INDEXES** `ix_payouts_status` (`organization_id`, `status`), `ix_payouts_event` (`event_id`)
- **A retry reuses the row.** US-FIN-04 requires that retrying a failed payout move *this* payout
  back to `processing` rather than create a second one, so the retry overwrites `gateway_ref` and
  `failure_reason` with the new attempt's and restarts `requested_at`, while `reference` and the
  amount stay put. Only the latest attempt is therefore readable here; each retry also writes an
  `audit_events` row in the same transaction, and that is where the per-attempt history lives.

#### `payout_items` — JUNCTION (payouts ⇄ payments)
Resolves the M:N between a payout and the payments it settles (net of refunds/fees).

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id | | |
| `payout_id` | bigint | no | FK→payouts.id, UK, IX | | |
| `payment_id` | uuid | no | FK→payments.id, UK, IX | | |
| `gross_satang` | bigint | no | | | Payment gross included. |
| `refund_satang` | bigint | no | | `0` | Refunds netted. |
| `fee_satang` | bigint | no | | `0` | Fees withheld. |
| `net_satang` | bigint | no | | | gross − refund − fee. |
| `created_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `payout_id`→`payouts(id)` **ON DELETE CASCADE**; `payment_id`→`payments(id)` **ON DELETE RESTRICT**
- **UNIQUE** (`payout_id`, `payment_id`), and (`payment_id`) — a payment settles in at most one payout
- **INDEXES** `ix_payout_items_payout` (`payout_id`), `ix_payout_items_payment` (`payment_id`)

#### `tax_periods`
Monthly VAT/WHT filing period. VAT recomputed from confirmed registrations net of refunds.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, UK, IX | | |
| `period` | text | no | UK | | Month label (e.g. "Jun"). |
| `year` | smallint | no | UK | | |
| `due_at` | date | no | | | 15th of the following month. |
| `sales_satang` | bigint | no | | | Taxable sales. |
| `vat_satang` | bigint | no | | | round(sales × 0.07). |
| `wht_satang` | bigint | no | | `0` | Withholding tax. |
| `remitted_satang` | bigint | no | | `0` | Non-zero only when filed. |
| `currency` | char(3) | no | | `'THB'` | |
| `status` | `tax_status` | no | | `'upcoming'` | filed/due/upcoming. |
| `filed_at` | timestamptz | yes | | | |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `version` | integer | no | | `1` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEY** `organization_id`→`organizations(id)` **ON DELETE CASCADE**
- **UNIQUE** (`organization_id`, `period`, `year`)
- **INDEX** `ix_tax_periods_org` (`organization_id`)

### Engagement & Messaging

#### `message_templates`
Automated email/SMS template (seeded slugs like `registration-confirmation`).

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, UK, IX | | |
| `slug` | text | no | UK | | e.g. `registration-confirmation`, `payment-receipt`. |
| `title` | text | no | | | |
| `description` | text | yes | | | Editor trigger line. |
| `channels` | `message_channel[]` | no | | `'{email}'` | Subset of {email, sms}. |
| `active` | boolean | no | | `true` | Automation enabled — the kill switch US-MSG-01 checks. |
| `tags` | text[] | no | | `'{}'` | Allowed merge tags (`COMMON_TAGS`). |
| `email_subject_en` | text | yes | | | Required when channels ∋ email. |
| `email_subject_th` | text | yes | | | |
| `email_body_en` | text | yes | | | |
| `email_body_th` | text | yes | | | |
| `sms_body_en` | text | yes | | | ≤160 chars/segment; Thai encodes shorter. |
| `sms_body_th` | text | yes | | | |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEY** `organization_id`→`organizations(id)` **ON DELETE CASCADE**
- **UNIQUE** (`organization_id`, `slug`)
- **INDEX** `ix_message_templates_org` (`organization_id`)
- **RLS** `tenant_isolation` on `organization_id` (migration `0029`)

**Built (migration `0028`), with three deliberate departures from the original spec:**
- **The wording columns are split `_en`/`_th`.** US-MSG-02 requires that "each message exists in English and Thai", which single `email_subject`/`email_body`/`sms_body` columns cannot express.
- **`icon` / `icon_class` are omitted.** They are presentation choices for the settings screen, not data the sender needs; the front-end can map them from `slug`.
- **No `deleted_at` / `version`.** A template is configuration keyed by (`organization_id`, `slug`), not an audited domain record — it is edited in place, and deleting one would just restore the built-in default.

**Scope is per WORKSPACE, per message kind — there is deliberately no `event_id`.** US-MSG-01's "given the organizer has turned a given message off" therefore means workspace-wide; per-event control of an automated message exists in neither the story nor this model.

**An ABSENT row means active.** Rows are not seeded on workspace creation, so a workspace that has never opened its message settings still sends its confirmations; the sender treats "no row" as on.

#### `announcements`
A one-off broadcast to an event's registrants. One row per broadcast, whether it has gone out, is waiting
for its hour, or was called off.

**Its three states are mutually exclusive and the database enforces it.** `ck_announcements_state` is a
single `CHECK` saying that a `sent` row has both a `sent_at` and a `recipient_count`, a `scheduled` row has
a `scheduled_for` and no `sent_at`, and a `cancelled` row has a `cancelled_at` and no `sent_at`. That is
what stops the two failure modes worth stopping: a "sent" broadcast with nothing recording that it went,
and a cancelled one that went anyway.

**Cancellation replaces soft delete here, and it is the better model.** Withdrawing a scheduled broadcast
is not an editing mistake to be hidden — it is a decision somebody made, with an actor
(`cancelled_by_user_id`) and a time, and a reader needs to see that it was called off rather than find a
gap. There is no `deleted_at` as a result.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | Tenant. |
| `event_id` | uuid | no | IX | | The event addressed. **No FK constraint** — see the note below. |
| `subject` | text | no | | | Subject line. |
| `body` | text | no | | | Message body. |
| `recipient_count` | bigint | yes | | | How many it actually reached; `NOT NULL` once `status = sent`, by CHECK. |
| `sent_by_user_id` | uuid | yes | FK→users.id | | Who sent it; null once that account is removed. |
| `sent_at` | timestamptz | yes | | | Set when sent; must be null while scheduled or cancelled. |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `status` | `announcement_status` | no | | `'sent'` | `scheduled` \| `sent` \| `cancelled`. |
| `scheduled_for` | timestamptz | yes | IX | | When it is due; required while scheduled. |
| `cancelled_at` | timestamptz | yes | | | Set when called off; required while cancelled. |
| `cancelled_by_user_id` | uuid | yes | FK→users.id | | Who called it off. |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `sent_by_user_id`→`users(id)` **ON DELETE SET NULL**; `cancelled_by_user_id`→`users(id)` **ON DELETE SET NULL**
- **CHECK** `ck_announcements_state` — the three-state rule above
- **INDEXES** `ix_announcements_event` (`organization_id`, `event_id`), `ix_announcements_org_sent` (`organization_id`, `sent_at`), partial `ix_announcements_due` (`scheduled_for`) WHERE `status = 'scheduled'` — the scheduler's poll
- **RLS** tenant isolation on `organization_id`
- `status` defaults to `'sent'` because the immediate send is the ordinary case; a scheduled broadcast says
  so explicitly.
- **`event_id` carries no foreign key.** It holds an `events.id` and the application joins on it, but
  PostgreSQL is not enforcing it, so a broadcast can name an event that does not exist and nothing cascades
  when one is removed. `erd.md` → *Unenforced references* tabulates this and the four other columns like
  it; adding the constraint is a schema change, not a documentation one.
- No `deleted_at` (cancellation instead, above) and no `version`.

> **Designed, not built.** The catalog specified `audience` (`announcement_audience`: All registrants /
> Checked-in attendees / Waitlist) and `channels` (a `message_channel[]`). Neither column exists and the
> `announcement_audience` enum was never created, so **a broadcast cannot be aimed at a subset of an
> event's audience and cannot choose SMS** — every announcement is email to everyone. `title` was built as
> `subject`, `recipients` as `recipient_count`, and `created_by` as `sent_by_user_id`; those are renames
> and are corrected above.

#### `message_deliveries`
One row per message handed to a provider, by a template send or a broadcast. Append-only.

**`sent_at` is separate from `created_at`** because the two can differ: the first is when the message was
handed over, the second when the row reached the database, and a worker replaying a backlog writes them
apart. `scan_attempts.scanned_at` splits from its `created_at` for the same reason.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | Tenant. |
| `event_id` | uuid | yes | | | The event the message was about, where there is one. **No FK constraint** — see below. |
| `kind` | text | no | | | Which message this was, by catalog slug (e.g. `registration-confirmation`). |
| `channel` | `message_channel` | no | | `'email'` | `email` \| `sms`. |
| `recipient_email` | text | no | | | Where it went. |
| `recipient_name` | text | yes | | | Addressed name, when known. |
| `status` | `delivery_status` | no | IX | | `sent` \| `failed`. |
| `error` | text | yes | | | The provider's reason, when `failed`. |
| `sent_at` | timestamptz | no | IX | | When it was handed to the provider. |
| `created_at` | timestamptz | no | | `now()` | When the row was written. |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEY** `organization_id`→`organizations(id)` **ON DELETE CASCADE**
- **INDEXES** `ix_message_deliveries_org_sent` (`organization_id`, `sent_at`), `ix_message_deliveries_org_status` (`organization_id`, `status`)
- **RLS** tenant isolation on `organization_id`
- `event_id` carries **no foreign key**, like `announcements.event_id` — see `erd.md` →
  *Unenforced references*.
- No `updated_at`, `deleted_at` or `version`: a delivery is a fact about one moment, written once.

> **Designed, not built, and this is the largest gap in the catalog.** Five columns were specified and none
> exists: `recipient_attendee_id`, `order_id`, `announcement_id` and `message_template_id` — the four
> source links — plus `provider_message_id`.
>
> **A delivery therefore cannot be traced to its cause.** There is no join from a send back to the
> broadcast or template that produced it, the order it confirmed, or the attendee CRM row it went to; only
> `kind` (a slug) and a bare `recipient_email`. The `CHECK` that "exactly one of announcement/template is
> set" cannot exist, because neither column does. Without `provider_message_id` there is also nothing to
> reconcile a send against the provider's own record, so bounces and complaints cannot be matched back —
> which is why `delivery_status` is only `sent`/`failed`: the designed `delivered` and `opened` need
> callbacks this table has no key to receive. `type` was built as `kind`, and `updated_at` was dropped;
> those are corrected above.

#### `notification_reads`
**How far one member has read their notification feed** (US-MSG-03). A watermark, not a row per
notification: everything at or before `read_at` is read, and the whole of "mark all read" is moving this
timestamp forward.

**One row per (organization, member).** A colleague clearing their own feed must not clear anybody else's,
and a member who belongs to two workspaces reads each of them separately — which is why the unique key is
the pair and not `user_id` alone.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, UK | | Tenant. |
| `user_id` | uuid | no | FK→users.id, UK | | The member. |
| `read_at` | timestamptz | no | | | Everything at or before this instant is read. |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `user_id`→`users(id)` **ON DELETE CASCADE**
- **UNIQUE** `uq_notification_reads_member` (`organization_id`, `user_id`)
- **RLS** tenant isolation on `organization_id`
- An **absent row means nothing has been read**, so a member who has never opened the feed sees all of it
  as unread without needing a row seeded at sign-up.

> **`notifications` — catalogued, and deliberately not built.** This catalog carried a `notifications`
> entry for a long time: one row per inbox item, with `kind` (`notification_kind`), `icon`, `title`, a
> `body` of rich `[{text, bold?}]` segments, and an `unread` flag per row. **No such table exists**, and
> the absence is a decision rather than an oversight.
>
> The two tables built instead are the evidence. `notification_reads` above only makes sense if the feed is
> **derived** — a single watermark has nothing to mark if every item is already a row of its own — and
> `notification_preferences` (under *Organization & Settings*) decides per category what to *send*, not
> what to store. So the inbox is re-read on each request from the domain activity that caused it
> (registrations, payments, payouts as they already stand), nothing writes a notification when those things
> happen, and `notification_kind` survives as the category key on `notification_preferences` rather than as
> a column on a stored notification.
>
> **What that costs, so a reader does not go looking for it:** there is no per-item read state (read is
> all-or-nothing up to an instant), no notification history independent of the records it is derived from,
> and nothing to render an inbox from on the server. If any of those is ever required the stored table
> comes back — and the watermark then has to go, because the two models cannot both own "unread".

#### `event_message_runs`
**Bookkeeping for the messages that go out on a schedule** (US-MSG-01/08) — the post-event thank-you and
the event reminder, sent by jobs in eventa-worker that run every hour.

A job that runs every hour runs again, and this row is what stops the second run emailing everybody twice.
One row per (`event_id`, `kind`) is the claim that a run began; `completed_at` is the separate statement
that every recipient was reached, so **a run that crashed halfway is resumed rather than treated as done**.
Per-recipient repeats inside a resumed run are prevented by the worker's own delivery ledger, not here.

One table for every scheduled kind rather than one per job: each has exactly this shape, so a third
scheduled message should be a new `kind`, not a new table.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id | | Tenant. |
| `event_id` | uuid | no | UK | | The event. **No FK constraint** — see below. |
| `kind` | text | no | UK | | Catalog slug of the message: `post-event-thankyou`, `event-reminder`. |
| `requested_at` | timestamptz | no | | | When the run was claimed. |
| `completed_at` | timestamptz | yes | | | Set once every recipient was reached; null means in progress or abandoned. |
| `created_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEY** `organization_id`→`organizations(id)` **ON DELETE CASCADE**
- **UNIQUE** `uq_event_message_runs` (`event_id`, `kind`) — the claim, and the whole no-double-send guarantee
- **RLS** tenant isolation on `organization_id`
- `event_id` carries **no foreign key** even though `uq_event_message_runs` indexes it — see `erd.md` →
  *Unenforced references*. The unique key deliberately omits `organization_id`: an event belongs to exactly
  one workspace, so the pair is already globally unique, and including the tenant would let the same event
  be claimed twice under two tenant ids.
- **WRITER** eventa-worker only (ADR-14). eventa-api never writes this table.

### Feedback & Surveys

#### `surveys`
A feedback survey attached to an event.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | Tenant. |
| `event_id` | uuid | no | IX | | The event surveyed. **No FK constraint** — see below. |
| `title` | text | no | | | |
| `status` | `survey_status` | no | | `'draft'` | `draft` \| `live` \| `closed`. |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEY** `organization_id`→`organizations(id)` **ON DELETE CASCADE**
- **INDEX** `ix_surveys_org_event` (`organization_id`, `event_id`)
- **RLS** tenant isolation on `organization_id`
- `event_id` carries **no foreign key** — see `erd.md` → *Unenforced references*.
- `responses_count`, `avg`, `nps`, `completion`, `dist` are **derived** aggregates.

> **Designed, not built.** `deleted_at` and `version` do not exist. A survey is therefore hard-deleted or
> not deleted, with no recoverable archive — and deleting one cascades its questions, responses and answers
> away with it, which for collected feedback is a sharper edge than the rest of this schema has anywhere
> else. The absent `version` means two organizers editing a survey at once do not get a `409`.

#### `survey_questions`
A question within a survey.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id | | Tenant. |
| `survey_id` | bigint | no | FK→surveys.id, IX | | |
| `position` | integer | no | IX | | Order within the survey. |
| `type` | `survey_question_type` | no | | | `rating` \| `text` \| `choice` \| `nps`. |
| `prompt` | text | no | | | Question text. |
| `options` | text[] | no | | `'{}'` | The choices, for `choice`; empty for the other three types. |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `survey_id`→`surveys(id)` **ON DELETE CASCADE**
- **INDEX** `ix_survey_questions_survey` (`survey_id`, `position`)
- **RLS** tenant isolation on `organization_id`
- `options` is `NOT NULL DEFAULT '{}'` rather than nullable, so "no choices" is an empty array and every
  read can skip the null case.
- **`nps`** is the 0–10 "how likely are you to recommend…" question NPS is computed from (US-MSG-08/09).
  It carries no `options`, like a rating; its answer goes in `survey_answers.score`, not `rating`.

> **Designed, not built.** `required` does not exist, so **no question can be made mandatory** — a
> survey's completeness is enforced only in the client, where it can be bypassed. There is also **no
> unique constraint on (`survey_id`, `position`)**, which the catalog specified: `ix_survey_questions_survey`
> indexes that pair but does not enforce it, so two questions can share a position and their order is then
> whatever the plan returns. `sort_order` was built as `position`; that is a rename and is corrected above.

#### `survey_responses`
One member's completed submission of a survey.

**One response per (survey, person), by `uq_survey_responses_person`.** A survey is answered once, and the
constraint is what makes every aggregate below it countable.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | Tenant. |
| `survey_id` | bigint | no | FK→surveys.id, UK | | |
| `event_id` | uuid | no | IX | | Denormalized from the survey for per-event reporting. **No FK constraint.** |
| `user_id` | uuid | no | FK→users.id, UK | | The respondent. |
| `submitted_at` | timestamptz | no | | | |
| `created_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `survey_id`→`surveys(id)` **ON DELETE CASCADE**; `user_id`→`users(id)` **ON DELETE CASCADE**
- **UNIQUE** `uq_survey_responses_person` (`survey_id`, `user_id`)
- **INDEX** `ix_survey_responses_org_event` (`organization_id`, `event_id`)
- **RLS** tenant isolation on `organization_id`
- `event_id` carries **no foreign key**, and nothing keeps it agreeing with `surveys.event_id` either —
  see `erd.md` → *Unenforced references*.

> **Remodelled, not renamed.** The catalog gave this table a nullable `attendee_id → attendees` and a
> `respondent_name`, for a survey that could be answered anonymously or by someone who is not a user.
> Neither column exists; the parent changed to a `NOT NULL` `user_id → users`. **So a response is never
> anonymous and never from a non-user** — answering requires a portal account, and the "optionally
> anonymous" survey the catalog described is not what was built. That is a different model rather than a
> gap to fill: with `user_id` `NOT NULL`, `uq_survey_responses_person` can exist, which is what stops one
> person answering twice.

#### `survey_answers`
One answer to one question within a response. Normalizes the prototype's flattened rating/text.

**One answer per question per response**, by `uq_survey_answers_question` (US-MSG-08). Every figure built
on this table counts *answers*, not people, so thirty scores of 10 inside a single response would be thirty
promoters in the NPS; the `(survey_id, user_id)` unique one level up cannot stop that, because the
duplicates all sit inside the one response it permits.

**Four value columns, one per question type, exactly one filled.** `score` is deliberately **not** `rating`:
every read of `rating` — the average, the star breakdown, the rating lifted into the responses list, the
rating filter — treats "rating is not null" as "a star rating" without joining to the question's type, so a
0 or a 9 stored there would drag the average and invent stars the 1–5 scale does not have. A column each
keeps every figure reading only what it measures.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id | | Tenant. |
| `response_id` | bigint | no | FK→survey_responses.id, UK, IX | | |
| `question_id` | bigint | no | FK→survey_questions.id, UK | | |
| `rating` | integer | yes | | | 1–5, for a `rating` question. |
| `answer_text` | text | yes | | | Free text, for a `text` question. |
| `choice` | text | yes | | | The selected option, for a `choice` question. |
| `created_at` | timestamptz | no | | `now()` | |
| `score` | smallint | yes | | | 0–10, for an `nps` question. |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `response_id`→`survey_responses(id)` **ON DELETE CASCADE**; `question_id`→`survey_questions(id)` **ON DELETE CASCADE**
- **UNIQUE** `uq_survey_answers_question` (`response_id`, `question_id`)
- **CHECK** `ck_survey_answers_score` — `score BETWEEN 0 AND 10`
- **INDEX** `ix_survey_answers_response` (`response_id`)
- **RLS** tenant isolation on `organization_id`
- The `CHECK` is on the value and names no question type, because the type it would name (`nps`) was added
  to its enum in the same migration transaction and PostgreSQL refuses to *use* a new enum value there.
- `text` was built as `answer_text`; that is a rename and is corrected above.

> **Designed, not built.** There is **no `CHECK` that `rating` falls between 1 and 5**, which the catalog
> specified — only `score` is range-checked. A 0 or a 7 in `rating` would be accepted by the database and
> would silently skew the average, so the 1–5 scale is an application rule, not a stored guarantee. The
> catalog also named an index on `question_id`; only `response_id` is indexed, so "every answer to this
> question across responses" is a scan. `question_id` is `ON DELETE CASCADE`, not the `RESTRICT` the
> catalog specified — deleting a question discards the answers given to it.

### Meetings

#### `meetings`
A coordination meeting an organizer books from the console (E12) — a venue walkthrough, a sponsor call, a
speaker briefing. Optionally tied to an event.

**Times are Bangkok wall clock, not instants.** `meeting_date` + `start_time` are what the organizer typed
and what the venue expects, so there is no `timestamptz` here; the UTC instant is derived when one is
needed. This mirrors `sessions`.

**The calendar is not in the write path.** Saving a meeting never waits on the external calendar: the row
commits with `sync_status = 'pending'` and an outbox event in the *same* transaction, and the worker does
the invite, the conference link and the reminder afterwards. That is what makes US-MTG-04's "the meeting is
still kept" true by construction rather than by a catch block — a calendar outage cannot lose a booking,
because the booking was never conditional on it.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | uuid | no | PK | `gen_random_uuid()` | |
| `organization_id` | bigint | no | FK→organizations.id, UK, IX | | Tenant. |
| `title` | text | no | | | |
| `meeting_date` | date | no | IX | | Bangkok calendar day. |
| `start_time` | time | no | IX | | Bangkok wall clock. |
| `end_time` | time | no | | | `> start_time`, by CHECK. |
| `type` | `meeting_type` | no | | | Venue/Sponsor/Vendor/Speaker/Internal. |
| `mode` | `meeting_mode` | no | | | Video/In person/Phone. |
| `status` | `meeting_status` | no | | `'scheduled'` | `scheduled` \| `cancelled`. |
| `person` | text | no | | | Counterparty name. |
| `role` | text | yes | | | Counterparty role label. |
| `guest_email` | citext | no | | | Where the invite goes; `citext` so a differently-cased address is the same guest. |
| `event_id` | uuid | yes | FK→events.id, IX | | Null means the meeting is general rather than about one event, which the story allows explicitly. |
| `link` | text | yes | | | The conference link, once the calendar sync has produced one (US-MTG-04). |
| `location` | text | yes | | | The venue for In person, "Phone call" for Phone, null for Video. |
| `notes` | text | yes | | | Organizer's own notes. |
| `cancellation_reason` | text | yes | | | Why it was called off; shown to the guest (US-MTG-06). |
| `cancelled_at` | timestamptz | yes | | | Set when called off. |
| `sync_status` | `meeting_sync_status` | no | IX | `'pending'` | `pending` \| `synced` \| `failed` \| `not_connected`. |
| `external_event_id` | text | yes | | | The calendar's own id, held so a reschedule **updates** the guest's existing invite instead of sending a second one (US-MTG-05). |
| `sync_error` | text | yes | | | The calendar's reason, when `sync_status = failed`. |
| `idempotency_key` | text | yes | UK | | The organizer's key for this booking. |
| `created_by` | uuid | yes | FK→users.id | | |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `deleted_at` | timestamptz | yes | | | Soft delete. |
| `version` | integer | no | | `1` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `event_id`→`events(id)` **ON DELETE SET NULL**; `created_by`→`users(id)` **ON DELETE SET NULL**
- **UNIQUE** `uq_meetings_org_idem` (`organization_id`, `idempotency_key`) — a double-submitted booking resolves to the one meeting it meant to create rather than booking the guest twice (US-MTG-03)
- **CHECK** `ck_meetings_times` — `end_time > start_time`
- **INDEXES** `ix_meetings_date` (`organization_id`, `meeting_date`, `start_time`) backing every list, its tabs, ordering and date filters; `ix_meetings_event` (`event_id`); `ix_meetings_sync` (`organization_id`, `sync_status`) for the retry sweep
- **RLS** tenant isolation on `organization_id`
- `event_id` is `SET NULL` rather than cascade: a meeting held *about* an event is still the record of a
  conversation that happened, and it outlives the event row.
- **`cancelled_at` and `deleted_at` are both present and mean different things** — called off (the guest is
  told, the reason is kept) versus removed from the organizer's list.

> **Designed, not built.** There is no `bucket` column. The catalog listed a stored
> `meeting_bucket` (`today`/`upcoming`/`past`) marked "derived", and a stored one is wrong from the moment
> the Bangkok day turns — it would need a nightly job to stay true. It is computed per request instead,
> the same call this schema makes for invoice ageing and session colour, and the enum was never created.
> The `status` column that occupies that name is a different thing: whether the meeting is going ahead.

### System & Audit

#### `audit_events`
Append-only, immutable security/finance audit trail. Never hard- or soft-deleted.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | |
| `type` | `audit_type` | no | IX | | signin/newdev/pwd/twofa/perm/xport/fail/apikey/revoke. |
| `title` | text | no | | | Human summary. |
| `meta` | text | yes | | | Context (device, IP, target). |
| `actor_user_id` | uuid | yes | FK→users.id, IX | | Null for anonymous/failed sign-ins. |
| `ip_address` | inet | yes | | | |
| `occurred_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE RESTRICT**; `actor_user_id`→`users(id)` **ON DELETE SET NULL**
- **INDEXES** `ix_audit_events_org_time` (`organization_id`, `occurred_at`), `ix_audit_events_type` (`type`), `ix_audit_events_actor` (`actor_user_id`)

#### `outbox_events`
Transactional outbox: a domain event written in the same DB transaction as its aggregate change; an outbox relay publishes it to RabbitMQ. High-volume, append-mostly Platform infrastructure.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id | | Tenant. |
| `aggregate_type` | text | no | | | Aggregate root kind (e.g. `order`, `payment`). |
| `aggregate_id` | text | no | IX | | Id of the changed aggregate. |
| `routing_key` | text | no | | | AMQP routing key (e.g. `order.confirmed`). |
| `payload` | jsonb | no | | | Serialized event body. |
| `created_at` | timestamptz | no | | `now()` | Written in the aggregate's transaction. |
| `published_at` | timestamptz | yes | IX | | Null until the relay publishes. |
| `attempts` | integer | no | | `0` | Publish attempts (retry counter). |
| `available_at` | timestamptz | no | | `now()` | Next eligible publish time (backoff). |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEY** `organization_id`→`organizations(id)` **ON DELETE CASCADE**
- **INDEXES** partial `ix_outbox_unpublished` (`published_at`) WHERE `published_at IS NULL` (relay poll), `ix_outbox_aggregate` (`aggregate_type`, `aggregate_id`)

#### `seat_holds`
Short-lived checkout reservation that expires and releases inventory if the buyer doesn't pay in time. Platform infrastructure.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id | | Tenant. |
| `event_id` | uuid | no | FK→events.id | | |
| `order_id` | uuid | yes | FK→orders.id, IX | | Pending order this hold backs; null while cart-only. |
| `ticket_type_id` | uuid | yes | FK→ticket_types.id | | For GA quantity holds. |
| `seat_id` | bigint | yes | FK→seats.id, UK | | For reserved-seat holds; one row per seat. |
| `quantity` | integer | no | | `1` | Seats/units held. |
| `status` | `hold_status` | no | | `'active'` | active/converted/expired/released. |
| `expires_at` | timestamptz | no | IX | | Hold TTL. |
| `created_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `event_id`→`events(id)` **ON DELETE CASCADE**; `order_id`→`orders(id)` **ON DELETE SET NULL**; `ticket_type_id`→`ticket_types(id)` **ON DELETE CASCADE**; `seat_id`→`seats(id)` **ON DELETE CASCADE**
- **UNIQUE** partial `uq_seat_hold_active` (`seat_id`) WHERE `status = 'active'` — a seat has at most one active hold
- **CHECK** `quantity >= 1` — a hold always reserves something, including the reserved-seat case where it is the single seat in `seat_id`
- **INDEXES** partial `ix_seat_holds_expiry` (`expires_at`) WHERE `status = 'active'` (expiry sweeper), `ix_seat_holds_order` (`order_id`)
- **A lapsed hold stops reserving the moment `expires_at` passes** — availability
  filters on it — so inventory is never held by a row this table has not yet
  retired. Retiring it is housekeeping, not correctness: it keeps the table from
  growing without bound and keeps "count of live holds" honest.
- **WRITERS** eventa-api (create, convert, release) and **eventa-worker**'s
  order-expiry sweep (`status = 'expired'`, alongside its order — ADR-14).

#### `webhook_events`
Inbound provider webhook log used for signature-verified, idempotent (exactly-once) processing. High-volume, append-mostly Platform infrastructure.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `provider` | text | no | UK | | `stripe` \| `promptpay` \| `email` \| `sms`. |
| `provider_event_id` | text | no | UK | | Provider's event id (dedup key). |
| `event_type` | text | no | | | Provider event type. |
| `payload` | jsonb | no | | | Raw signature-verified body. |
| `received_at` | timestamptz | no | | `now()` | |
| `processed_at` | timestamptz | yes | IX | | Null until handled. |
| `status` | `webhook_status` | no | | `'received'` | received/processed/failed. |
| `organization_id` | bigint | yes | FK→organizations.id | | Resolved tenant if known. |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEY** `organization_id`→`organizations(id)` **ON DELETE SET NULL**
- **UNIQUE** (`provider`, `provider_event_id`) — the dedup guarantee
- **INDEX** `ix_webhook_events_processed` (`processed_at`) — the whole column, not partial on `processed_at IS NULL`: a btree indexes nulls, so the "not yet handled" scan uses it as it stands

---

## Relationship summary

Complete inventory of every relationship. Type is read parent→child. "Via" names
the FK column or junction table.

**`(no FK)` marks a real relationship PostgreSQL is not enforcing** — a column
holding another table's `id` that the application joins on, with no `REFERENCES`
behind it, so an orphan is silent and nothing cascades or nulls when the parent
goes away. The line is listed because the relationship is real in the domain; the
marker is there because the constraint is missing. `erd.md` →
[Unenforced references](erd.md#unenforced-references) tabulates all of them.

| Parent | Child | Type | Via FK / junction | Notes |
|---|---|---|---|---|
| organizations | users | 1:N | users.organization_id | Tenant root. |
| organizations | memberships | 1:N | memberships.organization_id | |
| organizations | roles | 1:N | roles.organization_id | |
| organizations | api_keys | 1:N | api_keys.organization_id | |
| organizations | payment_settings | 1:1 | payment_settings.organization_id | Optional singleton per workspace. |
| organizations | payment_method_settings | 1:N | payment_method_settings.organization_id | Per checkout method. |
| organizations | categories | 1:N | categories.organization_id | |
| organizations | events | 1:N | events.organization_id | |
| organizations | payouts | 1:N | payouts.organization_id | |
| organizations | tax_periods | 1:N | tax_periods.organization_id | |
| organizations | audit_events | 1:N | audit_events.organization_id | RESTRICT. |
| organizations | *(all tenant tables)* | 1:N | `organization_id` | Every tenant-owned table. |
| users | memberships | 1:N | memberships.user_id | A user may belong to many orgs. |
| roles | memberships | 1:N | memberships.role_id | Role assigned to members. |
| roles | role_permissions | 1:N | role_permissions.role_id | |
| permissions | role_permissions | 1:N | role_permissions.permission_key | 15 fixed permissions. |
| roles ⇄ permissions | role_permissions | M:N | junction role_permissions | Preset matrix `ROLE_PERMS`. |
| users ⇄ organizations | memberships | M:N | junction memberships | User↔org with role. |
| users | auth_sessions | 1:N | auth_sessions.user_id | Devices/sessions. |
| users | social_identities | 1:N | social_identities.user_id | Linked OAuth/OIDC providers. |
| users | two_factors | 1:1 | two_factors.user_id (UK) | 0..1 enrollment. |
| two_factors | recovery_codes | 1:N | recovery_codes.two_factor_id | One-time codes. |
| users | notification_reads | 1:N | notification_reads.user_id | Feed read watermark, one row per (org, member); CASCADE. |
| users | notification_preferences | 1:N | notification_preferences.user_id | Per category. |
| users | audit_events | 1:N | audit_events.actor_user_id | Actor; nullable. |
| users | api_keys | 1:N | api_keys.created_by | Issuer. |
| attendees | users | 1:1 | users.attendee_id **(no FK)** | Portal persona link (optional); can dangle. |
| categories | events | 1:N | events.category_id | SET NULL. |
| landing_templates | events | 1:N | events.landing_template_id | Global seed; SET NULL. |
| organizations | events | 1:N | events.organization_id | |
| events | ticket_types | 1:N | ticket_types.event_id | CASCADE. |
| events | discount_codes | 1:N | discount_codes.event_id | Nullable for org-wide. |
| events | orders | 1:N | orders.event_id | RESTRICT. |
| events | speakers | 1:N | speakers.event_id | CASCADE. |
| events | sessions | 1:N | sessions.event_id | Agenda; CASCADE. |
| events | event_highlights | 1:N | event_highlights.event_id | Public-page bullets; CASCADE. |
| events | event_faqs | 1:N | event_faqs.event_id | Public-page Q&A; CASCADE. |
| events | surveys | 1:N | surveys.event_id **(no FK)** | Application-only link. |
| events | announcements | 1:N | announcements.event_id **(no FK)** | A broadcast can outlive or precede its event. |
| events | meetings | 1:N | meetings.event_id | Optional; SET NULL. |
| events | seat_maps | 1:1 | seat_maps.event_id (UK) | Reserved seating only. |
| events | tickets | 1:N | tickets.event_id | Denormalized for scans. |
| events | payments | 1:N | payments.event_id | Denormalized for reporting. |
| events | invoices | 1:N | invoices.event_id | |
| events | payouts | 1:N | payouts.event_id | Optional attribution. |
| events | check_ins | 1:N | check_ins.event_id | |
| events | scan_attempts | 1:N | scan_attempts.event_id | The event the station was bound to; RESTRICT, so door history outlives the event. |
| events | scan_attempts | 1:N | scan_attempts.ticket_event_id | Where the presented pass actually belonged, on a `wrong_event` refusal; SET NULL. |
| tickets | scan_attempts | 1:N | scan_attempts.ticket_id | Every scan a pass was part of; null for an unrecognised code; SET NULL. |
| users | scan_attempts | 1:N | scan_attempts.scanned_by | Staff who scanned; SET NULL. |
| users | saved_events | 1:N | saved_events.user_id | Cross-tenant personal bookmarks; CASCADE. |
| events | saved_events | 1:N | saved_events.event_id | Bookmark targets; CASCADE. |
| events | event_invitations | 1:N | event_invitations.event_id | One invite per recipient per event; CASCADE. |
| users | event_invitations | 1:N | event_invitations.invited_by | The organizer who sent it; SET NULL. |
| sessions ⇄ speakers | session_speakers | M:N | junction session_speakers | A speaker owns many sessions. |
| ticket_types | order_items | 1:N | order_items.ticket_type_id | RESTRICT. |
| ticket_types | tickets | 1:N | tickets.ticket_type_id | RESTRICT. |
| ticket_types | seats | 1:N | seats.ticket_type_id | Tier per seat; SET NULL. |
| discount_codes | orders | 1:N | orders.discount_code_id | Applied code; SET NULL. |
| discount_codes ⇄ orders | discount_redemptions | M:N | junction discount_redemptions | Idempotent per order. |
| attendees | orders | 1:N | orders.attendee_id | Null for guests; SET NULL. |
| users | orders | 1:N | orders.decided_by | Who approved or rejected the registration (US-REG-02); SET NULL. |
| users | orders | 1:N | orders.offered_by | Who offered the waitlist seat; null when the queue offered it itself (US-REG-04); SET NULL. |
| attendees | tickets | 1:N | tickets.attendee_id | Named holder. |
| users | survey_responses | 1:N | survey_responses.user_id | The respondent; one response per (survey, member); CASCADE. |
| orders | order_items | 1:N | order_items.order_id | CASCADE. |
| orders | tickets | 1:N | tickets.order_id | CASCADE. |
| orders | payments | 1:N | payments.order_id | RESTRICT. |
| orders | refunds | 1:N | refunds.order_id | RESTRICT. |
| orders | invoices | 1:N | invoices.order_id | RESTRICT. |
| users | invoices | 1:N | invoices.voided_by | The Admin who voided it (US-FIN-10); SET NULL. |
| orders | discount_redemptions | 1:N | discount_redemptions.order_id | |
| order_items | tickets | 1:N | tickets.order_item_id | One ticket per admitted seat. |
| seat_maps | seats | 1:N | seats.seat_map_id | CASCADE. |
| seats ⇄ tickets | seat_assignments | M:N | junction seat_assignments | One live assignment per seat/ticket. |
| tickets | seat_assignments | 1:1 | seat_assignments.ticket_id (UK) | Reserved-seat tickets. |
| tickets | check_ins | 1:1 | check_ins.ticket_id (UK) | At most one admission per pass — the door's concurrency guarantee; CASCADE. |
| payments | refunds | 1:N | refunds.payment_id | RESTRICT. |
| payments ⇄ payouts | payout_items | M:N | junction payout_items | Payment settles in one payout. |
| payouts | payout_items | 1:N | payout_items.payout_id | CASCADE. |
| refunds | *(finance ledger)* | — | refunds.issued_by→users | Requires `finRefund`. |
| surveys | survey_questions | 1:N | survey_questions.survey_id | CASCADE. |
| surveys | survey_responses | 1:N | survey_responses.survey_id | CASCADE. |
| survey_responses | survey_answers | 1:N | survey_answers.response_id | CASCADE. |
| survey_questions | survey_answers | 1:N | survey_answers.question_id | CASCADE — deleting a question discards its answers. |
| events | message_deliveries | 1:N | message_deliveries.event_id **(no FK)** | The send log's only link to the domain. The designed links back to the template, announcement, order and attendee were never built. |
| users | check_ins | 1:N | check_ins.checked_in_by | Staff who admitted them; SET NULL. |
| attendees | check_ins | 1:N | check_ins.attendee_id | The CRM row admitted; SET NULL. |
| meetings | users | N:1 | meetings.created_by | Organizer. |
| organizations | payment_credentials | 1:N | payment_credentials.organization_id | One key pair per mode; CASCADE. |
| users | announcements | 1:N | announcements.sent_by_user_id | Who sent the broadcast; SET NULL. |
| users | announcements | 1:N | announcements.cancelled_by_user_id | Who called it off; SET NULL. |
| organizations | event_message_runs | 1:N | event_message_runs.organization_id | Scheduled-send bookkeeping; CASCADE. |
| events | event_message_runs | 1:N | event_message_runs.event_id **(no FK)** | The run claim, unique per (event, kind). |
| events | survey_responses | 1:N | survey_responses.event_id **(no FK)** | Denormalized from the survey for reporting. |
| organizations | outbox_events | 1:N | outbox_events.organization_id | Transactional outbox; CASCADE. |
| organizations | seat_holds | 1:N | seat_holds.organization_id | CASCADE. |
| events | seat_holds | 1:N | seat_holds.event_id | CASCADE. |
| orders | seat_holds | 1:N | seat_holds.order_id | Null while cart-only; SET NULL. |
| ticket_types | seat_holds | 1:N | seat_holds.ticket_type_id | GA quantity holds; CASCADE. |
| seats | seat_holds | 1:N | seat_holds.seat_id | Reserved-seat holds; CASCADE. |
| organizations | webhook_events | 1:N | webhook_events.organization_id | Resolved tenant; SET NULL. |
