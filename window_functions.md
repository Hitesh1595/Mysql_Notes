# Window Functions in MySQL

A window function performs a calculation **across a set of rows related to the current row** without collapsing them into groups like `GROUP BY` does.

---

## Window Function vs GROUP BY

```sql
-- GROUP BY collapses rows
SELECT gender, AVG(salary) FROM employee_salary GROUP BY gender;
-- returns 2 rows (one per gender)

-- Window function keeps all rows
SELECT first_name, salary, AVG(salary) OVER(PARTITION BY gender) AS avg_salary
FROM employee_salary;
-- returns all rows + avg alongside each
```

> `GROUP BY` → fewer rows, aggregated
> Window function → same rows, extra calculated column added

---

## Core Syntax — OVER

`OVER()` defines the window (set of rows) the function operates on.

```sql
function_name() OVER (
    PARTITION BY column   -- divide rows into groups (optional)
    ORDER BY column       -- order within each group (optional)
)
```

| Clause         | Purpose                                      |
|----------------|----------------------------------------------|
| `PARTITION BY` | Split rows into groups (like GROUP BY but keeps all rows) |
| `ORDER BY`     | Define row order within each partition       |

---

## Tables Used

**employee_demographics**

| employee_id | first_name | last_name | gender |
|-------------|------------|-----------|--------|
| 1           | John       | Smith     | Male   |
| 2           | Alice      | Brown     | Female |
| 3           | Mark       | Johnson   | Male   |
| 4           | Anna       | White     | Female |
| 5           | Robert     | Davis     | Male   |

**employee_salary**

| employee_id | job_title        | salary |
|-------------|------------------|--------|
| 1           | Data Analyst     | 50000  |
| 2           | HR Manager       | 60000  |
| 3           | Senior Developer | 90000  |
| 4           | Recruiter        | 45000  |
| 5           | Team Lead        | 80000  |

---

## SUM OVER — Rolling Total

```sql
SELECT ed.first_name, ed.last_name, gender, salary,
    SUM(salary) OVER(PARTITION BY gender ORDER BY ed.employee_id) AS rolling_total
FROM employee_demographics ed
INNER JOIN employee_salary es
    ON ed.employee_id = es.employee_id;
```

| first_name | last_name | gender | salary | rolling_total |
|------------|-----------|--------|--------|---------------|
| Alice      | Brown     | Female | 60000  | 60000         |
| Anna       | White     | Female | 45000  | 105000        |
| John       | Smith     | Male   | 50000  | 50000         |
| Mark       | Johnson   | Male   | 90000  | 140000        |
| Robert     | Davis     | Male   | 80000  | 220000        |

> `PARTITION BY gender` → resets the running total for each gender
> `ORDER BY employee_id` → adds salary row by row in id order
> Each row shows the cumulative sum up to that point within the gender group

---

## ROW_NUMBER

Assigns a unique sequential number to each row within a partition. No ties — every row gets a different number.

```sql
SELECT first_name, gender, salary,
    ROW_NUMBER() OVER(PARTITION BY gender ORDER BY salary DESC) AS row_num
FROM employee_demographics ed
INNER JOIN employee_salary es ON ed.employee_id = es.employee_id;
```

| first_name | gender | salary | row_num |
|------------|--------|--------|---------|
| Alice      | Female | 60000  | 1       |
| Anna       | Female | 45000  | 2       |
| Mark       | Male   | 90000  | 1       |
| Robert     | Male   | 80000  | 2       |
| John       | Male   | 50000  | 3       |

> Row numbering restarts at 1 for each gender partition.
> Useful for getting the **top N rows per group**.

### Get top 1 salary per gender using ROW_NUMBER

```sql
SELECT first_name, gender, salary
FROM (
    SELECT first_name, gender, salary,
        ROW_NUMBER() OVER(PARTITION BY gender ORDER BY salary DESC) AS row_num
    FROM employee_demographics ed
    INNER JOIN employee_salary es ON ed.employee_id = es.employee_id
) AS ranked
WHERE row_num = 1;
```

