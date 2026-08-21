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
> explicit foreign keys with chosen `ON DELETE` behavior, and resolves every
> many-to-many with an associative (junction) table. All monetary values are
> stored as **integer minor units (satang)**, never floating point.
>
> **Schema status.** The 53 tables in this catalog are the **target physical
> model**, not a claim that every table has already shipped in API migrations.
> Treat the API's committed `pgTable` definitions as the implementation inventory
> and count them independently when reporting delivery progress.
>
> **Faithfulness.** Field names, enum values and monetary rules are taken
> verbatim from the SRS data model and the TypeScript source of truth under
> `src/features/*/data/*.ts`, `src/lib/eventCatalog.ts` and `src/lib/format.ts`.
> Display-only enum tokens are stored as their lowercase wire values; Title-cased
> variants are UI labels and are not persisted.

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
- **Multi-tenancy.** 46 of the 53 physical tables carry
  `organization_id bigint REFERENCES organizations(id)` (NOT NULL except the
  inbound `webhook_events` log while its tenant is unresolved). Every tenant
  read/write is filtered by the caller's organization (row-level isolation).
  The seven tables without that column are `organizations`, the global lookups
  `landing_templates` and `permissions`, the transitively-scoped children
  `role_permissions`, `recovery_codes`, and `session_speakers`, and the
  cross-tenant, user-scoped `saved_events` bookmark table.
