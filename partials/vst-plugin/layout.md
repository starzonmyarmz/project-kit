- `plugin/src/engine.rs`: the DSP, in f64, with no host types, so `cargo test` runs it offline.
- `plugin/src/params.rs`: the parameters. Their defaults come from `Settings::default()`, and `settings()` turns them into the engine's `Settings`.
- `plugin/src/lib.rs`: host plumbing, plugin IDs and `PAGE` (title, window size, version).
- `plugin/src/editor.rs`, `bridge.rs`: the webview editor and its message bridge. Don't add another channel to `editor.rs`; use `WebEditor::with_feed`.
{% if starter %}- `plugin/tests/`: engine tests, golden renders in `golden/`, real DI recordings in `di/`, and `mock.rs`, which keeps `ui/mock.json` in step with the params.
- `ui/index.html`: the editor page, compiled into the plugin. `ui/mock.json` drives it in a browser.
{%- elif goldens %}- `plugin/tests/`: engine tests, golden renders in `golden/` and real DI recordings in `di/`. `support/` has the WAV and golden helpers.
- `ui/index.html`: the whole editor, compiled into the plugin, with a hand-written mock bridge (`mockBridge()`) for running it in a browser.
{%- else %}- `plugin/tests/`: engine tests.
- `ui/index.html`: the whole editor, compiled into the plugin, with a hand-written mock bridge (`mockBridge()`) for running it in a browser.
{%- endif %}
- `xtask/`, `bundler.toml`: nice-plug's bundler.
{% if screenshot %}- `scripts/`: install, clap-validator, screenshot and release.
{%- else %}- `scripts/`: install, clap-validator and release.
{%- endif %}
{% if screenshot and goldens %}- `.claude/skills/`: `design-kickoff` (settle the look before building the editor), `screenshot` and `bless-goldens`. Use them rather than working from memory.
{%- elif screenshot %}- `.claude/skills/`: `design-kickoff` (settle the look before building the editor) and `screenshot`. Use them rather than working from memory.
{%- elif goldens %}- `.claude/skills/`: `design-kickoff` (settle the look before building the editor) and `bless-goldens`. Use them rather than working from memory.
{%- else %}- `.claude/skills/`: `design-kickoff`, to settle the look before building the editor. Use it rather than working from memory.
{%- endif %}