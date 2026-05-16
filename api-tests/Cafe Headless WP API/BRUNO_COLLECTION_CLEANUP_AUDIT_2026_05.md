# Bruno Collection Cleanup Audit - Phase 19 - 2026-05

Scope: `api-tests/Cafe Headless WP API/` only. No backend/runtime contracts were changed.

## Current Collection Organization

Primary branch-first folders:

- `Public_Menu`: public branch-owned reads for `/menu`, `/tiles`, and public branch lookup.
- `Admin_Branches`: branch metadata CRUD.
- `Admin_Branch_Categories`: branch-owned category CRUD.
- `Admin_Branch_Owned_Items`: branch-owned item CRUD.
- `Admin_Branch_Variants`: branch-owned variant CRUD.
- `Migration_And_Status`: health, branch-first readiness/status reports, and seed fixture setup.

Legacy audit folders intentionally remain split by old route family:

- `Legacy_Admin_Categories`
- `Legacy_Admin_Items`
- `Legacy_Admin_Item_Variants`

These are the `Legacy_Admin_Audit` equivalent for this collection. They stay separate so historical route ownership remains obvious while the branch-first workflow is the default.

## Cleanup Performed

- Renamed `Public` to `Public_Menu`.
- Renamed `Maintenance` to `Migration_And_Status`.
- Moved `Health` into `Migration_And_Status`.
- Added branch-first status and readiness smoke requests:
  - `GET /wp-json/cafe/v1/admin/branch-first-status`
  - `GET /wp-json/cafe/v1/admin/branch-first-readiness-report`
- Updated public menu/tile smoke requests to use `{{branchSlug}}` instead of a hard-coded branch slug.
- Removed the duplicate/malformed `Branches - With item` request. It pointed at `/menu?branch={{branchSlug}}` but contained branch slug examples, duplicating the public menu smoke path.
- Normalized the example environment variable names to the active collection convention: `baseUrl`, `username`, secret `password`, `branchId`, `branchSlug`, `categoryId`, `itemId`, and `variantId`.
- Assigned folder sequence values so branch-first folders appear before legacy audit folders.

## Branch-First Workflow Notes

Preferred smoke order:

1. `Migration_And_Status/Health`
2. `Migration_And_Status/Branch First Status`
3. `Migration_And_Status/Branch First Readiness Report`
4. `Public_Menu/Public Menu - Branch Owned Read`
5. `Admin_Branch_Categories/*`
6. `Admin_Branch_Owned_Items/*`
7. `Admin_Branch_Variants/*`

For branch-owned CRUD, use the `Admin_Branch_*` folders. The route family `/admin/branches/{branchId}/items` is branch-owned item CRUD in the stabilized contract, not the old override editor surface.

## Legacy Fallback Notes

The `Legacy_Admin_*` folders are preserved for rollback/debugging and compatibility auditing only. They cover global/item-scoped admin endpoints that still exist in the backend while legacy fallback remains intentional.

Do not add new branch workspace coverage under `Legacy_Admin_*`. Add new branch-first coverage under the relevant `Admin_Branch_*` folder instead.
