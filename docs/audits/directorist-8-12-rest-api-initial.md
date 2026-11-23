# Directorist 8.12 – staging REST API audit (initial)

**Environment:** `staging.dawsomegifts.co.uk`  
**Installed version:** 8.12 (confirmed by the WordPress Plugins screen)  
**Evidence:** the user supplied the live response from `/wp-json/directorist/v1` on 23 November 2026.

This is an audit of the routes exposed by the installed site's **v1 namespace**, not a complete audit of all WordPress REST routes or PHP extension hooks.

## Confirmed in the supplied v1 route index

| Area | Exposed operations | Notes |
|---|---|---|
| Directories | Collection GET/POST; item GET/POST/PUT/PATCH/DELETE | Supports name, slug, image URL, icon, default flag, new/edit listing status and expiry days. |
| Listings | Collection GET/POST; item GET; separate update-status POST | Listing create schema includes directory, author, categories, tags, locations, images, address, coordinates and contact fields. No item PUT/PATCH/DELETE route is shown in this v1 index. |
| Categories | Collection GET/POST; item GET/POST/PUT/PATCH/DELETE | Supports hierarchy, icon/image and directory associations. |
| Tags | Collection GET/POST; item GET/POST/PUT/PATCH/DELETE | Standard taxonomy management. |
| Locations | Collection GET/POST; item GET/POST/PUT/PATCH/DELETE | Supports hierarchy, image and directory associations. This is a taxonomy/location endpoint, not proof of a reusable venue entity with its own address/contact record. |
| Users | Collection GET/POST; item GET/POST/PUT/PATCH/DELETE | User creation schema includes email and password. Do not expose this to public clients or build a second authentication system around it. Prefer WordPress user/account flows and server-side permission checks. |
| Favorites | Add/remove listing favorite | User-specific operations. |
| Orders | Collection GET/POST; item GET | Exposes Directorist pricing-plan order fields and payment status. This is not evidence of a full class/session booking or WooCommerce order workflow. |
| Plans | Collection GET; item GET | Read-only plan endpoints in this namespace. |
| Directory Builder | GET builder tab configuration | Useful for reading per-directory builder configuration; no write endpoint shown here. |
| Media | POST temporary media upload | Requires field and directory slug. Must be tested with authenticated permissions before use. |
| Pages / MCP | GET pages, MCP status, config snippets and guide | Confirms MCP-related routes are exposed in this namespace. |

## Important implementation implications

1. **Directories are a good fit for the six sectors.** Each directory can have its own submission and edit moderation defaults. Keep directory creation in a controlled admin-only setup step.
2. **Listings need a further API check.** The v1 index shows listing creation at the collection route but does not show an item update/delete route. Do not assume the v1 endpoint is sufficient for leader CRUD.
3. **Check the v2 namespace next.** WordPress.org's Directorist changelog records v2 listing create/edit/delete REST improvements introduced in 8.5. The supplied v1 index does not establish whether those v2 routes are active on this staging site.
4. **Do not treat a Directorist location term as a Bubba Hub venue record.** The v1 routes only establish hierarchical location taxonomy. The separate venue model and its relationship to listings still need code/schema verification.
5. **Directorist orders are not class bookings.** Their exposed fields concern plans, listings and payment status. Bubba Hub's weekly sessions, term passes, child attendees, capacity and attendance still need a dedicated booking model unless a tested plugin supplies them.
6. **Security is a hard requirement.** Never trust client-supplied author/user IDs or listing status. Enforce current-user ownership and capabilities server-side; preserve Directorist moderation rules. The 8.11 release notes document security work around ownership, attachments, receipts, orders and approval bypasses.

## Next staging checks

- Inspect `/wp-json/directorist/v2` for listing create/edit/delete routes.
- Inspect the full `/wp-json/` index for additional namespaces and installed-plugin APIs.
- Test read-only directory, category and location requests while logged out and logged in.
- Verify which operations require authentication, what roles/capabilities are checked, and whether a leader can only access their own listings.
- Inspect the installed plugin's actual data model for venues, custom listing fields and directory associations before designing integrations.

## Reference documentation

- [Directorist API reference](https://directorist.com/docs/api-reference/)
- [Directorist action hooks](https://directorist.com/docs/action-hooks/)
- [Directorist Directory Builder overview](https://directorist.com/docs/overview/)
- [WordPress.org Directorist changelog](https://wordpress.org/plugins/directorist/)
