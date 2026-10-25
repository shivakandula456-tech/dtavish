# TAVISH & CO. Repository Audit and Rebuild Record
Date: 2026-10-25

## Scope
Repository reviewed: `shivakandula456-tech/dtavish` (default branch `main`).
The audit covered the storefront entry point, Admin entry and product editor, Admin JavaScript, local storage/data layer, bundled seed catalogue, Supabase catalogue adapter, Supabase schema, Supabase configuration, and README/release notes.

## Backup and branch safety
- Pre-rebuild snapshot branch: `backup/pre-rebuild-2026-10-25`.
- Verified that the README blob on the backup branch matches the README blob on `main` at audit time; the backup branch was created from `main` before implementation.
- Implementation is isolated on `rebuild/catalogue-publish-on-save`; `main` was not modified.

## Locked product decisions
1. A valid product save publishes immediately.
2. One product image is sufficient to publish; the editor and schema support up to 12 images, including 4+.
3. Migration preserves existing catalogue records. Matching remote SKUs are not overwritten by seed/local data during the migration import.
4. No catalogue product is deleted as part of this rebuild. Test products must be identified and reviewed before any deletion.

## Findings
- The repository is a static HTML/CSS/vanilla JavaScript storefront with browser-local storage for local catalogue and customer/admin state.
- A Supabase adapter exists, but `supabase-config.js` has blank URL and publishable key values. Live Supabase state, auth, Storage bucket configuration, remote catalogue contents, and cross-device publication cannot be confirmed from repository source alone.
- The bundled seed catalogue contains 24 products: 22 marked ACTIVE, 1 DRAFT, and 1 OUT_OF_STOCK. No seed product names contain test/demo/sample/dummy/placeholder markers. This is only a seed-data review; it does not establish whether browser-local or remote test products exist.
- Before this rebuild, image count was constrained to four in several places: product editor, client-side validation, status-change guard, and Supabase schema.
- Before this rebuild, the Admin save flow could retain a valid product as DRAFT unless the user selected ACTIVE; it did not consistently implement publish-on-save.
- Before this rebuild, the local-to-Supabase import upserted every local/backup/seed SKU and could overwrite an existing remote record with the same SKU.
- Product images uploaded from the browser are uploaded to Supabase Storage only when Supabase is configured and the Admin is authenticated. Without configuration, changes remain local to that browser.
- The project has no `package.json` or Node-based build/test setup. Automated runtime and end-to-end browser checks were not run in this audit.

## Changes made on the implementation branch
- Product saves now set status to ACTIVE automatically when required validation passes, except when the Admin intentionally selects HIDDEN, TRASH, or OUT_OF_STOCK. Incomplete required fields save as DRAFT with an explanation.
- Product image limits are aligned to 12 across editor controls, client validation, status changes, and the SQL schema. One image is sufficient.
- Supabase migration now fetches existing remote products and skips local/seed records whose normalized SKU already exists remotely, preventing migration-time overwrite of matching remote records.
- README release notes now record the rebuild decisions and verification limits.

## Pre-deletion review checklist
Do not delete any product until all of the following are completed:
- Export a fresh catalogue backup from the actual Admin/browser and, if configured, export the remote Supabase catalogue.
- Compare local, seed, and remote records by SKU and slug.
- Flag candidates using explicit markers (test/demo/sample/dummy/placeholder), obviously synthetic names/prices, duplicate SKUs/slugs, or internal notes.
- Manually confirm each candidate against the owner’s intended live catalogue.
- Record approved deletions separately and only then delete the individually approved records.

## Remaining verification / risks
- Configure a real Supabase project and test the SQL migration against a copy/staging database before applying it to production.
- Confirm Storage bucket policies and authenticated Admin role access.
- Test save-to-publish for a new and existing product with 1, 4, and 5+ images, plus intentionally hidden/out-of-stock products.
- Test migration with remote-only SKUs, local-only SKUs, duplicate SKUs, and existing remote records; verify no remote record is overwritten or deleted.
- Run desktop/mobile browser tests and inspect product image loading, storefront visibility, cart/wishlist behavior, and Admin error states.
- A source-level audit cannot inspect a customer’s browser localStorage or a live Supabase database when credentials are absent.
