A VST3 and CLAP plugin for macOS and Windows.

## Installing

Download the zip from [Releases](https://github.com/{{ owner }}/{{ name }}/releases/latest) and follow `INSTALL.txt` inside it. The plugin isn't signed or notarized, so macOS may need `xattr -dr com.apple.quarantine` on it first; `INSTALL.txt` has the exact command.

## Tests

`cargo test` runs the engine offline, in host-sized blocks, against stored golden renders and synthetic checks. After an intended change to the sound, rerun with `BLESS=1` to rewrite the renders, and listen to them before committing.