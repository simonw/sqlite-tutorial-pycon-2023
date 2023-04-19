# Exploring data with Datasette

[Datasette](https://datasette.io/) is "an open source multi-tool for exploring and publishing data".

## Installing Datasette locally

```
pip install datasette
```
Or if you prefer `pipx`:
```
pipx install datasette
```
Or Homebrew (on macOS):
```
brew install datasette
```
[More installations options](https://docs.datasette.io/en/stable/installation.html).

In Codespaces you should also install the `datasette-codespaces` plugin:

    datasette install datasette-codespaces

## Try a database: legislators.db

```
wget https://congress-legislators.datasettes.com/legislators.db
```

This is a database of US legislators, presidents and vice presidents.

You can explore it online at <https://congress-legislators.datasettes.com/legislators>

Open it in Datasette like this:

    datasette legislators.db

We'll follow this tutorial to explore Datasette's features: **[Exploring a database with Datasette](https://datasette.io/tutorials/explore)**

## Install some plugins

Datasette has over a hundred plugins: <https://datasette.io/plugins>

You can `pip install` them, but it's better to use `datasette install` as that ensures they will go in the correct virtual environment, especially useful if you used `pipx` or Homebrew to install Datasette itself.

    datasette install datasette-cluster-map

Now restart Datasette and visit the "offices" table to see the result.

You can review what plugins are installed with:

    datasette plugins

Or by visiting the `/-/plugins` page in Datasette.

Plugins can be uninstalled with:

    datasette uninstall datasette-cluster-map

## Learning SQL with Datasette

The "✎ View and edit SQL" link is a quick way to start learning basic SQL queries.

We'll follow this tutorial next: **[Learn SQL with Datasette](https://datasette.io/tutorials/learn-sql)**


