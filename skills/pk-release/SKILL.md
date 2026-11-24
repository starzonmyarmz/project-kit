---
name: pk-release
description: Release a project made from project-kit - preview the bump that scripts/release computes from the commits, get a yes, run it, and watch the release workflow until the GitHub release is up. Only when the user types /pk-release.
argument-hint: [patch|minor|major|X.Y.Z]
disable-model-invocation: true
---

# /pk-release

Arguments: `$ARGUMENTS`, an optional override for the bump.

Releasing is outward-facing: it pushes a tag, publishes a release and may update the Homebrew tap or itch.io. Nothing in this skill runs without the user's yes at step 2.

## 1. Preview

Run `scripts/release $ARGUMENTS --dry-run`.

- If it says the changelog is empty, run `/pk-changelog` first, then come back.
- If it says there's nothing to release, say so and stop, unless the user passed an explicit bump.
- Any other preflight error (not on main, dirty tree, behind origin): report it and stop. Don't fix it on your own.

## 2. Confirm

Ask with AskUserQuestion, showing:
- the version change, e.g. 0.3.1 → 0.4.0;
- why: the commits that decided the bump;
- the `[Unreleased]` section that becomes the release notes.

Options:
- **Release vX.Y.Z** (recommended)
- **Pick a different bump** (then ask which)
- **Not now**

Before 1.0, mention that a breaking change only bumps the minor version, and that going to 1.0.0 is the user's call: pass `1.0.0`.

## 3. Release

Run `scripts/release -y $ARGUMENTS`, adding the bump the user picked if it differs.

## 4. Watch it

```sh
gh run list --workflow release.yml --limit 1
gh run watch <id> --exit-status
```

When it passes:
- show the release URL (`gh release view vX.Y.Z --json url`);
- list the assets;
- say what the publish jobs updated (itch).

If the project ships to Homebrew (`.copier-answers.yml` lists `homebrew-cask` or `homebrew-formula`, or there's a `packaging/*.rb`), update the tap now rather than waiting for its hourly run. GitHub turns off a scheduled workflow after 60 days without activity in the repo, so turn it on first:

```sh
gh workflow enable update.yml -R starzonmyarmz/homebrew-tap
gh workflow run update.yml -R starzonmyarmz/homebrew-tap
gh run list --workflow update.yml -R starzonmyarmz/homebrew-tap --limit 1
gh run watch <id> -R starzonmyarmz/homebrew-tap --exit-status
```

Then confirm the tap's commit says `chore(<name>): update to X.Y.Z`. If the project isn't in the tap's `projects` file, say so and offer to add it.

When it fails, show the failing step (`gh run view <id> --log-failed`) and propose a fix. Don't delete or move the tag without asking: people may already have the release.
