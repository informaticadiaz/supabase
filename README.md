# supabase
Practical SQL recipes to inspect and document Supabase/PostgreSQL tables: columns, PK/FK (inbound &amp; outbound), enums, indexes, triggers, and dependencies.

Supabase SQL Introspection Kit is a collection of battle-tested SQL queries and helper functions to analyze and document your Supabase (PostgreSQL) schema. It covers table structure (columns, nullability, defaults), constraints (PK, UNIQUE, CHECK), foreign keys (outbound and incoming “referencing tables”), enum types and values, indexes, triggers (via information_schema and pg_catalog), object dependencies (views, materialized views, routines), and comments—ready to run in the Supabase SQL Editor or psql. Includes an optional one-shot inspect_table(schema, table) function to generate a comprehensive report.

## Highlights

- Table structure: columns, types, nullability, defaults

- Constraints: PK, UNIQUE, CHECK definitions

- Foreign keys: outbound and incoming (referencing tables)

- Enums: all enum types and values used by a table

- Indexes: definitions and coverage

- Triggers: names, timing, events, target function

- Dependencies: views/matviews/functions that reference a table

- Readable outputs using information_schema, with “pro” variants using pg_* catalogs

## Usage

Run any script from /sql directly in the Supabase SQL Editor.

Or install the optional /functions/inspect_table.sql and call:
SELECT * FROM inspect_table('public', 'your_table');

## Compatibility

PostgreSQL (Supabase) >= 13

Works with custom schemas (defaults to public—adjust as needed)

## License
MIT
