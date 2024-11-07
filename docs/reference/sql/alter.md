# ALTER

`ALTER` can be used to modify any table settings or data within the table:

* Add/Drop/Modify a column
* Rename a table

## Syntax

```sql
ALTER TABLE [db.]table
   [ADD COLUMN name type [options]
    | DROP COLUMN name
    | MODIFY COLUMN name type
    | MODIFY COLUMN name SET FULLTEXT [WITH <options>]
    | RENAME name
   ]
```

## Examples

### Add column

Adds a new column to the table:

```sql
ALTER TABLE monitor ADD COLUMN load_15 double;
```

Definition of column is the same as in [CREATE](./create.md).

We can set the new column's location. In first position for example:

```sql
ALTER TABLE monitor ADD COLUMN load_15 double FIRST;
```

After an existing column:

```sql
ALTER TABLE monitor ADD COLUMN load_15 double AFTER memory;
```

Adds a new column as a tag(primary key) with a default value:

```sql
ALTER TABLE monitor ADD COLUMN app STRING DEFAULT 'shop' PRIMARY KEY;
```

### Remove column

Removes a column from the table:

```sql
ALTER TABLE monitor DROP COLUMN load_15;
```

The removed column can't be retrieved immediately by all subsequent queries.

### Modify column type

Modify the date type of a column

```sql
ALTER TABLE monitor MODIFY COLUMN load_15 STRING;
```

The modified column cannot be a tag (primary key) or time index, and it must be nullable to ensure that the data can be safely converted (returns `NULL` on cast failures).

### Modify column fulltext index options

Modify the fulltext index options of a column

```sql
ALTER TABLE monitor MODIFY COLUMN load_15 SET FULLTEXT WITH (enable = 'true', analyzer = 'Chinese', case_sensitive = 'false');
```

You can specify the following options using `FULLTEXT WITH`:

- `enable`: Determines whether the full-text index is enabled. Supported values are `true` and `false`. Default is `true`.
- `analyzer`: Sets the language analyzer for the full-text index. Supported values are `English` and `Chinese`. Default is `English`.
- `case_sensitive`: Determines whether the full-text index is case-sensitive. Supported values are `true` and `false`. Default is `false`.

If `WITH` is not specified, `FULLTEXT` will use the default values.

Currently, the fulltext index can be altered when:

1. The column is of a string type.
2. Either the column has no fulltext index and the alter command is to enable it, or the column has a fulltext index and the alter command is to disable it.

### Rename table

Renames the table:

```sql
ALTER TABLE monitor RENAME monitor_new;
```

This command only renames the table; it doesn't modify the data within the table.
