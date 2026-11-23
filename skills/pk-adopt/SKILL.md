---
name: pk-adopt
description: Bring an existing repo onto project-kit in a worktree - render the kit over it with the repo's own answers, keep its version and IDs, move its CLAUDE.md knowledge into AGENTS.md and DESIGN.md, drop starter files it never had, and hand back a reviewed diff. Only when the user types /pk-adopt.
argument-hint: [path to repo, default the current one]
disable-model-invocation: true
---

# /pk-adopt

Arguments: `$ARGUMENTS`, the repo to adopt (default: the current repo).

The kit lives at `${PROJECT_KIT:-$HOME/Code/project-kit}`. Read its `CONVENTIONS.md` and the "Adopting an existing project" section of its README before starting.

Adopting rewrites a lot of files. Work in a worktree on a branch, and never push or merge without a yes.

## 1. Read the repo

Record the answers the kit would have asked for, taken from what the repo already says:

| Answer | Where it comes from |
|---|---|
| `name` | the folder or GitHub repo name |
| `description` | `gh repo view --json description` |
| `display_name` | `bundler.toml` `name`, the app's display name, or the README title |
| `stack` | `bundler.toml` + nice-plug means `vst-plugin`; `Package.swift` with an app target means `swift-app`; otherwise `other` |
| `app_id` | the shipped ID: `CLAP_ID` in `plugin/src/lib.rs`, or the bundle ID |
| `vst3_class_id` | `VST3_CLASS_ID` in `plugin/src/lib.rs` |
| `public` | `gh repo view --json visibility` |
| `ships` | what `release.yml` does today |
| `design_doc`, `voice_doc` | whether the repo has design or voice guidance (in its own file, or inside CLAUDE.md) |
| `year` | the year of the first commit |

**The IDs must be the shipped ones, never the kit's defaults.** Hosts and macOS find the project by them.

Show the table to the user and get a yes before rendering.

## 2. Render in a worktree

```sh
git -C <repo> status --short     # stop if it isn't clean
git -C <repo> worktree add ../<name>-adopt -b chore/adopt-project-kit
copier copy --defaults --overwrite --vcs-ref HEAD -d name=... -d app_id=... (every answer) <kit> ../<name>-adopt
```

## 3. Repair what the first copy gets wrong

Work through `git status` and `git diff` in the worktree:

1. **Version.** Put the source of truth back to the version on `HEAD`, e.g. `Cargo.toml`'s `version` (then `cargo update --workspace`). The kit starts new projects at 0.0.0.
2. **Starter files the repo never had.** Delete untracked files that the kit lists under `_skip_if_exists` in `copier.yml`. For example, the template's golden render, and `ui/mock.json` with `plugin/tests/mock.rs` when the repo's UI doesn't use a mock.
3. **CLAUDE.md knowledge.** The old CLAUDE.md (`git show HEAD:CLAUDE.md`) is now `@AGENTS.md`. Move everything specific to this project into the rendered AGENTS.md:
   - layout → Layout
   - lessons and gotchas → Invariants and footguns, under the stack's own sections
   - sound, design and controls notes → DESIGN.md's "This plugin" section, or Principles where they differ.
   Drop what the kit's text already says, and old rules the conventions replace (bump-after-every-change, plain commit messages, `bump.sh`). Nothing project-specific may be lost. List anything you left out and why.
4. **Protected files that need a kit feature.** Make the smallest change in files the kit won't overwrite. For a plugin's dev versions, `PAGE.version` in `lib.rs` becomes the `option_env!("PLUGIN_VERSION")` match from the kit.
5. **Scripts the kit replaces.** Remove `scripts/bump.sh` and anything else the kit's `scripts/release` supersedes.
6. **CHANGELOG.** The kit creates an empty one. If the repo has tags, offer to seed one version section per tag from its GitHub release notes (`gh release view vX.Y.Z --json body`), rewritten in user-facing words.

## 4. Check it

Run the stack's checks, the same ones `pk-new-project` lists, plus `scripts/release --dry-run`.

For a Rust repo, `cargo fmt --all` will reflow hand-wrapped code under the kit's `rustfmt.toml`. Do that as its own commit after the adoption commit, then add that commit's hash to `.git-blame-ignore-revs` (with a comment saying what it was) in a third commit.

## 5. Commit and hand over

Commits on `chore/adopt-project-kit`:
1. `chore: adopt project-kit`. The body lists what was carried over from CLAUDE.md, what was dropped, and every deliberate difference from the kit.
2. `style: format with the kit's rustfmt settings` (if needed).
3. `chore: ignore the formatting commit in blame` (if needed).

Show the user the diff stat and the summary, then ask whether to:
- fast-forward `main` and push;
- leave the branch for them;
- or throw it away (`git worktree remove`, `git branch -D`).

After merging, offer to normalize the GitHub metadata. This is outward-facing, so ask first:
- `gh repo edit --add-topic vibe-coded`, removing `vibecode` and `vibecoded`;
- set the description to the README one-liner.
