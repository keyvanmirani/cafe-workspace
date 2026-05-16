# Cafe v1.5 Release Candidate Workspace

This workspace contains the four surfaces used for the Cafe v1.5 release candidate:

- `cafe-headless-wp`: WordPress plugin and REST API.
- `cafe-admin`: Nuxt admin app for branch-first menu management.
- `cafe-public`: Nuxt public menu app.
- `api-tests/Cafe Headless WP API`: Bruno collection for API smoke and regression checks.

The current architecture is branch-first. Branch workspace CRUD creates and edits branch-owned categories, items, and variants. Public branch menu reads prefer those branch-owned rows. The backend intentionally keeps the legacy global menu and `branch_items` fallback path for compatibility, audit, and rollback during the v1.5 candidate.

For the current release-candidate audit, see [RELEASE_CANDIDATE_1_5_AUDIT_2026_05.md](./RELEASE_CANDIDATE_1_5_AUDIT_2026_05.md).
