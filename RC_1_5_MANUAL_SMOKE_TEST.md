# v1.5 RC Manual Smoke Test Checklist

Scope: v1.5 release-candidate manual smoke preparation for the current branch-first cafe ecosystem.

Projects:

- `cafe-headless-wp`
- `cafe-admin-front`
- `cafe-farda-v4-front`
- `cafe-api-bruno`

Rules for this smoke pass:

- Do not add new features during smoke testing.
- Do not redesign UI during smoke testing.
- Do not introduce inventory, stock, orders, payments, checkout, accounting, or ERP logic.
- Use a disposable or release-candidate-safe WordPress database.
- Live WordPress validation must be run manually on the developer machine where WP Studio is running. Codex static, source, and build validation can pass without proving that the WP Studio WordPress instance, local HTTPS certificate, REST routes, or live database are reachable.
- Record every failure with exact route, request, screen, branch, category, item, and variant where applicable.

For each item, fill:

- Status: `PASS` / `FAIL` / `SKIP`
- Notes:
- Screenshot needed: `yes` / `no`

## Test Context

- Date:
- Tester:
- Environment:
- Backend URL:
- Admin URL:
- Public URL:
- WordPress user:
- Branch ID:
- Branch slug:
- Category ID:
- Fixed item ID:
- Variable item ID:
- Variant ID:
- Ingredient ID:

## WP Studio Live Validation

Run this section manually on the developer machine that has the WP Studio site running. Replace the sample values with the actual WP Studio site URL and an active branch slug from that site.

```sh
export WP_STUDIO_SITE_URL="https://cafeapi.wp.local"
export RC_BRANCH_SLUG="mehrvila"
```

The `WP_STUDIO_SITE_URL` value is the WordPress site origin only. Do not include `/wp-json/cafe/v1` in that variable.

### Site And Public REST Checks

- [ ] Confirm the WP Studio site URL opens and returns an HTTP response.

```sh
curl -I "$WP_STUDIO_SITE_URL"
```

- [ ] Confirm the Cafe health endpoint returns HTTP 200 and a healthy JSON response.

```sh
curl -fsS "$WP_STUDIO_SITE_URL/wp-json/cafe/v1/health"
```

- [ ] Confirm public branches return HTTP 200 and include the branch intended for RC validation.

```sh
curl -fsS "$WP_STUDIO_SITE_URL/wp-json/cafe/v1/branches"
```

- [ ] Confirm the branch menu endpoint returns branch-scoped menu data.

```sh
curl -fsS "$WP_STUDIO_SITE_URL/wp-json/cafe/v1/menu?branch=$RC_BRANCH_SLUG"
```

- [ ] Confirm the branch tiles endpoint returns branch-scoped tile data.

```sh
curl -fsS "$WP_STUDIO_SITE_URL/wp-json/cafe/v1/tiles?branch=$RC_BRANCH_SLUG"
```

### Public App API Base URL And Image Audit

- [ ] Confirm the public app environment points at the WP Studio Cafe API base URL.

```sh
cd apps/cafe-public
printf '%s\n' "$NUXT_WORDPRESS_API_BASE_URL"
test "$NUXT_WORDPRESS_API_BASE_URL" = "$WP_STUDIO_SITE_URL/wp-json/cafe/v1"
```

If `NUXT_WORDPRESS_API_BASE_URL` is not exported in the shell, set it before running local public validation:

```sh
export NUXT_WORDPRESS_API_BASE_URL="$WP_STUDIO_SITE_URL/wp-json/cafe/v1"
```

- [ ] Run the RC image audit against the WP Studio public API.

```sh
cd apps/cafe-public
NUXT_WORDPRESS_API_BASE_URL="$WP_STUDIO_SITE_URL/wp-json/cafe/v1" pnpm run audit:images:rc
```

### Optional Backend Live Smoke

Run this only when a disposable or release-candidate-safe WP Studio database is available and credentials are available. The smoke runner writes live branch, category, and item data and then attempts cleanup.

```sh
cd apps/cafe-headless-wp
CAFE_BACKEND_INTEGRATION_SMOKE=1 \
CAFE_API_BASE_URL="$WP_STUDIO_SITE_URL" \
CAFE_API_USERNAME="admin" \
CAFE_API_PASSWORD="application-password-or-test-password" \
node tests/integration-smoke.test.mjs
```

Skip this command when any required credential or environment value is unavailable, and record the reason in the smoke notes.

### Optional Bruno Manual Run

Use this only if Bruno is installed and the local environment is configured with the WP Studio site URL and RC IDs/slugs.

```sh
cd tools/cafe-api-bruno
bru run --env "Local WordPress API"
```

At minimum, the Bruno environment should set:

