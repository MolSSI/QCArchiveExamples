# QCArchiveTutorials

Tips and help getting started with the QCArchive ecosystem.

## Build

The docs for this project are built with Sphinx. To compile the docs, first ensure that Sphinx, the ReadTheDocs theme, and nbsphinx are installed.

```
conda install sphinx sphinx_rtd_theme npsphinx 
```

Once installed, you can use the Makefile in this directory to compile static HTML pages by

```
make html
```

The compiled docs will be in the `_build` directory and can be viewed by
opening index.html (which may itself be inside a directory called html/
depending on what version of Sphinx is installed).

## Contributing

These jupyter notebooks should be able to be run from top to bottom using the
MolSSI QCArchive dataset. This particular set of examples should require
relatively minimal compute resources and should be limited to datasets of less
then 10,000 rows.
