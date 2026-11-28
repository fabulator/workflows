# Fabulator workflows

Shared [reusable GitHub Actions workflows](https://docs.github.com/en/actions/sharing-automations/reusing-workflows) for Fabulator's repositories.

Callers reference the moving major tag `v1`. Non-breaking changes move `v1`, breaking changes get `v2`.

## npm package – lint, test, release

`.github/workflows/main.yml` in the package:

```yaml
name: CI

on: [push]

permissions:
    contents: read
    id-token: write # npm trusted publishing (OIDC)

jobs:
    build:
        uses: fabulator/workflows/.github/workflows/npm-package.yml@v1
        secrets:
            release-token: ${{ secrets.GHUB_BOT_TOKEN }}
```

- Runs `npm ci`, `npm run lint`, `npm run typecheck` and `npm test` (each only if the script exists), and `npm run release` on `master`.
- Type checking is opt-in: add `"typecheck": "tsc --noEmit"` to the package's scripts. The name is intentionally not `tsc`, so repositories that already have a `tsc` script are not affected until they opt in.
- Publishes via **npm trusted publishing**. npm validates the **calling** workflow, so on npmjs.com configure the package's repository and workflow filename `main.yml` – not this repository.
- `release-token` is a PAT allowed to push to the release branch (semantic-release commit + tag, GitHub release).
- Inputs: `node-version` (default `26.13.1`), `release-branch` (default `master`).

## pnpm app – lint, type check, test, build

For pnpm (mono)repos that are deployed rather than published to npm. Deploy jobs stay in the caller and can `needs:` this job.

```yaml
name: CI

on:
    pull_request:

permissions:
    contents: read

jobs:
    check:
        uses: fabulator/workflows/.github/workflows/pnpm-app.yml@v1
        with:
            node-version: 24.x
        secrets:
            registry-token: ${{ secrets.G }} # only if private @fabulator packages are installed
```

- Runs `pnpm install --frozen-lockfile`, `pnpm run lint`, `typecheck` and `test` (each only if the script exists) and `pnpm -r run build` (disable with `build: false`).
- Type checking is opt-in via a `"typecheck": "tsc --noEmit"` script, same as in the npm package workflow.
- `registry-token` (optional) authenticates `registry-scope` (default `@fabulator`) on `npm.pkg.github.com`.
- Inputs: `node-version` (default `26.13.1`), `pnpm-version` (default `11`), `registry-scope`, `build` (default `true`).

## PR title

`.github/workflows/pr-title.yml` in the repository:

```yaml
name: PR title

on:
    pull_request:
        types: [opened, edited, synchronize, reopened]

permissions:
    contents: read

jobs:
    pr-title:
        uses: fabulator/workflows/.github/workflows/pr-title.yml@v1
```

Lints the PR title with [commitlint-config-fabulator](https://github.com/fabulator/commitlint-config-fabulator). It does not depend on the caller's dependencies.

## Releasing a change

```bash
git tag -f v1 && git push -f origin v1
```
