# Subquery in MySQL

A subquery is a query **nested inside another query**.
It runs first and passes its result to the outer query.

---

## Types of Subquery Placement

| Where Used  | Purpose                                      |
|-------------|----------------------------------------------|
| `SELECT` (column) | Calculate a value per row               |
| `WHERE IN`  | Filter rows based on a list from inner query |
| `FROM`      | Use query result as a temporary table        |

---

## 1. Subquery in SELECT (Column)

Used to calculate an extra value for each row.

```sql
SELECT
    first_name,
    age,
    (SELECT AVG(age) FROM employee_demographics) AS avg_age
FROM employee_demographics;
```

| first_name | age | avg_age |
|------------|-----|---------|
| John       | 25  | 34.4    |
| Alice      | 28  | 34.4    |
| Mark       | 52  | 34.4    |
| Anna       | 22  | 34.4    |
| Robert     | 45  | 34.4    |

> The subquery runs once and returns a single value (34.4).
> That value is shown alongside every row.

### Compare each employee age to the average

```sql
SELECT
    first_name,
    age,
    (SELECT AVG(age) FROM employee_demographics) AS avg_age,
    age - (SELECT AVG(age) FROM employee_demographics) AS diff_from_avg
FROM employee_demographics;
```

| first_name | age | avg_age | diff_from_avg |
|------------|-----|---------|---------------|
| John       | 25  | 34.4    | -9.4          |
| Alice      | 28  | 34.4    | -6.4          |
| Mark       | 52  | 34.4    | +17.6         |
| Anna       | 22  | 34.4    | -12.4         |
| Robert     | 45  | 34.4    | +10.6         |

---

## 2. Subquery in WHERE IN

Used to filter rows where a column value exists in the result of the inner query.

```sql
-- Get employees who have a salary record
SELECT first_name, age
FROM employee_demographics
WHERE emp_id IN (SELECT emp_id FROM employee_salary);
```

| first_name | age |
|------------|-----|
| John       | 25  |
| Alice      | 28  |
| Mark       | 52  |

> Inner query returns `(1, 2, 3)` — outer query keeps only those emp_ids.

### NOT IN — exclude matched rows

```sql
-- Get employees who do NOT have a salary record
SELECT first_name, age
FROM employee_demographics
WHERE emp_id NOT IN (SELECT emp_id FROM employee_salary);
```

| first_name | age |
|------------|-----|
| Anna       | 22  |
| Robert     | 45  |

---

### WHERE with a single value subquery

```sql
-- Get employees older than the average age
SELECT first_name, age
FROM employee_demographics
WHERE age > (SELECT AVG(age) FROM employee_demographics);
```

| first_name | age |
|------------|-----|
| Mark       | 52  |
| Robert     | 45  |

> Subquery returns one value (34.4). Outer query compares each row's age to it.

---

## 3. Subquery in FROM

The inner query result is used as a **temporary table** (must be given an alias).

```sql
-- Get genders where avg age is above 30
SELECT gender, avg_age
FROM (
    SELECT gender, AVG(age) AS avg_age
    FROM employee_demographics
    GROUP BY gender
) AS age_summary
WHERE avg_age > 30;
```

| gender | avg_age |
|--------|---------|
| Male   | 34.5    |

> Inner query groups and aggregates first.
> Outer query filters on that result using `WHERE` — which normally cannot filter on aggregates directly.
> Alias `age_summary` is **required** — MySQL will error without it.

### Another FROM subquery — join with derived table

```sql
SELECT ed.first_name, ed.age, top_earners.salary
FROM employee_demographics ed
INNER JOIN (
    SELECT emp_id, salary
    FROM employee_salary
    WHERE salary > 50000
) AS top_earners ON ed.emp_id = top_earners.emp_id;
```

| first_name | age | salary |
|------------|-----|--------|
| Alice      | 28  | 60000  |
| Mark       | 52  | 90000  |

> The subquery in FROM filters salary records first, then joins with demographics.

---

## Subquery vs JOIN

```sql
-- Using Subquery
SELECT first_name FROM employee_demographics
WHERE emp_id IN (SELECT emp_id FROM employee_salary WHERE salary > 50000);

-- Using JOIN (same result)
SELECT ed.first_name
FROM employee_demographics ed
INNER JOIN employee_salary es ON ed.emp_id = es.emp_id
WHERE es.salary > 50000;
```

| first_name |
|------------|
| Alice      |
| Mark       |

> Both return the same result. JOIN is generally faster for large datasets.
> Subquery is more readable for simple filtering.

---

## Note

> - Subquery in `SELECT` must return **exactly one value** (scalar) — multiple rows will error
> - Subquery in `WHERE IN` can return **multiple rows** — used as a list
> - Subquery in `FROM` can return **multiple rows and columns** — acts as a table, must have alias
> - Subqueries run **inside out** — inner query executes first, result passed to outer
> - Avoid deeply nested subqueries (3+ levels) — hard to read and slow to run, prefer JOINs
> - `NOT IN` with a subquery that returns `NULL` will give unexpected results — use `NOT EXISTS` in that case
