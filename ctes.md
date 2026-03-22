# CTEs (Common Table Expressions) in MySQL

A CTE is a **temporary named result set** defined at the top of a query using `WITH`.
It exists only for the duration of that query.

---

## Syntax

```sql
WITH cte_name AS (
    -- your subquery here
)
SELECT * FROM cte_name;
```

---

## CTE vs Subquery

```sql
-- Subquery in FROM (harder to read)
SELECT gender, avg_age
FROM (
    SELECT gender, AVG(age) AS avg_age
    FROM employee_demographics
    GROUP BY gender
) AS age_summary
WHERE avg_age > 30;

-- Same with CTE (cleaner)
WITH age_summary AS (
    SELECT gender, AVG(age) AS avg_age
    FROM employee_demographics
    GROUP BY gender
)
SELECT gender, avg_age
FROM age_summary
WHERE avg_age > 30;
```

> Both return the same result. CTE is easier to read and reuse.

---

## Example 1 — Basic CTE

Get employees older than the average age.

```sql
WITH avg_age_cte AS (
    SELECT AVG(age) AS avg_age FROM employee_demographics
)
SELECT first_name, age
FROM employee_demographics, avg_age_cte
WHERE age > avg_age_cte.avg_age;
```

| first_name | age |
|------------|-----|
| Mark       | 52  |
| Robert     | 45  |

---

## Example 2 — CTE with JOIN

```sql
WITH salary_info AS (
    SELECT emp_id, salary, job_title
    FROM employee_salary
    WHERE salary > 40000
)
SELECT ed.first_name, ed.gender, si.job_title, si.salary
FROM employee_demographics ed
INNER JOIN salary_info si ON ed.emp_id = si.emp_id;
```

| first_name | gender | job_title        | salary |
|------------|--------|------------------|--------|
| John       | Male   | Data Analyst     | 50000  |
| Alice      | Female | HR Manager       | 60000  |
| Mark       | Male   | Senior Developer | 90000  |

---

## Example 3 — Multiple CTEs

You can define more than one CTE by separating them with a comma.

```sql
WITH
male_employees AS (
    SELECT emp_id, first_name, age
    FROM employee_demographics
    WHERE gender = 'Male'
),
high_salary AS (
    SELECT emp_id, salary
    FROM employee_salary
    WHERE salary > 60000
)
SELECT me.first_name, me.age, hs.salary
FROM male_employees me
INNER JOIN high_salary hs ON me.emp_id = hs.emp_id;
```

| first_name | age | salary |
|------------|-----|--------|
| Mark       | 52  | 90000  |
| Robert     | 45  | 80000  |

> First CTE filters male employees.
> Second CTE filters high salary records.
> Final query joins both CTEs together.

---

## Example 4 — CTE with Window Function

CTEs pair well with window functions to filter on ranked results.

```sql
WITH ranked_salary AS (
    SELECT
        ed.first_name,
        ed.gender,
        es.salary,
        RANK() OVER(PARTITION BY ed.gender ORDER BY es.salary DESC) AS rnk
    FROM employee_demographics ed
    INNER JOIN employee_salary es ON ed.emp_id = es.emp_id
)
SELECT first_name, gender, salary
FROM ranked_salary
WHERE rnk = 1;
```

| first_name | gender | salary |
|------------|--------|--------|
| Alice      | Female | 60000  |
| Mark       | Male   | 90000  |

> CTE computes the rank first.
> Outer query simply filters `WHERE rnk = 1` — top earner per gender.
> This is cleaner than nesting a window function inside a subquery.

---

## Example 5 — CTE reused multiple times

A CTE can be referenced more than once in the same query — unlike a subquery.

```sql
WITH dept_avg AS (
    SELECT dept_name, AVG(salary) AS avg_salary
    FROM employee_salary es
    INNER JOIN department d ON es.emp_id = d.emp_id
    GROUP BY dept_name
)
SELECT
    d1.dept_name,
    d1.avg_salary,
    (SELECT MAX(avg_salary) FROM dept_avg) AS highest_avg
FROM dept_avg d1;
```

| dept_name   | avg_salary | highest_avg |
|-------------|------------|-------------|
| Analytics   | 50000      | 90000       |
| HR          | 60000      | 90000       |
| Engineering | 85000      | 90000       |

> `dept_avg` CTE is defined once but referenced twice in the outer query.

---

## When to Use CTE

| Situation                                | Use CTE                        |
|------------------------------------------|--------------------------------|
| Subquery used more than once             | Yes — define once, reuse       |
| Query has deeply nested subqueries       | Yes — flatten with CTEs        |
| Window function result needs filtering   | Yes — wrap in CTE, then filter |
| Breaking a complex query into steps      | Yes — one CTE per logical step |
| Simple one-time filter                   | Subquery is fine               |

---

## Note

> - CTE is defined with `WITH` and lasts only for that single query
> - Multiple CTEs are separated by commas, all under one `WITH`
> - CTE can reference a previously defined CTE in the same `WITH` block
> - Unlike a subquery in `FROM`, a CTE can be referenced multiple times
> - CTEs do not improve performance by themselves — they are a readability tool
> - For recursive queries (hierarchical data), MySQL supports `WITH RECURSIVE`