- **Audit columns.** Mutable domain tables normally carry `created_at timestamptz
  NOT NULL DEFAULT now()` (immutable) and `updated_at timestamptz NOT NULL DEFAULT
  now()` (touched on every mutation, via trigger). Aggregate roots and user-editable entities also
  carry `deleted_at timestamptz NULL` for **soft delete**; append-only ledgers
  (`payments`, `refunds`, `audit_events`, `check_ins`, `message_deliveries`) are
  **never** soft-deleted and omit the column. `created_by bigint NULL REFERENCES
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
| `seating_mode` | `reserved`, `ga` | `events.seating_mode`, `seat_maps` | `portal/events.ts`, `landing/types.ts` |
| `ticket_status` | `onsale`, `scheduled`, `paused`, `soldout` | `ticket_types.status` | `ticketing/types.ts`, `tickets.ts` |
| `admission_type` | `general_admission`, `reserved_seat` | `ticket_types.admission_type` | derived from `seating_mode` |
| `discount_type` | `percent`, `fixed` | `discount_codes.type` | `ticketing/types.ts` |
| `discount_status` | `active`, `scheduled`, `expired`, `disabled` | `discount_codes.status` | `ticketing/types.ts`, `discounts.ts`, `reportsDiscounts.ts` |
| `order_status` | `confirmed`, `pending`, `waitlisted`, `cancelled`, `rejected`, `expired` | `orders.status` | SRS `RegistrationStatus` (`registrations.ts`). `rejected` = an organizer turned it down (US-REG-02); `expired` = nobody paid before the seat hold lapsed (US-DISC-05). Both are deliberately distinct from `cancelled`, which is a withdrawal or a refund — a decision somebody made, not the absence of one. |
| `registration_status` | *(alias — realized as `order_status`)* | — | `registrations.ts` (naming note) |
| `issued_ticket_status` | `issued`, `checked_in`, `void`, `refunded`, `transferred` | `tickets.status` | derived from `RegStatus`/`ScanState` |
| `payment_status` | `paid`, `pending`, `refunded`, `failed` | `payments.status`, `orders.payment_status` | `payments.ts` |
| `payment_method` | `Card`, `PromptPay`, `Bank transfer`, `Apple Pay`, `Google Pay` | `payments.method`, `invoices.paid_via` | `payments.ts` |
| `payment_provider` | `stripe` | `payment_settings.provider` | `payment-settings.ts` |
| `payment_mode` | `test`, `live` | `payment_settings.mode` | `payment-settings.ts` |
| `payment_connection_status` | `disconnected`, `connected` | `payment_settings.status` | `payment-settings.ts` |
| `refund_status` | `pending`, `succeeded`, `failed` | `refunds.status` | derived from `TransactionStatus` (`reportsTransactions.ts`) |
| `payout_status` | `paid`, `processing`, `scheduled`, `failed` | `payouts.status` | `payouts.ts` (reports labels: Paid / In transit / Pending) |
| `invoice_status` | `paid`, `issued`, `overdue`, `void` | `invoices.status` | `invoices.ts` (paid/void terminal; issued/overdue derived) |
| `tax_status` | `filed`, `due`, `upcoming` | `tax_periods.status` | `taxes.ts` |
| `transaction_type` | `payment`, `refund` | insights union (view) | `reportsTransactions.ts` |
| `transaction_status` | `succeeded`, `refunded`, `failed` | insights union (view) | `reportsTransactions.ts` |
| `session_type` | `Keynote`, `Talk`, `Workshop`, `Panel`, `Break` | `sessions.type` | `agenda.ts`, `eventDetail.ts` |
| `session_color` | `green`, `amber`, `rose` | `sessions.color` | `agenda.ts` |
| `speaker_tone` | `green`, `blue`, `purple`, `amber`, `red`, `pink` | `speakers.tone` | `speakers.ts` |
| `meeting_type` | `Venue`, `Sponsor`, `Vendor`, `Speaker`, `Internal` | `meetings.type` | `meetings.ts` |
| `meeting_mode` | `Video`, `In person`, `Phone` | `meetings.mode` | `meetings.ts` |
| `meeting_bucket` | `today`, `upcoming`, `past` | `meetings.bucket` (derived) | `meetings.ts` |
| `template_id` | `aurora`, `noir`, `minimal`, `atlas` | `landing_templates.id`, `events.landing_template_id` | `landingTemplates.ts` |
| `message_channel` | `email`, `sms` | `message_templates.channels`, `announcements.channels`, `message_deliveries.channel` | `messagingTemplates.ts`, `announcements.ts`, `deliveryLog.ts` |
| `announcement_status` | `sent`, `scheduled` | `announcements.status` | `announcements.ts` |
| `announcement_audience` | `All registrants`, `Checked-in attendees`, `Waitlist` | `announcements.audience` | `announcements.ts` (`ANNOUNCEMENT_AUDIENCES`) |
| `delivery_status` | `delivered`, `opened`, `sent`, `failed` | `message_deliveries.status` | `deliveryLog.ts` |
| `notification_kind` | `registration`, `payment`, `sales`, `feedback`, `payout`, `alert`, `task`, `reminder`, `marketing` | `notifications.kind` | `notifications.ts` |
| `survey_status` | `live`, `closed`, `draft` | `surveys.status` | `feedback.ts` (`FeedbackStatus`) |
| `question_type` | `Rating`, `Text`, `Multiple choice` | `survey_questions.type` | `feedback.ts` |
| `member_role` | `Admin`, `Organizer`, `Staff`, `Attendee` | *(retired from `roles.name`/`memberships.role` — both are text since US-SET-13)* | `roles.ts` (`RoleName`) |
| `member_status` | `Active`, `Invited`, `Suspended` | `memberships.status`, `users.status` | `users.ts` (`UserStatus`) |
| `user_persona` | `admin`, `attendee` | `users.persona` | SRS §1.23 (personas never share a login) |
| `permission_key` | `evCreate`, `evPublish`, `evSpeakers`, `regView`, `regCheckin`, `regExport`, `finView`, `finRefund`, `finDiscount`, `setUsers`, `setSettings`, `setIntegrations` | `permissions.key`, `role_permissions.permission_key` | `roles.ts` (`PermKey`, 12 values) |
| `permission_group` | `Events`, `Registrations`, `Finance`, `Settings` | `permissions.group` | `roles.ts` (`PermGroup`) |
| `audit_type` | `signin`, `newdev`, `pwd`, `twofa`, `perm`, `xport`, `fail`, `apikey`, `revoke` | `audit_events.type` | `security.ts` (`AuditType`) |
| `attendee_tag` | `VIP`, `Speaker`, `Sponsor`, `Student` | `attendees.tag` (nullable) | `attendees.ts` (`TAG_BADGE`) |
| `scan_state` | `ok`, `dupe`, `invalid`, `wrong`, `void` | `check_ins.state` | `checkin-tool/checkin.ts` (`ScanState`) |
| `seat_status` | `available`, `held`, `reserved`, `sold`, `blocked` | `seats.status` | production (reserved-seating model) |
| `category_color` | `pink`, `blue`, `amber`, `brand`, `violet`, `indigo`, `teal`, `red` | `categories.color` | `categories.ts` |
| `locale` | `en`, `th` | `organizations.locale`, `users.locale` | `format.ts` / SRS §5.5 |
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

---

## Entities

Notation for the **Key** column: `PK` primary key · `FK→table.col` foreign key ·
`UK` part of a unique constraint · `IX` indexed. **Null** = *is the column
nullable?* (`no` = NOT NULL, `yes` = nullable). Every table also carries the
common audit columns from *Conventions* (`created_at`, `updated_at`, `version`,
and `deleted_at`/`created_by` where noted); they are shown per table for
completeness.

### Identity & Access

#### `organizations`
Tenant root / workspace. Not itself tenant-scoped; parent of everything else.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `name` | text | no | | | Workspace/company name. |
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
- **UNIQUE** (`slug`)
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
| `attendee_id` | bigint | yes | FK→attendees.id | | 1:1 link for portal persona. |
| `last_active_at` | timestamptz | yes | | | |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `deleted_at` | timestamptz | yes | | | |
| `version` | integer | no | | `1` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `attendee_id`→`attendees(id)` **ON DELETE SET NULL**
- **UNIQUE** (`organization_id`, `email`, `persona`) — same email may exist once per persona per org
- **INDEXES** `ix_users_org` (`organization_id`), `ix_users_email` (`email`)

#### `roles`
Named preset of the 12 permissions, per organization.

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
Global fixed lookup of the 12 permission keys and their group/label. Not tenant-scoped.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `key` | `permission_key` | no | PK | | e.g. `finRefund`. |
| `group` | `permission_group` | no | IX | | Events/Registrations/Finance/Settings. |
| `label` | text | no | | | e.g. "Issue refunds". |

- **PRIMARY KEY** (`key`)
- **INDEX** `ix_permissions_group` (`group`)
- Seeded with exactly 12 rows; no `organization_id`.

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
| `role_id` | bigint | no | FK→roles.id, IX | | Preset of 12 permissions. |
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
| `organization_id` | bigint | no | FK→organizations.id, IX | | Tenant. |
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

**PCI SAQ-A:** no card data and **no provider secret key** is stored. The connection is by reference only
— the provider's account id plus its publishable key, both non-secret. Charges are made with the platform
secret from config acting on behalf of `account_id`, so there is nothing sensitive to show back or mask.

`account_id` is a **reference, not a credential**: on its own it authorises nothing. `Payments` reads this
row through `MerchantAccountPort` and `Payouts` through `PayoutAccountPort`; neither reaches into the
table, and the two ports stay distinct because "can this workspace take money" and "can this workspace be
paid" are different answers at the provider (`charges_enabled` against `payouts_enabled`).

A workspace that is not set up to take money **cannot take paid registrations** — checkout refuses rather
than collecting somewhere the organizer cannot reach, which would issue a valid ticket against money they
can never claim. Free events are unaffected.

> **In flight.** Per-workspace API keys (`payment_credentials`, below) are replacing the shared-platform-key
> model this section was written for. `account_id` and the ports survive the change; the sentence about
> whose key signs the request does not. This note goes when the migration is finished.

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

- **PRIMARY KEY** (`id`)
- **FOREIGN KEY** `organization_id`→`organizations(id)` **ON DELETE CASCADE**
- **UNIQUE** `uq_payment_settings_org` (`organization_id`)
- **RLS** tenant isolation on `organization_id`

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
- **INDEXES** `ix_events_org_status` (`organization_id`, `status`), `ix_events_start_at` (`start_at`), `ix_events_type` (`type`), `ix_events_category` (`category_id`)
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

- **FOREIGN KEYS** both **ON DELETE CASCADE** · **INDEX** `ix_event_highlights_event` (`event_id`,`position`)
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

- **FOREIGN KEYS** both **ON DELETE CASCADE** · **INDEX** `ix_event_faqs_event` (`event_id`,`position`)
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

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `event_id`→`events(id)` **ON DELETE CASCADE**
- **INDEXES** `ix_speakers_event` (`event_id`), `ix_speakers_org` (`organization_id`)
- `sessions_count` is **derived** from `session_speakers`.

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
| `color` | `session_color` | yes | | | green/amber/rose. |
| `sort_order` | integer | no | | `0` | Ordering within day. |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `deleted_at` | timestamptz | yes | | | |
| `version` | integer | no | | `1` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `event_id`→`events(id)` **ON DELETE CASCADE**
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
- **UNIQUE** (`organization_id`, `event_id`, `code`) — code unique per event (case-insensitive via uppercase storage)
- **CHECK** `used >= 0`, `redemption_limit >= 0`, `per_person_limit >= 0`, `min_order_satang >= 0`; and the value matches its type — `percent` between 1 and 100, `fixed` at least 1 satang
- **INDEXES** `ix_discount_codes_event` (`event_id`), `ix_discount_codes_org` (`organization_id`)

#### `discount_redemptions` — JUNCTION (discount_codes ⇄ orders)
Records each application of a code to an order; enforces idempotent, once-per-order redemption and drives the `used` counter.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | |
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

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `event_id`→`events(id)` **ON DELETE RESTRICT**; `attendee_id`→`attendees(id)` **ON DELETE SET NULL**; `discount_code_id`→`discount_codes(id)` **ON DELETE SET NULL**; `created_by`→`users(id)` **ON DELETE SET NULL**
- **UNIQUE** (`organization_id`, `reference`)
- **CHECK** `seats BETWEEN 1 AND 8`, all money `>= 0`
- **INDEXES** `ix_orders_event` (`event_id`), `ix_orders_attendee` (`attendee_id`), `ix_orders_status` (`organization_id`, `status`), `ix_orders_discount` (`discount_code_id`)
- `checked_in` is derived from child `tickets`.
- **WRITERS — eventa-api owns this table, with one agreed exception.** The
  order-expiry sweep in **eventa-worker** (`modules/order-expiry`, ADR-14) sets
  `status = 'expired'` on `pending`/`pending` orders whose seat holds lapsed, and
  retires those holds in the same transaction. It is the only clock-driven write
  to an API aggregate in the platform, and the only place outside eventa-api that
  writes `orders`. **Consequence:** the worker's `orders` and `seat_holds` schema
  files are *write* mirrors, not read views — an enum value added here must be
  added there in the same change, because a write fails on a value a read would
  simply never have produced.

#### `order_items`
Line item: a quantity of one ticket type within an order.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | |
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
- **INDEXES** `ix_order_items_order` (`order_id`), `ix_order_items_ticket_type` (`ticket_type_id`)

#### `tickets`
One issued admission ticket per admitted person/seat, carrying a signed QR token. Check-in is per-ticket.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | uuid | no | PK | `gen_random_uuid()` | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | |
| `order_id` | uuid | no | FK→orders.id, IX | | |
| `order_item_id` | bigint | no | FK→order_items.id, IX | | Which line issued it. |
| `event_id` | uuid | no | FK→events.id, IX | | Denormalized for check-in scans. |
| `ticket_type_id` | uuid | no | FK→ticket_types.id, IX | | |
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
| `organization_id` | bigint | no | FK→organizations.id, IX | | |
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
| `organization_id` | bigint | no | FK→organizations.id, IX | | |
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
Append-only log of every door scan (successful or not). One row per scan attempt; carries the `ScanState` outcome.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | |
| `event_id` | uuid | no | FK→events.id, IX | | |
| `ticket_id` | uuid | yes | FK→tickets.id, IX | | Null for `invalid` scans (no match). |
| `scanned_qr` | text | yes | | | Raw scanned token/ref (`invalid` case). |
| `state` | `scan_state` | no | | | ok/dupe/invalid/wrong/void. |
| `scanned_by` | uuid | yes | FK→users.id | | Staff (needs `regCheckin`). |
| `other_event_id` | uuid | yes | FK→events.id | | For `wrong`-event scans. |
| `device_label` | text | yes | | | Scanner device. |
| `scanned_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `event_id`→`events(id)` **ON DELETE RESTRICT**; `ticket_id`→`tickets(id)` **ON DELETE SET NULL**; `scanned_by`→`users(id)` **ON DELETE SET NULL**; `other_event_id`→`events(id)` **ON DELETE SET NULL**
- **INDEXES** `ix_check_ins_event` (`event_id`), `ix_check_ins_ticket` (`ticket_id`), `ix_check_ins_scanned_at` (`event_id`, `scanned_at`)
- The authoritative first successful (`ok`) scan sets `tickets.status = checked_in` and `tickets.checked_in_at`.

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

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `order_id`→`orders(id)` **ON DELETE RESTRICT**; `event_id`→`events(id)` **ON DELETE RESTRICT**
- **UNIQUE** (`organization_id`, `txn`), (`organization_id`, `idempotency_key`)
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

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `payment_id`→`payments(id)` **ON DELETE RESTRICT**; `order_id`→`orders(id)` **ON DELETE RESTRICT**; `issued_by`→`users(id)` **ON DELETE RESTRICT**
- **UNIQUE** (`organization_id`, `idempotency_key`)
- **INDEXES** `ix_refunds_payment` (`payment_id`), `ix_refunds_order` (`order_id`)

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

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `order_id`→`orders(id)` **ON DELETE RESTRICT**; `event_id`→`events(id)` **ON DELETE RESTRICT**
- **UNIQUE** (`organization_id`, `number`)
- **INDEXES** `ix_invoices_order` (`order_id`), `ix_invoices_event` (`event_id`), `ix_invoices_status` (`organization_id`, `status`)
- `overdue` is computed (`status = issued AND due_at < today`), never stored free-hand.

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

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `event_id`→`events(id)` **ON DELETE SET NULL**
- **UNIQUE** (`organization_id`, `reference`)
- **INDEXES** `ix_payouts_status` (`organization_id`, `status`), `ix_payouts_event` (`event_id`)

