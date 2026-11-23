# Bubba Hub hybrid architecture blueprint

**Status:** Initial agreed direction — planning and verification required before implementation.

## 1. Product architecture

Bubba Hub will use a hybrid, plugin-first WordPress architecture:

- WordPress provides the shared platform foundation.
- Directorist provides public directories and listing/venue records.
- Established plugins provide standard capabilities where they meet the requirements.
- Custom Bubba Hub plugins provide specialist functionality and integrations that are missing or unsuitable in existing plugins.

The objective is to reduce custom security-sensitive code, avoid duplicated data entry, minimise plugin costs and make family and leader workflows straightforward.

## 2. Responsibility boundaries

| System | Owns | Does not own |
|---|---|---|
| WordPress | User accounts, authentication, roles/capabilities, pages, media and core site infrastructure | Bubba Hub-specific booking rules |
| Directorist | Public directory listings, categories, locations/venues and directory discovery | Booking transactions and family calendar records |
| WooCommerce (proposed) | Checkout, orders, payment gateway integration and supported refund/order workflows | Class schedules, attendance allocation and family planner logic |
| Bubba Hub Core (proposed) | Shared integration contracts, family/child relationships and common services where required | A second user authentication system or duplicate directory |
| Booking solution (to select/build) | Classes, recurring sessions, booking options, capacity, attendees and attendance | Public directory source records |
| Family Planner (to select/build) | Family calendar entries, personal commitments and links to saved/ booked records | Duplicate copies of listing or booking master data |
| Leader tools (to select/build) | Leader-facing workflows for their own listings, classes and bookings | Broad WordPress administrator access |

These are responsibility boundaries, not a commitment to create one plugin per row. Closely related custom modules may be packaged together where that is simpler to maintain.

## 3. Directorist integration contract

Directorist is the source of truth for public directory listings and venues.

Other plugins should link to Directorist records using stable identifiers, for example:

- `directorist_listing_id`
- `directorist_venue_id` (only after verifying the actual venue/location data model)
- `leader_user_id` (WordPress user ID)

Before implementation, inspect the exact Directorist version installed on staging and verify:

1. Listing post type, metadata and ownership model.
2. Venue/location storage and whether venues are separate records or listing fields.
3. Public and authenticated REST API endpoints and permissions.
4. Supported actions, filters and extension APIs.
5. Listing creation/editing workflows and ownership checks.
6. Compatibility requirements and supported versions.

Use Directorist's documented APIs, hooks and public functions. Do not write directly to its internal tables or rely on undocumented internals unless a deliberate, version-pinned adapter is reviewed and tested. Never edit Directorist core files.

## 4. Shared account and access model

- WordPress is the only identity provider and account store.
- Families and leaders use one WordPress account.
- Children are family profiles, not independent login accounts by default.
- Roles/capabilities grant broad feature access; record-level ownership checks restrict access to the correct leader, family and booking.
- A leader must only be able to view or change their own permitted listings, classes, sessions, attendees and bookings.
- A family member must only access the household/family records for which they have been authorised.
- Administrative access is separate from leader access.

All sensitive REST, AJAX, form and admin actions must perform server-side authentication, capability checks, ownership checks, nonce/CSRF protection where applicable, input validation, sanitisation and safe output escaping.

## 5. Booking requirements

The booking solution must be assessed against the following requirements before selection or custom implementation:

- Weekly recurring classes and individual dated sessions.
- Parent can book an individual session, several sessions, or a complete term.
- Leader can enable individual bookings, term bookings, or both per class.
- Example pricing: £8 per individual session or £42 for a six-week term.
- Multiple children in one family booking, with child-specific attendee details.
- Capacity management at the appropriate session/term level.
- Multiple independent leaders with restricted access to their own records.
- Confirmation, cancellation, refund and attendance workflows.
- Online payment and reliable payment status reconciliation.
- Booking references to the Directorist listing, venue and WordPress leader/family accounts.
- Confirmed sessions can be added to the Family Planner without duplicating master data.

A recurring event calendar is not sufficient on its own: the system must support purchasing and allocating a multi-session term pass.

## 6. Family Planner requirements

The intended planner is calendar-first and shared within a family/household.

It should support:

- Shared calendar access for invited parents/carers.
- Child profiles associated with the family.
- Personal appointments, birthdays and family commitments.
- Saved directory activities and events.
- Confirmed bookings and their individual session dates.
- Links back to the original listing, venue and booking.
- Updated display details when a linked source record changes, where appropriate.

A calendar plugin may be used if its sharing, permissions, API and data-linking capabilities pass review. Otherwise, a focused custom planner can be built.

## 7. Plugin selection policy

Before adopting an existing plugin, verify:

1. Compatibility with the installed WordPress and Directorist versions.
2. Whether the required feature is genuinely available in the free edition.
3. Whether the same WordPress accounts and permissions can be used.
4. Whether records can be linked through documented APIs/hooks.
5. Security, maintenance activity, support and upgrade path.
6. Total cost, including required extensions, gateway charges and transaction fees.
7. Data export and migration options.

Do not install multiple overlapping plugins to solve the same problem. Keep temporary evaluation plugins isolated to staging and remove them after data/export checks if they are not selected.

## 8. Proposed delivery sequence

1. **Foundation audit:** confirm WordPress, Directorist version, staging configuration, account flows, roles and security.
2. **Directorist audit:** document listing/venue data model, API, hooks and supported extension points.
3. **Plugin compatibility matrix:** compare booking, calendar, forms, payments and security plugins against real requirements and free/pro limits.
4. **Integration contract:** define IDs, relationships, permissions and data ownership before coding.
5. **Booking implementation:** prove recurring sessions, individual/term purchase, multiple children, capacity and leader isolation.
6. **Family Planner:** connect saved activities and bookings to shared family calendars.
7. **Leader workflows:** provide restricted, simple management screens.
8. **End-to-end testing:** test registration, login recovery, permissions, checkout, payment callbacks, cancellations, refunds and data integrity.
9. **Controlled release:** staging acceptance, backups and a documented production deployment/rollback.

## 9. Decisions still open

- Final booking plugin versus custom Bubba Hub Bookings implementation.
- Whether WooCommerce is the chosen payment/order engine.
- Final Family Planner implementation.
- Whether custom Core, Bookings, Planner and Leader functionality should be separate plugins or a smaller set of modules.
- Exact Directorist venue/location model and integration API, pending code and staging-version audit.

## 10. Non-goals and safeguards

- Do not rebuild WordPress authentication.
- Do not create a second user database.
- Do not replace Directorist's public directory responsibilities.
- Do not assume two plugins are integrated merely because both run on WordPress.
- Do not mark a booking paid based only on a browser redirect; reconcile verified server-side payment notifications.
- Do not allow client-submitted prices, capacities or ownership IDs to be trusted.
- Do not develop directly on the live site.
- Do not treat this blueprint as approval to install, purchase or deploy a plugin.
