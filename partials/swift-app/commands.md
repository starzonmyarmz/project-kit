{% if xcodegen -%}
```sh
{{ "%-46s" | format("swift test --package-path " ~ type_name ~ "Kit") }}# the {{ type_name }}Kit tests
xcodegen generate                             # regenerate {{ type_name }}.xcodeproj after editing project.yml
scripts/install.sh                            # build, replace /Applications/{{ display_name }}.app as X.Y.Z-dev+<sha>, open it
scripts/release --dry-run                     # what the next release would be
```

`scripts/install.sh` needs Xcode and XcodeGen (`brew install xcodegen`). It re-signs the build with `SIGN_IDENTITY` when that names a certificate, and leaves it ad hoc otherwise. Use one long-lived self-signed certificate per machine (Keychain Access → Certificate Assistant → Create a Certificate, type Code Signing), so macOS keeps the app's permissions across reinstalls.
{%- else -%}
```sh
swift test                                  # the {{ type_name }}Core tests
swift run                                   # run the app without bundling it
swift format --in-place --recursive Package.swift Sources Tests
scripts/install.sh                          # build, replace /Applications/{{ display_name }}.app as X.Y.Z-dev+<sha>, open it
scripts/build-app.sh 1.2.3                  # dist/{{ display_name }}.app and dist/{{ name }}-1.2.3.zip
scripts/release --dry-run                   # what the next release would be
```

`scripts/install.sh` and `build-app.sh` sign ad hoc unless `SIGN_IDENTITY` names a certificate. Use one long-lived self-signed certificate per machine (Keychain Access → Certificate Assistant → Create a Certificate, type Code Signing), so macOS keeps the app's permissions across reinstalls.
{%- endif %}