| first_name | gender | salary |
|------------|--------|--------|
| Alice      | Female | 60000  |
| Mark       | Male   | 90000  |

---

## RANK

Assigns a rank to each row within a partition ordered by a column.
**Tied rows get the same rank** and the next rank is skipped.

```sql
SELECT first_name, gender, salary,
    RANK() OVER(PARTITION BY gender ORDER BY salary DESC) AS rnk
FROM employee_demographics ed
INNER JOIN employee_salary es ON ed.employee_id = es.employee_id;
```

| first_name | gender | salary | rnk |
|------------|--------|--------|-----|
| Alice      | Female | 60000  | 1   |
| Anna       | Female | 45000  | 2   |
| Mark       | Male   | 90000  | 1   |
| Robert     | Male   | 80000  | 2   |
| John       | Male   | 50000  | 3   |

> If two employees had the same salary, both get rank 1 and rank 2 is skipped — next is rank 3.

---

## DENSE_RANK

Same as `RANK` but **does not skip** rank numbers after a tie.

```sql
SELECT first_name, gender, salary,
    DENSE_RANK() OVER(PARTITION BY gender ORDER BY salary DESC) AS dense_rnk
FROM employee_demographics ed
INNER JOIN employee_salary es ON ed.employee_id = es.employee_id;
```

| first_name | gender | salary | dense_rnk |
|------------|--------|--------|-----------|
| Alice      | Female | 60000  | 1         |
| Anna       | Female | 45000  | 2         |
| Mark       | Male   | 90000  | 1         |
| Robert     | Male   | 80000  | 2         |
| John       | Male   | 50000  | 3         |

### RANK vs DENSE_RANK with a tie example

Two employees both earn 80000:

| first_name | salary | RANK | DENSE_RANK |
|------------|--------|------|------------|
| Mark       | 90000  | 1    | 1          |
| Robert     | 80000  | 2    | 2          |
| John       | 80000  | 2    | 2          |
| Alice      | 50000  | 4    | 3          |

> `RANK` skips 3 after the tie → jumps to 4
> `DENSE_RANK` does not skip → continues with 3

---

## All Four Together

```sql
SELECT ed.first_name, ed.last_name, gender, salary,
    ROW_NUMBER() OVER(PARTITION BY gender ORDER BY salary DESC) AS row_num,
    RANK()       OVER(PARTITION BY gender ORDER BY salary DESC) AS rnk,
    DENSE_RANK() OVER(PARTITION BY gender ORDER BY salary DESC) AS dense_rnk,
    SUM(salary)  OVER(PARTITION BY gender ORDER BY ed.employee_id) AS rolling_total
FROM employee_demographics ed
INNER JOIN employee_salary es
    ON ed.employee_id = es.employee_id;
```

| first_name | gender | salary | row_num | rnk | dense_rnk | rolling_total |
|------------|--------|--------|---------|-----|-----------|---------------|
| Alice      | Female | 60000  | 1       | 1   | 1         | 60000         |
| Anna       | Female | 45000  | 2       | 2   | 2         | 105000        |
| Mark       | Male   | 90000  | 1       | 1   | 1         | 50000         |
| Robert     | Male   | 80000  | 2       | 2   | 2         | 140000        |
| John       | Male   | 50000  | 3       | 3   | 3         | 220000        |

---

## LAG

Pulls value from **previous row** in partition. Good for row-to-row comparison (e.g. previous salary).

```sql
SELECT ed.first_name, gender, salary,
    LAG(salary) OVER(PARTITION BY gender ORDER BY ed.employee_id) AS prev_salary
FROM employee_demographics ed
INNER JOIN employee_salary es ON ed.employee_id = es.employee_id;
```

