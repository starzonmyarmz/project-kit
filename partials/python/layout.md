- `pyproject.toml`: the package metadata, dependencies and entry points. The version lives in `[project] version`.
- `src/{{ crate }}/`: the package. `__version__` comes from the installed package's metadata, never a literal.{% if starter %} `cli.py` is the `{{ name }}` command.{% endif %}
- `tests/`: pytest tests. `test_version.py` checks `__version__` against `pyproject.toml`.
{%- if 'homebrew-formula' in ships %}
- `packaging/homebrew/{{ name }}.rb`: the formula template `starzonmyarmz/homebrew-tap` renders from each release.
{%- endif %}
- `.python-version`: the Python CI runs.
- `scripts/`: release tooling (`release`, `version.sh`, `changelog-notes`, `check-changelog`).