#### `payout_items` — JUNCTION (payouts ⇄ payments)
Resolves the M:N between a payout and the payments it settles (net of refunds/fees).

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | |
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
A one-off broadcast to an event audience.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | uuid | no | PK | `gen_random_uuid()` | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | |
| `event_id` | uuid | no | FK→events.id, IX | | |
| `title` | text | no | | | |
| `body` | text | yes | | | SMS branch ≤160/segment. |
| `audience` | `announcement_audience` | no | | | All registrants / Checked-in / Waitlist. |
| `recipients` | integer | no | | `0` | Resolved count at send (derived). |
| `channels` | `channel[]` | no | | | Subset of {email, sms}. |
| `status` | `announcement_status` | no | IX | `'scheduled'` | sent/scheduled. |
| `scheduled_for` | timestamptz | yes | | | Required when scheduled. |
| `sent_at` | timestamptz | yes | | | Set when sent. |
| `created_by` | uuid | yes | FK→users.id | | |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `deleted_at` | timestamptz | yes | | | |
| `version` | integer | no | | `1` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `event_id`→`events(id)` **ON DELETE CASCADE**; `created_by`→`users(id)` **ON DELETE SET NULL**
- **INDEXES** `ix_announcements_event` (`event_id`), `ix_announcements_status` (`organization_id`, `status`)

