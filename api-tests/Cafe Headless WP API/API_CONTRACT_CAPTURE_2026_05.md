# Cafe Headless WP API Contract Capture - 2026-05

Scope: Bruno regression coverage for the current `/wp-json/cafe/v1` v1.5 release-candidate behavior after branch-first stabilization and Bruno cleanup. These notes lock the current branch-first public/admin smoke expectations; do not normalize or redesign backend contracts as part of the release-candidate pass.

## Bruno Execution Notes

- Collection: `Cafe Headless WP API`
- Base URL variable: `{{baseUrl}}`
- Admin auth variables: use the environment's admin Application Password credentials.
- Dynamic IDs should be captured from create responses and reused in follow-up requests: `categoryId`, `itemId`, `variantId`, `branchId`, and `branchSlug`.
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
- Assert branch-owned categories/items/variants are reflected for the requested branch.
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
- Branch-owned items: `/admin/branches/{branch_id}/items`, `/admin/branches/{branch_id}/items/{item_id}`
- Items: `/admin/items`, `/admin/items/{id}`
- Variants: `/admin/items/{item_id}/variants`, `/admin/items/{item_id}/variants/{variant_id}`
- Seed: `/seed`

Expected auth-negative cases:

- No auth: request should not succeed; assert WordPress REST error shape with `code`, `message`, and `data.status`.
- Invalid Application Password: request should not succeed; assert WordPress REST error shape with `code`, `message`, and `data.status`.
- Authenticated user without `manage_options`, if available: request should not succeed; assert forbidden status and WordPress REST error shape.

## Legacy Admin Categories CRUD

### Create Group Category

- Request: `POST {{baseUrl}}/wp-json/cafe/v1/admin/categories`
- Body: `name`, unique `slug`, `parent_id: null`, `sort_order`.
- Expected success contract: normalized response with `status`, `api_version`, `generated_at`, and `category`; HTTP `201`.
- Capture `id` as `groupCategoryId`.

### Create Child Category

- Request: `POST {{baseUrl}}/wp-json/cafe/v1/admin/categories`
- Body: `name`, unique `slug`, `parent_id: {{groupCategoryId}}`, `sort_order`.
- Expected success contract: normalized response with `status`, `api_version`, `generated_at`, and `category`; HTTP `201`.
- Capture `id` as `childCategoryId`.

### List, Read, Update, Delete

- `GET /admin/categories`: normalized response with `categories`; empty list is `categories: []`.
- `GET /admin/categories/{id}`: normalized response with `category`.
- `PUT/PATCH/POST /admin/categories/{id}`: normalized response with `category`.
- `DELETE /admin/categories/{id}`: normalized response preserving delete internals such as `deleted` and IDs.
- Regression cases: duplicate slug on create/update, self-parent update rejection, blocked delete while children/items exist, successful delete after dependencies are removed.

## Admin Branches CRUD

- `GET /admin/branches`: currently reuses public list; wrapped response with `status`, `api_version`, `generated_at`, and `branches`; inactive branches are omitted.
- `POST /admin/branches`: normalized response with `branch`; HTTP `201`. Capture `id` as `branchId` and `slug` as `branchSlug`.
- `GET /admin/branches/{id}`: normalized response with `branch`.
- `PUT /admin/branches/{id}`: normalized response with `branch`.
- Regression cases: required `name` and `slug`, duplicate slug, boolean-ish `is_active`, and sort order coercion.

## Legacy Admin Items CRUD

- `GET /admin/items`: wrapped response with `status`, `api_version`, `generated_at`, and `items`.
- `POST /admin/items`: wrapped response with `status`, `api_version`, `generated_at`, and `item`; HTTP `201`. Capture `id` as `itemId`.
- `GET /admin/items/{id}`: normalized response with `item`.
- `PUT /admin/items/{id}`: normalized response with `item`.
- `DELETE /admin/items/{id}`: normalized response preserving delete internals such as `deleted` and IDs.
- Regression cases: fixed item requires price, variable item may be created for variant flow, blocked delete while variants exist, active item attaches to active branches.

