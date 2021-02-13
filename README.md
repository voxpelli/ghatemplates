# Templates for `ghat`

My personal GitHub Actions workflow templates for [`ghat`](https://github.com/fregante/ghat)

## Example Usage

### Initial setup

```bash
npx ghat voxpelli/ghatemplates/lint
npx ghat voxpelli/ghatemplates/nodejs --set jobs.test.strategy.matrix.node_version=\[12,14,15\]
```

### Update to latest version

```bash
npx ghat
```

Or _preferably_ document your use of `ghat`, and which version, by adding it as a dev dependency, then also add a dedicated npm script to indicate how to use it:

```bash
npm install --dev ghat
```

In `"scripts"` in `package.json`:

```json
"sync-gh-actions": "ghat",
```

Then sync using:

```bash
npm run sync-gh-actions
```

## Available actions

### Static code analysis

* `lint/lint.yml` – _uses `yarn`_ – installs npm dependencies and then runs the `check` npm script, which is expected to be available.  Has a basic matrix set, testing on a single Node.js version on a single OS, but can be overridden.

### Node.js CI

* `nodejs/nodejs.yml` – _uses `yarn`_ – basic one which installs npm dependencies and then runs the `test-ci` npm script across a matrix composed of Linux + Windows and a couple of relevant Node.js versions. The matrix is expected to be overridden with use specific settings.
* `nodejs-pg/nodejs.yml` – _uses `yarn`_ – similar to `nodejs/nodejs.yml`, but also sets up a Postgres database and sets a `DATABASE_URL` with a connection URI as eg. the port may differ and thus can't be hard coded but also since the username and password in CI is probably different from what one uses eg. locally. By default has a more limited test matrix than `nodejs/nodejs.yml` as it has an extra dimension which could mean a large amount of combinations.
* `nodejs-coveralls/nodejs.yml` – _uses `yarn`_ – _to be extracted_ – similar to `nodejs/nodejs.yml`, but uploads test coverage data to Coveralls

### Automatic Publishing

* `gh-publish/gh-publish.yml` – _uses `npm`_ – contains two steps: First step installs npm dependencies, runs the `build` npm script if available and finally runs the `test` npm script. Second step publishes the module to GitHub Package Registry.

### Basic ones

* `npm-test/npm-test.yml` – _uses `npm`_ – installs npm dependencies and then runs the `build` npm script if available and then the `test` npm script. Has a basic matrix set, testing on a single Node.js version on a single OS, but can be overridden.
