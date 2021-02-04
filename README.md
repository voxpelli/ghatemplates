# Templates for `ghat`

My personal GitHub Actions workflow templates for [`ghat`](https://github.com/fregante/ghat)

## Example Usage

```bash
npm --save-dev ghat npm-run-all
```

```json
"sync-gh-actions:lint": "ghat voxpelli/ghatemplates/lint",
"sync-gh-actions:nodejs": "ghat voxpelli/ghatemplates/nodejs --set jobs.test.strategy.matrix.node_version=\\[12,14,15\\]",
"sync-gh-actions": "run-p sync-gh-actions:*",
```

```bash
npm run sync-gh-actions
```

## Available actions

### Static code analysis

* `lint/lint.yml` – installs npm dependencies and then runs the `check` npm script, which is expected to be available. Has no test matrix set up as the static code analysis is expected to be the same no matter the Node.js version or OS

### Node.js CI

* `nodejs/nodejs.yml` – basic one which installs npm dependencies and then runs the `test-ci` npm script across a matrix composed of Linux + Windows and a couple of relevant Node.js versions. The matrix is expected to be overridden with use specific settings.
* `nodejs-pg/nodejs.yml` – similar to `nodejs/nodejs.yml`, but also sets up a Postgres database and sets a `DATABASE_URL` with a connection URI as eg. the port may differ and thus can't be hard coded but also since the username and password in CI is probably different from what one uses eg. locally. By default has a more limited test matrix than `nodejs/nodejs.yml` as it has an extra dimension which could mean a large amount of combinations.
* `nodejs-coveralls/nodejs.yml` – _to be extracted_ – similar to `nodejs/nodejs.yml`, but uploads test coverage data to Coveralls