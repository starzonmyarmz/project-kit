## Install
{% if 'homebrew-cask' in ships %}
On macOS, with [Homebrew](https://brew.sh):

```sh
brew install --cask {{ owner }}/tap/{{ name }}
```
{% endif %}
Downloads for macOS (a universal app), Windows (an installer or a zip) and Linux (a tarball with an install script) are on [Releases](https://github.com/{{ owner }}/{{ name }}/releases/latest){% if 'itch' in ships %} and itch.io{% endif %}.

{{ display_name }} is self-signed, not notarized{% if 'homebrew-cask' in ships %}: the cask clears the quarantine flag for you. If you downloaded the zip, run this once{% else %}, so run this once after moving it to Applications{% endif %}:

```sh
xattr -dr com.apple.quarantine "/Applications/{{ display_name }}.app"
```

Windows builds are unsigned, so SmartScreen warns on first run: click **More info** → **Run anyway**. On Linux, unpack the tarball and run `./install.sh`.