- `baseUrl`: WP Studio site origin, for example `https://cafeapi.wp.local`
- `username`: WordPress user for authenticated admin requests
- `password`: WordPress application password or test password
- `branchSlug`: active branch slug used for public menu and tiles requests
- `branchId`, `categoryId`, `itemId`, `variantId`, and `ingredientId` when running admin or mutation requests

## Backend/API

### Authenticated Admin Branch List/Read/Update

- [ ] Admin branch list returns authenticated branch collection.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Admin branch single read returns the selected branch by ID.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Admin branch update persists profile fields without affecting unrelated branches.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Active/inactive branch state persists after update and read-back.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] `sort_order` persists after update and affects returned ordering where supported.
  - Status:
  - Notes:
  - Screenshot needed:

### Branch Categories

- [ ] Create a branch leaf category under the selected branch.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Read the created branch category in the branch category list.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Update branch category name, active state, and `sort_order`.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Confirm category active/inactive persistence after read-back.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Confirm category `sort_order` behavior in list response.
  - Status:
  - Notes:
  - Screenshot needed:

### Fixed-Price Branch Item

- [ ] Create a fixed-price branch item in the branch leaf category.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Read the fixed-price item by ID.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Update fixed-price item fields, active state, and `sort_order`.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Confirm fixed-price item active/inactive persistence after read-back.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Confirm fixed-price item `sort_order` behavior in list/public response.
  - Status:
  - Notes:
  - Screenshot needed:

### Variable Branch Item And Variants

- [ ] Create a variable branch item in the branch leaf category.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Read the variable item by ID.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Create the first item variant.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Create a second item variant.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Update a variant name, price, active state, and `sort_order`.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Confirm variant active/inactive persistence after read-back.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Confirm variant `sort_order` behavior in list/public response.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Delete a variant and confirm it no longer appears.
  - Status:
  - Notes:
  - Screenshot needed:

### Branch Ingredients And Composition

- [ ] Create a branch ingredient for the selected branch.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Read the created ingredient in the branch ingredient list.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Assign item ingredients to a branch item.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Replace item ingredients with a changed composition.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Confirm item ingredient list reflects the replacement exactly.
  - Status:
  - Notes:
  - Screenshot needed:

### Retired Global Routes

- [ ] Global admin categories route returns `410`.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Global admin items route returns `410`.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Global admin item variants route returns `410`.
  - Status:
  - Notes:
  - Screenshot needed:

## Admin

### Login And Workspace

- [ ] Admin login succeeds with the release-candidate WordPress user.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Failed login shows a useful error without breaking the page.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Branch workspace opens from the branch list.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Workspace state clearly reflects the selected branch.
  - Status:
  - Notes:
  - Screenshot needed:

### Branch Profile

- [ ] Branch profile edit form loads existing branch values.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Saving branch profile changes persists after refresh.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Branch active/inactive change persists and remains visible after reload.
  - Status:
  - Notes:
  - Screenshot needed:

### Branch Categories

- [ ] Create a branch category from the admin UI.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Edit branch category fields from the admin UI.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Category loading, empty, and error states are understandable.
  - Status:
  - Notes:
  - Screenshot needed:

### Branch Items

- [ ] Create a branch item from the admin UI.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Edit branch item fields from the admin UI.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Fixed-price item flow saves price and displays the fixed-price state.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Variable item flow allows item creation without fixed price.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Variants can be created, edited, reordered by `sort_order`, activated/deactivated, and deleted.
  - Status:
  - Notes:
  - Screenshot needed:

### Ingredients And Composition UI

- [ ] Branch ingredient create/edit UI works for the selected branch.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Item composition UI assigns ingredients to an item.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Item composition UI replaces ingredients without keeping stale rows.
  - Status:
  - Notes:
  - Screenshot needed:

### Admin UX Sanity

- [ ] RTL layout is visually sane across branch workspace, categories, items, variants, and ingredients.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Toasts appear for successful create/update/delete actions.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] API errors appear as actionable UI messages.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Loading states appear during slow branch/category/item/variant/ingredient requests.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Mobile width sanity check passes for login and branch workspace.
  - Status:
  - Notes:
  - Screenshot needed:

## Public

### Branch Entry

- [ ] Homepage branch cards preserve `?branch=` when selecting/opening a branch path.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Menu page requires a `branch` query and handles missing branch query cleanly.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Invalid branch query displays a useful public error or empty state.
  - Status:
  - Notes:
  - Screenshot needed:

### Menu Navigation

- [ ] Category/group navigation displays branch-owned categories.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Category/group navigation scrolls or switches sections correctly.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Public menu respects active/inactive category, item, and variant state.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Public menu respects expected `sort_order` for categories, items, and variants.
  - Status:
  - Notes:
  - Screenshot needed:

