# Advanced SQL

## Aggregations

The simplest form of aggregation is the one Datasette does to implement its own faceting feature.

```sql
select
  party,
  count(*)
from
  executive_terms
where
  type = 'prez'
group by
  party
```
[Try that query here](https://congress-legislators.datasettes.com/legislators?sql=select%0D%0A++party%2C%0D%0A++count%28*%29%0D%0Afrom%0D%0A++executive_terms%0D%0Awhere%0D%0A++type+%3D+%27prez%27%0D%0Agroup+by%0D%0A++party).

The `group by` creates groups of rows, then the aggregation functions calculate a value across that entire group.

The most common aggregation functions are:

- `count(*)` - count the number of rows in each group
- `max(column)` - maximum value for a column
- `min(column)` - minimum value for a column
- `sum(column)` - sum up the values in the column

Here's an example of `sum()` and `count()` from [What's in the RedPajama-Data-1T LLM training set](https://simonwillison.net/2023/Apr/17/redpajama-data/):
```sql
select
  top_folders,
  sum(size_gb) as total_gb,
  count(*) as num_files
from raw
group by top_folders
order by sum(size_gb) desc
```
[Run that in Datasette Lite](https://lite.datasette.io/?install=datasette-copyable&json=https://gist.github.com/simonw/73d15c0dd1025d1196829740bacf4464#/data?sql=select%0A++top_folders%2C%0A++cast+%28sum%28size_gb%29+as+integer%29+as+total_gb%2C%0A++count%28*%29+as+num_files%0Afrom+raw%0Agroup+by+top_folders%0Aorder+by+sum%28size_gb%29+desc).

Change the `total_gb` line to this to round it to the nearest integer:
```sql
  cast (sum(size_gb) as integer) as total_gb,
```
## Subqueries

SQLite has excellent support for subqueries. You can use them in `where X in` clauses:

```sql
select html_url from releases where repo in (
  select id from repos where full_name in (
    select repo from plugin_repos
  )
)
order by created_at desc
```
[Run that on datasette.io](https://datasette.io/content?sql=select+html_url+from+releases+where+repo+in+%28%0D%0A++select+id+from+repos+where+full_name+in+%28%0D%0A++++select+repo+from+plugin_repos%0D%0A++%29%0D%0A%29%0D%0Aorder+by+created_at+desc). Sometimes I find these to be more readable than joins!

You can also use them directly in `select` clauses:

```sql
select
  full_name,
  (
    select
      html_url
    from
      releases
    where
      releases.repo = repos.id
    order by
      created_at desc
    limit
      1
  ) as latest_release
from
  repos
```
[Run that here](https://datasette.io/content?sql=select+full_name%2C+%28select+html_url+from+releases+where+releases.repo+%3D+repos.id+order+by+created_at+desc+limit+1%29+as+latest_release+from+repos).

## CTEs

CTE is a terrible name for an incredibly powerful feature. It stands for Common Table Expressions. Think of it as a way of creating an alias to a temporary table for the duration of a query.

```sql
with presidents as (
  select
    executives.name
  from
    executive_terms
    join executives
      on executive_terms.executive_id = executives.id
  where
    executive_terms.type = 'prez'
),
vice_presidents as (
  select
    executives.name
  from
    executive_terms
    join executives
      on executive_terms.executive_id = executives.id
  where
    executive_terms.type = 'viceprez'
)
select
  distinct name
from
  presidents
where name in vice_presidents
```
[Try this CTE query here](https://congress-legislators.datasettes.com/legislators?sql=with+presidents+as+%28%0D%0A++select%0D%0A++++executives.name%0D%0A++from%0D%0A++++executive_terms%0D%0A++++join+executives+on+executive_terms.executive_id+%3D+executives.id%0D%0A++where%0D%0A++++executive_terms.type+%3D+%27prez%27%0D%0A%29%2C%0D%0Avice_presidents+as+%28%0D%0A++select%0D%0A++++executives.name%0D%0A++from%0D%0A++++executive_terms%0D%0A++++join+executives+on+executive_terms.executive_id+%3D+executives.id%0D%0A++where%0D%0A++++executive_terms.type+%3D+%27viceprez%27%0D%0A%29%0D%0Aselect%0D%0A++distinct+name%0D%0Afrom%0D%0A++presidents%0D%0Awhere%0D%0A++name+in+vice_presidents).

## JSON

SQLite has excellent JSON functionality built in. Store JSON in a `text` column and you can query it using `json_extract()` - you can also build JSON values in `select` queries.

[Returning related rows in a single SQL query using JSON](https://til.simonwillison.net/sqlite/related-rows-single-query) shows some advanced tricks you can do with this.

```sql
select
  legislators.id,
  legislators.name,
  json_group_array(json_object(
    'type', legislator_terms.type,
    'state', legislator_terms.state,
    'start', legislator_terms.start,
    'end', legislator_terms.end,
    'party', legislator_terms.party
   )) as terms,
   count(*) as num_terms
from
  legislators join legislator_terms on legislator_terms.legislator_id = legislators.id
  group by legislators.id
order by
  id
limit
  10
```
[Run that query](https://congress-legislators.datasettes.com/legislators?sql=select%0D%0A++legislators.id%2C%0D%0A++legislators.name%2C%0D%0A++json_group_array(json_object(%0D%0A++++%27type%27%2C+legislator_terms.type%2C%0D%0A++++%27state%27%2C+legislator_terms.state%2C%0D%0A++++%27start%27%2C+legislator_terms.start%2C%0D%0A++++%27end%27%2C+legislator_terms.end%2C%0D%0A++++%27party%27%2C+legislator_terms.party%0D%0A+++))+as+terms%2C%0D%0A+++count(*)+as+num_terms%0D%0Afrom%0D%0A++legislators+join+legislator_terms+on+legislator_terms.legislator_id+%3D+legislators.id%0D%0A++group+by+legislators.id%0D%0Aorder+by%0D%0A++id%0D%0Alimit%0D%0A++10).

Paul Ford [said about SQLite's JSON support](https://simonwillison.net/2018/Jan/29/paul-ford/):

> The JSON interface is like, “we save the text and when you retrieve it we parse the JSON at several hundred MB/s and let you do path queries against it please stop overthinking it, this is filing cabinet.”

## Window functions

I wanted to run a query that would return the following:

- The repository name
- The date of the most recent release from that repository (the releases table is a many-to-one against repos)
- The total number of releases
- **The three most recent releases** (as a JSON array of objects)


```sql
with cte as (
  select
    repos.full_name,
    releases.created_at,
    releases.id as rel_id,
    releases.name as rel_name,
    releases.created_at as rel_created_at,
    rank() over (partition by repos.id order by releases.created_at desc) as rel_rank
  from repos
    left join releases on releases.repo = repos.id
)
select
  full_name,
  max(created_at) as max_created_at,
  count(rel_id) as releases_count,
  json_group_array(
    json_object(
      'id', rel_id,
      'name', rel_name,
      'created_at', rel_created_at
    )
  ) filter (where rel_id is not null and rel_rank <= 3) as recent_releases
from cte
group by full_name
order by releases_count desc
```
[Run that query here](https://datasette.io/content?sql=with+cte+as+%28%0D%0A++select%0D%0A++++repos.full_name%2C%0D%0A++++releases.created_at%2C%0D%0A++++releases.id+as+rel_id%2C%0D%0A++++releases.name+as+rel_name%2C%0D%0A++++releases.created_at+as+rel_created_at%2C%0D%0A++++rank%28%29+over+%28partition+by+repos.id+order+by+releases.created_at+desc%29+as+rel_rank%0D%0A++from+repos%0D%0A++++left+join+releases+on+releases.repo+%3D+repos.id%0D%0A%29%0D%0Aselect%0D%0A++full_name%2C%0D%0A++max%28created_at%29+as+max_created_at%2C%0D%0A++count%28rel_id%29+as+releases_count%2C%0D%0A++json_group_array%28%0D%0A++++json_object%28%0D%0A++++++%27id%27%2C+rel_id%2C%0D%0A++++++%27name%27%2C+rel_name%2C%0D%0A++++++%27created_at%27%2C+rel_created_at%0D%0A++++%29%0D%0A++%29+filter+%28where+rel_id+is+not+null+and+rel_rank+%3C%3D+3%29+as+recent_releases%0D%0Afrom+cte%0D%0Agroup+by+full_name%0D%0Aorder+by+releases_count+desc).

[Running this smaller query](https://datasette.io/content?sql=with+cte+as+(%0D%0A++select%0D%0A++++repos.full_name%2C%0D%0A++++releases.created_at%2C%0D%0A++++releases.id+as+rel_id%2C%0D%0A++++releases.name+as+rel_name%2C%0D%0A++++releases.created_at+as+rel_created_at%2C%0D%0A++++rank()+over+(partition+by+repos.id+order+by+releases.created_at+desc)+as+rel_rank%0D%0A++from+repos%0D%0A++++left+join+releases+on+releases.repo+%3D+repos.id%0D%0A)%0D%0Aselect+*+from+cte) helps show what's going on with that `rel_rank` column:

```sql
with cte as (
  select
    repos.full_name,
    releases.created_at,
    releases.id as rel_id,
    releases.name as rel_name,
    releases.created_at as rel_created_at,
    rank() over (partition by repos.id order by releases.created_at desc) as rel_rank
  from repos
    left join releases on releases.repo = repos.id
)
select * from cte
```
