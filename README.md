# ludwig-cf.github.io

This is the repository for the Ludwig static web site.

## Local preview

In order to see anything other than the current site via github pages, e.g. a branch, you will need a local preview. To arrange this, `sphinx` is required.

An easy way to install this is via `miniconda`. See https://docs.conda.io/en/latest/miniconda.html

Following the installation of miniconda, it should be possible to
```
$ conda install sphinx
```

## Rendering the pages locally

Check out the relevant branch. From the top level directory
```
$ make html
```
and then the pages should be accessible via `./_build/html/index.html`


By agreement with The University of Edinburgh this site is available via the
URL http://ludwig.epcc.ed.ac.uk/.
