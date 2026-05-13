# Cafe Headless WP API Contract Capture - 2026-05

Scope: Bruno regression coverage for the current `/wp-json/cafe/v1` behavior before PHP cleanup/refactor. These notes intentionally lock the mixed response shapes documented in `BACKEND_AUDIT_2026_05.md`; do not normalize expectations until Nuxt/client compatibility is coordinated.

## Bruno Execution Notes

- Collection: `Cafe Headless WP API`
- Base URL variable: `{{baseUrl}}`
- Admin auth variables: use the environment's admin Application Password credentials.
- Dynamic IDs should be captured from create responses and reused in follow-up requests: `groupCategoryId`, `childCategoryId`, `itemId`, `variantId`, `branchId`, and `branchSlug`.
- Negative auth should be run with three environment/auth variants where available: no auth, invalid Application Password, and authenticated user without `manage_options`.

## Public Regression Scenarios

### `GET /health`

- Request: `GET {{baseUrl}}/wp-json/cafe/v1/health`
- Expected success contract: wrapped-ish health response with `status` and `api_version`; `generated_at` is intentionally absent.
- Assert either installed health details such as `tables`, or missing-table `message`, depending on fixture state.
- Assert public access succeeds without auth.

### `GET /menu`

- Request: `GET {{baseUrl}}/wp-json/cafe/v1/menu`
- Expected success contract: wrapped response with `status`, `api_version`, `generated_at`, and `categories`.
- Assert `categories` is an array.
- Assert public access succeeds without auth.

### `GET /menu?branch={slug}`

- Request: `GET {{baseUrl}}/wp-json/cafe/v1/menu?branch={{branchSlug}}`
- Expected success contract: wrapped response with `status`, `api_version`, `generated_at`, and `categories`.
- Assert branch overrides are reflected for the requested branch.
- Missing or inactive branch fixtures should be documented separately because current public behavior depends on stored branch state.

### `GET /tiles`

- Request: `GET {{baseUrl}}/wp-json/cafe/v1/tiles`
- Expected success contract: wrapped response with `status`, `api_version`, `generated_at`, and `groups`.
- Assert `groups` is an array.
- Assert public access succeeds without auth.

### `GET /tiles?branch={slug}`

- Request: `GET {{baseUrl}}/wp-json/cafe/v1/tiles?branch={{branchSlug}}`
- Expected success contract: wrapped response with `status`, `api_version`, `generated_at`, and `groups`.
- Assert branch-filtered tile groups mirror branch menu availability and pricing.

### `GET /branches`

- Request: `GET {{baseUrl}}/wp-json/cafe/v1/branches`
- Expected success contract: wrapped response with `status`, `api_version`, `generated_at`, and `branches`.
- Assert only active branches are returned.
- Assert public access succeeds without auth.

### `GET /branches/{slug}`

- Request: `GET {{baseUrl}}/wp-json/cafe/v1/branches/{{branchSlug}}`
- Expected success contract: wrapped response with `status`, `api_version`, `generated_at`, and `branch`.
- Assert inactive branches are not exposed by public slug lookups.

## Admin Auth Failure Matrix

Run the same auth-negative expectations across every admin route group:

- Categories: `/admin/categories`, `/admin/categories/{id}`
- Branches: `/admin/branches`, `/admin/branches/{id}`
- Branch item overrides: `/admin/branches/{branch_id}/items`, `/admin/branches/{branch_id}/items/{item_id}`
- Items: `/admin/items`, `/admin/items/{id}`
- Variants: `/admin/items/{item_id}/variants`, `/admin/items/{item_id}/variants/{variant_id}`
- Seed: `/seed`

Expected auth-negative cases:

- No auth: request should not succeed; assert WordPress REST error shape with `code`, `message`, and `data.status`.
- Invalid Application Password: request should not succeed; assert WordPress REST error shape with `code`, `message`, and `data.status`.
- Authenticated user without `manage_options`, if available: request should not succeed; assert forbidden status and WordPress REST error shape.

## Admin Categories CRUD

### Create Group Category

- Request: `POST {{baseUrl}}/wp-json/cafe/v1/admin/categories`
- Body: `name`, unique `slug`, `parent_id: null`, `sort_order`.
- Expected success contract: raw category object returned through direct `WP_REST_Response`; HTTP `201`; no `status`, `api_version`, or `generated_at`.
- Capture `id` as `groupCategoryId`.

### Create Child Category

- Request: `POST {{baseUrl}}/wp-json/cafe/v1/admin/categories`
- Body: `name`, unique `slug`, `parent_id: {{groupCategoryId}}`, `sort_order`.
- Expected success contract: raw category object; HTTP `201`; no wrapper metadata.
- Capture `id` as `childCategoryId`.

### List, Read, Update, Delete

- `GET /admin/categories`: raw array; empty list is `[]`; no wrapper metadata.
- `GET /admin/categories/{id}`: raw category object; no wrapper metadata.
- `PUT/PATCH/POST /admin/categories/{id}`: raw category object; no wrapper metadata.
- `DELETE /admin/categories/{id}`: partial status response with `status` and `deleted`; no `api_version` or `generated_at`.
- Regression cases: duplicate slug on create/update, self-parent update rejection, blocked delete while children/items exist, successful delete after dependencies are removed.

