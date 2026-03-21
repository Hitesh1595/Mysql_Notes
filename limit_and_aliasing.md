# LIMIT and Aliasing in MySQL

---

## LIMIT

`LIMIT` restricts the number of rows returned by a query.

### Syntax

```sql
SELECT columns FROM table LIMIT n;
```

### Example 1 — Get Top 3 Employees by Age

```sql
SELECT first_name, age
FROM employee_demographics
ORDER BY age DESC
LIMIT 3;
```

| first_name | age |
|------------|-----|
| Mark       | 52  |
| Robert     | 45  |
| Karen      | 45  |

---

### Example 2 — Get Youngest Employee

```sql
SELECT first_name, age
FROM employee_demographics
ORDER BY age ASC
LIMIT 1;
```

| first_name | age |
|------------|-----|
| Anna       | 22  |

---

### Example 3 — LIMIT with OFFSET

`OFFSET` skips a number of rows before starting to return.

```sql
-- Skip first 2 rows, then return next 3
SELECT first_name, age
FROM employee_demographics
ORDER BY age DESC
LIMIT 3 OFFSET 2;
```

| first_name | age |
|------------|-----|
| Karen      | 45  |
| Alice      | 34  |
| Daniel     | 30  |

> Skipped top 2 (Mark 52, Robert 45), then returned the next 3.

Short syntax for `LIMIT offset, count`:

```sql
LIMIT 2, 3  -- same as LIMIT 3 OFFSET 2
```

---

### Example 4 — LIMIT with GROUP BY

```sql
-- Top 1 gender by count
SELECT gender, COUNT(*) AS total
FROM employee_demographics
GROUP BY gender
ORDER BY total DESC
LIMIT 1;
```

| gender | total |
|--------|-------|
| Male   | 6     |

---

## Aliasing

An alias gives a column or table a **temporary name** for that query.
It does not rename the actual column in the database.

### Syntax

```sql
SELECT column AS alias_name FROM table;
```

`AS` keyword is optional — both work:

```sql
SELECT age AS employee_age ...
SELECT age employee_age ...   -- same result
```

---

### Example 1 — Column Alias

```sql
SELECT first_name AS name, age AS years
FROM employee_demographics;
```

| name  | years |
|-------|-------|
| John  | 25    |
| Alice | 28    |
| Mark  | 52    |

---

### Example 2 — Alias on Aggregate

```sql
SELECT gender, AVG(age) AS avg_age, COUNT(*) AS total
FROM employee_demographics
GROUP BY gender;
```

| gender | avg_age | total |
|--------|---------|-------|
| Male   | 34.5    | 6     |
| Female | 30.2    | 4     |

> Without alias it would show `AVG(age)` as the column header — harder to read.

---

### Example 3 — Table Alias

Useful when joining tables or writing long queries.

```sql
SELECT ed.first_name, ed.age
FROM employee_demographics AS ed
WHERE ed.age > 30;
```

| first_name | age |
|------------|-----|
| Mark       | 52  |
| Robert     | 45  |
| Karen      | 45  |

> `ed` is the alias for `employee_demographics` — shorter to type in long queries.

---

### Example 4 — Alias in ORDER BY

```sql
SELECT first_name, age * 12 AS age_in_months
FROM employee_demographics
ORDER BY age_in_months DESC;
```

| first_name | age_in_months |
|------------|---------------|
| Mark       | 624           |
| Robert     | 540           |
| Alice      | 336           |

---

## Note

> - `LIMIT` always goes at the **end** of a query, after `ORDER BY`
> - Without `ORDER BY`, `LIMIT` returns arbitrary rows — always pair them
> - `OFFSET` is useful for **pagination** (page 1 = LIMIT 10 OFFSET 0, page 2 = LIMIT 10 OFFSET 10)
> - Aliases are only valid within the **same query** — cannot be reused in `WHERE` (use `HAVING` for aggregates)
> - `AS` keyword is optional but recommended for readability
> - Table aliases are especially useful in **JOIN** queries to avoid repeating long table names
