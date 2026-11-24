---
name: pk-new-project
description: Start a new project from project-kit - ask what it is, render the kit into ~/Code/<name>, make the first commit, create the GitHub repo with its description and topics, set up release secrets, and watch the first CI run. Only when the user types /pk-new-project.
argument-hint: [name] ["one-line description"]
disable-model-invocation: true
---

# /pk-new-project

Arguments: `$ARGUMENTS`, optionally the repo name and a quoted one-line description.

The kit lives at `${PROJECT_KIT:-$HOME/Code/project-kit}`. Its `CONVENTIONS.md` is the source of truth for everything below; read it if a step here seems to disagree with it, and follow it.

## 1. Check the tools

Run each as its own command: `copier --version`, `gh auth status`, `git -C <kit> status --short`.

- If copier is missing: `brew install copier`.
- If the kit has uncommitted changes, say so: they'll be included, which is fine for trying kit changes but not for a real project.

## 2. Ask what it is

The name and the description come from the arguments. Ask in plain text for whichever is missing:

- **Name:** kebab-case, used for the repo and folder. Suggest one if the user only describes the idea.
- **Description:** one sentence, for the README and GitHub.

Stop if `~/Code/<name>` exists or `gh repo view starzonmyarmz/<name>` finds a repo.

Then ask the rest in one AskUserQuestion call:

1. **Stack** (single choice):
   - VST plugin (VST3 + CLAP)
   - Mac app
   - Something else
   Read the current choices from `copier.yml` rather than this list, since stacks get added.
2. **Visibility:** Public (recommended) or Private.
3. **Ships to** (multi-select): GitHub release, Homebrew cask, Homebrew formula, itch.io, GitHub Pages, or, for a Mac app only you will run, Nowhere (`local`, on its own): you build and install it yourself, and GitHub never builds it.
4. **Extras** (multi-select):
   - DESIGN.md (preselect for anything with a UI)
   - VOICE.md
   - Sponsor button (FUNDING.yml; public only, preselect when public)
   - Community files (CONTRIBUTING, SECURITY, code of conduct, issue forms; public only)

The display name defaults to the title-cased name (`sputnik` becomes `Sputnik`). Mention the default and let the user change it. The same goes for the app ID: `com.iamdanielmarino.<name>` for plugins and apps.

## 3. Render

One `copier copy` with every answer passed as data, so it asks nothing:

```sh
copier copy --defaults --vcs-ref HEAD \
  -d name=<name> -d "description=<description>" -d "display_name=<Display Name>" \
  -d year=<current year> -d stack=<stack> -d public=<true|false> \
  -d 'ships=["github-release", ...]' -d design_doc=<bool> -d voice_doc=<bool> -d sponsors=<bool> -d oss_community=<bool> \
  <kit> ~/Code/<name>
```

Then in `~/Code/<name>`:

- `git init -b main`
- `git add -A`
- `git commit -m "chore: start from project-kit"`, with the session's attribution lines.

Check that it builds before going public:

| Stack | Check |
|---|---|
| vst-plugin | `cargo fmt --all --check`, `cargo clippy --workspace --all-targets -- -D warnings`, `cargo test --workspace` (the first build takes a few minutes) |
| swift-app | `swift format lint --strict --recursive Package.swift Sources Tests scripts/make-icon.swift`, `swift test`, `scripts/build-app.sh` |
| rust-app | `cargo fmt --all --check`, `cargo clippy --workspace --all-targets -- -D warnings`, `cargo test --workspace` (the first Bevy build takes several minutes) |
| other | `scripts/release --dry-run` exits with "nothing to release" |

If a check fails, the kit has a bug. Fix it in the kit, commit it there, and render again. Don't patch the new project by hand.

## 4. Create the GitHub repo

This is outward-facing. Show the user, in one message:
- the exact `gh repo create` command;
- the description;
- the topics.

Wait for a yes before running anything.

```sh
gh repo create starzonmyarmz/<name> --<public|private> --description "<description>" --source ~/Code/<name> --push
gh repo edit starzonmyarmz/<name> --enable-wiki=false --enable-projects=false --delete-branch-on-merge --add-topic vibe-coded --add-topic <topic> ...
```

Topics: `vibe-coded`, plus the ones for the stack and the places it ships.

| Stack or target | Topics |
|---|---|
| vst-plugin | `audio-plugin`, `vst3`, `clap`, `rust`, `nice-plug`, `music-production` |
| swift-app | `swift`, `macos` (add `swiftui`, `appkit`, `menu-bar-app` as they fit) |
| rust-app | `rust`, `bevy-engine`, `egui`, `macos`, `windows`, `linux` (add `game` if it is one) |
| python | `python` (add `cli` if it's a command-line tool) |
| Homebrew cask | `homebrew-cask` |
| Homebrew formula | `homebrew-formula` |
| itch.io | `itch-io` |

With `sponsors`, turn on the repo's Sponsorships setting too, which `gh repo edit` has no flag for. Without it GitHub hides the Sponsor button:

```sh
gh api graphql -f query='mutation($id:ID!){ updateRepository(input:{repositoryId:$id, hasSponsorshipsEnabled:true}) { repository { hasSponsorshipsEnabled } } }' -f id=<id from gh repo view --json id>
```

Add one or two topics for what it does (`cpu-monitor`, `voxel-editor`), and say which.

## 5. Secrets

Never ask the user to paste a secret into the chat. For each secret the project's README lists under Releases, tell the user to type it with the `!` prefix, so the value goes straight to `gh` without passing through this conversation:

```
! gh secret set BUTLER_API_KEY -R starzonmyarmz/<name>
```

- `SIGNING_CERT_P12` and `SIGNING_CERT_PASSWORD` (swift-app and rust-app, but not a `local` Mac app): the user's long-lived self-signed certificate. Reuse one certificate for every app, and set the P12 base64-encoded: `! base64 -i cert.p12 | gh secret set SIGNING_CERT_P12 -R starzonmyarmz/<name>`.
- For itch.io, also set the variable yourself: `gh variable set ITCH_TARGET -R starzonmyarmz/<name> --body "starzonmyarmz/<name>:<channel>"`. A rust-app pushes each platform to its own channel, so its `ITCH_TARGET` is just `starzonmyarmz/<name>`.

Skip this step when nothing is needed. Then confirm with `gh secret list -R starzonmyarmz/<name>`.

Homebrew needs no secret. For a Homebrew cask or formula, add a line for each `packaging/*.rb` to the `projects` file in `~/Code/homebrew-tap` (`<name>  packaging/<name>.rb  Casks/<name>.rb`, or `Formula/` for a formula), commit it as `feat(<name>): add to the tap`, and push. The tap renders it after the first release.

## 6. Watch the first CI run

`gh run list -R starzonmyarmz/<name> --limit 3`, then `gh run watch <id> -R starzonmyarmz/<name> --exit-status` on the Check run. A `local` Mac app has no Check run: run `swift test` and `scripts/install.sh` locally instead.

If it fails, read the failing step's log (`gh run view <id> --log-failed`). Fix the kit if the failure comes from the template, then fix the project in a `fix(CI): …` commit.

## 7. Hand off

Finish with:
- the repo URL and the folder;
- what was set up;
- what's left to the user.

Then suggest the first step for the stack:
- vst-plugin: `design-kickoff`.
- A project with DESIGN.md or VOICE.md: fill in "The look in one line" / "The voice in one line" together before building.
- A multi-step build: offer a `docs/spec/` roadmap with acceptance checks, as in keepup.
