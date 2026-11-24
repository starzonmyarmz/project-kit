# Conventions

How every project starts and ships. Templates, skills and `/pk-doctor` follow this file. When the workflow changes, change this file first, then the kit, then run `copier update` in the projects that should catch up.

Each rule says where it came from so it can be judged later. A rule marked **(review)** was picked during the first draft, not proven in a project yet.

## Repo basics

- Default branch `main`. Solo work pushes straight to `main`. PRs are optional, used when a change wants a review or CI before landing.
- `LICENSE` is MIT, `Copyright (c) <year> Daniel Marino`, for every repo, public or private.
- `.worktreeinclude` lists whatever a fresh worktree needs copied (e.g. `node_modules`, `.venv`).
- `.gitignore` always covers `.DS_Store`, `.playwright-mcp/`, `.claude/settings.local.json`, `.env`, build output and screenshot scratch dirs.
- `.github/CODEOWNERS` is `* @starzonmyarmz`.
- Public repos have `.github/FUNDING.yml` with `github: [starzonmyarmz]` and `ko_fi: danielmarino`, for the Sponsor button, and the repo's Sponsorships setting on (GitHub hides the button without it). A project can opt out.
- Public repos keep real names, logins, URLs and work data out of fixtures, docs, changelogs and commits. Use stand-ins (`alice`, `/Users/alice`, `example.com`). (keepup)

## GitHub metadata

Set by the wizard with `gh repo create` and `gh repo edit`, and checked by `/pk-doctor`.

- **Description:** one sentence, the same as the README one-liner. No trailing period unless it's two sentences.
- **Topics:**
  - Always `vibe-coded`: it replaces `vibecode` and `vibecoded`.
  - The language or framework: `rust`, `swift`, `swiftui`, `typescript`, `python`, `preact`, `bevy-engine`.
  - The platform or kind: `macos`, `menu-bar-app`, `audio-plugin`, `vst3`, `clap`, `alfred-workflow`, `game`, `cli`.
  - The distribution: `homebrew-cask` or `homebrew-formula`, `itch-io`.
- **Homepage:** the Pages, docs or itch URL when there is one.
- **Wiki and Projects:** off. Docs live in the repo.
- **Automatically delete head branches:** on.

## Commits

Conventional Commits with Title-Case scopes. (keepup, roxel, hogs, riff-raff)

```
type(Scope)!: imperative subject, about 50 characters

Body in prose, explaining why. The rationale goes here, not in code comments.
```