| first_name | gender | salary | prev_salary |
|------------|--------|--------|--------------|
| Alice      | Female | 60000  | NULL         |
| Anna       | Female | 45000  | 60000        |
| John       | Male   | 50000  | NULL         |
| Mark       | Male   | 90000  | 50000        |
| Robert     | Male   | 80000  | 90000        |

> First row per partition has no previous row -> `prev_salary` NULL
> Optional args: `LAG(column, offset, default)` -> e.g. `LAG(salary, 1, 0)` returns `0` instead of `NULL` for first row
> `LEAD()` same idea but pulls value from **next** row instead

### Consecutive Months Case

Common use: find employees active in **consecutive months** (e.g. login/sales log).

**employee_activity**

| employee_id | activity_month |
|-------------|-----------------|
| 1           | 2026-01         |
| 1           | 2026-02         |
| 1           | 2026-04         |
| 2           | 2026-01         |
| 2           | 2026-02         |
| 2           | 2026-03         |

```sql
SELECT employee_id, activity_month,
    LAG(activity_month) OVER(PARTITION BY employee_id ORDER BY activity_month) AS prev_month,
    PERIOD_DIFF(DATE_FORMAT(activity_month, '%Y%m'), DATE_FORMAT(LAG(activity_month) OVER(PARTITION BY employee_id ORDER BY activity_month), '%Y%m')) AS month_diff
FROM employee_activity;
```

| employee_id | activity_month | prev_month | month_diff |
|-------------|-----------------|------------|------------|
| 1           | 2026-01         | NULL       | NULL       |
| 1           | 2026-02         | 2026-01    | 1          |
| 1           | 2026-04         | 2026-02    | 2          |
| 2           | 2026-01         | NULL       | NULL       |
| 2           | 2026-02         | 2026-01    | 1          |
| 2           | 2026-03         | 2026-02    | 1          |

> `month_diff = 1` -> consecutive month, no gap
> `month_diff > 1` -> gap in activity (employee 1 skipped 2026-03)
> wrap in subquery + `WHERE month_diff = 1` to filter only consecutive-month rows
> `PERIOD_DIFF` needs `YYYYMM` format ints, hence `DATE_FORMAT(..., '%Y%m')`

**DATEDIFF for knowledge** — `DATEDIFF(date1, date2)` returns **days** between two dates (date1 - date2), not months/years. Use for consecutive-**day** checks instead:

```sql
SELECT employee_id, activity_date,
    LAG(activity_date) OVER(PARTITION BY employee_id ORDER BY activity_date) AS prev_date,
    DATEDIFF(activity_date, LAG(activity_date) OVER(PARTITION BY employee_id ORDER BY activity_date)) AS day_diff
FROM employee_activity;
```

> `day_diff = 1` -> consecutive day (streak logic, e.g. daily login streak)
> `PERIOD_DIFF` -> month granularity only, works on `YYYYMM` ints
> `DATEDIFF` -> day granularity, works on actual `DATE`/`DATETIME` values

---

## Quick Reference

| Function       | Ties handled        | Skips rank? | Use case                          |
|----------------|---------------------|-------------|-----------------------------------|
| `ROW_NUMBER()` | No ties, always unique | N/A      | Top N per group, pagination       |
| `RANK()`       | Same rank for ties  | Yes         | Competition-style ranking         |
| `DENSE_RANK()` | Same rank for ties  | No          | Ranking without gaps              |
| `SUM() OVER`   | N/A                 | N/A         | Running/rolling totals            |
| `LAG()`        | N/A                 | N/A         | Compare row to previous row       |

---

## Note

> - Window functions do **not** reduce rows — all original rows are kept
> - `PARTITION BY` is optional — omitting it treats the entire result as one window
> - `ORDER BY` inside `OVER()` is separate from `ORDER BY` at the end of the query
> - Window functions cannot be used in `WHERE` or `HAVING` — wrap in a subquery to filter on them
> - `SUM() OVER` without `ORDER BY` gives the total for the whole partition on every row
> - `SUM() OVER` with `ORDER BY` gives a cumulative/rolling total row by row
