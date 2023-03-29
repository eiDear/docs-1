---
title: SHOW RANGES
summary: The SHOW RANGES statement shows information about the ranges that comprise the data for a table, index, or entire database.
toc: true
docs_area: reference.sql
---

The `SHOW RANGES` [statement](sql-statements.html) shows information about the [ranges](architecture/overview.html#architecture-range) that comprise the data for a table, index, or entire database. This information is useful for verifying how SQL data maps to underlying ranges, and where the replicas for ranges are located. If `SHOW RANGES` displays `NULL` for both the start and end keys of a range, the range is empty and has no splits.

{{site.data.alerts.callout_info}}
To show range information for a specific row in a table or index, use the [`SHOW RANGE ... FOR ROW`](show-range-for-row.html) statement.
{{site.data.alerts.end}}

## Synopsis

<div>
{% remote_include https://raw.githubusercontent.com/cockroachdb/generated-diagrams/{{ page.release_info.crdb_branch_name }}/grammar_svg/show_ranges.html %}
</div>

## Required privileges

To use the `SHOW RANGES` statement, a user must either be a member of the [`admin`](security-reference/authorization.html#admin-role) role (the `root` user belongs to the `admin` role by default) or have the `ZONECONFIG` [privilege](security-reference/authorization.html#managing-privileges) defined.

## Parameters

Parameter | Description
----------|------------
[`table_name`](sql-grammar.html#table_name) | The name of the table you want range information about.
[`table_index_name`](sql-grammar.html#table_index_name) | The name of the index you want range information about.
[`database_name`](sql-grammar.html#database_name) | The name of the database you want range information about.
[`opt_show_ranges_options`](sql-grammar.html#show_ranges_options) | The options used to configure what fields appear in the [response](#response).

## Options

The following options

[xxx](): WRITE ME

## Response

The response varies depending on the values passed as [`opt_show_ranges_options`](sql-grammar.html#show_ranges_options).  The following fields are returned:

Field | Description
------|------------
`start_key` | The start key for the range.
`end_key` | The end key for the range.
`range_id` | The range ID.
`voting_replicas` | The nodes that contain the range [replicas](architecture/overview.html#architecture-range).
`non_voting_replicas` | The nodes that contain the range [replicas](architecture/overview.html#architecture-range).
`replicas` | The nodes that contain the range [replicas](architecture/overview.html#architecture-range).
`replica_localities` | The [locality](cockroach-start.html#locality) of the range.

`range_size_mb` | The size of the range.
`lease_holder` | The node that contains the range's [leaseholder](architecture/overview.html#architecture-range).
`lease_holder_locality` | The [locality](cockroach-start.html#locality) of the leaseholder.


`table_name` | The name of the table.

{{site.data.alerts.callout_info}}
If both `start_key` and `end_key` show `NULL`, the range is empty and has no splits.
{{site.data.alerts.end}}

## Examples

{% include {{page.version.version}}/sql/movr-statements-geo-partitioned-replicas.md %}

### Show ranges for a table (primary index)

{% include_cached copy-clipboard.html %}
~~~ sql
WITH x as (SHOW RANGES FROM TABLE vehicles) SELECT * FROM x WHERE "start_key" NOT LIKE '%Prefix%';
~~~
~~~
     start_key     |          end_key           | range_id | range_size_mb | lease_holder |  lease_holder_locality   | replicas |                                 replica_localities
+------------------+----------------------------+----------+---------------+--------------+--------------------------+----------+------------------------------------------------------------------------------------+
  /"new york"      | /"new york"/PrefixEnd      |       58 |      0.000304 |            2 | region=us-east1,az=c     | {1,2,5}  | {"region=us-east1,az=b","region=us-east1,az=c","region=us-west1,az=b"}
  /"washington dc" | /"washington dc"/PrefixEnd |      102 |      0.000173 |            2 | region=us-east1,az=c     | {1,2,3}  | {"region=us-east1,az=b","region=us-east1,az=c","region=us-east1,az=d"}
  /"boston"        | /"boston"/PrefixEnd        |       63 |      0.000288 |            3 | region=us-east1,az=d     | {1,2,3}  | {"region=us-east1,az=b","region=us-east1,az=c","region=us-east1,az=d"}
  /"seattle"       | /"seattle"/PrefixEnd       |       97 |      0.000295 |            4 | region=us-west1,az=a     | {4,5,6}  | {"region=us-west1,az=a","region=us-west1,az=b","region=us-west1,az=c"}
  /"los angeles"   | /"los angeles"/PrefixEnd   |       55 |      0.000156 |            5 | region=us-west1,az=b     | {4,5,6}  | {"region=us-west1,az=a","region=us-west1,az=b","region=us-west1,az=c"}
  /"san francisco" | /"san francisco"/PrefixEnd |       71 |      0.000309 |            6 | region=us-west1,az=c     | {1,5,6}  | {"region=us-east1,az=b","region=us-west1,az=b","region=us-west1,az=c"}
  /"amsterdam"     | /"amsterdam"/PrefixEnd     |       59 |      0.000305 |            9 | region=europe-west1,az=d | {7,8,9}  | {"region=europe-west1,az=b","region=europe-west1,az=c","region=europe-west1,az=d"}
  /"paris"         | /"paris"/PrefixEnd         |       62 |      0.000299 |            9 | region=europe-west1,az=d | {7,8,9}  | {"region=europe-west1,az=b","region=europe-west1,az=c","region=europe-west1,az=d"}
  /"rome"          | /"rome"/PrefixEnd          |       67 |      0.000168 |            9 | region=europe-west1,az=d | {7,8,9}  | {"region=europe-west1,az=b","region=europe-west1,az=c","region=europe-west1,az=d"}
(9 rows)
~~~

{% include {{page.version.version}}/sql/show-ranges-output-deprecation-notice.md %}

~~~
                                      start_key                                     |                                      end_key                                      | range_id | replicas |                                 replica_localities                                 | voting_replicas | non_voting_replicas | learner_replicas |    split_enforced_until
------------------------------------------------------------------------------------+-----------------------------------------------------------------------------------+----------+----------+------------------------------------------------------------------------------------+-----------------+---------------------+------------------+-----------------------------
  …/<TableMin>                                                                      | …/1/"amsterdam"                                                                   |      105 | {3,4,8}  | {"region=us-east1,az=d","region=us-west1,az=a","region=europe-west1,az=c"}         | {3,8,4}         | {}                  | {}               | NULL
  …/1/"amsterdam"                                                                   | …/1/"amsterdam"/PrefixEnd                                                         |      220 | {7,8,9}  | {"region=europe-west1,az=b","region=europe-west1,az=c","region=europe-west1,az=d"} | {7,8,9}         | {}                  | {}               | NULL
  …/1/"boston"                                                                      | …/1/"boston"/"\"\"\"\"\"\"B\x00\x80\x00\x00\x00\x00\x00\x00\x02"                  |      227 | {1,2,3}  | {"region=us-east1,az=b","region=us-east1,az=c","region=us-east1,az=d"}             | {3,1,2}         | {}                  | {}               | NULL
  …/1/"boston"/"\"\"\"\"\"\"B\x00\x80\x00\x00\x00\x00\x00\x00\x02"                  | …/1/"boston"/"333333D\x00\x80\x00\x00\x00\x00\x00\x00\x03"                        |       75 | {1,2,3}  | {"region=us-east1,az=b","region=us-east1,az=c","region=us-east1,az=d"}             | {3,2,1}         | {}                  | {}               | 2262-04-11 23:47:16.854776
  …/1/"boston"/"333333D\x00\x80\x00\x00\x00\x00\x00\x00\x03"                        | …/1/"boston"/PrefixEnd                                                            |       74 | {1,2,3}  | {"region=us-east1,az=b","region=us-east1,az=c","region=us-east1,az=d"}             | {3,1,2}         | {}                  | {}               | 2262-04-11 23:47:16.854776
  …/1/"los angeles"                                                                 | …/1/"los angeles"/PrefixEnd                                                       |      115 | {4,5,6}  | {"region=us-west1,az=a","region=us-west1,az=b","region=us-west1,az=c"}             | {5,4,6}         | {}                  | {}               | NULL
  …/1/"new york"                                                                    | …/1/"new york"/"\x11\x11\x11\x11\x11\x11A\x00\x80\x00\x00\x00\x00\x00\x00\x01"    |      117 | {1,2,3}  | {"region=us-east1,az=b","region=us-east1,az=c","region=us-east1,az=d"}             | {3,1,2}         | {}                  | {}               | NULL
  …/1/"new york"/"\x11\x11\x11\x11\x11\x11A\x00\x80\x00\x00\x00\x00\x00\x00\x01"    | …/1/"new york"/PrefixEnd                                                          |       73 | {1,2,3}  | {"region=us-east1,az=b","region=us-east1,az=c","region=us-east1,az=d"}             | {3,2,1}         | {}                  | {}               | 2262-04-11 23:47:16.854776
  …/1/"paris"                                                                       | …/1/"paris"/PrefixEnd                                                             |      134 | {7,8,9}  | {"region=europe-west1,az=b","region=europe-west1,az=c","region=europe-west1,az=d"} | {7,9,8}         | {}                  | {}               | NULL
  …/1/"rome"                                                                        | …/1/"rome"/PrefixEnd                                                              |      137 | {7,8,9}  | {"region=europe-west1,az=b","region=europe-west1,az=c","region=europe-west1,az=d"} | {7,9,8}         | {}                  | {}               | NULL
  …/1/"san francisco"                                                               | …/1/"san francisco"/"wwwwwwH\x00\x80\x00\x00\x00\x00\x00\x00\a"                   |      199 | {4,5,6}  | {"region=us-west1,az=a","region=us-west1,az=b","region=us-west1,az=c"}             | {6,4,5}         | {}                  | {}               | NULL
  …/1/"san francisco"/"wwwwwwH\x00\x80\x00\x00\x00\x00\x00\x00\a"                   | …/1/"san francisco"/"\x88\x88\x88\x88\x88\x88H\x00\x80\x00\x00\x00\x00\x00\x00\b" |      109 | {4,5,6}  | {"region=us-west1,az=a","region=us-west1,az=b","region=us-west1,az=c"}             | {5,6,4}         | {}                  | {}               | 2262-04-11 23:47:16.854776
  …/1/"san francisco"/"\x88\x88\x88\x88\x88\x88H\x00\x80\x00\x00\x00\x00\x00\x00\b" | …/1/"san francisco"/PrefixEnd                                                     |       72 | {4,5,6}  | {"region=us-west1,az=a","region=us-west1,az=b","region=us-west1,az=c"}             | {4,5,6}         | {}                  | {}               | 2262-04-11 23:47:16.854776
  …/1/"seattle"                                                                     | …/1/"seattle"/"UUUUUUD\x00\x80\x00\x00\x00\x00\x00\x00\x05"                       |       94 | {4,5,6}  | {"region=us-west1,az=a","region=us-west1,az=b","region=us-west1,az=c"}             | {5,4,6}         | {}                  | {}               | NULL
  …/1/"seattle"/"UUUUUUD\x00\x80\x00\x00\x00\x00\x00\x00\x05"                       | …/1/"seattle"/"ffffffH\x00\x80\x00\x00\x00\x00\x00\x00\x06"                       |       91 | {4,5,6}  | {"region=us-west1,az=a","region=us-west1,az=b","region=us-west1,az=c"}             | {5,4,6}         | {}                  | {}               | 2262-04-11 23:47:16.854776
  …/1/"seattle"/"ffffffH\x00\x80\x00\x00\x00\x00\x00\x00\x06"                       | …/1/"seattle"/PrefixEnd                                                           |       90 | {4,5,6}  | {"region=us-west1,az=a","region=us-west1,az=b","region=us-west1,az=c"}             | {5,6,4}         | {}                  | {}               | 2262-04-11 23:47:16.854776
  …/1/"washington dc"                                                               | …/1/"washington dc"/"DDDDDDD\x00\x80\x00\x00\x00\x00\x00\x00\x04"                 |      128 | {1,2,3}  | {"region=us-east1,az=b","region=us-east1,az=c","region=us-east1,az=d"}             | {3,2,1}         | {}                  | {}               | NULL
  …/1/"washington dc"/"DDDDDDD\x00\x80\x00\x00\x00\x00\x00\x00\x04"                 | …/1/"washington dc"/PrefixEnd                                                     |      119 | {1,2,3}  | {"region=us-east1,az=b","region=us-east1,az=c","region=us-east1,az=d"}             | {3,1,2}         | {}                  | {}               | 2262-04-11 23:47:16.854776
  …/2/"amsterdam"                                                                   | …/2/"amsterdam"/PrefixEnd                                                         |      139 | {7,8,9}  | {"region=europe-west1,az=b","region=europe-west1,az=c","region=europe-west1,az=d"} | {9,8,7}         | {}                  | {}               | NULL
  …/2/"boston"                                                                      | …/2/"boston"/PrefixEnd                                                            |      141 | {1,2,3}  | {"region=us-east1,az=b","region=us-east1,az=c","region=us-east1,az=d"}             | {3,1,2}         | {}                  | {}               | NULL
  …/2/"los angeles"                                                                 | …/2/"los angeles"/PrefixEnd                                                       |      143 | {4,5,6}  | {"region=us-west1,az=a","region=us-west1,az=b","region=us-west1,az=c"}             | {6,5,4}         | {}                  | {}               | NULL
  …/2/"new york"                                                                    | …/2/"new york"/PrefixEnd                                                          |      145 | {1,2,3}  | {"region=us-east1,az=b","region=us-east1,az=c","region=us-east1,az=d"}             | {3,2,1}         | {}                  | {}               | NULL
  …/2/"paris"                                                                       | …/2/"paris"/PrefixEnd                                                             |      147 | {7,8,9}  | {"region=europe-west1,az=b","region=europe-west1,az=c","region=europe-west1,az=d"} | {9,8,7}         | {}                  | {}               | NULL
  …/2/"rome"                                                                        | …/2/"rome"/PrefixEnd                                                              |      319 | {7,8,9}  | {"region=europe-west1,az=b","region=europe-west1,az=c","region=europe-west1,az=d"} | {9,8,7}         | {}                  | {}               | NULL
  …/2/"san francisco"                                                               | …/2/"san francisco"/PrefixEnd                                                     |      321 | {4,5,6}  | {"region=us-west1,az=a","region=us-west1,az=b","region=us-west1,az=c"}             | {6,5,4}         | {}                  | {}               | NULL
  …/2/"seattle"                                                                     | …/2/"seattle"/PrefixEnd                                                           |      323 | {4,5,6}  | {"region=us-west1,az=a","region=us-west1,az=b","region=us-west1,az=c"}             | {5,6,4}         | {}                  | {}               | NULL
  …/2/"washington dc"                                                               | …/2/"washington dc"/PrefixEnd                                                     |      325 | {1,2,3}  | {"region=us-east1,az=b","region=us-east1,az=c","region=us-east1,az=d"}             | {3,2,1}         | {}                  | {}               | NULL
(27 rows)
~~~

### Show ranges for an index

{% include_cached copy-clipboard.html %}
~~~ sql
WITH x AS (SHOW RANGES FROM INDEX vehicles_auto_index_fk_city_ref_users) SELECT * FROM x WHERE "start_key" NOT LIKE '%Prefix%';
~~~

~~~
     start_key     |          end_key           | range_id | range_size_mb | lease_holder |  lease_holder_locality   | replicas |                                 replica_localities
+------------------+----------------------------+----------+---------------+--------------+--------------------------+----------+------------------------------------------------------------------------------------+
  /"washington dc" | /"washington dc"/PrefixEnd |      188 |      0.000089 |            2 | region=us-east1,az=c     | {1,2,3}  | {"region=us-east1,az=b","region=us-east1,az=c","region=us-east1,az=d"}
  /"boston"        | /"boston"/PrefixEnd        |      141 |      0.000164 |            3 | region=us-east1,az=d     | {1,2,3}  | {"region=us-east1,az=b","region=us-east1,az=c","region=us-east1,az=d"}
  /"new york"      | /"new york"/PrefixEnd      |      168 |      0.000174 |            3 | region=us-east1,az=d     | {1,2,3}  | {"region=us-east1,az=b","region=us-east1,az=c","region=us-east1,az=d"}
  /"los angeles"   | /"los angeles"/PrefixEnd   |      165 |      0.000087 |            6 | region=us-west1,az=c     | {4,5,6}  | {"region=us-west1,az=a","region=us-west1,az=b","region=us-west1,az=c"}
  /"san francisco" | /"san francisco"/PrefixEnd |      174 |      0.000183 |            6 | region=us-west1,az=c     | {4,5,6}  | {"region=us-west1,az=a","region=us-west1,az=b","region=us-west1,az=c"}
  /"seattle"       | /"seattle"/PrefixEnd       |      186 |      0.000166 |            6 | region=us-west1,az=c     | {4,5,6}  | {"region=us-west1,az=a","region=us-west1,az=b","region=us-west1,az=c"}
  /"amsterdam"     | /"amsterdam"/PrefixEnd     |      137 |       0.00017 |            9 | region=europe-west1,az=d | {7,8,9}  | {"region=europe-west1,az=b","region=europe-west1,az=c","region=europe-west1,az=d"}
  /"paris"         | /"paris"/PrefixEnd         |      170 |      0.000162 |            9 | region=europe-west1,az=d | {7,8,9}  | {"region=europe-west1,az=b","region=europe-west1,az=c","region=europe-west1,az=d"}
  /"rome"          | /"rome"/PrefixEnd          |      172 |       0.00008 |            9 | region=europe-west1,az=d | {7,8,9}  | {"region=europe-west1,az=b","region=europe-west1,az=c","region=europe-west1,az=d"}
(9 rows)
~~~

Starting with CockroachDB v23.1, the above output is deprecated.  You can enable the output shown below by changing the [cluster setting](cluster-settings.html) `sql.show_ranges_deprecated_behavior.enabled` to `false`. This output will become the default in CockroachDB v23.2 and later.

[XXX](): WRITE STUFF HERE

### Show ranges for a database

{% include_cached copy-clipboard.html %}
~~~ sql
> WITH x as (SHOW RANGES FROM database movr) SELECT * FROM x WHERE "start_key" NOT LIKE '%Prefix%';
~~~
~~~
          table_name         |    start_key     |          end_key           | range_id | range_size_mb | lease_holder |  lease_holder_locality   | replicas |                                 replica_localities
+----------------------------+------------------+----------------------------+----------+---------------+--------------+--------------------------+----------+------------------------------------------------------------------------------------+
  users                      | /"amsterdam"     | /"amsterdam"/PrefixEnd     |       47 |      0.000562 |            7 | region=europe-west1,az=b | {7,8,9}  | {"region=europe-west1,az=b","region=europe-west1,az=c","region=europe-west1,az=d"}
  users                      | /"boston"        | /"boston"/PrefixEnd        |       51 |      0.000665 |            3 | region=us-east1,az=d     | {1,2,3}  | {"region=us-east1,az=b","region=us-east1,az=c","region=us-east1,az=d"}
  users                      | /"chicago"       | /"los angeles"             |       83 |             0 |            4 | region=us-west1,az=a     | {2,4,8}  | {"region=us-east1,az=c","region=us-west1,az=a","region=europe-west1,az=c"}
  users                      | /"los angeles"   | /"los angeles"/PrefixEnd   |       45 |      0.000697 |            4 | region=us-west1,az=a     | {4,5,6}  | {"region=us-west1,az=a","region=us-west1,az=b","region=us-west1,az=c"}
  users                      | /"new york"      | /"new york"/PrefixEnd      |       48 |      0.000664 |            1 | region=us-east1,az=b     | {1,2,3}  | {"region=us-east1,az=b","region=us-east1,az=c","region=us-east1,az=d"}
  users                      | /"paris"         | /"paris"/PrefixEnd         |       52 |      0.000628 |            8 | region=europe-west1,az=c | {7,8,9}  | {"region=europe-west1,az=b","region=europe-west1,az=c","region=europe-west1,az=d"}

  ...

  user_promo_codes           | /"washington dc" | /"washington dc"/PrefixEnd |      144 |             0 |            2 | region=us-east1,az=c     | {1,2,3}  | {"region=us-east1,az=b","region=us-east1,az=c","region=us-east1,az=d"}
(73 rows)
~~~

## See also

- [`SHOW RANGE ... FOR ROW`](show-range-for-row.html)
- [`ALTER TABLE ... SPLIT AT`](alter-table.html#split-at)
- [`ALTER INDEX ... SPLIT AT`](alter-index.html#split-at)
- [`CREATE TABLE`](create-table.html)
- [`CREATE INDEX`](create-index.html)
- [Indexes](indexes.html)
- [Partitioning tables](partitioning.html)
- [Architecture Overview](architecture/overview.html)
