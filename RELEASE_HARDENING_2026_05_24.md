# Release Hardening Checklist

Date: 2026-05-24
Scope: branch-first recovery integration hardening only.

## Current Repository State

- Root repository: `keyvanmirani/cafe-workspace`, branch `main`, commit `2ab0e2a`.
- Root working tree has updated submodule pointers for:
  - `apps/cafe-admin` at `5636917` on `recovery/branch-first-final-stabilization`
  - `apps/cafe-headless-wp` at `23a76f1` on `recovery/branch-first-final-stabilization`
  - `apps/cafe-public` at `b5546a2` on `recovery/branch-first-final-stabilization`
- Root has no `.github/workflows` directory.
- Frontend CI workflows exist only in:
  - `apps/cafe-admin/.github/workflows/ci.yml`
  - `apps/cafe-public/.github/workflows/ci.yml`
- `cafe-workspace` has no open PRs visible through the GitHub API.
- `cafe-admin-front`, `cafe-farda-v4-front`, and `cafe-headless-wp` PR state could not be audited through unauthenticated API calls; GitHub returned `404`, consistent with private or inaccessible repositories.
- `gh` is not installed locally, so authenticated PR/check rollup could not be confirmed from this workspace.

## Workflow Audit

Admin frontend CI runs on PRs to `main` and pushes to `recovery/*` or `feature/*`:

- install with Node 24 and pnpm 10.28.2
- `pnpm run lint`
- `pnpm run typecheck`
- `pnpm run test:unit`
- `pnpm run build`

Public frontend CI runs on PRs to `main` and pushes to `recovery/*` or `feature/*`:

- install with Node 24 and pnpm 10.30.2
- `pnpm run lint`
- `pnpm run typecheck`
- `pnpm run test:unit`
- `pnpm run build`
- PR-only `public-smoke-a11y` job with `pnpm run test:smoke`, Chromium install, and `pnpm run test:axe`

Required frontend CI checks cannot be marked confirmed green from local evidence alone. Treat merge readiness as conditional on the latest GitHub check runs passing on the relevant PR heads.

## Branch Separation

- Recovery work is separated from app `main` branches on `recovery/branch-first-final-stabilization`.
- Root `main` currently records submodule pointers to recovery commits, so root merge readiness depends on each app repository accepting those recovery heads first or intentionally pinning them as release candidates.
- Admin and public `origin/main` already contain prior recovery-final merges, but local `main` branches are behind their remotes. Do not use local app `main` as release evidence until refreshed.
- Backend `origin/HEAD` points to `origin/admin-api-categories`, not `origin/main`. This should be fixed before relying on default-branch automation or reviewer expectations.

## Merge Readiness

Status: conditionally ready for integration hardening review, not ready for stable release tagging.

Merge blockers before stable release:

- Confirm live GitHub PR status and check rollups with authenticated access.
- Run frontend CI commands in both frontend apps or confirm green required checks from GitHub.
- Run backend source regression tests.
- Run the opt-in live backend integration smoke against a disposable WordPress instance.
- Confirm root submodule pointer merge order after app repositories merge their recovery branches.
- Fix or consciously accept backend remote default branch pointing at `admin-api-categories`.

## Local Validation Performed

Backend source checks passed:

- `node tests/branch-first-contract.test.mjs`
- `node tests/public-read-selection.test.mjs`
- `node tests/admin-active-persistence.test.mjs`
- `node tests/schema-tightening.test.mjs`
- `node tests/integration-smoke.test.mjs` skip path confirmed when live smoke env is absent

Frontend checks passed locally:

- Admin: `pnpm run lint`
- Admin: `pnpm run typecheck`
- Admin: `pnpm run test:unit`
- Admin: `pnpm run build`
- Public: `pnpm run lint`
- Public: `pnpm run typecheck`
- Public: `pnpm run test:unit`
- Public: `pnpm run build`
- Public: `pnpm run test:smoke` passed after allowing localhost dev-server bind

Validation still pending:

- Public: `pnpm run test:axe` could not complete locally because Playwright Chromium is not installed and `pnpm exec playwright install chromium` failed repeatedly with `ECONNRESET`.
- Backend live integration smoke still needs a disposable WordPress instance and `CAFE_BACKEND_INTEGRATION_SMOKE=1`.
- GitHub PR/check rollups still need authenticated GitHub access.

Whitespace:

- `git diff --check` passed at the workspace root.
- `git -C apps/cafe-headless-wp diff --check` passed for backend changes.

## Branch Protection Recommendations

Protect `main` in each app repository and in `cafe-workspace`:

- Require pull requests before merge.
- Require one approving review for recovery/release work; require two for schema, auth, deployment, or destructive data changes.
- Dismiss stale approvals when new commits are pushed.
- Require review from code owners once ownership files exist.
- Require branches to be up to date before merge for frontend apps and the workspace repo.
- Require conversation resolution before merge.
- Block force pushes.
- Block branch deletion.
- Restrict who can push directly to `main`; prefer no direct pushes except emergency maintainers.
- Require signed tags for release tags if available on the plan tier.

## Required Status Checks

`cafe-admin-front`:

- `CI / ci`

`cafe-farda-v4-front`:

- `CI / ci`
- `CI / public-smoke-a11y`

`cafe-headless-wp`:

- backend source regression command, once represented in CI
- opt-in live integration smoke is required before release approval, but should not block every PR until stable infrastructure exists

`cafe-workspace`:

- submodule pointer audit
- `git diff --check`
- release checklist review

## Merge Strategy

- Prefer squash merge for feature, recovery, and hardening PRs into app repositories.
- Use merge commits only for the workspace repository when preserving submodule pointer integration history is useful.
- Disable rebase merge for protected branches unless the team explicitly wants linear history and all contributors are comfortable with it.
- Do not merge app recovery branches into workspace release pointers until the corresponding app PR checks are green.

## Tag And Release Strategy

- Use annotated tags.
- Tag app repositories first, then tag `cafe-workspace` with the exact submodule SHAs.
- Release candidate format: `v1.5.0-rc.N`.
- Stable format: `v1.5.0`.
- Create a GitHub Release for each stable tag with:
  - app commit SHA
  - workspace submodule SHA
  - database migration notes
  - smoke validation result
  - rollback note

## Semantic Commit Expectations

Use conventional commit prefixes:

- `fix:` for release-blocking bug fixes
- `test:` for test and smoke coverage
- `docs:` for release hardening docs
- `ci:` for workflow-only changes
- `chore:` for repository metadata and release prep
- `refactor:` only when behavior is intended to stay unchanged

Avoid new `feat:` commits during release hardening unless the release scope is explicitly reopened.

## Release Checklist

Fresh install:

- Clone each app repository and the workspace with submodules.
- Install frontend dependencies with frozen lockfiles.
- Activate the WordPress plugin on a disposable or staging WordPress instance.
- Confirm `/wp-json/cafe/v1/health` is HTTP 200.

DB reset/reseed:

- Take a database backup first.
- Reset only a disposable or approved staging database.
- Run the branch-first seed endpoint or documented seed operation.
- Confirm branches, branch-owned categories, branch-owned items, and variants exist.

Branch-first schema validation:

- Confirm `categories.branch_id` and `items.branch_id` are non-null for branch-owned runtime rows.
- Confirm `categories` has unique `(branch_id, slug)`.
- Confirm no release-critical path depends on retired `chm_branch_items` as the primary branch menu source.
- Confirm schema tightening status option has no unresolved blocker.

Frontend CI expectations:

- Admin: lint, typecheck, unit tests, build.
- Public: lint, typecheck, unit tests, build, smoke, axe.
- Public smoke and axe are PR-only in CI; run manually before stable release if no PR is open.

Smoke validation:

- Run backend source regression tests.
- Run `CAFE_BACKEND_INTEGRATION_SMOKE=1 node tests/integration-smoke.test.mjs` from `apps/cafe-headless-wp` against a disposable WordPress instance.
- Run Bruno collection checks manually if the live smoke runner is not available.
- Confirm retired global admin routes return HTTP 410.

Rollback basics:

- Keep the previous plugin package and frontend deployment artifacts.
- Keep the pre-release database backup.
- Roll back frontend deployments first if the issue is display-only.
- Roll back the plugin only after confirming database compatibility.
- Do not delete branch-owned data during rollback unless a restore plan has been approved.

Required environment variables:

- Admin frontend API base URL.
- Public frontend API base URL.
- WordPress URL.
- WordPress admin/application-password credentials for release smoke.
- Any hosting-specific build/deploy tokens.

Known remaining risks:

- GitHub PR/check state for private app repos still needs authenticated confirmation.
- Backend has no always-on CI workflow in this workspace.
- Live integration smoke requires a disposable WordPress instance and credentials.
- Ingredient runtime remains deferred/quarantined; do not market ingredient management as stable.
- Backend default branch metadata appears inconsistent because `origin/HEAD` points to `origin/admin-api-categories`.
- Schema tightening depends on existing production data being clean enough for non-null and unique constraints.

## Recommended Merge Order

1. Merge backend recovery/hardening into `cafe-headless-wp/main` after backend source tests and live smoke pass.
2. Merge public frontend recovery into `cafe-farda-v4-front/main` after public CI and smoke/a11y checks pass.
3. Merge admin frontend recovery into `cafe-admin-front/main` after admin CI passes.
4. Update `cafe-workspace` submodule pointers to the merged app `main` commits.
5. Merge the workspace release pointer PR into `cafe-workspace/main`.

## Recommended Release Sequence

1. Freeze feature work.
2. Refresh all branches from origin.
3. Run backend source tests.
4. Reset/reseed disposable WordPress.
5. Run backend live integration smoke.
6. Run admin frontend CI commands.
7. Run public frontend CI commands, including smoke and axe.
8. Merge app repositories in the recommended order.
9. Update and merge workspace submodule pointers.
10. Tag app repositories as `v1.5.0-rc.N`.
11. Tag workspace as `v1.5.0-rc.N`.
12. Deploy release candidate to staging.
13. Repeat smoke validation against staging.
14. Promote to stable `v1.5.0` only after staging signoff.
