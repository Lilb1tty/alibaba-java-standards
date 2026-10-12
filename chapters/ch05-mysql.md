# Chapter 5: MySQL and ORM Conventions

**Source:** Printed pp. 38-43: Schema 38; Indexes 39-40; SQL 41; ORM 42-43. **Levels:** M = mandatory, R = recommended, F = reference.

## Core Idea

Put stable meaning and integrity in schemas and explicit mappings. Choose indexes from actual access patterns and inspect execution behavior rather than assuming an index guarantees speed.

## Techniques and Criteria

- **Business uniqueness — M, Indexes rule 1:** Identify the complete business key, including composite columns. Enforce it with a unique index even when application validation exists. Concurrent callers can both pass check-then-insert; a database constraint arbitrates the conflict.
- **Index discrimination — M/R, Indexes rules 3/9:** Choose string prefix length from `COUNT(DISTINCT LEFT(column, length)) / COUNT(*)`. The source cites length 20 often achieving over 90% discrimination, not a guarantee. Prefer high-discrimination columns but put equality predicates before ranges in mixed conditions. Check representative data and plans.
- **Covering index — R, Indexes rule 6:** Select required fields and determine whether the index supplies them without table lookup. This is a query-plan property, not a separate index type; the source points to `Using index` in EXPLAIN.
- **Delayed association — R, Indexes rule 7:** Locate page IDs through a narrow query, then fetch the wider rows. Deep offsets still scan/discard work; limit page depth or rewrite above a justified threshold. Preserve deterministic ordering.
- **Explicit ORM mapping — M, ORM rules 2/3:** Use resultMap for named columns and typed DO properties. Preserve `is_deleted` to `deleted` and check complete numeric domains.

## Schema Rules

**M:** Boolean columns use `is_xxx`, unsigned TINYINT, 1 for yes and 0 for no; Java properties omit `is`. Nonnegative fields use unsigned types in this edition. Names are lowercase letters/digits with meaningful underscores, never leading digits or a number alone between underscores (`level3_name`, not `level_3_name`). Tables are singular; avoid reserved words. Index naming uses `pk_`, `uk_`, and `idx_`. Runtime note: MySQL reports the primary index as `PRIMARY`; that engine behavior is distinct from the naming convention.

**M:** Fractional decimal columns use DECIMAL, not FLOAT/DOUBLE. Use CHAR for near-fixed-length strings. VARCHAR should not exceed 5000 under the edition; longer TEXT moves to a separate primary-key-linked table. Include `id`, `create_time`, `update_time`. The source specifies BIGINT UNSIGNED primary-key id, auto-increment step one for a single table, and DATETIME for both timestamps.

**R:** Name tables by business/purpose and align database/app naming. Keep column comments current. Limited redundancy can reduce queries if values change infrequently, are not unique-index fields, and are not huge VARCHAR/TEXT; define consistency ownership. Recommend sharding only beyond five million rows or 2 GB per table; avoid early sharding if the next three years will not approach that scale. These are source heuristics, not measured universal capacity limits.

**F:** Choose storage widths for actual ranges. The source uses utf8, with utf8mb4 for emoji; inspect current charset definitions rather than assuming utf8 means full Unicode. Distinguish byte and character lengths.

## Reference Tables

Reconstructed unsigned-range table from p. 38. Precise limits clarify the source's rounded large-number examples.

| Unsigned type | Bytes | Range |
|---|---|---|
| TINYINT | 1 | 0-255 |
| SMALLINT | 2 | 0-65,535 |
| INT | 4 | 0-4,294,967,295 |
| BIGINT | 8 | 0-18,446,744,073,709,551,615 |

Java Long ends at 9,223,372,036,854,775,807. The source's generic BIGINT/Long advice needs a validated-domain check for unsigned columns; choose a suitable representation or supported subset explicitly.

| Choice | Source preference | Verify |
|---|---|---|
| Fixed/variable text | CHAR for near-fixed; VARCHAR otherwise | Distribution and access needs |
| Long text | Beyond VARCHAR 5000, separate TEXT table | Edition/project policy |
| Composite index | Equality first, useful discrimination, matching order | EXPLAIN with representative data |
| Deep pages | Narrow IDs, then association | Stable ordering and retained offset cost |
| Money | Integer smallest units | Currency and overflow domain |
| Fractional decimals | DECIMAL | Required precision/scale |
| Logical deletion | `is_deleted` mapped to `deleted` | Explicit types and resultMap |

## Index and Query Rules

**M:** At most three joined tables. Joined types match exactly and appropriate join columns have indexes. VARCHAR indexes have deliberate prefix lengths under the source policy. For unique indexes, verify full business uniqueness: prefix collisions can reject distinct values, so document any necessary project exception. No leading-wildcard/fully fuzzy page searches; use a search engine where this edition requires it.

**R:** Align ORDER BY with composite index order. Equality predicates before the ordering column can preserve usable order; an earlier range can prevent it. Prefer covering queries and delayed association for deep pages. Avoid implicit type conversions that defeat index access. Target at least range, preferably ref and ideally const when appropriate. The source spells the last `consts`; actual EXPLAIN reports `const`. Access type `index` means an index scan, not proof of efficient lookup. Cardinality, sorting, actual rows, and latency still matter.

