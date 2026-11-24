# project-kit

Starts new projects with my conventions already in place: CI, tests, changelog, releases, docs and agent skills. That way the first session can go straight to the fun part.

[CONVENTIONS.md](CONVENTIONS.md) is the source of truth. Everything here follows from it.

## Why

I build a lot of small side projects, mostly with coding agents and across very different stacks: audio plugins in Rust, menu-bar apps in Swift, games, CLIs, Python tools. Each one used to start with the same chores, like setting up CI, picking a commit style, writing a release script and explaining the repo to the agent. Each one solved them a little differently. Then a fix in one repo, like a better release flow or a version rule, never made it back to the others.

The kit fixes that in three ways:

- **One place for the rules.** CONVENTIONS.md writes down how a project starts and ships, and notes which project each rule came from, so a rule can be judged later.
- **Projects start ready.** A new repo already has CI, tests, a changelog, a release script, AGENTS.md and the `pk-*` skills, so the agent and I can begin on the actual idea.
- **Projects keep up.** Because the kit is a Copier template, `copier update` brings later improvements into existing projects. `/pk-doctor` shows where a repo has drifted, and `/pk-adopt` brings in a project that started before the kit.

## How it works

The kit is one [Copier](https://copier.readthedocs.io) template. `copier.yml` asks for the name, the stack (`vst-plugin`, `swift-app`, `rust-app`, `python`, `other`, …), where it ships, and which optional docs to add. Files that only some projects need have conditional names (`{% if oss_community %}SECURITY.md{% endif %}.jinja`). Shared files such as AGENTS.md and README pull their stack-specific parts from `partials/<stack>/`.

[Copier](https://copier.readthedocs.io) is used, rather than a GitHub template repo or cookiecutter, because it can update a project after creating it. It records the answers and kit version in `.copier-answers.yml`. Later, `copier update` re-renders the template at the new version and does a three-way merge against what the project changed. Those tools stop at the first copy. Copier also has the questions, conditional files and Jinja includes that let one template serve every stack.

```
copier.yml       the questions
template/        what gets rendered into a project
partials/<stack> commands, layout, invariants and install text per stack
```

```sh
copier copy gh:starzonmyarmz/project-kit my-app   # start a project
copier update                                     # later, inside my-app, to catch up with the kit
```

Every stack gets the same release contract: `scripts/release` plus a stack-specific `scripts/version.sh` that defines `read_version` and `write_version`.

A new stack needs:
- a choice in `copier.yml`;
- `partials/<stack>/`;
- a branch in `version.sh.jinja`;
- its files under conditional names.

The `/pk-new-project` skill asks the questions, runs these, creates the GitHub repo with description and topics, sets secrets, pushes, and watches the first CI run. `/pk-doctor` reports how an existing repo has drifted from the conventions.

## Starter files

The files listed in `_skip_if_exists` in `copier.yml` belong to the project once it exists: README, CHANGELOG, DESIGN/VOICE, `Cargo.lock`, and the plugin's code, tests and UI. Copier creates them once and never overwrites them. Everything else is managed: `copier update` merges kit changes into it three ways, keeping the project's own edits.

## Adopting an existing project

Running the kit over a repo that didn't start from it needs a judgment pass. A trial adoption showed what that pass has to do:

1. Run `copier copy --overwrite` in a worktree, with answers that match the repo (name, description and the existing plugin IDs).
2. Put the version back to the last tag's. The first copy resets the source of truth (e.g. `Cargo.toml`) to the template's `0.0.0`, and an old bump-after-every-change rule may have left `HEAD` ahead of anything released.
3. Move the old CLAUDE.md's project knowledge into AGENTS.md (invariants, layout) and DESIGN.md (the look), since CLAUDE.md becomes `@AGENTS.md`.
4. Delete starter files the repo never had (e.g. the template's golden render, `ui/mock.json`, `tests/mock.rs`).
5. Adapt the protected files by hand where a kit feature needs them (e.g. `PAGE.version` in `lib.rs` for dev versions).
6. Review the whole diff, then commit it as one `chore: adopt project-kit` commit.

## Changing a convention

1. Edit CONVENTIONS.md.
2. Update the templates and skills to match.
3. Run `copier update` in the projects that should catch up, and review the diff. If a template file now sits behind an answer, the first update deletes a project's own file at that path, even a protected one: restore it with `git checkout HEAD -- <path>` before committing.
