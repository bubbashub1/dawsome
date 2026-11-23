# Directorist v2 public listing response – staging test

**Date:** 23 November 2026  
**Site:** `staging.dawsomegifts.co.uk`  
**Endpoint tested:** `GET /wp-json/directorist/v2/listings?per_page=5`  
**Evidence:** response pasted by the site administrator in the project conversation after being asked to test the endpoint while logged out. Treat the response as the observed public-view result.

## Confirmed response behaviour

The endpoint returned five listing records successfully. All five had:

- A numeric listing ID and slug.
- Permalink and created/modified dates.
- Directory ID `3`.
- Status `publish`.
- Author ID `1`.
- Title, description and tagline.
- Categories, locations and tags.
- Address, latitude/longitude and map visibility.
- Price and price type/range.
- Website, phone and email.
- Listing image URL and video URL.
- Custom fields `custom-date` and `custom-time`.
- Related listing IDs, view count, popularity/new flags and review metadata.

This is a strong basis for rendering directory cards and detail pages through a Bubba Hub front end.

## Privacy and data minimisation

The observed response includes contact details (email and phone), an author user ID, address and precise coordinates. These may be intended as public listing fields, but Bubba Hub must make an explicit field-by-field decision before importing real listings.

Recommended:
- Do not expose private leader account details or internal user metadata in custom endpoints.
- Only publish contact fields the listing owner has explicitly chosen to display.
- Use contact forms or controlled contact actions where appropriate.
- Do not assume that hiding a field in the front-end removes it from the REST response.
- Test the actual public response for each configured directory and field set.

## Data-quality observation

The supplied records contain apparent demo/test content (for example, restaurant/holiday-property titles, generic placeholder descriptions and contact information, plus Melbourne and Dubai locations). Their coordinates are not Devon/Cornwall data. Do not import these records into the real Bubba Hub directory.

## What is not yet proven

- Whether unpublished/pending/private listings are hidden from logged-out requests.
- Whether `status=any` is restricted to privileged users.
- Whether `context=edit` reveals additional private fields or is correctly permission-gated.
- Whether a logged-in leader can see only their own non-public listings.
- Whether API writes enforce ownership, validation and moderation.

## Next safe read-only test

While logged out, request:

`https://staging.dawsomegifts.co.uk/wp-json/directorist/v2/listings?status=any&per_page=10`

Check whether the response contains only published listings or also pending, draft, private, expired or trashed records. Do not attempt any write or status change during this test.
