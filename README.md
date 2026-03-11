# nbstripout-action-test

A regression test repository that validates the fix for a special character filename bug in the [nbstripout](https://github.com/kynan/nbstripout) GitHub Action.

## The Bug

The official [`kynan/nbstripout@main`](https://github.com/kynan/nbstripout) action passes file paths directly to bash, causing a **syntax error** when notebook filenames contain spaces or parentheses:

```
bash: syntax error near unexpected token `('
```

This affects any filename like `My Analysis (draft).ipynb` or paths with spaces such as `notebooks/Model Training v2.ipynb`.

## The Fix

[`kunalagra/nbstripout@refactor/gh-actions-refactor`](https://github.com/kunalagra/nbstripout/tree/refactor/gh-actions-refactor) resolves this by:

- Using **Python** (via `pathlib.Path.glob`) for file discovery instead of bash glob expansion
- Passing filenames as **null-delimited** entries to `xargs -0`, which safely handles any characters in filenames
- Supporting `**` glob patterns correctly across nested directories

## Test Notebooks

| File | Description |
|------|-------------|
| `clean.ipynb` | Stripped notebook — no outputs |
| `dirty.ipynb` | Has outputs — should fail verification |
| `My Analysis (draft).ipynb` | Stripped — special characters (spaces + parentheses) |
| `notebooks/Model Training v2.ipynb` | Stripped — nested path with spaces |

## Workflows

### [`test-official.yml`](.github/workflows/test-official.yml) — uses `kynan/nbstripout@main`

| Job | Expected Result | Reason |
|-----|----------------|--------|
| `test-clean` | ✅ Pass | Clean notebook, simple filename |
| `test-dirty` | ❌ Fail | Notebook has outputs |
| `test-special-chars` | ❌ Fail | Bash syntax error on `(` in filename |
| `test-all` | ❌ Fail | Bash syntax error + dirty notebook |

### [`test-fixed.yml`](.github/workflows/test-fixed.yml) — uses `kunalagra/nbstripout@refactor/gh-actions-refactor`

| Job | Expected Result | Reason |
|-----|----------------|--------|
| `test-clean` | ✅ Pass | Clean notebook |
| `test-dirty` | ❌ Fail | Notebook has outputs (correct behavior) |
| `test-special-chars` | ✅ Pass | Special characters handled correctly |
| `test-all` | ❌ Fail | Only fails on dirty notebook — no bash errors |

The key contrast is `test-special-chars`: the official action crashes with a bash error, while the fixed fork verifies the notebook successfully.
