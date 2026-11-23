---
name: pk-doctor
description: Report how a repo has drifted from project-kit's conventions - kit version, commits, changelog, release tooling, CI, docs and GitHub metadata - without changing anything. Use when the user types /pk-doctor or asks whether a project follows the conventions.
argument-hint: [path to repo, default the current one]
---

# /pk-doctor

Arguments: `$ARGUMENTS`, the repo to check (default: the current repo).

Read the kit's `CONVENTIONS.md` (`${PROJECT_KIT:-$HOME/Code/project-kit}`) first: it's the standard to check against. **Change nothing**: no edits, no commits, no `gh repo edit`. Only report.

## Checks

Run each with a simple command. Mark it ✅, ⚠️ (drifted, still works) or ❌ (broken or missing).

1. **Kit**
   - `.copier-answers.yml` exists. Without it, the only fix is `/pk-adopt`.
   - Its `_commit` vs the kit's `HEAD`. If they differ, say how many kit commits behind it is, and list their subjects (`git -C <kit> log --oneline <_commit>..HEAD`).
2. **Agent docs**
   - `CLAUDE.md` is `@AGENTS.md`.
   - AGENTS.md has these sections: Commands, Layout, Invariants and footguns, Conventions, Releasing.
3. **Commits.** The share of the last 30 subjects that are Conventional Commits with Title-Case scopes (`git log -30 --format=%s`). List the ones that aren't.
4. **Changelog**
   - `CHANGELOG.md` exists with an `## [Unreleased]` section.
   - Every `feat`, `fix` or `perf` commit since the last tag is covered by it.
5. **Versions and tags**
   - The last `v*` tag matches the source of truth for the stack (CONVENTIONS' table).
   - No tags without the `v`.
   - `scripts/release --dry-run` runs.
6. **CI.** These workflows exist: `check.yml`, `release.yml` and `changelog.yml`. Check the latest runs on `main` with `gh run list --branch main --limit 5`.
7. **Basics**
   - LICENSE is MIT with the right name.
   - `.gitignore` covers the baseline in CONVENTIONS.
   - `.worktreeinclude` exists where the stack needs one.
   - CODEOWNERS is present.
8. **GitHub** (`gh repo view --json description,repositoryTopics,homepageUrl,visibility`)
   - The description matches the README one-liner.
   - Topics include `vibe-coded`, not `vibecode` or `vibecoded`, plus the stack's topics from `pk-new-project`.
   - Secrets for what the project ships are set (`gh secret list`: names only).
9. **IDs.** The app ID follows `com.iamdanielmarino.<name>`. A project that shipped with another ID keeps it: report it as ⚠️ "kept on purpose", never as ❌.

## Report

Show one table: check, status, what's off. Under it:
- the fixes, cheapest first;
- whether each is a `copier update`, `/pk-adopt`, `/pk-changelog`, a one-line edit, or a GitHub setting;
- which ones are outward-facing.

Offer to make the fixes, but leave them for the user to pick.
