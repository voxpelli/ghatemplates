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
