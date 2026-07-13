Git PR Workflow - A Definitive Step-by-Step Guide
===================================================

This Git Workflow guide was made with [Neovim's CONTRIBUTING.md](https://github.com/neovim/neovim/blob/master/CONTRIBUTING.md) in mind;
therefore, all examples and general workflow will try to mimic their
documentation.

## Prerequisite

- `git`
- `lazygit`: this guide was made with lazygit in mind, but it can be
  reproduced using normal git commands.
- `GitHub CLI (gh)`: the official gh-cli is preferred, but working from their
  web UI will also work.
- neovim, and [snacks plugin](https://github.com/folke/snacks.nvim) is mentioned but not required, you can still do 
  everything from any terminal, using `gh-cli` or their web UI.

## Guidelines

Every step has **[Policy]** (what the rule is) and **[Technique]** (how to do it
in lazygit/gh-cli).

Steps appended with (IMPORTANT) may need more time to understand or come back
to, and that's okay.

## For Contributors

### Contents
- [Step 1: Create the branch](#step-1-create-the-branch)
- [Step 2: Commit your work (IMPORTANT)](#step-2-commit-your-work-important)
- [Step 3: Open a PR (IMPORTANT)](#step-3-open-a-pr-important)
- [Step 4: Review](#step-4-review)
- [Step 5: Address feedback](#step-5-address-feedback)
- [Step 6: Decide your final commit shape](#step-6-decide-your-final-commit-shape)
- [Step 7: Force-push](#step-7-force-push)

## Step 1: Create the branch

**[Policy]**

Use a feature branch, not `main`; and always create a branch FROM
`main`.

**[Technique]**

Branches panel `3` → `n` → `feat/add-horse` → enter.

→ Continue to [Step 2](#step-2-commit-your-work-important)

## Step 2: Commit your work (IMPORTANT)

**[Policy]:**

Avoid unrelated cosmetic changes in a commit, and the final commit(s) must
eventually satisfy the format below:

```
type(scope): subject

Problem:

Solution:
```

*except* fixup commits (will be explain in later steps) or commits you already
know will be squashed away.

See [Conventional Commits](#references) in References for the general
convention this format is based on.

> [!NOTE]
> What "unrelated cosmetic changes" means: don't let incidental
> formatting/style diffs ride along in a functional commit just because your
> editor or LSP auto-touched something nearby, e.g. adding a feature while your
> editor also auto-sorts unrelated imports or reformats a block you didn't
> mean to touch. If a line's diff has nothing to do with what the commit message
> says it does, it doesn't belong in that commit. Reviewers (and git blame
> later) shouldn't have to filter formatter noise out of the functional diff.

### Commit Rules

There's no rule saying you must write bad messages now and clean up later. This
step covers only the time before you push or open a PR, nobody sees these
commits yet, so how you write them is entirely on you.

**Ask yourself** do you already know what this commit's final message should
say?

- **Yes** → write `type(scope): subject + Problem/Solution` now. No reason to
  write it twice, if this commit later gets squashed away, the message gets
  replaced anyway, so nothing was wasted; if it survives to the final PR, you're
  already done.
- **No, you're not sure this code will even survive** (trying an approach that
  might get thrown out, figuring out the right structure as you go) → commit
  however you want for now, `wip`, `asdf`, whatever. But before [Step 3](#step-3-open-a-pr-important)
  (pushing / opening the PR), this branch must be cleaned up to meet the
  `type(scope): subject + Problem/Solution` format shown above. Nobody outside
  your machine should ever see a commit that doesn't follow it.

**[Technique]** 

1. Files panel `2`
2. `space` to stage.
3. `c` to commit.

→ Continue to [Step 3](#step-3-open-a-pr-important)

## Step 3: Open a PR (IMPORTANT)

**[Policy]**

Send a reasonably complete PR. Mark it Draft while you are not yet requesting
feedback and still working; convert to Ready when it's actually ready for
review.

**Ask yourself** are you confident this is done and correct right now?

- **Yes** code works, tests pass, you've read your own diff, you're not
  expecting to change the approach → open it directly as a normal (non-Draft)
  PR. There's no rule requiring you to sit in Draft first if you're not actually
  iterating anymore.
- **No** you want it pushed for CI feedback, or you want to keep working and
  don't want to accidentally get reviewed/merged mid-change → open as Draft,
  convert to Ready (`gh pr ready`) once you hit the "Yes" case above.

> [!NOTE]
> CI = Continuous Integration, automated checks that run against your code every
> time you push (build, tests, lint, analyzers). It runs whether you're Draft
> or not — pushing to a Draft PR still gets you CI feedback before anyone
> reviews it.

**PR title:** should match what the PR description already says. Same
principle as the export example below, since the `feat` commit already
tells the whole story (Case 1), the PR title is just that commit's
subject: `feat(export): add PDF export option`.

### PR Description

**One commit:** nothing to write. `gh pr create --fill` uses that commit's
subject/body directly as the PR itself. Since they're the same content,
there's nothing separate to produce.

**More than one commit:** how much you write depends on whether one
commit already covers the whole point, or the PR needs its own
explanation the commits don't individually give.

- **Case 1** a commit already tells the story. Example, say a PR has these two
  commits:

  ```
  refactor(export): extract shared formatting logic for CSV and JSON export

  Problem: CSV and JSON export duplicate the same row-formatting logic.

  Solution: extract a shared formatter used by both.
  ```

  ```
  feat(export): add PDF export option

  Problem: users can only export data as CSV or JSON, not PDF.

  Solution: add an "Export as PDF" button using the shared formatter.
  ```

  The `feat` commit already covers why this PR exists; the refactor is
  just what it needed first. So the PR description is that commit's body.

  ```
  Problem: users can only export data as CSV or JSON, not PDF.

  Solution: add an "Export as PDF" button using the shared formatter.
  ```

  Real neovim example of this exact pattern:
  [#40619](https://github.com/neovim/neovim/pull/40619). The maintainer
  didn't summarize or write anything new, just copied the primary commit's
  Problem/Solution in as the PR body directly.

- **Case 2** no single commit covers it. Several related commits building
  toward one larger effort, where the overall point involves things no
  individual commit captures on its own like design tradeoffs, breaking
  changes, open questions. Here you write a separate `Problem` / `Solution` at
  the whole PR level. See [#40621](https://github.com/neovim/neovim/pull/40621) and [#40647](https://github.com/neovim/neovim/pull/40647). Both write substantially more
  at the PR level than any single commit says.

**[Technique]**

`P` to push, then:

- `gh pr create` (interactive prompt for title/body)
- `gh pr create --draft` for Draft, `gh pr ready` later to flip it
- `gh pr create --fill` only for single-commit PRs

→ Continue to [Step 4](#step-4-review)

## Step 4: Review

Did the reviewer request changes?

- **Yes** → go to [Step 5](#step-5-address-feedback)
- **No, approved** → go to [Step 6](#step-6-decide-your-final-commit-shape)

## Step 5: Address feedback

**[Policy]**

This uses a rebase approach: your branch's history is treated as
rewritable, not append-only. Instead of stacking a new commit for every
review comment, you rewrite the existing commit(s) and force-push the
corrected result.

**[Technique]**

1. Make the fix, stage it.
2. `ctrl-f` in the Files panel to jump to the commit it belongs to.
3. `shift+F` to create a `fixup!` commit targeting it.
4. Immediately `shift+S` to autosquash right there, before pushing anything.
5. Force-push the now-clean branch.

The fixup commit never leaves your machine; nobody sees it. Real example:
[#40621](https://github.com/neovim/neovim/pull/40621), every force-push
in its timeline already shows clean commits (`fix(ui2)!:`,
`feat(statusline):`, `docs(ui2):`), and the author even comments "rebased
onto master and added news" directly. No `fixup!` commit ever appears in
the pushed history.

→ Return to [Step 4](#step-4-review)

## Step 6: Decide your final commit shape

**[Policy]**

By this point your commits are already pushed (from [Step 3](#step-3-open-a-pr-important), and likely
several more times from [Step 5](#step-5-address-feedback)'s review rounds), this step rewrites them
locally, then [Step 7](#step-7-force-push) force-pushes the result back.

For most PRs this isn't really a fresh decision, you already knew the shape back
in [Step 2](#step-2-commit-your-work-important) while writing the commits. You can treat this as a final checkpoint: 
confirm what you already knew, or catch cases where the commits had changed,
reviews added fixups that blur a boundary, or a commit you thought was
independent but turned out not to be.

Is this really one meaningful change, or several independently meaningful
ones?

- **Single meaningful commit** → go to [Step 6a](#step-6a-squash-to-one-commit)
- **Multiple meaningful commits** → go to [Step 6b](#step-6b-reword-each-commit)

## Step 6a: Squash to one commit

**[Technique]**

In the Commits panel (`4`):

1. `i` to start interactive rebase from the branch base.
2. Mark every commit but the first as `s` (squash).
3. `m` to continue.
4. Write the combined message when prompted; it should match your PR title
  [Step 3](#step-3-open-a-pr-important). You will notice all the commits you have squashed will end up
  in the base commit's body, so make sure to rewrite the body of the commit to
  `Problem/Solution`.

Real examples that are all single-parent squash commits.
[#40653](https://github.com/neovim/neovim/pull/40653),
[#40601](https://github.com/neovim/neovim/pull/40601),
[#40600](https://github.com/neovim/neovim/pull/40600),
[#40585](https://github.com/neovim/neovim/pull/40585),
[#40607](https://github.com/neovim/neovim/pull/40607)

→ Continue to [Step 7](#step-7-force-push)

## Step 6b: Reword each commit

**[Technique]**

In the Commits panel (`4`):

1. Select a commit.
2. `r` to reword it.
3. Write its own `type(scope): subject + Problem/Solution`.
4. Repeat for each remaining commit.

Real example: [#40619](https://github.com/neovim/neovim/pull/40619), a real 2-parent merge commit preserving both 
original commits.

→ Continue to [Step 7](#step-7-force-push)

## Step 7: Force-push

**[Policy]**

Force-pushing after addressing review or rebasing is expected.

**[Technique]**

`P`, lazygit will prompt since history was rewritten; confirm.

→ Continue to [Step 8](#step-8-merge)

## For Maintainers

### Contents

- [Step 8: Merge](#step-8-merge)

## Step 8: Merge

**[Policy]**

- If the branch ended up as a single commit ([Step 6a](#step-6a-squash-to-one-commit)) →
  Squash Merge. In the snacks.nvim picker: `gh_squash`.
- If the branch ended up as multiple commits ([Step 6b](#step-6b-reword-each-commit)) →
  Merge (real merge commit). In the snacks.nvim picker: `gh_merge`.

## Quick reference

### Contributors

| Step                                        | Situation                                                           |
| ------------------------------------------- | ------------------------------------------------------------------- |
| [1](#step-1-create-the-branch)              | Create feature branch                                               |
| [2](#step-2-commit-your-work-important)     | Commit                                                              |
| [3](#step-3-open-a-pr-important)            | Open PR                                                             |
| [4](#step-4-review)                         | Review decision point                                               |
| [5](#step-5-address-feedback)               | Changes requested → local fixup + immediate autosquash + force-push |
| [6](#step-6-decide-your-final-commit-shape) | Decide commit count                                                 |
| [6a](#step-6a-squash-to-one-commit)         | Squash to one, full format required                                 |
| [6b](#step-6b-reword-each-commit)           | Reword each, full format required                                   |
| [7](#step-7-force-push)                     | Force-push                                                          |

### Maintainers

| Step               | Situation                           |
| ------------------ | ----------------------------------- |
| [8](#step-8-merge) | `gh_squash` (6a) or `gh_merge` (6b) |

## References

### Further Reading

- [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) —
  the general commit message convention the [Step 2](#step-2-commit-your-work-important)
  format is based on.

### Real PRs

Real neovim/neovim PRs used to verify this guide against actual practice.

| PR                                                    | Author      | Commits |
| ----------------------------------------------------- | ----------- | ------- |
| [#40653](https://github.com/neovim/neovim/pull/40653) | echasnovski | 1       |
| [#40619](https://github.com/neovim/neovim/pull/40619) | justinmk    | 2       |
| [#40522](https://github.com/neovim/neovim/pull/40522) | bfredl      | 1       |
| [#38546](https://github.com/neovim/neovim/pull/38546) | dchinmay2   | 2       |
| [#40525](https://github.com/neovim/neovim/pull/40525) | zeertzjq    | 6       |
| [#40601](https://github.com/neovim/neovim/pull/40601) | barrettruth | 1       |
| [#40600](https://github.com/neovim/neovim/pull/40600) | barrettruth | 1       |
| [#40585](https://github.com/neovim/neovim/pull/40585) | prwang      | 1       |
| [#40607](https://github.com/neovim/neovim/pull/40607) | barrettruth | 1       |
| [#40647](https://github.com/neovim/neovim/pull/40647) | barrettruth | 8       |
| [#40621](https://github.com/neovim/neovim/pull/40621) | epithet     | 6       |

## LICENSE

[MIT (c) Calsjunior](LICENSE)