#### `message_deliveries`
Per-recipient delivery record materialized by a template send or announcement. Append-only; provider webhooks drive `status`.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | uuid | no | PK | `gen_random_uuid()` | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | |
| `recipient_name` | text | no | | | |
| `recipient_email` | citext | yes | | | |
| `recipient_attendee_id` | bigint | yes | FK→attendees.id, IX | | |
| `order_id` | uuid | yes | FK→orders.id, IX | | Source booking (confirmation/receipt). |
| `announcement_id` | uuid | yes | FK→announcements.id, IX | | Source announcement. |
| `message_template_id` | bigint | yes | FK→message_templates.id, IX | | Source template. |
| `type` | text | no | | | Message type label. |
| `channel` | `channel` | no | | | email/sms. |
| `status` | `delivery_status` | no | IX | `'sent'` | delivered/opened/sent/failed. |
| `provider_message_id` | text | yes | | | ESP/SMS gateway id for webhooks. |
| `sent_at` | timestamptz | no | | `now()` | |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `recipient_attendee_id`→`attendees(id)` **ON DELETE SET NULL**; `order_id`→`orders(id)` **ON DELETE SET NULL**; `announcement_id`→`announcements(id)` **ON DELETE SET NULL**; `message_template_id`→`message_templates(id)` **ON DELETE SET NULL**
- **CHECK** exactly one of (`announcement_id`, `message_template_id`) set
- **INDEXES** `ix_deliveries_order` (`order_id`), `ix_deliveries_announcement` (`announcement_id`), `ix_deliveries_template` (`message_template_id`), `ix_deliveries_status` (`organization_id`, `status`)

