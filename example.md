> **Short repo description:** Practical SQL recipes to inspect and document Supabase/PostgreSQL tables: columns, PK/FK (inbound & outbound), enums, indexes, triggers, and dependencies.

---

# Supabase SQL Introspection Kit

A collection of battle-tested SQL queries and helper functions to analyze and document your Supabase (PostgreSQL) schema. It covers table structure (columns, nullability, defaults), constraints (PK, UNIQUE, CHECK), foreign keys (outbound and incoming “referencing tables”), enum types and values, indexes, triggers (via `information_schema` and `pg_catalog`), object dependencies (views, materialized views, routines), and comments—ready to run in the Supabase SQL Editor or `psql`. Includes an optional one-shot `inspect_table(schema, table)` function to generate a comprehensive report.

## Highlights

* **Structure:** columns, data types, nullability, defaults
* **Constraints:** PK, UNIQUE, CHECK (with full definitions)
* **Foreign keys:** outbound and **incoming** (referencing tables)
* **Enums:** enum types and their values used by a table
* **Indexes:** names, definitions, coverage
* **Triggers:** names, timing, events, target function
* **Dependencies:** views/materialized views/functions that reference a table
* **Two flavors:** readable `information_schema` queries + “pro” `pg_*` catalog versions

## Repository layout (suggested)

```
/sql
  columns.sql
  constraints.sql
  fks_outbound.sql
  fks_inbound.sql
  enums_by_table.sql
  indexes.sql
  triggers_info_schema.sql
  triggers_pg_catalog.sql
  dependencies_views.sql
  dependencies_functions.sql

/functions
  inspect_table.sql   # optional: one-shot report function
```

## Quick start

### Run in Supabase SQL Editor

Copy any script from `/sql` and run it directly in the SQL Editor.

### Run with psql

```bash
psql "$SUPABASE_DB_URL" -f sql/columns.sql
```

## Usage examples

### Table structure

```sql
SELECT column_name, data_type, is_nullable, column_default
FROM information_schema.columns
WHERE table_schema = 'public' AND table_name = 'your_table'
ORDER BY ordinal_position;
```

### Foreign keys (outbound from `your_table`)

```sql
SELECT
  tc.table_name AS source_table,
  kcu.column_name AS source_column,
  ccu.table_name AS target_table,
  ccu.column_name AS target_column,
  tc.constraint_name
FROM information_schema.table_constraints tc
JOIN information_schema.key_column_usage kcu
  ON tc.constraint_name = kcu.constraint_name
JOIN information_schema.constraint_column_usage ccu
  ON ccu.constraint_name = tc.constraint_name
WHERE tc.constraint_type = 'FOREIGN KEY'
  AND tc.table_schema = 'public'
  AND tc.table_name = 'your_table'
ORDER BY target_table, source_column;
```

### Incoming foreign keys (tables referencing `your_table`)

```sql
SELECT
  tc.table_name      AS referencing_table,
  kcu.column_name    AS referencing_column,
  ccu.table_name     AS referenced_table,
  ccu.column_name    AS referenced_column,
  tc.constraint_name
FROM information_schema.table_constraints tc
JOIN information_schema.key_column_usage kcu
  ON tc.constraint_name = kcu.constraint_name
JOIN information_schema.constraint_column_usage ccu
  ON ccu.constraint_name = tc.constraint_name
WHERE tc.constraint_type = 'FOREIGN KEY'
  AND ccu.table_schema = 'public'
  AND ccu.table_name = 'your_table'
ORDER BY referencing_table, referencing_column;
```

### Enums used by `your_table`

```sql
SELECT 
  c.table_name,
  c.column_name,
  c.udt_name AS enum_name,
  e.enumlabel AS enum_value
FROM information_schema.columns c
JOIN pg_type t ON t.typname = c.udt_name
JOIN pg_enum e ON t.oid = e.enumtypid
WHERE c.table_schema = 'public'
  AND c.table_name = 'your_table'
ORDER BY c.column_name, e.enumlabel;
```

### Triggers (information_schema version)

```sql
SELECT 
  trigger_name,
  event_object_table AS table_name,
  event_manipulation AS event,
  action_timing AS timing,
  action_statement AS function_called
FROM information_schema.triggers
WHERE event_object_table = 'your_table'
ORDER BY trigger_name;
```

### Dependencies – views that reference `your_table`

```sql
SELECT view_schema, view_name
FROM information_schema.view_table_usage
WHERE table_schema = 'public'
  AND table_name = 'your_table'
ORDER BY view_schema, view_name;
```

## Optional: one-shot inspector function

Install `/functions/inspect_table.sql` and then:

```sql
SELECT * FROM inspect_table('public', 'your_table');
```

This returns a consolidated report (structure, constraints, FKs, enums, indexes, triggers, dependencies) in a set of result tables.

## Naming conventions

* **Outbound foreign keys**: FKs defined **by** `your_table`
* **Inbound/Incoming foreign keys**: FKs **referencing** `your_table`
* Also referred to as **referencing tables** or **incoming FKs**

## Compatibility

* PostgreSQL (Supabase) ≥ 13
* Works with custom schemas (defaults to `public`—adjust filters as needed)

## License

MIT
