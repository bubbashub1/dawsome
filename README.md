# Dawsome / Bubba Hub platform

This repository is the working home for the new hybrid WordPress architecture and Bubba Hub extensions.

## Direction

Use WordPress and established plugins for standard platform foundations. Build custom Bubba Hub plugins only for specialist workflows and integrations that existing plugins cannot provide adequately.

### Platform responsibilities

- **WordPress:** authentication, user accounts, roles/capabilities, pages, media and core site infrastructure.
- **Directorist:** public directory listings and venue/location records.
- **WooCommerce (proposed):** payment checkout, orders and gateway integrations, subject to a compatibility and fee review.
- **Established plugins:** use where they meet requirements, integrate cleanly and remain affordable.
- **Bubba Hub extensions:** bookings, family planner, leader workflows and cross-plugin integration where required.

## Architecture principles

1. One WordPress account per person across the platform.
2. Directorist remains the source of truth for public directory listings and venues.
3. Specialist plugins own their own data and reference other records by stable IDs.
4. Prefer documented APIs, WordPress hooks and supported extension points; do not modify vendor plugin core files.
5. Do not duplicate data unnecessarily.
6. Enforce authorization server-side on every sensitive operation.
7. Prefer proven plugins over custom code when compatibility, security, cost and maintainability are acceptable.
8. Develop and test on staging before controlled release to production.
9. No paid extensions or recurring costs without an explicit review and decision.
10. Keep the architecture modular, but avoid splitting every small feature into a separate plugin.

## Initial documentation

- [Hybrid architecture blueprint](docs/architecture/overview.md)

## Status

Planning and compatibility investigation. No production implementation is authorised by this document alone.
