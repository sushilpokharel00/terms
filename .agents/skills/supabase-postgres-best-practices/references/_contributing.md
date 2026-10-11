# Contribution Guidelines for Postgres References

Use these guidelines when creating or updating a Postgres best-practice
reference. Each reference should give AI agents and developers a clear,
accurate, and actionable path from a problematic pattern to a better one.

## Authoring Principles

### 1. Show Concrete Transformations

Describe the specific change to make and show it in SQL or application code.
Avoid advice that is too broad to act on.

**Good:** "Add a partial index for rows that match the query's active-record
filter."

**Avoid:** "Design good schemas."

### 2. Present the Problem Before the Solution

Show the problematic pattern first, explain its impact, and then provide the
recommended alternative. This structure helps readers recognize when the
guidance applies.

```markdown
**Problematic (sequential queries):**
[Example of the problematic pattern]

**Recommended (batched query):**
[Example of the improved pattern]
```

### 3. Quantify Impact Responsibly

Use concrete measurements when they are available, and include the workload,
dataset, or conditions behind them. Do not present estimates or illustrative
figures as guaranteed results.

**Good:** "In a test with 1 million rows, the partial index was 50% smaller
than the full index."

**Avoid:** "This is much faster."

### 4. Make Examples Self-Contained

Include enough context for readers to understand and adapt each example. When
the schema affects the behavior, provide the relevant table or column
definitions.

```sql
create table users (
  id bigint PRIMARY KEY,
  email text NOT NULL,
  deleted_at timestamptz
);

create index users_active_email_idx
  on users (email)
  where deleted_at is null;
```

### 5. Use Meaningful Names

Choose names that communicate the role of each table, column, and object.

**Good:** `users`, `email`, `created_at`, `is_active`

**Avoid:** `table1`, `col1`, `field`, `flag`

---

## Code Example Standards

### SQL Formatting

Use lowercase SQL keywords, consistent indentation, and clear line breaks.

```sql
create index users_active_email_idx
  on users (email)
  where deleted_at is null;
```

Avoid cramped formatting or inconsistent capitalization:

```sql
CREATE INDEX USERS_EMAIL_IDX ON USERS(EMAIL) WHERE DELETED_AT IS NULL;
```

### Comments

- Explain why a pattern matters rather than restating what the code does.
- Call out relevant performance implications and common pitfalls.
- Keep comments concise and ensure they remain accurate as examples change.

### Language Tags

Use the fenced-code language tag that matches the example:

- `sql` for SQL statements and queries
- `plpgsql` for PL/pgSQL functions
- `typescript` for TypeScript application code
- `python` for Python application code

---

## When to Include Application Code

Keep references SQL-focused by default. Include application code when the
guidance depends on application behavior, such as connection pooling,
transaction management, ORM query patterns, or prepared statements.

For mixed examples, label each example by the problem it demonstrates and the
recommended approach:

````markdown
**Problematic (N+1 queries in application code):**

```typescript
for (const user of users) {
  const posts = await db.query("select * from posts where user_id = $1", [
    user.id,
  ]);
}
```

**Recommended (fetch related rows in one query):**

```typescript
const posts = await db.query("select * from posts where user_id = any($1)", [
  userIds,
]);
```
````

---

## Impact Level Guidance

Choose an impact level that reflects the likely benefit and scope of the
problem. Treat the ranges below as guidance, not a promise; actual results
depend on the workload, data distribution, and database configuration.

| Level | Typical improvement | Example use cases |
|-------|--------------------|-------------------|
| **CRITICAL** | 10-100x | Missing indexes or connection exhaustion under significant load |
| **HIGH** | 5-20x | An unsuitable index type or an inefficient access strategy |
| **MEDIUM-HIGH** | 2-5x | N+1 queries, inefficient pagination, or costly RLS policies |
| **MEDIUM** | 1.5-3x | Redundant indexes or avoidable query-plan instability |
| **LOW-MEDIUM** | 1.2-2x | Vacuum or configuration tuning |
| **LOW** | Incremental or workload-specific | Advanced patterns and specialized edge cases |

Do not assign a level based only on this table. Explain the conditions that
make the impact relevant, and avoid numeric claims unless they are supported
by a measurement or a clearly identified source.

---

## Sources and Related References

Prefer primary documentation and verify that each linked source supports the
guidance in the reference.

- [PostgreSQL documentation](https://www.postgresql.org/docs/current/)
- [Supabase documentation](https://supabase.com/docs)
- [PostgreSQL wiki: Performance Optimization](https://wiki.postgresql.org/wiki/Performance_Optimization)
- [Supabase database overview](https://supabase.com/docs/guides/database/overview)
- [Supabase Row Level Security guide](https://supabase.com/docs/guides/auth/row-level-security)
- [Reference template](./_template.md)
- [Postgres reference sections](./_sections.md)

---

## Review Checklist

Before submitting a reference, confirm that:

- [ ] The title is clear and action-oriented.
- [ ] The impact level is appropriate, and any quantified claim is supported.
- [ ] The impact description states relevant conditions or evidence.
- [ ] The explanation is concise and describes why the guidance matters.
- [ ] At least one problematic example and one recommended example are included.
- [ ] SQL and application-code examples use meaningful names and correct language tags.
- [ ] Comments explain why a pattern matters without duplicating the code.
- [ ] Relevant trade-offs and edge cases are addressed.
- [ ] Source links are relevant, accurate, and accessible.
- [ ] The reference follows the [reference template](./_template.md) and is assigned to the appropriate [section](./_sections.md).
- [ ] `pnpm test` passes.
