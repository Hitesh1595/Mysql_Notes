# HAVING in MySQL

`HAVING` filters rows **after** `GROUP BY` has been applied.
It is used with aggregate functions (`COUNT`, `AVG`, `MAX`, `MIN`, `SUM`).

---

## Why not WHERE?

| Clause  | Filters             | Works with aggregates? |
|---------|---------------------|------------------------|
| `WHERE` | Individual rows     | No                     |
| `HAVING`| Grouped results     | Yes                    |

`WHERE` runs **before** grouping — it cannot see aggregated values.
`HAVING` runs **after** grouping — it can filter on `AVG`, `COUNT`, etc.

---

## Example 1 — HAVING with AVG

Find genders where the average age is greater than 30.

```sql
SELECT gender, AVG(age) AS avg_age
FROM employee_demographics
GROUP BY gender
HAVING AVG(age) > 30;
```

| gender | avg_age |
|--------|---------|
| Male   | 34.5    |

> Female avg_age was 30.2 — not greater than 30, so excluded.

---

## Example 2 — HAVING with COUNT

Find genders that have more than 3 employees.

```sql
SELECT gender, COUNT(*) AS total
FROM employee_demographics
GROUP BY gender
HAVING COUNT(*) > 3;
```

| gender | total |
|--------|-------|
| Male   | 6     |

---

## Example 3 — HAVING with MAX Age

Find genders where the oldest employee is above 50.

```sql
SELECT gender, MAX(age) AS oldest
FROM employee_demographics
GROUP BY gender
HAVING MAX(age) > 50;
```

| gender | oldest |
|--------|--------|
| Male   | 52     |

---

## WHERE vs HAVING — Used Together

`WHERE` filters rows first, then `GROUP BY` groups, then `HAVING` filters groups.

```sql
SELECT gender, AVG(age) AS avg_age
FROM employee_demographics
WHERE age > 25
GROUP BY gender
HAVING AVG(age) > 30;
```

**Step-by-step execution:**

1. `WHERE age > 25` — removes employees aged 25 or below from the dataset
2. `GROUP BY gender` — groups remaining rows by gender
3. `HAVING AVG(age) > 30` — keeps only groups where average age exceeds 30

| gender | avg_age |
|--------|---------|
| Male   | 36.2    |

---

## Execution Order in MySQL

```
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
```

> `WHERE` acts on raw rows before any grouping.
> `HAVING` acts on grouped results after `GROUP BY`.

---

## Wrong Way — Using WHERE with Aggregate (will error)

```sql
-- This will FAIL
SELECT gender, AVG(age)
FROM employee_demographics
WHERE AVG(age) > 30
GROUP BY gender;
```

```
ERROR 1111: Invalid use of group function
```

> You cannot use aggregate functions inside `WHERE`. Use `HAVING` instead.

---

## Note

> - Use `WHERE` to filter individual rows before grouping
> - Use `HAVING` to filter groups after `GROUP BY`
> - `HAVING` without `GROUP BY` is valid but rare — treats entire table as one group
> - You can use column alias in `HAVING` in some MySQL versions but using the full aggregate is safer
> - Always remember the execution order: `WHERE` → `GROUP BY` → `HAVING`
