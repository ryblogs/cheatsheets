# PostgreSQL vs DuckDB SQL Cheatsheet

A side-by-side comparison of common SQL operations in PostgreSQL and DuckDB.

## Table of Contents
- [Basic Queries](#basic-queries)
- [Data Types & Casting](#data-types--casting)
- [JSON Operations](#json-operations)
- [Date & Time Operations](#date--time-operations)
- [String Functions](#string-functions)
- [Aggregate Functions](#aggregate-functions)
- [Window Functions](#window-functions)
- [Array/List Operations](#arraylist-operations)
- [CTEs & Subqueries](#ctes--subqueries)
- [Other Operations](#other-operations)

---

## Basic Queries

| Operation | PostgreSQL | DuckDB |
|-----------|-----------|---------|
| **Select all** | `SELECT * FROM table;` | `SELECT * FROM table;` |
| **Limit results** | `SELECT * FROM table LIMIT 10;` | `SELECT * FROM table LIMIT 10;` |
| **Offset** | `SELECT * FROM table OFFSET 5;` | `SELECT * FROM table OFFSET 5;` |
| **Distinct** | `SELECT DISTINCT column FROM table;` | `SELECT DISTINCT column FROM table;` |
| **Order by** | `SELECT * FROM table ORDER BY col DESC;` | `SELECT * FROM table ORDER BY col DESC;` |
| **Qualified table names** | `SELECT * FROM "schema"."table";` | `SELECT * FROM "database"."schema"."table";` |
| **Case sensitivity** | Case-insensitive unless quoted | Case-insensitive unless quoted |

---

## Data Types & Casting

| Operation | PostgreSQL | DuckDB |
|-----------|-----------|---------|
| **PostgreSQL-style cast** | `column::INTEGER` | `column::INTEGER` |
| **SQL standard cast** | `CAST(column AS INTEGER)` | `CAST(column AS INTEGER)` |
| **String to int** | `'123'::INT` | `'123'::INT` |
| **Date type** | `DATE '2023-01-01'` | `DATE '2023-01-01'` |
| **Timestamp** | `TIMESTAMP '2023-01-01 12:00:00'` | `TIMESTAMP '2023-01-01 12:00:00'` |
| **Boolean** | `TRUE`, `FALSE` | `TRUE`, `FALSE` |
| **JSON type** | `JSONB` | `JSON` |
| **Array type** | `INTEGER[]` | `INTEGER[]` or `LIST` |
| **Division** | `5 / 2` = `2` (integer division) | `5 / 2` = `2.5` (float division) |
| **Integer division** | `5 / 2` | `5 // 2` |
| **Equality** | `=` only | `=` or `==` (both work) |

**Notes:**
- DuckDB performs implicit casting more liberally than PostgreSQL
- DuckDB division always returns float; use `//` for integer division
- Both support `::type` casting syntax (PostgreSQL-style)

---

## JSON Operations

| Operation | PostgreSQL | DuckDB |
|-----------|-----------|---------|
| **Extract as JSON** | `data->'field'` | `data->'field'` or `json_extract(data, '$.field')` |
| **Extract as text** | `data->>'field'` | `data->>'field'` or `json_extract_string(data, '$.field')` |
| **Nested field** | `data->'obj'->'field'` | `data->'obj'->'field'` or `json_extract(data, '$.obj.field')` |
| **Array element** | `data->'array'->0` | `data->'array'->0` or `json_extract(data, '$.array[0]')` |
| **Array length** | `jsonb_array_length(data->'array')` | `json_array_length(data->'array')` |
| **JSONPath notation** | `data->'$.field'` (limited support) | `data->'$.field'` or `data->'field'` (both work) |
| **Keys from object** | `jsonb_object_keys(data)` | `json_keys(data)` |
| **Type checking** | `jsonb_typeof(data)` | `json_type(data)` |
| **Filter by JSON field** | `WHERE (data->>'number')::int > 5` | `WHERE (data->>'number')::int > 5` |
| **Array length filter** | `WHERE jsonb_array_length(data->'arr') > 3` | `WHERE json_array_length(data->'arr') > 3` |

**Notes:**
- PostgreSQL uses `jsonb_*` functions; DuckDB uses `json_*` functions
- DuckDB supports both JSONPath (`$.field`) and shorthand (`field`) notation
- DuckDB JSON uses 0-based indexing (unlike PostgreSQL arrays which are 1-based)
- Both `->` and `->>` work similarly in both databases

---

## Date & Time Operations

| Operation | PostgreSQL | DuckDB |
|-----------|-----------|---------|
| **Current date** | `CURRENT_DATE` | `CURRENT_DATE` |
| **Current timestamp** | `CURRENT_TIMESTAMP` or `NOW()` | `CURRENT_TIMESTAMP` or `NOW()` |
| **Current time** | `CURRENT_TIME` | `CURRENT_TIME` |
| **Add days** | `date + 5` or `date + INTERVAL '5 days'` | `date + 5` or `date + INTERVAL 5 DAY` |
| **Subtract days** | `date - 5` or `date - INTERVAL '5 days'` | `date - 5` or `date - INTERVAL 5 DAY` |
| **Date difference** | `date1 - date2` (returns integer days) | `date1 - date2` (returns integer days) |
| **Extract year** | `EXTRACT(YEAR FROM date)` or `date_part('year', date)` | `EXTRACT(YEAR FROM date)` or `date_part('year', date)` |
| **Extract month** | `EXTRACT(MONTH FROM date)` | `EXTRACT(MONTH FROM date)` |
| **Date truncation** | `date_trunc('month', timestamp)` | `date_trunc('month', timestamp)` |
| **Format date** | `to_char(date, 'YYYY-MM-DD')` | `strftime(date, '%Y-%m-%d')` |
| **Parse date string** | `to_date('2023-01-01', 'YYYY-MM-DD')` | `strptime('2023-01-01', '%Y-%m-%d')` |
| **Day of week** | `EXTRACT(DOW FROM date)` (0=Sunday) | `EXTRACT(DOW FROM date)` (0=Sunday) |
| **Epoch timestamp** | `EXTRACT(EPOCH FROM timestamp)` | `EXTRACT(EPOCH FROM timestamp)` |
| **From epoch (ms)** | `to_timestamp(ms / 1000.0)` | `epoch_ms(ms)` |

**Notes:**
- PostgreSQL uses `to_char`/`to_date`; DuckDB uses `strftime`/`strptime`
- Both support standard SQL `EXTRACT` and `date_part` functions
- Interval syntax differs slightly: PostgreSQL uses quotes around units

---

## String Functions

| Operation | PostgreSQL | DuckDB |
|-----------|-----------|---------|
| **Concatenation** | `str1 \|\| str2` or `CONCAT(str1, str2)` | `str1 \|\| str2` or `CONCAT(str1, str2)` |
| **Uppercase** | `UPPER(str)` | `UPPER(str)` |
| **Lowercase** | `LOWER(str)` | `LOWER(str)` |
| **Length** | `LENGTH(str)` or `CHAR_LENGTH(str)` | `LENGTH(str)` or `LEN(str)` |
| **Substring** | `SUBSTRING(str, start, length)` | `SUBSTRING(str, start, length)` |
| **Replace** | `REPLACE(str, 'old', 'new')` | `REPLACE(str, 'old', 'new')` |
| **Trim** | `TRIM(str)` | `TRIM(str)` |
| **Left trim** | `LTRIM(str)` | `LTRIM(str)` |
| **Right trim** | `RTRIM(str)` | `RTRIM(str)` |
| **Split string** | `string_to_array(str, ',')` | `string_split(str, ',')` or `str_split(str, ',')` |
| **String contains** | `str LIKE '%pattern%'` | `str LIKE '%pattern%'` |
| **Regex match** | `str ~ 'pattern'` | `regexp_full_match(str, 'pattern')` |
| **Regex extract** | `regexp_substr(str, 'pattern')` | `regexp_extract(str, 'pattern')` |
| **Position/index** | `POSITION('sub' IN str)` | `POSITION('sub' IN str)` or `STRPOS(str, 'sub')` |

**Notes:**
- Both support standard SQL string functions
- DuckDB's regex functions differ: `~` maps to `regexp_full_match`
- DuckDB returns empty strings instead of NULLs in `regexp_extract` when no match

---

## Aggregate Functions

| Operation | PostgreSQL | DuckDB |
|-----------|-----------|---------|
| **Count** | `COUNT(*)` | `COUNT(*)` |
| **Sum** | `SUM(column)` | `SUM(column)` |
| **Average** | `AVG(column)` | `AVG(column)` |
| **Min/Max** | `MIN(column)`, `MAX(column)` | `MIN(column)`, `MAX(column)` |
| **String aggregation** | `STRING_AGG(column, ',')` | `STRING_AGG(column, ',')` or `LIST_AGG(column, ',')` |
| **Array aggregation** | `ARRAY_AGG(column)` | `ARRAY_AGG(column)` or `LIST(column)` |
| **Distinct count** | `COUNT(DISTINCT column)` | `COUNT(DISTINCT column)` |
| **Standard deviation** | `STDDEV(column)` | `STDDEV(column)` |
| **Variance** | `VARIANCE(column)` | `VARIANCE(column)` |
| **Median** | `PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY col)` | `MEDIAN(column)` or `QUANTILE_CONT(column, 0.5)` |
| **Mode** | `MODE() WITHIN GROUP (ORDER BY col)` | `MODE(column)` |
| **Filter clause** | `SUM(amount) FILTER (WHERE status = 'active')` | `SUM(amount) FILTER (WHERE status = 'active')` |
| **Group by** | `SELECT col, COUNT(*) FROM t GROUP BY col;` | `SELECT col, COUNT(*) FROM t GROUP BY col;` |
| **Having** | `HAVING COUNT(*) > 5` | `HAVING COUNT(*) > 5` |

**Notes:**
- Both support standard SQL aggregate functions
- DuckDB has simpler syntax for median/quantiles
- Both support the FILTER clause for conditional aggregation

---

## Window Functions

| Operation | PostgreSQL | DuckDB |
|-----------|-----------|---------|
| **Row number** | `ROW_NUMBER() OVER (ORDER BY col)` | `ROW_NUMBER() OVER (ORDER BY col)` |
| **Rank** | `RANK() OVER (ORDER BY col)` | `RANK() OVER (ORDER BY col)` |
| **Dense rank** | `DENSE_RANK() OVER (ORDER BY col)` | `DENSE_RANK() OVER (ORDER BY col)` |
| **Lag** | `LAG(col) OVER (ORDER BY date)` | `LAG(col) OVER (ORDER BY date)` |
| **Lead** | `LEAD(col) OVER (ORDER BY date)` | `LEAD(col) OVER (ORDER BY date)` |
| **First value** | `FIRST_VALUE(col) OVER (...)` | `FIRST_VALUE(col) OVER (...)` |
| **Last value** | `LAST_VALUE(col) OVER (...)` | `LAST_VALUE(col) OVER (...)` |
| **Partition by** | `OVER (PARTITION BY group ORDER BY col)` | `OVER (PARTITION BY group ORDER BY col)` |
| **Running total** | `SUM(amount) OVER (ORDER BY date)` | `SUM(amount) OVER (ORDER BY date)` |
| **Moving average** | `AVG(val) OVER (ORDER BY date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)` | `AVG(val) OVER (ORDER BY date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)` |
| **Named window** | `WINDOW w AS (PARTITION BY group)` then `OVER w` | `WINDOW w AS (PARTITION BY group)` then `OVER w` |
| **Qualify clause** | Not supported (use subquery/CTE) | `QUALIFY ROW_NUMBER() OVER (...) = 1` |
| **Frame types** | `ROWS`, `RANGE`, `GROUPS` | `ROWS`, `RANGE`, `GROUPS` |

**Notes:**
- Both support standard window functions
- DuckDB has the `QUALIFY` clause to filter window results directly (no CTE needed)
- Both support complex frame specifications with `ROWS`, `RANGE`, and `GROUPS`

---

## Array/List Operations

| Operation | PostgreSQL | DuckDB |
|-----------|-----------|---------|
| **Create array literal** | `ARRAY[1, 2, 3]` or `'{1,2,3}'::int[]` | `[1, 2, 3]` or `LIST_VALUE(1, 2, 3)` |
| **Access element** | `array[1]` (1-indexed) | `list[1]` (1-indexed) or `list[0]` (0-indexed with lists) |
| **Array length** | `array_length(array, 1)` or `CARDINALITY(array)` | `array_length(list)` or `LEN(list)` |
| **Array concatenation** | `array1 \|\| array2` | `list1 \|\| list2` or `list_concat(list1, list2)` |
| **Unnest array** | `SELECT * FROM unnest(ARRAY[1,2,3])` | `SELECT * FROM unnest([1,2,3])` |
| **Contains element** | `value = ANY(array)` | `list_contains(list, value)` or `value IN list` |
| **Array aggregation** | `ARRAY_AGG(column)` | `LIST(column)` or `ARRAY_AGG(column)` |
| **Array slice** | `array[1:3]` | `list[1:3]` or `list_slice(list, 1, 3)` |
| **Array position** | `array_position(array, element)` | `list_position(list, element)` |
| **Filter array** | N/A (use unnest + filter) | `list_filter(list, x -> x > 5)` |
| **Transform array** | N/A (use unnest + transform) | `list_transform(list, x -> x * 2)` |

**Notes:**
- PostgreSQL arrays are 1-indexed; DuckDB lists can be 0 or 1-indexed depending on context
- DuckDB has more built-in list manipulation functions
- DuckDB supports lambda functions for list operations
- Both use `unnest()` to expand arrays/lists into rows

---

## CTEs & Subqueries

| Operation | PostgreSQL | DuckDB |
|-----------|-----------|---------|
| **Basic CTE** | `WITH cte AS (SELECT ...) SELECT * FROM cte;` | `WITH cte AS (SELECT ...) SELECT * FROM cte;` |
| **Multiple CTEs** | `WITH cte1 AS (...), cte2 AS (...) SELECT ...` | `WITH cte1 AS (...), cte2 AS (...) SELECT ...` |
| **Recursive CTE** | `WITH RECURSIVE cte AS (...) SELECT ...` | `WITH RECURSIVE cte AS (...) SELECT ...` |
| **Subquery in SELECT** | `SELECT (SELECT ...) AS sub FROM ...` | `SELECT (SELECT ...) AS sub FROM ...` |
| **Subquery in WHERE** | `WHERE col IN (SELECT ...)` | `WHERE col IN (SELECT ...)` |
| **Correlated subquery** | `WHERE EXISTS (SELECT 1 FROM t2 WHERE t2.id = t1.id)` | `WHERE EXISTS (SELECT 1 FROM t2 WHERE t2.id = t1.id)` |

**Notes:**
- Both fully support CTEs and recursive queries
- Syntax is identical for most CTE operations

---

## Other Operations

| Operation | PostgreSQL | DuckDB |
|-----------|-----------|---------|
| **Create table** | `CREATE TABLE t (id INT, name TEXT);` | `CREATE TABLE t (id INT, name TEXT);` |
| **Insert** | `INSERT INTO t VALUES (1, 'Alice');` | `INSERT INTO t VALUES (1, 'Alice');` |
| **Update** | `UPDATE t SET name = 'Bob' WHERE id = 1;` | `UPDATE t SET name = 'Bob' WHERE id = 1;` |
| **Delete** | `DELETE FROM t WHERE id = 1;` | `DELETE FROM t WHERE id = 1;` |
| **Upsert** | `INSERT ... ON CONFLICT ... DO UPDATE ...` | `INSERT OR REPLACE INTO ...` or `INSERT ... ON CONFLICT ... DO UPDATE ...` |
| **COALESCE** | `COALESCE(col, 'default')` | `COALESCE(col, 'default')` or `col.ifnull('default')` |
| **NULLIF** | `NULLIF(col1, col2)` | `NULLIF(col1, col2)` |
| **CASE** | `CASE WHEN ... THEN ... ELSE ... END` | `CASE WHEN ... THEN ... ELSE ... END` |
| **Join types** | `INNER`, `LEFT`, `RIGHT`, `FULL OUTER`, `CROSS` | `INNER`, `LEFT`, `RIGHT`, `FULL OUTER`, `CROSS` |
| **ASOF join** | Limited support | `FROM t1 ASOF JOIN t2 ON t1.time >= t2.time` |
| **Lateral join** | `FROM t1, LATERAL (SELECT ...) t2` | `FROM t1, LATERAL (SELECT ...) t2` |
| **Union** | `SELECT ... UNION SELECT ...` | `SELECT ... UNION SELECT ...` |
| **Union all** | `SELECT ... UNION ALL SELECT ...` | `SELECT ... UNION ALL SELECT ...` |
| **Intersect** | `SELECT ... INTERSECT SELECT ...` | `SELECT ... INTERSECT SELECT ...` |
| **Except** | `SELECT ... EXCEPT SELECT ...` | `SELECT ... EXCEPT SELECT ...` |
| **Pivot** | `crosstab()` extension | `PIVOT table ON column USING agg(value)` |
| **Query from file** | `\copy` or `COPY` command | `SELECT * FROM 'file.csv'` or `FROM 'file.parquet'` |
| **Explain query** | `EXPLAIN SELECT ...` | `EXPLAIN SELECT ...` |
| **Transaction** | `BEGIN; ... COMMIT;` | `BEGIN; ... COMMIT;` |

**Notes:**
- DuckDB can query files directly (CSV, Parquet, JSON) without loading
- DuckDB has native PIVOT support; PostgreSQL requires extensions
- Both support standard SQL joins and set operations
- DuckDB's ASOF join is more powerful for time-series data

---

## Key Differences Summary

### When to use PostgreSQL:
- Need ACID transactions with high concurrency
- Building multi-user applications
- Require extensive extensions ecosystem
- Need stored procedures and triggers
- Production OLTP workloads

### When to use DuckDB:
- Analytical queries on large datasets
- Embedded analytics in applications
- Data science and exploration work
- Processing Parquet, CSV files directly
- Single-user analytical workloads
- Need for columnar storage benefits

### Syntax Compatibility:
DuckDB is highly PostgreSQL-compatible, supporting:
- PostgreSQL-style casting (`::type`)
- Most PostgreSQL functions and syntax
- Window functions and CTEs
- JSON operations (with slightly different function names)

### Main Syntax Differences:
1. **Division**: PostgreSQL does integer division; DuckDB does float division (use `//` for integer division)
2. **Date formatting**: PostgreSQL uses `to_char`/`to_date`; DuckDB uses `strftime`/`strptime`
3. **JSON functions**: PostgreSQL uses `jsonb_*`; DuckDB uses `json_*`
4. **Equality**: DuckDB supports both `=` and `==`; PostgreSQL only `=`
5. **QUALIFY clause**: DuckDB has it; PostgreSQL doesn't
6. **File querying**: DuckDB can query files directly; PostgreSQL cannot

---

## Resources

- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [DuckDB Documentation](https://duckdb.org/docs/)
- [DuckDB PostgreSQL Compatibility](https://duckdb.org/docs/sql/dialect/postgresql_compatibility)

---

## Contributing

Feel free to submit issues or pull requests to improve this cheatsheet!

## License

This cheatsheet is provided as-is for educational purposes.
