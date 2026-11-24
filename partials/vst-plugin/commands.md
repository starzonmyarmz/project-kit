```sh
{% if goldens %}cargo test                                  # engine tests and golden renders, offline
{% if starter %}BLESS=1 cargo test                          # rewrite golden renders and ui/mock.json after an intended change
{%- else %}BLESS=1 cargo test                          # rewrite golden renders after an intended change to the sound
{%- endif %}
{%- else %}cargo test --workspace                      # engine tests, offline
{%- endif %}
cargo fmt --all && cargo clippy --workspace --all-targets -- -D warnings
cargo xtask bundle {{ name }} --release     # target/bundled/{{ display_name }}.{vst3,clap}
scripts/install.sh                          # bundle, sign ad hoc and install on macOS as X.Y.Z-dev+<sha>
scripts/clap-validator.sh "target/bundled/{{ display_name }}.clap"
{% if screenshot %}scripts/screenshot.sh                       # retake docs/{{ name }}.png
{% endif %}scripts/release --dry-run                   # what the next release would be
```

Work on the editor in a browser against the mock: `cd ui && python3 -m http.server 8765`, then open http://127.0.0.1:8765 (add `?version=1.2.0` to show a version). To load the page live in the plugin, set `{{ crate | upper }}_UI_URL` to that URL before starting the host.