- The bundle ID `{{ app_id }}` never changes. macOS ties permissions, Launch at Login and preferences to it.
{% if xcodegen -%}
- The version lives only in `MARKETING_VERSION` in `project.yml`, and only `scripts/release` changes it. `scripts/install.sh` builds as `X.Y.Z-dev+<sha>`, with the commit count as the build number. Don't bump it by hand.
- Edit `project.yml`, never the generated Xcode project, Info.plist or entitlements: `xcodegen generate` overwrites them.
- Sign local builds with the same certificate every time (`SIGN_IDENTITY`). Ad hoc signing gives each build a new identity, so macOS asks for permissions again.
{%- else -%}
- The version isn't stored in the repo. It comes from the release tag (`scripts/version.sh` reads the last one), and `build-app.sh` writes it into `Info.plist`. Don't add a version anywhere else.
- Sign releases with the same certificate every time (the `SIGNING_CERT_P12` secret). A new certificate, or ad hoc signing, makes macOS forget the app's permissions on update.
- To change the app icon, edit `packaging/icon.svg` or the tile colours at the top of `scripts/make-icon.swift`, and rebuild. Never commit a generated `.icns`.
{%- endif %}
- After a change to the app, run `scripts/install.sh` so the user can try the real bundle.