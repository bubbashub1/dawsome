# Directorist 8.12 – v2 REST API audit

**Environment:** `https://staging.dawsomegifts.co.uk`  
**Installed version:** 8.12  
**Evidence:** route index supplied by the site administrator on 23 November 2026.

## Major finding: native listing CRUD is exposed

The installed site's `directorist/v2` namespace exposes:

| Route | Methods | What it gives Bubba Hub |
|---|---|---|
| `/listings` | GET, POST | Search/filter listings and create new listings |
| `/listings/{id}` | GET, POST, PUT, PATCH | Read and update an individual listing |
| `/listings/timezones` | GET | Timezone options for listing hours |
| `/orders` | GET, POST | User-facing order retrieval/creation |
| `/orders/{id}` | GET | Retrieve an order |
| `/orders/{id}/cancel` | POST | Cancel an order |
| `/checkout` | GET, POST | Payment and featured-listing checkout |
| `/checkout/retry-payment` | POST | Retry an order payment |
| `/admin/orders` | GET, POST | Admin order listing and status creation/update |
| `/admin/orders/{id}` | GET, POST, DELETE | Admin order retrieval/update/deletion |
| `/admin/orders/{id}/status` | POST | Change order status |
| `/admin/orders/{order_id}/refunds` | GET, POST | Admin refund list/create |
| `/admin/orders/{order_id}/refunds/{id}` | GET, POST, PUT, PATCH, DELETE | Admin refund management |
| `/admin/payments` | GET, POST | Admin payment lookup/creation |
| `/admin/payments/{id}` | GET, POST, DELETE | Admin payment retrieval/update/deletion |

## Listing create/update data exposed

The `POST /listings` and item update schemas expose:

- Directory ID, slug, expiry, featured flag and plan.
- Listing title, tagline, description and excerpt.
- Categories, tags and locations.
- Social links.
- Price type, price and price range.
- Address, postcode/ZIP, latitude and longitude.
- Phone, secondary phone, fax, email and website.
- Video URL and listing images.
- Contact form visibility and map visibility.
- Opening-hours-related fields, including always-open and timezone.
- Dynamic custom fields via `[field_key]`.
- Listing type.

The collection GET route can filter by directory, author, status, category, tag, location, price, featured state and search term. Pagination supports up to 100 records per page.

## What this means for the Bubba Hub build

**Use Directorist as the listing engine.** The Leader Hub can provide a simpler, branded interface that calls these endpoints rather than duplicating the underlying listing storage and CRUD logic.

Recommended division:

- Directorist: directories, listings, taxonomies, listing fields, listing ownership data, basic orders/plans and checkout.
- Bubba Hub integration layer: leader-friendly workflows, ownership/capability checks, shared venue and schedule relationships, family planner, class/session bookings, child attendee data, term passes and family-facing planning.
- WordPress: shared user accounts, authentication, roles and permissions.

## Critical caveats – not yet proven by the route index

1. **Authorization:** route registration does not prove that leaders can safely edit only their own listings. Test permissions using separate administrator, leader and family accounts. Enforce ownership server-side; never rely on hiding edit buttons in the browser.
2. **Listing status/moderation:** the v2 create/update schema does not show a direct status field. The v1 namespace exposes a separate status operation. Determine the intended approval workflow and whether Directorist's own submission rules apply to API writes.
3. **Custom fields:** the schema accepts dynamic `[field_key]` fields, but each directory's actual configured field keys, validation and required-field behaviour still need testing.
4. **Directory-specific fields:** verify whether the Directory Builder configuration changes the API's accepted/returned fields for each of the six directories.
5. **Delete/trash:** no DELETE method is shown for `/listings/{id}`. Do not assume the v2 item route supports permanent deletion or trashing.
6. **Bookings:** the v2 orders endpoints concern Directorist listing plans, checkout and payments. They do not establish recurring class schedules, session capacity, child attendees, term bookings or attendance.
7. **Venue records:** address and coordinates are listing fields, but this is not proof of a reusable venue entity that multiple activities can share.

## Recommended next tests

1. Test logged-out and logged-in GET requests to `/directorist/v2/listings?per_page=5`.
2. Test a leader account's GET request with `author={their_user_id}`, then test whether the API enforces ownership rather than merely filtering.
3. On a disposable test listing, test GET with `context=edit` and a safe PATCH to a non-critical field; verify the change in WordPress admin.
4. Test whether a leader can update another user's listing by ID. This must be denied.
5. Create one unpublished test listing through the API and verify its resulting post status and moderation behaviour.
6. Inspect each directory's Directory Builder fields and map their field keys to the Bubba Hub forms.

**Do not run destructive tests or change live listings.** Use staging and disposable records only.

## Decision

The v2 API is sufficient to justify building the Leader Hub as a custom front-end over Directorist listing CRUD. Keep the distinctive Bubba Hub features as integrations, rather than rebuilding directory functionality.