#### `notifications`
In-app inbox notification for an admin user.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | uuid | no | PK | `gen_random_uuid()` | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | |
| `user_id` | uuid | no | FK→users.id, IX | | Recipient. |
| `kind` | `notification_kind` | no | | | registration/payment/sales/feedback/payout/alert/task. |
| `icon` | text | yes | | | Hugeicons slug. |
| `title` | text | no | | | |
| `body` | jsonb | no | | | Rich segments `[{text, bold?}]`. |
| `unread` | boolean | no | | `true` | |
| `created_at` | timestamptz | no | | `now()` | Drives grouping. |
| `updated_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `user_id`→`users(id)` **ON DELETE CASCADE**
- **INDEXES** `ix_notifications_user` (`user_id`), partial `ix_notifications_unread` (`user_id`) WHERE `unread`

### Feedback & Surveys

#### `surveys`
A feedback survey attached to an event.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | uuid | no | PK | `gen_random_uuid()` | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | |
| `event_id` | uuid | no | FK→events.id, IX | | |
| `title` | text | no | | | |
| `status` | `survey_status` | no | | `'draft'` | live/closed/draft. |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `deleted_at` | timestamptz | yes | | | |
| `version` | integer | no | | `1` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `event_id`→`events(id)` **ON DELETE CASCADE**
- **INDEXES** `ix_surveys_event` (`event_id`), `ix_surveys_org` (`organization_id`)
- `responses_count`, `avg`, `nps`, `completion`, `dist` are **derived** aggregates.

#### `survey_questions`
A question within a survey.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | uuid | no | PK | `gen_random_uuid()` | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | |
| `survey_id` | uuid | no | FK→surveys.id, UK, IX | | |
| `prompt` | text | no | | | Question text. |
| `type` | `question_type` | no | | | Rating/Text/Multiple choice. |
| `sort_order` | integer | no | | `0` | Position within survey. |
| `options` | text[] | yes | | | Required when Multiple choice. |
| `required` | boolean | no | | `false` | |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `survey_id`→`surveys(id)` **ON DELETE CASCADE**
- **UNIQUE** (`survey_id`, `sort_order`)
- **INDEX** `ix_survey_questions_survey` (`survey_id`)

#### `survey_responses`
One completed submission of a survey (may be anonymous).

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | uuid | no | PK | `gen_random_uuid()` | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | |
| `survey_id` | uuid | no | FK→surveys.id, IX | | |
| `attendee_id` | bigint | yes | FK→attendees.id, IX | | Null when anonymous. |
| `respondent_name` | text | yes | | | May be anonymous. |
| `submitted_at` | timestamptz | no | | `now()` | |
| `created_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `survey_id`→`surveys(id)` **ON DELETE CASCADE**; `attendee_id`→`attendees(id)` **ON DELETE SET NULL**
- **INDEXES** `ix_survey_responses_survey` (`survey_id`), `ix_survey_responses_attendee` (`attendee_id`)

