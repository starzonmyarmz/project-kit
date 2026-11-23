## Install
{% if 'homebrew-cask' in ships %}
```sh
brew install --cask {{ owner }}/tap/{{ name }}
```

Or download the zip from [Releases](https://github.com/{{ owner }}/{{ name }}/releases/latest).
{%- else %}
Download the zip from [Releases](https://github.com/{{ owner }}/{{ name }}/releases/latest), unzip it and move {{ display_name }} to Applications.
{%- endif %}

Requires macOS 14 or later on Apple silicon. {{ display_name }} is self-signed, not notarized{% if 'homebrew-cask' in ships %}: the cask clears the quarantine flag for you. If you downloaded the zip, run this once{% else %}, so run this once after moving it{% endif %}:

```sh
xattr -dr com.apple.quarantine "/Applications/{{ display_name }}.app"
```