### Product Display

- [ ] Product modal opens from a menu item.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Product modal closes cleanly and restores page interaction.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Fixed-price item displays the correct fixed price.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Variable item displays variants and variant prices correctly.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Image fallback behavior works when an item image is missing.
  - Status:
  - Notes:
  - Screenshot needed:

### Public UX Sanity

- [ ] Mobile menu navigation is usable at phone width.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] Mobile product modal is usable at phone width.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] RTL visual sanity passes on homepage, menu, navigation, and modal.
  - Status:
  - Notes:
  - Screenshot needed:

## Bruno

### Required Environment Variables

Confirm the selected Bruno environment includes:

- [ ] `baseUrl`
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] `username`
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] `password` stored as a secret value where supported.
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] `branchId`
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] `branchSlug`
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] `categoryId`
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] `itemId`
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] `variantId`
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] `ingredientId`
  - Status:
  - Notes:
  - Screenshot needed:

### Requests To Run First

Run these before mutating branch menu data:

- [ ] `Core_Smoke/Health`
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] `Public_Menu/Branches - List`
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] `Public_Menu/Branches - Single by Slug`
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] `Admin_Branches/Branches - Admin List`
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] `Admin_Branches/Branches - Admin Single by ID`
  - Status:
  - Notes:
  - Screenshot needed:

### Suggested Smoke Order

1. `Core_Smoke/Health`
2. `Public_Menu/Branches - List`
3. `Public_Menu/Branches - Single by Slug`
4. `Admin_Branches/Branches - Admin List`
5. `Admin_Branches/Branches - Admin Single by ID`
6. `Admin_Branches/Branches - Admin Update`
7. `Admin_Branch_Categories/Branch Categories - Admin Create`
8. `Admin_Branch_Categories/Branch Categories - Admin List`
9. `Admin_Branch_Categories/Branch Categories - Admin Single`
10. `Admin_Branch_Categories/Branch Categories - Admin Update`
11. `Admin_Branch_Owned_Items/Branch Owned Items - Admin Create Fixed`
12. `Admin_Branch_Owned_Items/Branch Owned Items - Admin Create Variable`
13. `Admin_Branch_Owned_Items/Branch Owned Items - Admin List`
14. `Admin_Branch_Owned_Items/Branch Owned Items - Admin Single`
15. `Admin_Branch_Owned_Items/Branch Owned Items - Admin Update`
16. `Admin_Branch_Variants/Branch Variants - Admin Create`
17. `Admin_Branch_Variants/Branch Variants - Admin List`
18. `Admin_Branch_Variants/Branch Variants - Admin Single`
19. `Admin_Branch_Variants/Branch Variants - Admin Update`
20. `Admin_Branch_Variants/Branch Variants - Admin Delete`
21. `Admin_Branch_Ingredients/Branch Ingredients - Admin Create`
22. `Admin_Branch_Ingredients/Branch Ingredients - Admin List`
23. `Admin_Branch_Ingredients/Branch Ingredients - Admin Single`
24. `Admin_Branch_Ingredients/Branch Ingredients - Admin Update`
25. `Admin_Branch_Ingredients/Branch Item Ingredients - Admin Replace`
26. `Admin_Branch_Ingredients/Branch Item Ingredients - Admin List`
27. `Public_Menu/Public Menu - Branch Required`
28. `Public_Menu/Public Tiles - Branch Required`
29. `Public_Menu/Public Menu - Branch Owned Read`
30. `Public_Menu/Public Tiles - Branch Owned Read`
31. `Removed_Admin_Routes/Removed Admin Categories - 410`
32. `Removed_Admin_Routes/Removed Admin Items - 410`
33. `Removed_Admin_Routes/Removed Admin Item Variants - 410`

Cleanup requests, if run against disposable data only:

- [ ] `Admin_Branch_Owned_Items/Branch Owned Items - Admin Delete`
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] `Admin_Branch_Categories/Branch Categories - Admin Delete`
  - Status:
  - Notes:
  - Screenshot needed:
- [ ] `Admin_Branch_Ingredients/Branch Ingredients - Admin Delete`
  - Status:
  - Notes:
  - Screenshot needed:

## Release Decision

- Can tag `v1.5.0-rc.1`?
  - Status:
  - Notes:
  - Screenshot needed:
- Blocking issues:
  - Status:
  - Notes:
  - Screenshot needed:
- Non-blocking polish:
  - Status:
  - Notes:
  - Screenshot needed:
- Follow-up after RC:
  - Status:
  - Notes:
  - Screenshot needed:

Decision owner:

Decision date:

Final decision:
