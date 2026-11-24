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

- Runs `npm ci`, `npm run lint` and `npm test` (both only if the script exists), and `npm run release` on `master`.
- Publishes via **npm trusted publishing**. npm validates the **calling** workflow, so on npmjs.com configure the package's repository and workflow filename `main.yml` – not this repository.
- `release-token` is a PAT allowed to push to the release branch (semantic-release commit + tag, GitHub release).
- Inputs: `node-version` (default `26.13.1`), `release-branch` (default `master`).

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
