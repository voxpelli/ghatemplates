# Workflows

My personal reusable GitHub Actions workflows (geared towards node.js)

(_No versioning strategy for these ones, use as inspiration or at your own risk_)

## Reusable Workflows

### Code Quality & Analysis

* [dependency-review.yml](./.github/workflows/dependency-review.yml) – Wrapper around [`actions/dependency-review-action`](https://github.com/actions/dependency-review-action) thar configures it to eg. fail for GPL and AGPL dependencies (as is typical in the JS ecosystem)
* [lint.yml](./.github/workflows/lint.yml) – Runs the npm script `check` on your codebase for linting etc.
* [type-check.yml](./.github/workflows/type-check.yml) – Performs TypeScript type checking across multiple TS versions.

### Publishing & Release Automation

* [release-please-4.yml](./.github/workflows/release-please-4.yml) – Full flow for [googleapis/release-please-action@v4`](https://github.com/googleapis/release-please-action) – both PR generation and NPM publishing.
* [release-please-bot.yml](./.github/workflows/release-please-bot.yml) – Like `release-please-4.yml` but uses a bot account rather than the `GITHUB_TOKEN` when it generates PR:s etc.
* [release-please-oidc.yml](./.github/workflows/release-please-oidc.yml) – Like `release-please-bot.yml` but uses OIDC authentication rather than a `NPM_TOKEN`.

### Testing

* [exit-silently-on-unsupported.yml](./.github/workflows/exit-silently-on-unsupported.yml) – Tests that CLI tools exists silently on unsupported Node.js versions (see eg. [version-guard](https://github.com/voxpelli/version-guard)).
* [simple-test.yml](./.github/workflows/simple-test.yml) – Runs `npm test` on `lts/*` - nothing more, nothing less.
* [test-pg.yml](./.github/workflows/test-pg.yml) – Like `test.yml` but starts a Postgres service before and includes Postgres version as another dimension in the test matrix.
* [test.yml](./.github/workflows/test.yml) – Runs a npm test script (defaulting to `test-ci`) across an OS + Node.js version matrix.

### Utility

* [reusable-npm-run.yml](./.github/workflows/reusable-npm-run.yml) – Runs arbitrary npm scripts as a reusable workflow.
* [sync-reusable.yml](./.github/workflows/sync-reusable.yml) – Reusable flow that runs an npm script and generates a PR whenever the outcome is new.

### Deprecated

* [codeql-analysis.yml](./.github/workflows/codeql-analysis.yml) – Deprecated: Use the built-in default one in GitHub instead.
* [gh-publish.yml](./.github/workflows/gh-publish.yml) – Deprecated: Use the release-please workflows above instead.
* [release-please.yml](./.github/workflows/release-please.yml) – Deprecated: Use the newer release-please workflows above.

## Context

### Environments

These environments are typically expected to be locked down to only be allowed to be used from the main branch.

* `npm` – used by all the `release-please` actions when publishing to NPM. `release-please-4.yml` and `release-please-bot.yml` has a `NODE_TOKEN` secret saved in it while `release-please-oidc.yml` instead has the environment configured in the NPM Trusted Publisher config for the package.
* `pr-bot` – used by `release-please-bot.yml`, `release-please-oidc.yml` and `sync-reusable.yml` to find the `APP_PEM` to use with [`actions/create-github-app-token`](https://github.com/actions/create-github-app-token) to create the token needed to create a PR.

### Scripts

These are the default/required [NPM scripts](https://docs.npmjs.com/cli/using-npm/scripts) in the above workflows:

* `check` – used by `lint.yml` for linting and code quality checks.
* `test` – used by `simple-test.yml` for running all tests (often includes eg. linting).
* `test-ci` – used by `test.yml` and `test-pg.yml` to run all non-linting / non-check tests.
* (arbitrary script) – used by `reusable-npm-run.yml` and `sync-reusable.yml` for custom automation.

## Setup

### Release Please

1. Add the new files:
    - Either by using [a patch](./release-please.patch):
        1. Download it to `~/Downloads/release-please.patch`
        2. In your project, apply the patch: `git apply --reject ~/Downloads/release-please.patch`
        3. Find any `.rej` files and apply their changes manually
    - Or by adding [all the files](#release-please-files) manually
4. Run `npm install -D validate-conventional-commit` to add the dependency of `.husky/commit-msg`
5. Change `{".":"0.0.0"}` in `.github/release-please/manifest.json` to the latest tagged version of the module
6. Configure the repo as [Trusted Publisher](https://docs.npmjs.com/trusted-publishers#for-github-actions) in NPM:
    1. Go to [`npmjs.com/package/installed-check/access`](https://www.npmjs.com/package/installed-check/access)
    2. Set `release-please.yml` as the `Workflow filename`
    3. Set `npm` as the `Environment name`
    1. (Out of scope but still good to check on that page: Check that the module requires 2FA to publish)
7. Go to [`/settings/environments`](https://github.com/voxpelli/node-bunyan-adaptor/settings/environments) in your repository:
    1. Create a `npm` environment and **importantly** set its `Deployment branches and tags` to either `Protected branches only` or to `Selected branch and tags`, adding only `main`
    2. Create a `pr-bot` environment, add the [`APP_PEM`](https://docs.github.com/en/issues/planning-and-tracking-with-projects/automating-your-project/automating-projects-using-actions#example-workflow-authenticating-with-a-github-app) secret for the GitHub App that will create the PR:s. Also, **importantly**, set its `Deployment branches and tags` to either `Protected branches only` or to `Selected branch and tags`, adding only `main`
8. If a pre-existing repository: Ensure that no tag protection is on in [`/settings/rules`](https://github.com/voxpelli/async-htm-to-string/settings/tag_protection) (previously [`/settings/tag_protection`](https://github.com/voxpelli/async-htm-to-string/settings/rules))
9. Go to [`/settings/actions`](https://github.com/voxpelli/node-bunyan-adaptor/settings/actions) in your repository:
    1. Since we use a GitHub app to create PR:s, we can unclick `Allow GitHub Actions to create and approve pull requests `
    2. Then we should set `Allow voxpelli, and select non-voxpelli, actions and reusable workflows ` + `Allow actions created by GitHub` and then add these to the `Allow or block specified actions and reusable workflows` list:
        ```
        googleapis/release-please-action@v4,
        mtfoley/pr-compliance-action@*,
        ```
10. Commit your changes as semantic versioning (eg. `ci: added automatic release flow`) and push it
11. You should now be done :tada: And Release Please will be creating PR:s and triggering releases for you.

### Release Please Config

Optional [additional config](https://github.com/googleapis/release-please/blob/d5f2ca8a2cf32701f1d87a85bbc37493b1db65c2/docs/cli.md#bootstrapping) to be added to `.github/release-please/config.json`:

```json
  "bump-minor-pre-major": true,
  "bump-patch-for-minor-pre-major": true,
```

### Release Please Files

Instead of downloading and applying [the patch](./release-please.patch), you can add these files manually:

<details>
<summary><code>.github/release-please/config.json</code></summary>

```json
{
  "$schema": "https://raw.githubusercontent.com/googleapis/release-please/v16.12.0/schemas/config.json",
  "release-type": "node",
  "include-component-in-tag": false,
  "changelog-sections": [
    { "type": "feat", "section": "🌟 Features", "hidden": false },
    { "type": "fix", "section": "🩹 Fixes", "hidden": false },
    { "type": "docs", "section": "📚 Documentation", "hidden": false },

    { "type": "chore", "section": "🧹 Chores", "hidden": false },
    { "type": "perf", "section": "🧹 Chores", "hidden": false },
    { "type": "refactor", "section": "🧹 Chores", "hidden": false },
    { "type": "test", "section": "🧹 Chores", "hidden": false },

    { "type": "build", "section": "🤖 Automation", "hidden": false },
    { "type": "ci", "section": "🤖 Automation", "hidden": true }
  ],
  "packages": {
    ".": {}
  }
}
```

</details>

<details>
<summary><code>.github/release-please/manifest.json</code></summary>

```json
+{".":"0.0.0"}
```

</details>

<details>
<summary><code>.github/workflows/compliance.yml</code></summary>

```yml
name: Compliance

on:
  pull_request_target:
    types: [opened, edited, reopened]

permissions:
  pull-requests: write

jobs:
  compliance:
    runs-on: ubuntu-latest
    steps:
      - uses: mtfoley/pr-compliance-action@11b664f0fcf2c4ce954f05ccfcaab6e52b529f86
        with:
          body-auto-close: false
          body-regex: '.*'
          ignore-authors: |
            renovate
            renovate[bot]
          ignore-team-members: false
```

</details>

<details>
<summary><code>.github/workflows/release-please.yml</code></summary>

```yml
name: Release Please

on:
  push:
    branches:
      - main

  workflow_dispatch:
    inputs:
      force-release:
        description: 'Force release to npm'
        required: false
        type: boolean

permissions:
  contents: read
  id-token: write

jobs:
  release-please:
    uses: voxpelli/ghatemplates/.github/workflows/release-please-oidc.yml@main
    secrets: inherit
    with:
      app-id: '1082006'
      force-release: ${{ inputs.force-release }}

```

</details>

<details>
<summary><code>.husky/commit-msg</code></summary>

```sh
#!/usr/bin/env sh

npx --no validate-conventional-commit < .git/COMMIT_EDITMSG
```

</details>




## License

Licensed under [MIT](./LICENSE).

## Similar projects

* [`fastify/workflows`](https://github.com/fastify/workflows) – reusable workflows for use in the Fastify organization 
* [`mdn/workflows`](https://github.com/mdn/workflows) – reusable GitHub Actions workflows 
* [`SocketDev/workflows`](https://github.com/SocketDev/workflows) – reusable workflows for use in the SocketDev organization

## See also

* [GitHub reusable workflows announcement blog post](https://github.blog/2021-11-29-github-actions-reusable-workflows-is-generally-available/)
* [GitHub reusable workflows docs](https://docs.github.com/en/actions/learn-github-actions/reusing-workflows)
