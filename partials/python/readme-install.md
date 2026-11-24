## Install
{% if 'homebrew-formula' in ships %}
With [Homebrew](https://brew.sh):

```sh
brew install {{ owner }}/tap/{{ name }}
```

Or from source:
{% endif %}
```sh
pip install git+https://github.com/{{ owner }}/{{ name }}
```
