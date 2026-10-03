---
name: push
description: Commit the working tree and push it to main, in this repo's commit-message style. Use when asked to commit, push, or "commit and push" changes here.
---

# Commit and push

This repo has no branches and no PRs. Work is committed on `main` and pushed
straight to `origin`. Do not create a branch, do not open a PR, do not ask
whether to.

## Before committing

1. `git status` and read the **whole** diff -- `git diff` and `git diff --cached`,
   plus `cat` of any new file, since a new file shows as one undifferentiated
   block. You cannot write the message below from a summary -- nor decide how
   many commits this is, which is the next question and is answered under
   **Scope** below, before anything is staged.
2. Check that it runs. The tools here are nix-shell scripts in `scripts/`, so
   at minimum every changed Python script must parse:

   ```
   nix-shell -p python3 --pure --run 'python3 -c "import ast; ast.parse(open(\"scripts/<name>\").read())"'
   ```

   Then run whatever the change actually touches, as far as it goes without
   hardware or side effects: `--help`/usage output always, `granola models`
   and `granola-orders summary` (read-only against Shopify) where changed.
   Flashing, nuking and anything needing a controller on the bus is left to
   Mike -- say it was not exercised.
3. Report failures. A script that does not run is not a commit that goes in
   quietly with a note attached.

## The commit message

Match what is already in `git log`. The house style is a subject line, a blank
line, and then **prose paragraphs that explain why**, not a bullet list of what
changed. The diff already says what changed.

- Subject: lowercase after any `component:` prefix, no trailing period, under
  ~72 characters. `mod+/: one list of keybindings, generating both halves`,
  `pve-console: SPICE consoles outside the browser`.
- Body wrapped at 76ish columns.
- Write the reasoning that is not recoverable from the code: what was tried and
  rejected and why, which constraint forced a choice, what will break later and
  what the reader should do when it does. If a comment in the diff already
  argues a point at length, the message says the same thing shorter -- it does
  not link to it.
- Name the specific failure that motivated the change where there was one.
  "RFP masks the modifier state on key events, so noVNC never learns Ctrl was
  held" beats "fixes keyboard issues".
- No "This commit...", no "Changes include:", no emoji, no Generated-with line.
- End with:

  ```
  Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>
  ```

  Pass the message via `git commit -F -` and a heredoc; `-m` mangles the
  wrapping.

Small mechanical changes get a one-line subject and no body -- `terminal
colors, add local games share`. Reserve the long form for changes that made a
decision.

## The author

Commits go out under the GitHub noreply address, never a personal email --
these repos are public, or meant to be:

```
Mike Switzer <13549851+michaelswitzer@users.noreply.github.com>
```

The global git config still says otherwise, so do not trust it. Before the
first commit, check `git config user.email`; if it is not the address above,
set it for this repo:

```
git config user.name "Mike Switzer"
git config user.email 13549851+michaelswitzer@users.noreply.github.com
```

and confirm with `git log -1 --format='%an <%ae>'` after committing.

## Pushing

```
git push origin main
```

If it is rejected as non-fast-forward, `git pull --rebase` and push again;
report the rebase if it was not clean. Never force-push.

## Scope: one commit per distinct change

A dirty tree is not automatically one commit. Before staging anything, work out
how many separate things are in it, and commit them separately -- each with its
own message, in an order where every commit stands on its own.

Split when the working tree holds more than one of these:

- two features that could have been built in either order, or a month apart
- a fix and the unrelated feature that happened to be in progress alongside it
- a refactor and a behaviour change riding on top of it -- these especially,
  since a large mechanical diff will otherwise bury the two lines that actually
  changed what the machine does
- tooling, or docs that are not about the change beside them

Keep together what one message would have to explain as a whole: a module, the
hosts that import it, the README section describing it and the options it
exposes are one commit, however many files that is. File count is not the test;
whether the reasoning is one argument or two is.

The test, per commit: could it be reverted on its own without taking something
unrelated with it, and does its subject name one thing rather than joining two
with "and"? A subject reaching for an "and" -- or a body with a second "also"
paragraph that owes nothing to the first -- is usually two commits.

Stage precisely when splitting. `git add <path>` per commit where the split
falls on file boundaries, `git add -p` where two changes share a file, and
`git diff --cached` before each commit to confirm what is in it. Build between
commits wherever an intermediate one could plausibly be broken -- this is a
script, and a commit that does not run is one nobody can bisect through.

Do not leave a dirty tree behind without saying what is still in it and why.
