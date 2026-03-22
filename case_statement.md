# CASE Statement in MySQL

`CASE` is like an `if-else` inside a SQL query.
It evaluates conditions and returns a value based on the first match.

---

## Syntax

### Simple CASE — compare one column to values
```sql
CASE column
    WHEN value1 THEN result1
    WHEN value2 THEN result2
    ELSE default_result
END
```

### Searched CASE — evaluate conditions freely
```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ELSE default_result
END
```

> `ELSE` is optional — if no condition matches and no `ELSE`, returns `NULL`.

---

## Example 1 — Label Gender

```sql
SELECT first_name,
    CASE gender
        WHEN 'Male'   THEN 'M'
        WHEN 'Female' THEN 'F'
        ELSE 'Unknown'
    END AS gender_short
FROM employee_demographics;
```

| first_name | gender_short |
|------------|--------------|
| John       | M            |
| Alice      | F            |
| Mark       | M            |

---

## Example 2 — Age Group Categorization

One of the most common real-world use cases — bucketing numeric data into labels.

```sql
SELECT first_name, age,
    CASE
        WHEN age < 25 THEN 'Junior'
        WHEN age BETWEEN 25 AND 40 THEN 'Mid-Level'
        WHEN age > 40 THEN 'Senior'
    END AS age_group
FROM employee_demographics;
```

| first_name | age | age_group |
|------------|-----|-----------|
| John       | 25  | Mid-Level |
| Alice      | 28  | Mid-Level |
| Mark       | 52  | Senior    |
| Anna       | 22  | Junior    |
| Robert     | 45  | Senior    |

---

## Example 3 — Salary Band

```sql
SELECT first_name, salary,
    CASE
        WHEN salary < 40000 THEN 'Low'
        WHEN salary BETWEEN 40000 AND 70000 THEN 'Medium'
        WHEN salary > 70000 THEN 'High'
    END AS salary_band
FROM employee_demographics ed
INNER JOIN employee_salary es ON ed.emp_id = es.emp_id;
```

| first_name | salary | salary_band |
|------------|--------|-------------|
| John       | 50000  | Medium      |
| Alice      | 60000  | Medium      |
| Mark       | 90000  | High        |

---

## Example 4 — CASE with ORDER BY

Sort by a custom priority instead of alphabetical or numeric order.

```sql
SELECT first_name, gender
FROM employee_demographics
ORDER BY
    CASE gender
        WHEN 'Female' THEN 1
        WHEN 'Male'   THEN 2
        ELSE 3
    END;
```

| first_name | gender |
|------------|--------|
| Alice      | Female |
| Anna       | Female |
| John       | Male   |
| Mark       | Male   |
| Robert     | Male   |

> Females appear first not because of alphabetical order but because of the custom priority defined in CASE.

---

## Example 5 — CASE with GROUP BY (Pivot-like)

Count employees in each age group — commonly used for reporting.

```sql
SELECT
    COUNT(CASE WHEN age < 25 THEN 1 END)              AS junior_count,
    COUNT(CASE WHEN age BETWEEN 25 AND 40 THEN 1 END) AS mid_count,
    COUNT(CASE WHEN age > 40 THEN 1 END)              AS senior_count
FROM employee_demographics;
```

| junior_count | mid_count | senior_count |
|--------------|-----------|--------------|
| 1            | 2         | 2            |

> Each `CASE` acts as a conditional counter — returns `1` when condition matches, `NULL` otherwise.
> `COUNT` ignores `NULL` so only matched rows are counted.

---

## Example 6 — CASE in UPDATE

Update values conditionally without multiple UPDATE statements.

```sql
UPDATE employee_salary
SET salary =
    CASE
        WHEN salary < 40000 THEN salary * 1.20   -- 20% raise for low earners
        WHEN salary BETWEEN 40000 AND 70000 THEN salary * 1.10  -- 10% raise
        ELSE salary * 1.05                        -- 5% raise for high earners
    END;
```

> One single `UPDATE` handles all salary tiers at once.

---

## Common Use Cases

| Use Case                        | How CASE helps                              |
|---------------------------------|---------------------------------------------|
| Categorize numeric data         | Age groups, salary bands, score grades      |
| Custom sort order               | Priority-based ORDER BY                     |
| Conditional counting (pivot)    | Count per category in one row               |
| Label/display formatting        | Show 'M'/'F' instead of 'Male'/'Female'     |
| Conditional updates             | Different logic per row in one UPDATE       |
| Replace multiple queries        | One query with branching instead of many    |

---

## Note

> - MySQL evaluates conditions **top to bottom** and stops at the first match
> - Always put the most specific condition first
> - `ELSE` is optional but recommended — without it, unmatched rows return `NULL`
> - `CASE` can be used in `SELECT`, `WHERE`, `ORDER BY`, `GROUP BY`, and `UPDATE`
> - Simple `CASE` (comparing one column) is cleaner but Searched `CASE` is more flexible
> - `CASE` always ends with `END` — forgetting it causes a syntax error
