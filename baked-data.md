# Baked Data

The [The Baked Data architectural pattern](https://simonwillison.net/2021/Jul/28/baked-data/) describes this approach, which is key to taking full advantage of SQLite and Datasette.

I like to build my databases in GitHub Actions.

## Niche Museums and TILs

- <https://www.niche-museums.com/> is published from the <https://github.com/simonw/museums> repository - one big YAML file for the content.
- <https://til.simonwillison.net/> is published <https://github.com/simonw/til> - separate Markdown files for each item.

Both of these sites have Atom feeds that are defined using a Datasette [canned query](https://docs.datasette.io/en/stable/sql_queries.html#canned-queries), in conjunction with the [datasette-atom](https://datasette.io/plugins/datasette-atom) plugin.

- <https://www.niche-museums.com/browse/feed>
- <https://til.simonwillison.net/tils/feed>

## Generating a newsletter with an Observable notebook

I wrote about this in [Semi-automating a Substack newsletter with an Observable notebook](https://simonwillison.net/2023/Apr/4/substack-observable/):

- <https://datasette.simonwillison.net/simonwillisonblog> is a Datasette/SQLite copy of my Django blog, created using [db-to-sqlite](https://datasette.io/tools/db-to-sqlite) by my <https://github.com/simonw/simonwillisonblog-backup> GitHub repository.
- <https://observablehq.com/@simonw/blog-to-newsletter> is my Observable notebook that assembles a newsletter from that data.
- <https://simonw.substack.com/> is the Substack newsletter that I copy that content into.
