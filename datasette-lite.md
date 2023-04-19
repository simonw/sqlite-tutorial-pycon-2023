# Datasette Lite

It's Datasette... running entirely in your web browser with WebAssembly and Pyodide!

<https://lite.datasette.io/>

## Loading SQLite, CSV and JSON data

- SQLite: <https://lite.datasette.io/?url=https://github.com/simonw/scrape-hmb-traffic/blob/main/hmb.db?&install=datasette-copyable#/hmb?sql=with+item1+as+(%0A++select%0A++++time(datetime(commits.commit_at%2C+'-7+hours'))+as+t%2C%0A++++duration_in_traffic+%2F+60+as+mins_in_traffic%0A++from%0A++++item_version%0A++++join+commits+on+item_version._commit+%3D+commits.id%0A++order+by%0A++++commits.commit_at%0A)%2C%0Aitem2+as+(%0A++select%0A++++time(datetime(commits.commit_at%2C+'-7+hours'))+as+t%2C%0A++++duration_in_traffic+%2F+60+as+mins_in_traffic%0A++from%0A++++item2_version%0A++++join+commits+on+item2_version._commit+%3D+commits.id%0A++order+by%0A++++commits.commit_at%0A)%0Aselect%0A++item1.*%2C%0A++item2.mins_in_traffic+as+mins_in_traffic_other_way%0Afrom%0A++item1%0A++join+item2+on+item1.t+%3D+item2.t> - see [Measuring traffic during the Half Moon Bay Pumpkin Festival](https://simonwillison.net/2022/Oct/19/measuring-traffic/)
- CSV: <https://lite.datasette.io/?csv=https://raw.githubusercontent.com/fivethirtyeight/data/master/fight-songs/fight-songs.csv>
- JSON: <https://lite.datasette.io/?json=https://gist.github.com/simonw/73d15c0dd1025d1196829740bacf4464>

## Installing plugins

Add `?install=name-of-plugin` to `pip install` that plugin into your browser's environment!

This only works with a subset of plugins.

- <https://lite.datasette.io/?install=datasette-copyable&json=https://gist.github.com/simonw/73d15c0dd1025d1196829740bacf4464>

## Further reading

- [Datasette Lite: a server-side Python web application running in a browser](https://simonwillison.net/2022/May/4/datasette-lite/)
- [Plugin support for Datasette Lite](https://simonwillison.net/2022/Aug/17/datasette-lite-plugins/)
- [Joining CSV files in your browser using Datasette Lite](https://simonwillison.net/2022/Jun/20/datasette-lite-csvs/)