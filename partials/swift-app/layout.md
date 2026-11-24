{% if xcodegen -%}
- `project.yml`: the XcodeGen spec (targets, packages, Info.plist properties, entitlements, build scripts and the version). `{{ type_name }}.xcodeproj` and the Info.plist and entitlements it names are generated from it.
- `{{ type_name }}Kit/`: the Swift package with all the logic, with no AppKit or SwiftUI, so `swift test` covers it. Tests use Swift Testing (`@Suite`, `@Test`, `#expect`).
- The app target's sources (its `sources` in `project.yml`): the app shell (SwiftUI scenes, AppKit glue). Keep it thin: anything worth testing moves into the package.
- `scripts/`: `install.sh` and release tooling.
{%- else -%}
- `Sources/{{ type_name }}Core/`: all the logic, with no AppKit or SwiftUI, so `swift test` covers it.
- `Sources/{{ type_name }}/`: the app shell (SwiftUI scenes, AppKit glue). Keep it thin: anything worth testing moves into Core.
- `Tests/{{ type_name }}CoreTests/`: Swift Testing (`@Suite`, `@Test`, `#expect`).
- `packaging/`: `Info.plist` (the version is `@VERSION@`, filled in at build time), `icon.svg` (the glyph `scripts/make-icon.swift` draws on the app icon tile){% if 'homebrew-cask' in ships %}, and `{{ name }}.rb`, the cask template `starzonmyarmz/homebrew-tap` renders from each release{% endif %}.
- `scripts/`: `build-app.sh`, `install.sh`, `make-icon.swift` and release tooling.
{%- endif %}