#### `survey_answers`
One answer to one question within a response. Normalizes the prototype's flattened rating/text.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | bigint identity | no | PK | | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | |
| `response_id` | uuid | no | FK→survey_responses.id, UK, IX | | |
| `question_id` | uuid | no | FK→survey_questions.id, UK, IX | | |
| `rating` | smallint | yes | | | 1–5; for Rating questions. |
| `text` | text | yes | | | Free-text answer. |
| `choice` | text | yes | | | Selected option for Multiple choice. |
| `created_at` | timestamptz | no | | `now()` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `response_id`→`survey_responses(id)` **ON DELETE CASCADE**; `question_id`→`survey_questions(id)` **ON DELETE RESTRICT**
- **UNIQUE** (`response_id`, `question_id`)
- **CHECK** `rating BETWEEN 1 AND 5` when present
- **INDEXES** `ix_survey_answers_response` (`response_id`), `ix_survey_answers_question` (`question_id`)

### Meetings

#### `meetings`
Operational/coordination meeting, optionally tied to an event.

| Column | Type | Null | Key | Default | Notes |
|---|---|---|---|---|---|
| `id` | uuid | no | PK | `gen_random_uuid()` | |
| `organization_id` | bigint | no | FK→organizations.id, IX | | |
| `title` | text | no | | | |
| `meeting_date` | date | no | | | `RawMeeting.date`. |
| `start_time` | time | no | | | |
| `end_time` | time | no | | | `> start_time`. |
| `type` | `meeting_type` | no | | | Venue/Sponsor/Vendor/Speaker/Internal. |
| `role` | text | yes | | | Counterparty role label. |
| `person` | text | no | | | Counterparty name. |
| `event_id` | uuid | yes | FK→events.id, IX | | Optional; event-agnostic meetings allowed. |
| `mode` | `meeting_mode` | no | | | Video/In person/Phone. |
| `bucket` | `meeting_bucket` | no | | | Derived: today/upcoming/past. |
| `link` | text | yes | | | Required when mode = Video. |
| `location` | text | yes | | | Venue/"Phone call" for non-Video. |
| `created_by` | uuid | yes | FK→users.id | | |
| `created_at` | timestamptz | no | | `now()` | |
| `updated_at` | timestamptz | no | | `now()` | |
| `deleted_at` | timestamptz | yes | | | |
| `version` | integer | no | | `1` | |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEYS** `organization_id`→`organizations(id)` **ON DELETE CASCADE**; `event_id`→`events(id)` **ON DELETE SET NULL**; `created_by`→`users(id)` **ON DELETE SET NULL**
- **INDEXES** `ix_meetings_event` (`event_id`), `ix_meetings_date` (`organization_id`, `meeting_date`)

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
| `organization_id` | bigint | no | FK→organizations.id, IX | | Tenant. |
| `aggregate_type` | text | no | | | Aggregate root kind (e.g. `order`, `payment`). |
| `aggregate_id` | text | no | IX | | Id of the changed aggregate. |
| `routing_key` | text | no | IX | | AMQP routing key (e.g. `order.confirmed`). |
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
| `organization_id` | bigint | no | FK→organizations.id, IX | | Tenant. |
| `event_id` | uuid | no | FK→events.id, IX | | |
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
| `organization_id` | bigint | yes | FK→organizations.id, IX | | Resolved tenant if known. |

