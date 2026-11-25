---
name: repo-setup
description: Set up a new repository — package manager, layout, ignores, hooks and checks — so the first commit already builds and runs its checks. Use when the user asks to "start a new repo", "initialize a project", "scaffold a project", "set up a new codebase", or a fresh directory needs tooling before real work. Not for adding features to an existing repo.
---

# repo-setup

Set up a repository so the first commit runs its checks and a new contributor
(human or agent) can build, test and verify without asking how.

The work is decisions, not boilerplate: package manager, which checks run when,
what enters version control. Make each decision once, record it where the tool
reads it, and stop.

## Before creating anything

Ask only what the directory does not answer (a `pyproject.toml` gives the
language, a lockfile the package manager, `CLAUDE.md`/`AGENTS.md` the
conventions):

1. What is the repo for, in one sentence? This shapes the layout and first
   dependency.
2. Language and runtime? If unchosen, propose one for the user to confirm.
   Never guess a framework.
3. Does a repo exist at the path, or does `git init` start it? Never
   initialize over an existing `.git`.

## Package manager

- **Python: uv** — `uv run pytest`, `uv sync`.
- **JavaScript: pnpm** — `pnpm test`, never npm.
- Elsewhere use the ecosystem's tool (cargo, go).

If the user names another manager, theirs wins. Every documented command and
every command you run goes through it; `npm test` in a pnpm repo's README is a
bug.

Pin the runtime the way the ecosystem expects. Ask the user's preference;
otherwise pick a stable or LTS release at minor-version level, preferring what
is installed locally.

## Layout

Use the layout the ecosystem's tools expect, with tests mirroring the package
so each module and its tests are findable from either side:

- Python: `src/<package>/`; the tests of `pipeline/pack.py` live at
  `tests/pipeline/pack/`.
- JavaScript: `src/`, tests beside the code or mirrored under `tests/`.

Create only directories that have an occupant; an empty `scripts/` is a guess.

## Version control

- `git init`; the first commit holds manifests, lockfile, ignores and README,
  and builds.
- Write `.gitignore` before the first install, so caches and build output never
  enter history. Ignore what tools generate, not what they read.
- Local notes, scratchpads and personal TODOs stay uncommitted and out of
  commit messages; put them in `.git/info/exclude`, not `.gitignore`.
- Commit hooks come from prek (`prek.toml`), never hand-written `.git/hooks`
  scripts.

## Checks

Checks that are more than a bare command are scripts in the manifest
(`[project.scripts]` in `pyproject.toml`, `scripts` in `package.json`). Each
runs in seconds on the code at hand. A check in the wrong tier costs the day
(too early) or the review (too late):

1. **Pre-commit** — fixable in seconds, each pointing at one file: formatters,
   linters, import sorting, whitespace, end-of-file newlines, TOML/JSON
   validity, merge-conflict markers. No build.
2. **Pre-push** — needs the whole codebase: test suite, type checker, build.
   On every commit, a ten-commit branch pays ten full runs to learn what the
   last one says.
3. **Manual** — needs credentials, network, a service, or minutes. Runs in CI
   or as a documented command.

prek runs tiers 1 and 2; see [references/checks.md](references/checks.md) for
the wiring, committed-artifact checks, hook limits and hook failures.

## Agent documentation

Write a `CLAUDE.md` or `AGENTS.md` only when asked: what the user names, in
their words. No generic scaffold.

## When to stop

Setup is done when the first commit exists and builds, the hooks run and pass,
and someone can clone, install and run the checks from the README alone. The
first feature, CI config and release process are later work with their own
review.
