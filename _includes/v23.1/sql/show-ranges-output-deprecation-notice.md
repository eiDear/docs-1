Starting with CockroachDB v23.1, storage [ranges](architecture/overview.html#architecture-range) can be shared across multiple [indexes](indexes.html) and/or [tables](tables.html). As a result, there is no more guarantee that there is at most one [schema object](schema-design-overview.html) per range.

This will affect the output of:

- [`SHOW RANGES`](show-ranges.html)
- Any queries against the internal tables [`crdb_internal.ranges`](crdb-internal.html#ranges) and [`crdb_internal.ranges_no_leases`](crdb-internal.html#ranges_no_leases)

{% if page.title == crdb_internal %}
Because a range cannot be attributed to a single table or index any more, the following columns in `crdb_internal.ranges` and `crdb_internal.ranges_no_leases` have become meaningless, and will be removed in v23.2:
- `table_id`
- `database_name`
- `schema_name`
- `table_name`
- `index_name`

Similarly, the output of [`SHOW RANGES`]() will be changed to reflect this fact as well.  [xxx](): REWRITE THIS
{% endif %}

Therefore, the output shown on this page is deprecated.  It will continue to function in CockroachDB v23.1, but will be the subject of a backwards-incompatible change in v23.2.  You can enable the output that will become the default in v23.2 by changing the [cluster setting](cluster-settings.html) `sql.show_ranges_deprecated_behavior.enabled` to `false`.
