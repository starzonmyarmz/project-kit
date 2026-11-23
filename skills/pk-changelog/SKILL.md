---
name: pk-changelog
description: Catch CHANGELOG.md's [Unreleased] section up with the feat, fix and perf commits since the last release, written for the people who use the project. Use when /pk-release or scripts/release says the changelog is empty, or when the user types /pk-changelog.
---

# /pk-changelog

Normally each change writes its own changelog line in the same commit. This skill catches up when that didn't happen.

1. **Find what's missing.**
   - List the commits since the last tag: `git log --format='%h %s%n%b' $(git describe --tags --abbrev=0 --match 'v[0-9]*' 2>/dev/null || git rev-list --max-parents=0 HEAD)..HEAD`.
   - Keep the `feat`, `fix`, `perf` and breaking ones.
   - Drop any already covered under `## [Unreleased]`.
2. **Draft lines for people, not for developers.** Say what someone can now do, see or stop worrying about. Skip internals.
   - Group them under `### Added`, `### Changed`, `### Fixed` and `### Removed` (or the project's own sections, if AGENTS.md defines them).
   - Merge commits that tell one story into one line.
   - Leave out commits nobody using the project would notice, and say which you left out.
3. **Public repos:** check the draft has no real names, logins, paths or work data.
4. **Show the draft**, and write it under `## [Unreleased]` once the user is happy with it.
5. **Commit** it alone as `docs(Changelog): catch up on unreleased changes`.
