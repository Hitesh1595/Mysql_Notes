# DISTINCT in SQL

Removes duplicate rows from the result set.

## Single Field

```sql
SELECT DISTINCT city FROM users;
```

| city     |
|----------|
| Mumbai   |
| Delhi    |
| Chennai  |

---

## Two Fields — how it works

With 2 fields, DISTINCT treats the **combination** of both columns as one unit. A row is only removed if **both values** are identical.

```sql
SELECT DISTINCT city, role FROM users;
```

**Raw data:**

| city   | role  |
|--------|-------|
| Mumbai | Admin |
| Mumbai | User  |
| Delhi  | User  |
| Delhi  | User  |

**Result after DISTINCT:**

| city   | role  |
|--------|-------|
| Mumbai | Admin |
| Mumbai | User  |
| Delhi  | User  |

> `Delhi, User` appeared twice — removed to one. `Mumbai` rows kept both because `role` differs.

---

## DISTINCT vs GROUP BY

| DISTINCT | GROUP BY |
|----------|----------|
| Only removes duplicates | Groups rows to apply aggregate functions (`COUNT`, `SUM`, etc.) |
| No aggregation | Used with `COUNT`, `MAX`, etc. |

```sql
-- DISTINCT: just deduplicate
SELECT DISTINCT city FROM users;

-- GROUP BY: count per city (different purpose)
SELECT city, COUNT(*) FROM users GROUP BY city;
```

> Use **DISTINCT** when you only want unique rows.
> Use **GROUP BY** when you need to aggregate data.
