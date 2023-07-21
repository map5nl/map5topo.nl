# map5topo documentation

The files and dirs below contain the map5topo documentation source.

## Building the docs locally

The documentation is powered by [MkDocs](https://www.mkdocs.org) which facilitates easy management
of content and publishing. Docs content is written in Markdown.


## Setting up the manual environment locally

```bash
# build a virtual Python environment in isolation
python3 -m venv .
. bin/activate
# fork or clone from GitHub
git clone https://github.com/map5/map5topo.git
cd docs
# install required dependencies
pip install -r requirements.txt
# build the website
mkdocs build
# serve locally
mkdocs serve  # website is made available on http://localhost:8000
```

## Deploying to live site

Website updates are automatically published via GitHub Actions. To publish manually:

```bash
# NOTE: you require access privileges to the GitHub repository
# to publish live updates
mkdocs gh-deploy -m 'add new page on topic x'
```
