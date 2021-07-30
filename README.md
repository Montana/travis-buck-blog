---
title: "Using Sphinx & Travis CI Together for Seamless Documentation"
created_at: Fri Jul 09 2021 15:00:00 EDT
author: Montana Mendy
layout: post
permalink: 2021-07-09-buck
category: news
excerpt_separator: <!-- more --> 
tags:
  - news
  - feature
  - infrastructure
  - community
---

![Sphinx](https://user-images.githubusercontent.com/20936398/127698282-d19efe0f-5246-4dd3-8c1f-9626b0ba3c0d.png)


Sphinx is a documentation generator or a tool that translates a set of plain text source files into various output formats, automatically producing cross-references, indices, etc. That is, if you have a directory containing a bunch of reStructuredText or Markdown documents, Sphinx can generate a series of HTML files, a PDF file (via LaTeX), man pages and much more. Let's integrate Travis CI for automatic document generation and make things a bit easier!

<!-- more --> 

## Quickstart

Let's install Sphinx using the CLI: 

```bash
pip install --user travis-sphinx
export PATH=$HOME/.local/bin:$PATH
```
Make sure you have a `gh-pages` branch: 

```bash
git checkout -b gh-pages
git rm -rf .
git push --set-upstream origin gh-pages
```
You'll then need to get an `access_token` from GitHub - so you'll need to generate a token to use. Head over to your GitHub account settings:

<img width="953" alt="Screen Shot 2021-07-30 at 11 45 46 AM" src="https://user-images.githubusercontent.com/20936398/127698217-a72e7e12-3397-4135-9307-41cd8dd3cad3.png">


## The setup 

Given the things in this guide we have, the `.travis.yml` file will look a bit like this:

```yaml
language: python
python:
- '3.7'
- '3.8'
- 3.8-dev
- nightly
install:
- pip install -r dev-requirements.txt
script:
- make -C docs/ html
- touch docs/build/html/.nojekyll
deploy:
  - provider: pypi
    user: "__token__"
    password: $PYPI_TOKEN
    skip_existing: true
  - provider: pages:git
    verbose: true
    token: $GITHUB_TOKEN
    edge: true
    local_dir: ./docs/build/html/
    keep_history: true
 ```
 
 Now let's set your `env vars` in the Travis CI GUI: 
 
 ![1_hNIPvvrBAdPakiYx6wOthw](https://user-images.githubusercontent.com/20936398/127698526-06d3c779-a181-430f-8cf6-247a71ea5d61.png)
 
 ## Building
 
 Here's a sample `build.py` for Sphinx, that you may want to use to speed the process up:
 
 ```python
import logging

import click
try:
    # Sphinx 1.7+
    from sphinx.cmd.build import build_main
    sphinx_version_at_least_1_7 = True
except AttributeError:
    from sphinx import build_main
    sphinx_version_at_least_1_7 = False

from .main import main

_logger = logging.getLogger(__name__)


@click.command(
    help="Build sphinx documentation."
)
@click.option(
    '-s', '--source',
    type=click.Path(dir_okay=True, file_okay=False, exists=True),
    help='Source directory of sphinx docs',
    default='docs/source',
    show_default=True
)
@click.option(
    '-n', '--nowarn',
    type=click.BOOL,
    is_flag=True,
    help='Do not error on warnings',
)
@click.pass_context
def build(ctx, source, nowarn):
    """
    Build documentation from ``source``, placing built files in
    ``target``.
    """
    _logger.info('building documentation')
    outdir = ctx.obj['outdir']

    if sphinx_version_at_least_1_7:
    
        args = ['-b', 'html']
    else:
        args = ['-b html']

    if not nowarn:
        args.append('-W')
    if build_main(args + [source, outdir]):
        raise click.ClickException("Error building sphinx doc")
```
## Specifying a custom deployment repository

Now, to specify a custom deployment repository if you're using a fork (which most people reading this might be), for working on documentation. To do so, under your Travis environment variables, use the following `constant`:

```bash
GH_REPO_SLUG = 'syntaf/fork-of-my-repo'
```

Here's an example from Vim on how the config (the `.travis.yml`) looks currently from my machine:

<img width="534" alt="Screen Shot 2021-07-30 at 12 00 13 PM" src="https://user-images.githubusercontent.com/20936398/127699835-552132c4-1b40-4b05-8360-cdd61be2ff43.png">

## Congratulations

You've done it, here's a working [example:](https://nasa-develop.github.io/dnppy/) that NASA is using, [https://develop.larc.nasa.gov/](https://develop.larc.nasa.gov/).

If you need any more resources or help, please email me personally at [montana@travis-ci.com](mailto:montana@travis-ci.com), or look at the official Sphinx [documentation](https://www.sphinx-doc.org/en/master/usage/quickstart.html) to get a better idea of the `.travis.yml` I have laid out above. Happy building! 
