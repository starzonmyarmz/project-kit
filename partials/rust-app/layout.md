- `Cargo.toml`: the app crate and the workspace. The version lives in `[workspace.package]`; `[package.metadata.bundle]` is what `cargo bundle` turns into the macOS app.
- `src/main.rs`: the Bevy `App`: plugins, startup and the systems. `bevy_egui` draws the UI in `EguiPrimaryContextPass`.
{% if starter %}- `src/version.rs`: the version people see, `X.Y.Z` or the `X.Y.Z-dev+<sha>` that `scripts/install.sh` sets through `APP_VERSION`.
{% endif -%}
- `packaging/`: `windows/{{ name }}.iss` (the Inno Setup installer), `linux/` (the tarball's `install.sh` and `.desktop` file){% if 'homebrew-cask' in ships %}, and `{{ name }}.rb`, the cask template `starzonmyarmz/homebrew-tap` renders from each release{% endif %}.
- `scripts/`: `install.sh` and release tooling.
