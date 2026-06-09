# Cafe v1.5 Release Candidate Workspace

This workspace contains the four surfaces used for the Cafe v1.5 release candidate:

- `cafe-headless-wp`: WordPress plugin and REST API.
- `cafe-admin`: Nuxt admin app for branch-first menu management.
- `cafe-public`: Nuxt public menu app.
- `api-tests/Cafe Headless WP API`: Bruno collection for API smoke and regression checks.

The current architecture is branch-first. Branch workspace CRUD creates and edits branch-owned categories, items, ingredients, and variants. Public branch menu reads branch-owned rows through the v1.5 REST contracts.

Useful current docs:

- [Release hardening checklist](./RELEASE_HARDENING_2026_05_24.md)
- [Manual smoke checklist](./RC_1_5_MANUAL_SMOKE_TEST.md)
- [Admin setup](./apps/cafe-admin/README.md)
- [Public setup](./apps/cafe-public/README.md)
- [Backend setup](./apps/cafe-headless-wp/README.md)
