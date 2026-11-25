---
name: commit-discipline
description: Make git commits a reviewer can judge from the diff in minutes — atomic commits, specific staging, subject format, amend instead of stack, merge policy. Use whenever committing, writing or rewording commit messages, or preparing a branch for review or merge ("commit", "clean up the branch", "reword a commit", "prepare this for merge").
---

# commit-discipline

A commit is a unit of review. The reviewer — a person now, an archaeologist in
two years, a bisect tonight — sees only the diff and the message. Every rule
here keeps that pair sufficient to judge the change.

## Staging: name the files, never `-a`

- Never `git commit -a`, `git add -A`, `git add .` or anything that stages
  everything. Stage the specific files: `git add src/pack.py tests/pipeline/pack/`.
- Check `git status` and `git diff --staged` before committing. A file the
  subject does not explain either changes the subject or leaves the commit.
- Untracked personal files (scratchpads, local TODO lists) stay unstaged and
  unmentioned; they are not in history, so no message may refer to them.

## Shape: one reviewable step per commit

- Each commit stands alone: reviewable, logical, a step toward the goal — even
  if the branch will be squash-merged later.
- Split where each part can be judged on its own, not by size. New code
  usually belongs with its first use, so the reviewer sees why it exists; an
  abstraction with its own tests can stand alone.
- Mixed commits are the failure mode: a feature plus the formatting it touched
  plus the dependency it needed. Formatting and dependency get their own
  commits; the feature reads alone.
- Commit build and tooling changes with the code that needs them, so every
  commit builds: a rebuilt bundle goes in the commit that changed its source.
- An unmerged branch is an essay stating intent without asides. Rewrite it
  before merge with `rebase -i`, `fixup`, `amend`. Exception: a pushed branch
  with extensive review comments — use judgment, or ask.
- Don't run the test suite during a rebase, neither per stopped commit nor
  after. The rebase verifies itself (clean apply, no conflicts); the final
  tree is tested once, before push. Exception: when a fixup changed code its
  commit's tests pin, run those test files once. After a large rebase, at
  most run the test files the changed commits touch, once, before push.

## Subject

```
area: imperative sentence
```

- The area names the subsystem — `pipeline:`, `viewer:`, `cli:`, `tests:`,
  `docs:` — not the module file. Without an agreed prefix list, use the
  subsystem's name.
- Imperative mood: "Fix bug", "Add feature", "Remove unused import" — never
  "Fixing", "Adds", "Removed".
- Target 50–60 characters; then only a blank line and an optional body.

## Body

- Write one only when the diff cannot speak alone.
- Each bullet is an imperative instruction followed by why: "Move X to Y,
  because Z". Describe the end result, not the path to it.
- Cover only what the diff shows. Never mention intermediate states — entries
  added then removed, failed attempts, things "corrected" — they never existed
  in git and mean nothing to readers of history. Never reference the commit's
  own review conversation.

## Amend, don't stack

Review found a fault in an unmerged commit, whoever wrote it?
`git commit --amend`, or `rebase -i` with `fixup` when it sits deeper.
Stacking a fix makes the reader reconcile two commits to see one change.

## Merging

- Merge a feature branch into `main` with `git merge --no-ff`, never squashed:
  its commits are the review record.
- A one-commit branch may fast-forward.
- Push only when asked.

## Attribution

The `Co-Authored-By` trailer names the model that wrote the commit; a subagent
names its own model. Never name anyone who did not contribute to the diff.

## Repo-specific rules

Before a repo's first commit in a session, read its agent guidance
(`CLAUDE.md`, `AGENTS.md`). Its rules (prefixes, changelog, merge queue)
refine these; on conflict the repo wins.