## Admin Branches CRUD

- `GET /admin/branches`: currently reuses public list; wrapped response with `status`, `api_version`, `generated_at`, and `branches`; inactive branches are omitted.
- `POST /admin/branches`: raw branch object through direct `WP_REST_Response`; HTTP `201`; no wrapper metadata. Capture `id` as `branchId` and `slug` as `branchSlug`.
- `GET /admin/branches/{id}`: raw branch object; no wrapper metadata.
- `PUT /admin/branches/{id}`: raw branch object; no wrapper metadata.
- Regression cases: required `name` and `slug`, duplicate slug, boolean-ish `is_active`, and sort order coercion.

## Admin Items CRUD

- `GET /admin/items`: wrapped response with `status`, `api_version`, `generated_at`, and `items`.
- `POST /admin/items`: wrapped response with `status`, `api_version`, `generated_at`, and `item`; HTTP `201`. Capture `id` as `itemId`.
- `GET /admin/items/{id}`: raw item object; no wrapper metadata.
- `PUT /admin/items/{id}`: raw item object; no wrapper metadata.
- `DELETE /admin/items/{id}`: partial status response with `status` and `deleted`; no `api_version` or `generated_at`.
- Regression cases: fixed item requires price, variable item may be created for variant flow, blocked delete while variants exist, active item attaches to active branches.

## Admin Variants CRUD

- Parent item must exist and be variable.
- `GET /admin/items/{item_id}/variants`: wrapped response with `status`, `api_version`, `generated_at`, and `variants`.
- `POST /admin/items/{item_id}/variants`: wrapped response with `status`, `api_version`, `generated_at`, and `variant`; HTTP `201`. Capture `id` as `variantId`.
- `PUT /admin/items/{item_id}/variants/{variant_id}`: raw variant object; no wrapper metadata.
- `DELETE /admin/items/{item_id}/variants/{variant_id}`: partial status response with `status` and `deleted`; no `api_version` or `generated_at`.
- Regression cases: reject variants for fixed items, require name and price/price_raw, delete before deleting variable parent item.

## Admin Branch Item Overrides

- `GET /admin/branches/{branch_id}/items`: wrapped response with `status`, `api_version`, `generated_at`, and `items`.
- `POST/PUT/PATCH /admin/branches/{branch_id}/items/{item_id}`: wrapped response with `status`, `api_version`, `generated_at`, and `item`.
- Body fields: `is_available` as strict boolean, `price_override_raw` as string or null, `sale_price_override_raw` as string or null, `sort_order` as strict integer.

Expected public branch-menu assertions after override updates:

- `is_available: false` hides the item from `GET /menu?branch={{branchSlug}}`.
- `price_override_raw` changes the public item price for that branch.
- `sale_price_override_raw` changes the public sale price for that branch.
- `sort_order` changes branch menu ordering.
- Reset overrides before cleanup when later delete requests depend on fixture visibility.

## Seed Endpoint

- Request: `POST {{baseUrl}}/wp-json/cafe/v1/seed`
- Auth: admin user with `manage_options`.
- Expected success contract: direct `rest_ensure_response()` body with counters such as `inserted`, `updated`, and branch counters; no `status/api_version/generated_at` envelope and no API version header.
- Basic validation scenarios: invalid JSON, empty payload, duplicate slugs, top-level list, `{ "items": [] }`, `{ "data": [] }`, `{ "branches": [] }`, variable items, and branch-item backfill.
- Auth-negative cases: no auth, invalid Application Password, and user without `manage_options` if available.

## Dynamic ID Flow

Recommended end-to-end order:

1. Create group category; capture `groupCategoryId`.
2. Create child category under `groupCategoryId`; capture `childCategoryId`.
3. Create a variable item in `childCategoryId`; capture `itemId`.
4. Create a variant for `itemId`; capture `variantId`.
5. Create an active branch; capture `branchId` and `branchSlug`.
6. Update branch item override for `branchId` and `itemId`.
7. Verify `GET /menu?branch={{branchSlug}}` reflects availability, price override, sale price override, and sort order.
8. Verify `GET /tiles?branch={{branchSlug}}` remains consistent with branch menu filtering.
9. Cleanup in reverse order where possible: delete variant, delete item, delete child category, delete group category. Branch delete is not currently in the audited route inventory, so created branch cleanup may require fixture reset or database cleanup outside the runtime API.

## Response Shape Lock

Current expected shapes to preserve during Phase 0:

- Fully wrapped: public `/menu`, `/tiles`, `/branches`, `/branches/{slug}`; admin item list/create; admin variant list/create; admin branch item list/update.
- Health special case: `GET /health` has `status` and `api_version` but intentionally omits `generated_at`.
- Raw admin arrays/objects: admin categories list/read/update; admin branch create/read/update; admin item read/update; admin variant update.
- Delete partials: category, item, and variant delete responses include `status` and `deleted` but omit `api_version` and `generated_at`.
- Seed direct response: seed counters/errors are not wrapped in the versioned response envelope.
