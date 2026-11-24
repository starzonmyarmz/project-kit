```sh
python3 -m venv .venv && .venv/bin/pip install -e . pytest   # once
.venv/bin/pytest                                            # the tests
uvx ruff check . && uvx ruff format --check .               # lint and formatting, as CI runs them
scripts/release --dry-run                                   # what the next release would be
```
