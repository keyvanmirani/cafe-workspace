# Cafe v1.5 Release Candidate Audit - 2026-05

Scope: release-candidate documentation and stabilization pass after Phase 19 Bruno cleanup. This is not a feature phase and does not change backend contracts.

## Current Architecture Summary

- Backend: WordPress plugin exposing `/wp-json/cafe/v1`.
- Admin: Nuxt app using the WordPress REST API for authenticated branch and menu management.
- Public: Nuxt app rendering branch-aware public menu views.
- API tests: Bruno collection organized around public reads, branch-first admin CRUD, migration/status checks, and legacy audit folders.

The architecture direction is branch-first. Operational menu data belongs to a branch through branch-owned categories, branch-owned items, and branch-owned variants. Public branch menu rendering is branch-owned when a branch has branch-owned rows.

## Branch-First Workflow Summary

1. Create or select an active branch in admin.
2. Manage categories from that branch workspace.
3. Manage items from that branch workspace.
4. Manage variants for branch-owned variable items.
5. Validate public `/menu?branch={slug}` and `/tiles?branch={slug}` for the same branch.

New branch workspace coverage and manual testing should use:

- `Admin_Branches`
- `Admin_Branch_Categories`
- `Admin_Branch_Owned_Items`
- `Admin_Branch_Variants`
- `Public_Menu`

## Legacy Fallback Explanation

Legacy fallback intentionally remains in the backend only. Global categories/items, item-scoped variants, and `branch_items` compatibility rows are preserved for audit, compatibility, and rollback.

Public branch reads prefer branch-owned rows when present. Branches without branch-owned menu data can still fall back to the legacy branch read path. Admin branch workspace pages should not create new legacy menu data.

## Current Known Limitations

- Native `WP_Error` responses keep WordPress error shape instead of the success envelope.
- Admin branch list remains active-branch oriented.
- Legacy/global admin pages remain available for audit and rollback comparison.
- Image base URL customization helpers are not consistently used by every formatter.
- Public app API proxy targets are configured directly in `cafe-public/nuxt.config.ts`.
- Branch deletion is not part of the current Bruno dynamic cleanup flow.

## Recommended Smoke-Test Flow

Backend/API through Bruno:

1. `Migration_And_Status/Health`
2. `Migration_And_Status/Branch First Status`
3. `Migration_And_Status/Branch First Readiness Report`
4. `Public_Menu/Branches - List`
5. `Public_Menu/Branches - Single by Slug`
6. `Admin_Branch_Categories` create/list/update/delete flow
7. `Admin_Branch_Owned_Items` create/list/update/delete flow
8. `Admin_Branch_Variants` create/list/update/delete flow for a variable item
9. `Public_Menu/Public Menu - Branch Owned Read`
10. `Public_Menu/Public Tiles - Branch Owned Read`

Frontend checks:

- Admin: login, open `/branches`, open a branch workspace, create or edit a branch-owned category, create or edit a branch-owned item, and verify variant handling for variable items.
- Public: open `/`, select a branch, verify `/menu?branch={branchSlug}`, nested category navigation, classic menu, image fallback, and offline/error states.

Quality commands:

- `cd cafe-admin && pnpm lint && pnpm test:unit && pnpm typecheck && pnpm build`
- `cd cafe-public && pnpm lint && pnpm test:unit && pnpm typecheck && pnpm test:smoke && pnpm test:axe && pnpm build`

## Deploy/Runtime Prerequisites

- WordPress with `cafe-headless-wp` active and plugin tables installed.
- WordPress admin user with `manage_options`.
- Application Password credentials for admin API and Bruno.
- Admin frontend:
  - `NUXT_PUBLIC_API_BASE=https://{host}/wp-json/cafe/v1`
  - `NUXT_PUBLIC_WP_REST_BASE=https://{host}/wp-json`
- Bruno environment:
  - `baseUrl`
  - `username`
  - secret `password`
  - `branchId`
  - `branchSlug`
  - `categoryId`
  - `itemId`
  - `variantId`
- Public frontend dev proxy target aligned to the deployed WordPress host.

## Rollback Considerations

- Do not delete legacy global rows or `branch_items` compatibility rows during v1.5 rollout.
- If branch-owned public rendering exposes a branch-specific issue, remove or quarantine that branch's branch-owned rows only after confirming legacy fallback behavior.
- Admin branch workspace can be rolled back at the frontend routing/import level because legacy global admin pages still exist.
- Backend rollback should preserve response contracts and avoid schema cleanup until a dedicated post-v1.5 removal phase.
- Backfill/repair operations are high-impact and should be used only against known fixtures or with a database backup.

## Known Issues / Deferred Items

- Branch analytics.
- Inventory.
- Multi-admin permissions and roles beyond `manage_options`.
- Laravel migration evaluation.
- Advanced media management.
- Public/admin virtualization and performance improvements for large menus.
- Advanced transaction handling around multi-step branch-owned CRUD.
- Formal removal of legacy fallback and compatibility tables after a separate readiness gate.

## Audit Notes

- Bruno folder naming is branch-first first, legacy audit second.
- Environment variable naming is consistent in Bruno around `baseUrl`, `username`, `password`, `branchId`, `branchSlug`, `categoryId`, `itemId`, and `variantId`.
- Admin runtime environment naming is `NUXT_PUBLIC_API_BASE` and `NUXT_PUBLIC_WP_REST_BASE`.
- Remaining uses of "legacy", "fallback", and "branch-owned" in docs are intentional where they describe migration state or API ownership.
