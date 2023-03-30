CockroachDB implements SQL {% if page.name == "known-limitations.md" %} [cursor](cursors.html) {% else %} cursor {% endif %} support with the following limitations:

- `DECLARE` only supports forward cursors.
- `FETCH` supports forward, relative, and absolute variants, but only for forward cursors.
- `BINARY CURSOR`, which returns data in the Postgres binary format, is not supported.
- `MOVE`, which allows advancing the cursor without returning any rows, is not supported.
- `WITH HOLD`, which allows keeping a cursor open for longer than a transaction by writing its results into a buffer, is not supported.
- Scrollable cursor (also known as reverse `FETCH`) is not supported.
- [`SELECT ... FOR UPDATE`](select-for-update.html) is not supported.
- Respect for [`SAVEPOINT`s](savepoint.html) is not supported. Cursor definitions do not disappear properly if rolled back to a `SAVEPOINT` from before they were created.
