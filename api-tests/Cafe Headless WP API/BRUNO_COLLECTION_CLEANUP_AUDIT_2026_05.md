# Bruno Collection Cleanup Audit - 2026-05

Scope: `api-tests/Cafe Headless WP API/` only. No backend/runtime code was changed.

## Current Collection Structure Before Cleanup

The collection contained these active route groups:

- `Public`
- `Maintenance`
- `Admin_Branches`
- `Admin_Categories`
- `Admin_Items`
- `Admin_Variant`
- `Admin_Branch_Items`
- `Admin_Branch_Categories`
- `Admin_Branch_Owned_Items`
- `Admin_Branch_Variants`

The conflict was `Admin_Branch_Items`: it used `/admin/branches/{branchId}/items` for legacy `chm_branch_items` override management, while `Admin_Branch_Owned_Items` uses the same route family for branch-owned item CRUD. The current branch-first backend registers that route family to branch-owned item CRUD.

## Obsolete Folders/Files

Removed:

- `Admin_Branch_Items/folder.bru`
- `Admin_Branch_Items/Branch Items - Admin List.bru`
- `Admin_Branch_Items/Branch Item - Admin Update.bru`

Reason: these requests sent/read legacy override payloads (`is_available`, `price_override_raw`, `sale_price_override_raw`) against route paths that now map to branch-owned item CRUD. Keeping them as first-class requests would create false failures and hide the branch-first contract.

## Renamed Transitional Folders

Renamed for clarity:

- `Admin_Categories` -> `Legacy_Admin_Categories`
- `Admin_Items` -> `Legacy_Admin_Items`
- `Admin_Variant` -> `Legacy_Admin_Item_Variants`

Reason: these global/item-scoped admin endpoints still exist and are useful for transitional compatibility, migration verification, and regression checks, but they are no longer the preferred branch workspace CRUD surface.

## Transitional Folders/Files Preserved

Preserved:

- `Legacy_Admin_Categories`: global category CRUD compatibility and validation.
- `Legacy_Admin_Items`: global item CRUD compatibility, including legacy active-item branch attachment behavior.
- `Legacy_Admin_Item_Variants`: item-scoped variant compatibility.
- `Admin_Branches`: branch metadata CRUD remains current.
- `Public`: public menu/tiles/branches compatibility remains current during migration.
- `Maintenance`: seed behavior remains useful for fixture setup and legacy migration checks.
- `API_CONTRACT_CAPTURE_2026_05.md`: preserved as a historical capture. Its branch item override section is no longer represented by active Bruno requests because that route family is now branch-owned item CRUD.

## Branch-First Folders Preserved

Current branch-first API test folders:

- `Admin_Branch_Categories`
- `Admin_Branch_Owned_Items`
- `Admin_Branch_Variants`

These are the primary branch workspace CRUD collections and should be the default smoke-test path for new backend work.

## Recommended Final Structure

Recommended folder order:

- `Public`
- `Maintenance`
- `Admin_Branches`
- `Admin_Branch_Categories`
- `Admin_Branch_Owned_Items`
- `Admin_Branch_Variants`
- `Legacy_Admin_Categories`
- `Legacy_Admin_Items`
- `Legacy_Admin_Item_Variants`
- `environments`

Recommended policy:

- Add new branch workspace CRUD tests only under the `Admin_Branch_*` folders.
- Keep `Legacy_Admin_*` folders until global endpoints are formally deprecated or removed.
- Do not reintroduce branch item override requests under `/admin/branches/{branchId}/items` unless a distinct compatibility route is restored.

## What Was Deleted

Deleted the obsolete legacy branch item override folder:

```text
api-tests/Cafe Headless WP API/Admin_Branch_Items/
```

## What Was Intentionally Preserved

Preserved global admin CRUD coverage under `Legacy_Admin_*` because the backend still exposes these endpoints during the transition.

Preserved public menu and seed coverage because public behavior and seed fixture behavior have not yet been rewritten to branch-first reads.