- **Types:** `feat`, `fix`, `perf`, `refactor`, `revert`, `style`, `test`, `docs`, `build`, `ci`, `chore`. A project may add its own types in AGENTS.md (riff-raff's `balance`).
- **Scope:** an area or feature name in Title Case (`Sessions`, `Engine`, `UI`). It's optional when a change has no clear area.
- **Reserved scopes:**
  - `chore(Release)`: the release commit.
  - `docs(Spec)`: spec and acceptance checks, written before the code.
  - `docs(Backlog)`: ideas.
- One logical change per commit, following the `dm-organize-commits` skill.
- Never reference PR or issue numbers.

## Versions

SemVer, with one source of truth per stack:

| Stack | Where the version lives |
|---|---|
| Rust | `[workspace.package] version` in the root `Cargo.toml` |
| Swift (SwiftPM) | nowhere in the repo: it's baked into `Info.plist` from the tag at build time (hogs) |
| Swift (XcodeGen) | `MARKETING_VERSION` in `project.yml`; the build number is `git rev-list --count HEAD` (keepup) |
| JS/TS | `package.json`; any Tauri `Cargo.toml` is synced by the release script |
| Python | `pyproject.toml` |
| Alfred | nowhere in the repo: it's baked into `info.plist` from the tag at build time (the Alfred workflows) |

Tags are always `v` + the exact version (`v1.4.0`). A tag without the `v` never triggers CI. A version with `-` (e.g. `1.0.0-beta.1`) is published as a pre-release.

Versions change only at release. Builds installed locally between releases call themselves `X.Y.Z-dev+<short sha>`, and a Mac app installed from a clean checkout of a release tag calls itself `X.Y.Z`: a plugin panel or About window shows that, so you can tell which build is installed. Release builds show `X.Y.Z`. Mac apps leave the hash out: their build number is the commit count (`git rev-list --count HEAD`), which already says which commit it is, so About reads `Version 1.2.0-dev (412)` between releases and `Version 1.2.0 (412)` for a release. (keepup) This replaces the plugins' bump-after-every-change rule.

Spec-driven projects (keepup) number roadmap steps on their own (`v0.N` in `docs/spec/roadmap/`). Step numbers aren't versions, and finishing a step doesn't release anything.

### Deciding the bump

Commits since the last tag decide it; a human triggers the release.

| Commits since the last tag | Bump at 1.0+ | Bump before 1.0 |
|---|---|---|
| any `!` or `BREAKING CHANGE:` footer | major | minor |
| any `feat` | minor | minor |
| any `fix`, `perf` or `revert` | patch | patch |
| only other types | no release | no release |

- Before 1.0, a breaking change bumps the minor version. Going to 1.0.0 is a deliberate call, never computed.
- "Breaking" means whatever the project's users depend on. For plugins, that's renamed parameter IDs or presets that won't load. For apps, that's saved files or preferences that no longer open. For CLIs, that's flags or output. Each project's AGENTS.md says what counts. (patina)
- `/pk-release` computes the bump, shows the commits behind it, and waits for a yes. `patch`, `minor`, `major` or an exact version overrides it.
- Agents never release, tag, or edit a tap formula on their own. Releasing is outward-facing and only happens when asked. (hum2midi)

## Changelog

`CHANGELOG.md`, [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), hand-written for users, in every project, small tools included. (roxel, riff-raff, keepup)

```markdown
# Changelog

What changed for people using <Name>, newest first. Code-only work stays in the git history.

## [Unreleased]

### Added
- Plain-language line about what someone can now do or see.

## [1.3.0] - 2026-11-23
```

- Sections are `Added`, `Changed`, `Fixed` and `Removed`. A project may add its own, such as riff-raff's `Balance`.
- **Write the entry in the same commit as the change.** Any `feat`, `fix` or `perf` commit that someone using the project would notice adds a line under `[Unreleased]`. Refactors, tests, docs and CI don't.
- CI checks this on PRs: a `feat`/`fix`/`perf` commit with no CHANGELOG change fails. To opt out, add the `no-changelog` label or put `[skip changelog]` in the subject. `scripts/release` runs the same check locally, because most work goes straight to `main`.
- At release time, `[Unreleased]` becomes `## [X.Y.Z] - YYYY-MM-DD` and a fresh empty `[Unreleased]` is added. Those bullets become the GitHub release body.
- Public repos: read the new section against the scrubbing rule above before committing.

## Release flow

The same contract for every stack, with a different implementation per stack:

```
scripts/release [auto|patch|minor|major|X.Y.Z] [--dry-run] [-y]
```

1. Preflight: on `main`, clean tree, local `main` equal to `origin/main`, the tag doesn't exist yet, and `[Unreleased]` isn't empty if there are `feat`/`fix`/`perf` commits.
2. Work out the version, write it to the source of truth, and finalize the changelog.
3. Commit `chore(Release): X.Y.Z`, create an annotated tag `vX.Y.Z`, and push `main` and the tag.
4. The tag-triggered `release.yml` takes over:
   - Check the tag equals the version.
   - Build and test.
   - Package the artifacts as `<name>-X.Y.Z.zip`, with the repo's lowercase name, never the display name. Builds for several platforms add the platform: `<name>-X.Y.Z-macos.zip`, `-windows.zip`, `-setup.exe`, `-linux.tar.gz`.
   - `gh release create` titled `<Display Name> X.Y.Z`, with the changelog section as notes, plus `SHA256SUMS`.
   - Then the publish jobs (below).

Publish jobs, depending on how the project ships:

| Target | How | Secret or variable |
|---|---|---|
| Homebrew tap | No job. `packaging/<name>.rb` is a template (`@VERSION@`, `@SHA256:<file>@`) listed in `starzonmyarmz/homebrew-tap`'s `projects` file. The tap's Update workflow renders it from the latest release's `SHA256SUMS` hourly, or when `/pk-release` starts it | none |
| itch.io | Push with butler and `--userversion X.Y.Z` | `BUTLER_API_KEY`, variable `ITCH_TARGET` |
| GitHub Pages | Separate `pages.yml` on push to `main` (sites and docs aren't versioned) | none |
| Nowhere (`local`, Mac apps only) | No build on GitHub at all, not even a check workflow: the app is built and installed from source with `scripts/install.sh`. The release is notes only (keepup) | none |

macOS builds are self-signed, not notarized:
- One long-lived certificate per app, so Launch at Login and other permissions survive updates. (hogs)
- Its secrets are `SIGNING_CERT_P12` and `SIGNING_CERT_PASSWORD`. A release fails without them rather than signing ad hoc.
- Casks remove the quarantine flag in a `postflight`.
- The README and `INSTALL.txt` explain the `xattr -dr com.apple.quarantine` workaround.

Every app's ID is `com.iamdanielmarino.<name>`, whatever kind of app it is: the bundle ID for Mac apps, the CLAP ID for plugins. A VST3 class ID is 16 raw bytes rather than a reverse-DNS name, so plugins use `<TypeName>DanielMarino`, cut to 16 bytes or padded with `_` (`SputnikDanielMar`). Plugins name their vendor `Daniel Marino`.

The kit asks for these IDs with those defaults and saves them in `.copier-answers.yml`, so a later change to the kit can't change them by accident. Projects from before this rule move to it when they're adopted, even though it breaks things: hosts see a plugin with a new ID as a different plugin, so saved projects need it added again, and a Mac app with a new bundle ID loses its permissions and preferences. If a release was already out, say so in the changelog. Before a first release there's no one to tell, so skip it.

## CI

`check.yml` runs on push to `main` and on PRs:
- `concurrency` cancels earlier runs on the same ref.
- Formatting and lint are strict errors: `cargo fmt --check` + `clippy -D warnings`, `swift format lint`, prettier + stylelint, `ruff`.
- Tests run.
- A build checks that the artifact can be produced (e.g. clap-validator for plugins).
- The changelog gate runs on PRs only.

Optional workflows:
- `screenshot.yml`: retakes README screenshots on UI changes and commits them as `docs: retake the screenshot`. Path-filtered and skipped in template repos.
- `sync-*.yml`: daily cron + `workflow_dispatch`. Regenerates data, force-pushes a fixed `sync-<thing>` branch, then opens one PR or updates it. (the Alfred workflows)

Every workflow pins its toolchain (`rust-toolchain.toml`, `.node-version`, `.python-version`), so local runs match CI.

## Tests

- Pure logic is kept apart from the UI or host shell, and that's what gets tested:
  - Rust: the engine vs. plugin plumbing.
  - Swift: a `*Core` package vs. the app.
  - Web: game rules vs. components.
- Tests ship in the same commit as the feature.
- Snapshot and golden tests are rewritten with `BLESS=1`. A person checks the result (listens, looks) before it's committed, and it's committed together with the change that caused it. (plugins)
- Randomness is seeded so goldens and simulations are deterministic.
- Optional git hooks in `.githooks/` mirror CI and are turned on with `git config core.hooksPath .githooks`. (roxel)

## Docs

### README

The first lines are always the same. Sections after that are used only when they apply, in this order:

1. `# Name`, with the app icon in the heading when there is one: `# <img src="…" width="40" height="40" alt="" align="top"> Name`. Without `align="top"`, GitHub sits the icon on the baseline and it rides above the title.
2. The one-liner, word for word the GitHub description.
3. The hero screenshot (`docs/<name>.png` or `.github/assets/`).
4. A short pitch: what it is and who it's for.
5. **Features** or **How it works** / **Controls** / **Settings** (a table).
6. **Install**: Homebrew first, then Releases/itch. Include the Gatekeeper note when the build is self-signed.
7. **Develop**: the commands for running, testing and building.
8. **Releases**: the bump table, what CI does, and the secrets it needs.
9. **Credits** / **License**.

- No badges, except a single CI badge on public OSS projects.
- Screenshots are made by a script or the screenshot skill, never by hand. That way they can be retaken.

### Agent instructions

- `AGENTS.md` holds the content and `CLAUDE.md` is a one-liner, `@AGENTS.md`, so opencode and Claude read the same file. (riff-raff, hum2midi)
- AGENTS.md sections, in order:
  1. One paragraph on what the project is.
  2. Read-before pointers (e.g. DESIGN.md before styles, VOICE.md before copy).
  3. Commands.
  4. Layout / architecture map.
  5. Invariants and footguns: what was learned the hard way.
  6. Conventions specific to this project.
  7. Releasing.
- It only holds what's specific to the project. Rules that apply everywhere live in the global `~/.claude/CLAUDE.md` and the kit's skills.
- Big projects add a `CLAUDE.md` in a subdirectory for its own area, loaded only when work happens there. (roxel)

### Optional design docs

The wizard offers these per project:

| File | When | Shape |
|---|---|---|
| `DESIGN.md` | anything with a UI | the look in one line, tokens (color/type/space), components, patterns, "Adding something new" checklist, **Known drift** |
| `VOICE.md` | anything with copy or personality | the voice in one line, do/don't, before-and-after, off-limits, **Known drift** |
| `docs/spec/` | multi-step builds where Claude does most of the work | keepup's spec layout: numbered spec files, `roadmap.md`, `roadmap/v0.N.md` with acceptance checks, human checkpoint, "Not in v0.N" |
| `design-doc.md` | games | riff-raff's 17-section game design doc |

### Public OSS extras

Only for public projects that want outside contributors (roxel):
- `CONTRIBUTING.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md`.
- Issue forms with blank issues off.
- A PR template.
- An mdBook site in `docs/`, deployed to Pages.

## Skills

- **Kit** skills live in `project-kit/skills/`, so they change together with the templates. They're linked into `~/dotfiles/claude/.claude/skills`, the way `til` is.
- **Project** skills ship in the template only when they're stack-specific.
- Kit skills are prefixed `pk-` (`dm-` is kept for dotfiles skills), and run only when invoked by name.

| Skill | Where | What |
|---|---|---|
| `pk-new-project` | kit | the wizard |
| `pk-adopt` | kit | brings an existing repo onto the kit in a worktree, following the README's adoption steps |
| `pk-release` | kit | runs `scripts/release`, shows the computed bump, waits for a yes |
| `pk-changelog` | kit | drafts `[Unreleased]` lines from commits since the tag in user-facing words |
| `pk-doctor` | kit | reports drift from this file, changes nothing |
| `screenshot`, `design-kickoff` | vst-plugin | (generalize when a second stack needs them) |
| `bless-goldens` | vst-plugin | |
| `playtest`, `balance`, `devlog` | game | |

Skills that run multi-step rituals use `disable-model-invocation: true`, so they only run when invoked by name.

## Retrofitting existing projects

Done project by project with `copier update` (or `copier copy` over the repo the first time), reviewing the diff before committing.

- **Plugins** (halfwit, ghosted, altitude, hahaha, sputnik, patina):
  - Drop the bump-after-every-change rule; `scripts/install.sh` builds `-dev+<sha>`.
  - Add a CHANGELOG, `fmt`/`clippy` in CI, and `rust-toolchain.toml`.
  - Bring the drifted workflows and skills back in line with the template.
- **keepup:**
  - Stop the `chore(Release): bump the version to 0.N.0` per roadmap step.
  - Tag the current version (`v0.149.0`) as the starting point.
  - Convert `CHANGELOG.md` headings from `## 0.N.0 — Title` to Keep a Changelog, keeping the titles as the first bullet or dropping them.
  - Add `scripts/release`.
  - Update CLAUDE.md's step-end ritual: a step ends at the human checkpoint, and releases happen when asked.
- **hogs:** switch from automatic release-on-push to `scripts/release`.
- **Homebrew projects:** drop the release's tap job and the `HOMEBREW_TAP_TOKEN` secret, switch `packaging/*.rb` to `@SHA256:<file>@`, and list the project in the tap's `projects` file.
- **Commit style:** existing history stays as it is; new commits follow this file.
- **GitHub metadata** for every repo: normalize topics to `vibe-coded`, and add descriptions to riff-raff, pod and umpire.