- **PRIMARY KEY** (`id`)
- **FOREIGN KEY** `organization_id`→`organizations(id)` **ON DELETE SET NULL**
- **UNIQUE** (`provider`, `provider_event_id`) — the dedup guarantee
- **INDEXES** partial `ix_webhook_unprocessed` (`processed_at`) WHERE `processed_at IS NULL`

---

## Relationship summary

Complete inventory of every relationship (the ERD is generated from this table).
Type is read parent→child. "Via" names the FK column or junction table.

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
| permissions | role_permissions | 1:N | permissions.permission_key | 12 fixed permissions. |
| roles ⇄ permissions | role_permissions | M:N | junction role_permissions | Preset matrix `ROLE_PERMS`. |
| users ⇄ organizations | memberships | M:N | junction memberships | User↔org with role. |
| users | auth_sessions | 1:N | auth_sessions.user_id | Devices/sessions. |
| users | social_identities | 1:N | social_identities.user_id | Linked OAuth/OIDC providers. |
| users | two_factors | 1:1 | two_factors.user_id (UK) | 0..1 enrollment. |
| two_factors | recovery_codes | 1:N | recovery_codes.two_factor_id | One-time codes. |
| users | notifications | 1:N | notifications.user_id | Inbox. |
| users | notification_preferences | 1:N | notification_preferences.user_id | Per category. |
| users | audit_events | 1:N | audit_events.actor_user_id | Actor; nullable. |
| users | api_keys | 1:N | api_keys.created_by | Issuer. |
| attendees | users | 1:1 | users.attendee_id | Portal persona link (optional). |
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
| events | surveys | 1:N | surveys.event_id | CASCADE. |
| events | announcements | 1:N | announcements.event_id | CASCADE. |
| events | meetings | 1:N | meetings.event_id | Optional; SET NULL. |
| events | seat_maps | 1:1 | seat_maps.event_id (UK) | Reserved seating only. |
| events | tickets | 1:N | tickets.event_id | Denormalized for scans. |
| events | payments | 1:N | payments.event_id | Denormalized for reporting. |
| events | invoices | 1:N | invoices.event_id | |
| events | payouts | 1:N | payouts.event_id | Optional attribution. |
| events | check_ins | 1:N | check_ins.event_id | |
| users | saved_events | 1:N | saved_events.user_id | Cross-tenant personal bookmarks; CASCADE. |
| events | saved_events | 1:N | saved_events.event_id | Bookmark targets; CASCADE. |
| sessions ⇄ speakers | session_speakers | M:N | junction session_speakers | A speaker owns many sessions. |
| ticket_types | order_items | 1:N | order_items.ticket_type_id | RESTRICT. |
| ticket_types | tickets | 1:N | tickets.ticket_type_id | RESTRICT. |
| ticket_types | seats | 1:N | seats.ticket_type_id | Tier per seat; SET NULL. |
| discount_codes | orders | 1:N | orders.discount_code_id | Applied code; SET NULL. |
| discount_codes ⇄ orders | discount_redemptions | M:N | junction discount_redemptions | Idempotent per order. |
| attendees | orders | 1:N | orders.attendee_id | Null for guests; SET NULL. |
| attendees | tickets | 1:N | tickets.attendee_id | Named holder. |
| attendees | survey_responses | 1:N | survey_responses.attendee_id | Null when anonymous. |
| attendees | message_deliveries | 1:N | message_deliveries.recipient_attendee_id | |
| orders | order_items | 1:N | order_items.order_id | CASCADE. |
| orders | tickets | 1:N | tickets.order_id | CASCADE. |
| orders | payments | 1:N | payments.order_id | RESTRICT. |
| orders | refunds | 1:N | refunds.order_id | RESTRICT. |
| orders | invoices | 1:N | invoices.order_id | RESTRICT. |
| orders | message_deliveries | 1:N | message_deliveries.order_id | Confirmations/receipts. |
| orders | discount_redemptions | 1:N | discount_redemptions.order_id | |
| order_items | tickets | 1:N | tickets.order_item_id | One ticket per admitted seat. |
| seat_maps | seats | 1:N | seats.seat_map_id | CASCADE. |
| seats ⇄ tickets | seat_assignments | M:N | junction seat_assignments | One live assignment per seat/ticket. |
| tickets | seat_assignments | 1:1 | seat_assignments.ticket_id (UK) | Reserved-seat tickets. |
| tickets | check_ins | 1:N | check_ins.ticket_id | Scan log; SET NULL. |
| payments | refunds | 1:N | refunds.payment_id | RESTRICT. |
| payments ⇄ payouts | payout_items | M:N | junction payout_items | Payment settles in one payout. |
| payouts | payout_items | 1:N | payout_items.payout_id | CASCADE. |
| refunds | *(finance ledger)* | — | refunds.issued_by→users | Requires `finRefund`. |
| surveys | survey_questions | 1:N | survey_questions.survey_id | CASCADE. |
| surveys | survey_responses | 1:N | survey_responses.survey_id | CASCADE. |
| survey_responses | survey_answers | 1:N | survey_answers.response_id | CASCADE. |
| survey_questions | survey_answers | 1:N | survey_answers.question_id | RESTRICT. |
| message_templates | message_deliveries | 1:N | message_deliveries.message_template_id | Materialized sends; SET NULL. |
| announcements | message_deliveries | 1:N | message_deliveries.announcement_id | Materialized sends; SET NULL. |
| users | check_ins | 1:N | check_ins.scanned_by | Staff scanner; SET NULL. |
| events | check_ins | 1:N | check_ins.other_event_id | `wrong`-event scans; SET NULL. |
| meetings | users | N:1 | meetings.created_by | Organizer. |
| organizations | outbox_events | 1:N | outbox_events.organization_id | Transactional outbox; CASCADE. |
| organizations | seat_holds | 1:N | seat_holds.organization_id | CASCADE. |
| events | seat_holds | 1:N | seat_holds.event_id | CASCADE. |
| orders | seat_holds | 1:N | seat_holds.order_id | Null while cart-only; SET NULL. |
| ticket_types | seat_holds | 1:N | seat_holds.ticket_type_id | GA quantity holds; CASCADE. |
| seats | seat_holds | 1:N | seat_holds.seat_id | Reserved-seat holds; CASCADE. |
| organizations | webhook_events | 1:N | webhook_events.organization_id | Resolved tenant; SET NULL. |