**F:** Avoid both “index every query” and “refuse indexes because writes cost more.” Do not replace required unique indexes with application-only checks.

## SQL Rules and Null Semantics

**M:** Use `COUNT(*)` for row counts under this edition. `COUNT(column)` excludes nulls; `COUNT(DISTINCT column)` counts distinct nonnull values. Multi-column distinct counting excludes rows with null components. SUM returns null when no usable values exist; default explicitly only when semantically correct, such as `IFNULL(SUM(amount_minor), 0)`.

**M:** The edition prefers `ISNULL(column)`. Runtime clarification: `IS NULL` is valid SQL; `= NULL` and `<> NULL` yield unknown. The source's ISNULL speed explanation is not a universal optimizer guarantee.

**M:** Return early when a pagination count is zero. This edition prohibits foreign keys/cascades and stored procedures for its distributed/high-concurrency context; implement referential behavior in application logic. These are source policies, not universal claims about SQL features. Before correction UPDATE/DELETE, SELECT and verify the exact target set. Qualify columns by table/alias whenever multiple tables participate, including selections and mutations.

**R:** Use `AS` and clear aliases, commonly t1/t2/t3. Avoid IN where appropriate; when used, keep its list within 1000 elements. **F:** LENGTH counts bytes, CHARACTER_LENGTH counts characters. Avoid application TRUNCATE: MySQL implicit-commit/no-delete-trigger behavior differs from rollback-safe DELETE.

## ORM and Transactions

**M:** Select named required fields rather than `*`; `COUNT(*)` is the separate aggregate rule. Define resultMap, including boolean mapping. Bind values with MyBatis `#{}` or legacy iBATIS `#param#`; do not use `${}` for user SQL values. Avoid iBATIS queryForList start/size pagination that retrieves everything before subList. Do not expose raw HashMap/Hashtable query outputs instead of typed mappings. Every record update also updates update_time to the current time.

**R:** Update only intended changed fields, not every column in a loosely populated POJO. **F:** Use `@Transactional` deliberately; consider rollback/compensation for caches, search indexes, messages, and statistics. Legacy iBATIS `isEqual`, `isNotEmpty`, `isNotNull` tags have distinct comparison/null semantics and are not modern MyBatis tag names.

## Key Concepts

- **Unique index:** Database enforcement of a business invariant.
- **Discrimination:** Fraction of distinct candidate values/prefixes.
- **Leftmost prefix:** Useful initial portion of a composite index.
- **Covering query:** Required data is available from index access.
- **Delayed association:** Select narrow keys, then fetch wider rows.
- **ResultMap:** Explicit column/property/type mapping.
- **SQL unknown:** Third logical result arising from null comparisons.
- **Partial update:** Mutate selected intended fields, preserving others.

## Mental Models

- Schema is a durable API; changing types/names is costly.
- An index is a query-specific path, not a general speed sticker.
- Missing information is not automatically zero.
- A database transaction does not automatically cover every downstream resource.

## Code Examples

Adaptations of ORM mapping/binding and SQL aggregate rules. The mapper fragment assumes the illustrated UserDO and is not a complete XML file.

```xml
<resultMap id="userMap" type="com.example.user.UserDO">
    <id column="id" property="id" />
    <result column="is_deleted" property="deleted" />
</resultMap>

<select id="getUser" resultMap="userMap">
    SELECT id, is_deleted
    FROM app_user
    WHERE id = #{id}
</select>
```

```sql
SELECT IFNULL(SUM(amount_minor), 0)
FROM payment_order
WHERE account_id = ?;
```

Bind account_id with the actual driver. Zero defaulting assumes the business meaning of no amounts is zero.

## Worked Example

Source scenario: an unqualified `name` in a multi-table query works until another table gains that column; then “Column 'name' in field list is ambiguous.” Prevent the ambiguity:

```sql
SELECT t1.name
FROM table_first AS t1
JOIN table_second AS t2 ON t1.id = t2.id;
```

1. Qualify all selected/filtered/updated columns in multi-table operations.
2. Verify join types and indexes, even for two tables.
3. Name required output fields and map them explicitly.
4. Inspect EXPLAIN and the schema-change case; old successful execution does not establish future unambiguity.

The source's delayed-association example selects `t1.*`; an implementation adaptation should name needed output fields in accordance with its stricter ORM rule.

## Anti-patterns

- Check-then-insert alone enforces uniqueness.
- Unsigned BIGINT becomes Integer or unvalidated Long.
- `${userValue}` inserts executable user SQL.
- SELECT * changes mappings implicitly after schema evolution.
- A large partial-update POJO resets untouched columns.
- An indexed query is approved without examining access/row counts.
- A correction changes data before its target SELECT is checked.

## Key Takeaways

1. Enforce uniqueness in the database.
2. Match numeric domains and column/property meanings.
3. Bind values and constrain query structure.
4. Inspect plans under the actual workload.
5. Label edition policies and platform caveats explicitly.

## Connects To

- [Ch01](ch01-programming.md): Money, wrappers, locking, and POJO defaults.
- [Ch02](ch02-errors-and-logging.md): Transaction catches and SQL failures.
- [Ch03](ch03-unit-testing.md): Repeatable fixtures/null boundaries.
- [Ch04](ch04-security.md): SQL injection and validation.
- [Ch07](ch07-design.md): Reviewed storage/schema changes.
