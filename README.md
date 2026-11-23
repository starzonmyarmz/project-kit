# project-kit

Starts new projects with my conventions already in place: CI, tests, changelog, releases, docs and agent skills. That way the first session can go straight to the fun part.

[CONVENTIONS.md](CONVENTIONS.md) is the source of truth. Everything here follows from it.

## How it works

The kit is one [Copier](https://copier.readthedocs.io) template. `copier.yml` asks for the name, the stack (`rust-plugin`, `swift-app`, `other`, …), where it ships, and which optional docs to add. Files that only some projects need have conditional names (`{% if oss_community %}SECURITY.md{% endif %}.jinja`). Shared files such as AGENTS.md and README pull their stack-specific parts from `partials/<stack>/`.

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

The `/new-project` skill asks the questions, runs these, creates the GitHub repo with description and topics, sets secrets, pushes, and watches the first CI run. `/doctor` reports how an existing repo has drifted from the conventions.

## Changing a convention

1. Edit CONVENTIONS.md.
2. Update the templates and skills to match.
3. Run `copier update` in the projects that should catch up, and review the diff.
