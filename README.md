# libmonero.github.io

Source of the [Libmonero](https://github.com/libmonero) website, published at <https://libmonero.github.io/> with GitHub Pages.

It is a [MkDocs](https://www.mkdocs.org/) site with the [Material](https://squidfunk.github.io/mkdocs-material/) theme. The pages are in [docs/](docs/) and the configuration is in [mkdocs.yml](mkdocs.yml).

## Build locally

```bash
python3 -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt
mkdocs serve        # live preview on http://127.0.0.1:8000
mkdocs build --strict
```

Every push to `main` builds the site and deploys it, see [.github/workflows/pages.yml](.github/workflows/pages.yml). The project sites of the libraries, such as [monero-python](https://libmonero.github.io/monero-python/), are published from their own repositories under the same domain.

## License

[MIT](LICENSE)
