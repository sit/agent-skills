# Checks

Wiring the SKILL.md tiers into `prek.toml`.

## Example: Python + JavaScript

```toml
default_install_hook_types = ["pre-commit", "pre-push"]
default_stages = ["pre-commit"]

[[repos]]
repo = "builtin"
hooks = [
  { id = "trailing-whitespace" },
  { id = "end-of-file-fixer" },
  { id = "mixed-line-ending" },
  { id = "check-toml" },
]

[[repos]]
repo = "local"

[[repos.hooks]]
id = "ruff-check"
name = "ruff check"
language = "system"
entry = "uv run --frozen ruff check"
types = ["python"]

[[repos.hooks]]
id = "ruff-format"
name = "ruff format"
language = "system"
entry = "uv run --frozen ruff format"
types = ["python"]

[[repos.hooks]]
id = "oxlint"
name = "oxlint"
language = "system"
entry = "pnpm exec oxlint"
types = ["javascript"]
exclude = "^src/.*/dist/"

[[repos.hooks]]
id = "ty"
name = "ty check"
language = "system"
entry = "uv run --frozen ty check"
types = ["python"]
pass_filenames = false
stages = ["pre-push"]

[[repos.hooks]]
id = "pytest"
name = "pytest"
language = "system"
entry = "uv run --frozen pytest -q"
pass_filenames = false
always_run = true
stages = ["pre-push"]
```

- `language = "system"` with `uv run --frozen` / `pnpm exec` runs the tool from
  the project's own lockfile, not a prek-managed copy that drifts.
- `pass_filenames = false` on whole-program checks (`ty`, `pytest`): passing
  changed files changes the error messages, not the coverage.
- `default_stages` keeps every hook at pre-commit unless it opts into
  `stages = ["pre-push"]`, so a new hook is fast until proven slow.
- `exclude` keeps generated output out of lint; linting a bundle reports the
  bundler, not the code.

## Committed artifacts: bundle-fresh

When build output is committed (a bundled viewer, generated docs), a hook
rebuilds it and fails if the committed copy differs:

```toml
[[repos.hooks]]
id = "bundle-fresh"
name = "viewer bundle is fresh"
language = "system"
entry = "sh -c 'pnpm --silent bundle && git diff --exit-code -- src/viewer/dist/viewer.js'"
files = "^(src/viewer/(src|dist)/|package\\.json$|pnpm-lock\\.yaml$)"
pass_filenames = false
```

`files` limits it to inputs of the output, so unrelated commits skip the
rebuild. A commit that changes a source carries the rebuilt artifact.

## Never in a hook

- Anything slower than the review it protects: ten seconds per commit costs
  minutes a day; ten minutes costs the day.
- Anything needing network, credentials or a service: hooks run offline and in
  worktrees, and a check that cannot reach its backend cannot fail closed.
- Edits beyond the checked files or beyond formatting: that is an unreviewed
  rewrite.

## When a hook fails

Don't run hooks by hand to check work; they run at commit and push. If a hook
reformats, rerun the commit — the output is the fix. If it reports an error,
fix the error, not the hook.
