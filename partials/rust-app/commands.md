```sh
cargo run                                   # dev build: the app at opt-level 1, dependencies unoptimized
cargo run --features dev                    # link Bevy dynamically for faster rebuilds
cargo check                                 # iterate with this, not cargo build
cargo test --workspace                      # the pure-logic tests
cargo fmt --all && cargo clippy --workspace --all-targets -- -D warnings
scripts/install.sh                          # bundle, replace /Applications/{{ display_name }}.app as X.Y.Z-dev+<sha>, open it (needs cargo-bundle)
scripts/release --dry-run                   # what the next release would be
```

`scripts/install.sh` signs ad hoc unless `SIGN_IDENTITY` names a certificate. Use one long-lived self-signed certificate per machine (Keychain Access → Certificate Assistant → Create a Certificate, type Code Signing), so macOS keeps the app's permissions across reinstalls. On Linux, CI installs `libwayland-dev` and `libxkbcommon-dev` for Bevy; do the same locally.