## Legacy Admin Variants CRUD

- Parent item must exist and be variable.
- `GET /admin/items/{item_id}/variants`: wrapped response with `status`, `api_version`, `generated_at`, and `variants`.
- `POST /admin/items/{item_id}/variants`: wrapped response with `status`, `api_version`, `generated_at`, and `variant`; HTTP `201`. Capture `id` as `variantId`.
- `PUT /admin/items/{item_id}/variants/{variant_id}`: normalized response with `variant`.
- `DELETE /admin/items/{item_id}/variants/{variant_id}`: normalized response preserving delete internals such as `deleted` and IDs.
- Regression cases: reject variants for fixed items, require name and price/price_raw, delete before deleting variable parent item.

## Admin Branch-Owned Items

- `GET /admin/branches/{branch_id}/items`: wrapped response with `status`, `api_version`, `generated_at`, and `items`.
- `POST /admin/branches/{branch_id}/items`: creates a branch-owned item.
- `GET/PUT/PATCH/DELETE /admin/branches/{branch_id}/items/{item_id}`: reads, updates, or deletes a branch-owned item.
- Body fields follow the branch-owned item CRUD contract: `category_id`, `name`, `description`, `image`, `price_raw`, `sale_price_raw`, `is_variable`, `is_active`, `sort_order`, `ingredients`, and `data`.

Expected public branch-menu assertions after branch-owned CRUD:

- Active branch-owned items appear in `GET /menu?branch={{branchSlug}}`.
- Inactive branch-owned items stay hidden from `GET /menu?branch={{branchSlug}}`.
- `price_raw` and `sale_price_raw` drive branch menu pricing for that branch-owned item.
- `sort_order` changes branch menu ordering.
- Clean up branch-owned variants before deleting variable branch-owned items.

## Seed Endpoint

- Request: `POST {{baseUrl}}/wp-json/cafe/v1/seed`
- Auth: admin user with `manage_options`.
- Expected success contract: direct `rest_ensure_response()` body with counters such as `inserted`, `updated`, and branch counters; no `status/api_version/generated_at` envelope and no API version header.
- Basic validation scenarios: invalid JSON, empty payload, duplicate slugs, top-level list, `{ "items": [] }`, `{ "data": [] }`, `{ "branches": [] }`, variable items, and branch-item backfill.
- Auth-negative cases: no auth, invalid Application Password, and user without `manage_options` if available.

## Dynamic ID Flow

Recommended end-to-end order:

1. Create or select an active branch; capture `branchId` and `branchSlug`.
2. Create a branch-owned category for `branchId`; capture `categoryId`.
3. Create a variable branch-owned item in `categoryId`; capture `itemId`.
4. Create a branch-owned variant for `itemId`; capture `variantId`.
5. Verify `GET /menu?branch={{branchSlug}}` reflects branch-owned availability, pricing, and sort order.
6. Verify `GET /tiles?branch={{branchSlug}}` remains consistent with branch menu filtering.
7. Cleanup in reverse order where possible: delete variant, delete item, delete category. Branch delete is not currently in the audited route inventory, so created branch cleanup may require fixture reset or database cleanup outside the runtime API.

## Response Shape Lock

Current expected shapes to preserve for the v1.5 release candidate:

- Fully wrapped success responses: public `/menu`, `/tiles`, `/branches`, `/branches/{slug}`; admin branches; legacy admin categories/items/variants; branch-owned categories/items/variants; migration/status reads.
- Health special case: `GET /health` has `status` and `api_version` but intentionally omits `generated_at`.
- Native WordPress errors: auth, validation, lookup, and many database failures remain `WP_Error` shapes with `code`, `message`, and `data.status`.
- Delete responses: preserve legacy delete internals such as `deleted` and related IDs inside the normalized success envelope.
- Seed direct response: seed counters/errors are not wrapped in the versioned response envelope.
