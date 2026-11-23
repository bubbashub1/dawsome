# Directorist 8.12 staging API audit

**Environment:** `https://staging.dawsomegifts.co.uk`  
**Installed plugin:** Directorist – Business Directory Solution  
**Installed version:** 8.12 (WordPress Plugins screen)  
**API inspected:** `/wp-json/directorist/v1`  
**Evidence:** Route and argument schema copied from the staging REST index by the site owner on 2026-11-23.

## Executive assessment

Directorist 8.12 exposes useful native REST resources for directories, listings, categories, tags, locations, users, favourites, plans and Directorist orders. This is a strong foundation for Bubba Hub's six directory sectors.

The route index describes available routes and request arguments; it does **not** prove that a given request is publicly accessible, which roles can use it, whether ownership is enforced, or what data a successful request returns. Those require authenticated, role-based tests and inspection of the installed plugin's permission callbacks.

## Confirmed route inventory

| Resource | Route | Methods shown | Notes |
|---|---|---|---|
| API index | `/directorist/v1` | GET | Namespace index |
| Listings collection | `/listings` | GET, POST | GET filters include directory, author, status, category, tag and location |
| Listing item | `/listings/{id}` | GET | No item PUT/PATCH/DELETE shown in this route index |
| Listing status | `/listings/{id}/update-status` | POST | Status values shown: publish/private |
| Listing actions | `/listings/{listing_id}/actions` | POST | report/contact |
| Reviews | `/listings/reviews`, `/listings/reviews/{id}` | GET | Read routes shown |
| Categories | `/listings/categories`, `/listings/categories/{id}` | GET, POST, PUT, PATCH, DELETE | Directory filtering supported |
| Tags | `/listings/tags`, `/listings/tags/{id}` | GET, POST, PUT, PATCH, DELETE | General listing tags |
| Locations | `/listings/locations`, `/listings/locations/{id}` | GET, POST, PUT, PATCH, DELETE | Hierarchical taxonomy-style fields; directory filtering supported |
| Users | `/users`, `/users/{id}` | GET, POST, PUT, PATCH, DELETE | User fields and role filters exposed |
| Favourites | `/users/{user_id}/favorites` and item route | POST, DELETE | Listing save/favourite support |
| Directories | `/directories`, `/directories/{id}` | GET, POST, PUT, PATCH, DELETE | Directory settings include submission/edit status and expiry |
| Orders | `/orders`, `/orders/{id}` | GET, POST; item GET | Directorist package/order model |
| Plans | `/plans`, `/plans/{id}` | GET | Listing plans |
| Builder | `/builder/{builder_tab}` | GET | Listing/search/single-listing builder data |
| Pages | `/pages` | GET | Page data |
| Temporary media | `/temp-media-upload` | POST | Requires field and directory |
| MCP info | `/mcp/status`, `/mcp/config-snippets`, `/mcp/guide` | GET | Informational MCP routes |

## Key integration findings

### 1. Six directory sectors

The `directories` resource supports multiple directory records. Listing queries accept a `directory` filter, and category/location creation and queries can also be scoped to directory IDs.

This appears to support the agreed six sectors:
1. Listings
2. Events
3. Businesses
4. Childcare & Schools
5. Places & Days Out
6. School Clubs

Before configuration, retrieve the actual directory records and IDs from staging. Do not assume IDs or create duplicate directory types.

### 2. Listing data

The listing create schema includes:
- Name, slug, description and short description
- Address, postcode/zip, latitude and longitude
- Phone, email, website and social links
- Price and pricing type
- Directory ID
- Categories, tags and locations
- Images
- Author ID
- Status and expiry fields

This is enough to establish a basic listing integration contract. Bubba Hub-specific fields (for example age suitability, term dates, session schedule, booking options and accessibility detail) need a field-builder/custom-field audit before deciding whether to use Directorist custom fields or a separate Bubba Hub data model.

**Important API gap to verify:** the route index shows GET only for `/listings/{id}`; it does not show PUT/PATCH/DELETE for individual listings. Collection POST creates listings and a separate status endpoint changes publish/private status. Do not assume editing/deletion is supported through this REST namespace. Inspect installed source and test supported WordPress/Directorist frontend or admin flows before implementation.

### 3. Locations are not yet proven to be shared venues

The `locations` schema has taxonomy-like fields: name, slug, parent, description, image and directory. It does not expose a dedicated venue record with a full address, postcode, coordinates, contact details or reusable venue relationship.

The listing itself has address, postcode, latitude and longitude fields. Therefore, do not treat Directorist Locations as a confirmed shared venue database. We need to inspect the installed data model and determine whether a separate venue entity/plugin is needed, or whether Bubba Hub should own a shared venue record referenced by listings and bookings.

### 4. Accounts and ownership

The API exposes WordPress user records and listing `author` filters/fields. This supports the intended single WordPress account per person.

However, the route schema alone does not prove:
- A leader can only edit their own listings.
- A family cannot access another family's details.
- A user can safely change a listing author.
- Custom Bubba Hub roles are accepted by the user endpoint.
- Whether the endpoint exposes private user data to unauthorised requests.

The user collection's role enum lists standard roles and WooCommerce roles, but not arbitrary custom roles. This is a schema limitation to investigate, not proof that custom roles cannot exist in WordPress.

Do not expose the Directorist user-creation endpoint to public forms or client-side code. Account creation and role assignment must use trusted server-side WordPress APIs and explicit permission checks.

### 5. Directorist orders are not class bookings

The `orders` schema contains customer, plan, directory, listing, amount, currency, payment status and gateway fields. It is clearly oriented around Directorist listing plans/packages.

It does not describe class session allocation, term passes, child attendees, per-session capacity, attendance, or family booking records. Use it for its intended listing/package purpose only. Do not use it as the Bubba Hub class-booking ledger.

### 6. Schema quality observations

The copied route schema contains apparent type/documentation typos:
- Listing query `directory` is typed as `integar`.
- Listing `never_expired` is typed as `boolen`.

Treat these as documentation/schema defects until the installed implementation is checked. Validate request payloads against actual responses and source code rather than blindly generating clients from this schema.

## Required next tests (read-only first)

1. Fetch the actual directory records and record the six directory IDs.
2. Fetch representative listing records using `per_page=1` and `context=view`; inspect the returned field names and custom fields.
3. Fetch category, tag and location records; determine their actual hierarchy and directory associations.
4. Inspect one listing in `context=edit` while authenticated as an administrator, then compare with a leader account.
5. Test listing collection and item routes as logged-out visitor, family account, leader account and administrator. Record status codes and redacted responses.
6. Inspect installed source for permission callbacks and listing update/delete routes. Never bypass permissions or write directly to Directorist internal tables.
7. Inspect how Directorist stores listing author/owner and whether the installed plugin has a supported edit flow.
8. Inspect the location model and decide the shared venue strategy.

## Implementation guardrails

- Use Directorist IDs as references; do not duplicate its listing/category/directory records in Bubba Hub tables.
- Keep class bookings, session occurrences, family/child attendee data and planner links in a separate Bubba Hub-owned model.
- Use documented APIs, WordPress APIs and supported hooks; do not edit Directorist core.
- Enforce authorisation server-side on every read/write.
- Do not make destructive changes or create live records during this audit.
- Test on staging and retain backups before any migration or bulk operation.

## Current conclusion

**Directorist 8.12 is a viable directory foundation, but the API index is not enough to approve implementation yet.** The next decisive checks are actual directory IDs, listing field responses, listing edit permissions, and whether a proper shared venue entity exists.
