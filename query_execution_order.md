# SQL Query Execution Order in MySQL

When you write a SQL query, MySQL does **not** execute it in the order you write it.
It follows a fixed internal execution order.

---

## How You Write It vs How MySQL Runs It

### Written Order
```sql
SELECT ed.gender, AVG(ed.age) AS avg_age, es.job_title
FROM employee_demographics ed
INNER JOIN employee_salary es ON ed.emp_id = es.emp_id
WHERE ed.age > 22
GROUP BY ed.gender
HAVING AVG(ed.age) > 30
ORDER BY avg_age DESC
LIMIT 2;
```

### Execution Order
```
1. FROM + JOIN
2. WHERE
3. GROUP BY
4. HAVING
5. SELECT
6. ORDER BY
7. LIMIT
```

---

## Step-by-Step Breakdown

### Step 1 — FROM + JOIN
MySQL loads the table first, then immediately applies JOINs to combine tables.

```sql
FROM employee_demographics ed
INNER JOIN employee_salary es ON ed.emp_id = es.emp_id
```

**employee_demographics**

| emp_id | first_name | age | gender |
|--------|------------|-----|--------|
| 1      | John       | 25  | Male   |
| 2      | Alice      | 28  | Female |
| 3      | Mark       | 52  | Male   |
| 4      | Anna       | 22  | Female |
| 5      | Robert     | 45  | Male   |

**employee_salary**

| emp_id | job_title        | salary |
|--------|------------------|--------|
| 1      | Data Analyst     | 50000  |
| 2      | HR Manager       | 60000  |
| 3      | Senior Developer | 90000  |
| 6      | Intern           | 20000  |

**After INNER JOIN on emp_id:**

| emp_id | first_name | age | gender | job_title        | salary |
|--------|------------|-----|--------|------------------|--------|
| 1      | John       | 25  | Male   | Data Analyst     | 50000  |
| 2      | Alice      | 28  | Female | HR Manager       | 60000  |
| 3      | Mark       | 52  | Male   | Senior Developer | 90000  |

> Anna (emp_id 4), Robert (emp_id 5), and Intern (emp_id 6) are excluded — no match in both tables.
> All further steps work on this joined result.

---

### Step 2 — WHERE
Filters individual rows from the joined result. No aggregates allowed here.

```sql
WHERE ed.age > 22
```

| emp_id | first_name | age | gender | job_title        | salary |
|--------|------------|-----|--------|------------------|--------|
| 1      | John       | 25  | Male   | Data Analyst     | 50000  |
| 2      | Alice      | 28  | Female | HR Manager       | 60000  |
| 3      | Mark       | 52  | Male   | Senior Developer | 90000  |

> All three pass the filter (all are above 22).

---

### Step 3 — GROUP BY
Groups the filtered rows into buckets.

```sql
GROUP BY ed.gender
```

| gender | rows included                  |
|--------|--------------------------------|
| Male   | John(25), Mark(52)             |
| Female | Alice(28)                      |

> Rows are collapsed into groups. Only grouped column or aggregates usable after this.

---

### Step 4 — HAVING
Filters groups using aggregate conditions.

```sql
HAVING AVG(ed.age) > 30
```

| gender | AVG(age) | kept? |
|--------|----------|-------|
| Male   | 38.5     | yes   |
| Female | 28.0     | no    |

> Female group removed — avg age 28 is not > 30.

---

### Step 5 — SELECT
MySQL now picks which columns to return and applies aliases.

```sql
SELECT ed.gender, AVG(ed.age) AS avg_age, es.job_title
```

| gender | avg_age | job_title        |
|--------|---------|------------------|
| Male   | 38.5    | Data Analyst     |

> Alias `avg_age` is created here — this is why alias **cannot** be used in WHERE or HAVING (they run before SELECT).

---

### Step 6 — ORDER BY
Sorts the final result. Alias from SELECT **can** be used here.

```sql
ORDER BY avg_age DESC
```

| gender | avg_age | job_title    |
|--------|---------|--------------|
| Male   | 38.5    | Data Analyst |

---

### Step 7 — LIMIT
Cuts the result to N rows after sorting.

```sql
LIMIT 2
```

> Returns at most 2 rows from the final sorted result.

---

## Full Visual Flow

```
┌──────────────────────────────────────────────────┐
│  FROM + JOIN  → load tables, apply join condition │
│       ↓                                           │
│  WHERE        → filter rows (no aggregates)       │
│       ↓                                           │
│  GROUP BY     → collapse rows into groups         │
│       ↓                                           │
│  HAVING       → filter groups (aggregates ok)     │
│       ↓                                           │
│  SELECT       → pick columns, assign aliases      │
│       ↓                                           │
│  ORDER BY     → sort result (alias usable here)   │
│       ↓                                           │
│  LIMIT        → cut to N rows                     │
└──────────────────────────────────────────────────┘
```

---

## Common Mistakes Explained by Execution Order

### 1 — Using alias in WHERE (fails)
```sql
-- WRONG
SELECT age * 12 AS age_months
FROM employee_demographics
WHERE age_months > 300;
```
```
ERROR: Unknown column 'age_months' in 'where clause'
```
**Fix:** Use the expression directly.
```sql
WHERE age * 12 > 300;
```

---

### 2 — Using aggregate in WHERE (fails)
```sql
-- WRONG
SELECT gender FROM employee_demographics
WHERE AVG(age) > 30;
```
```
ERROR 1111: Invalid use of group function
```
**Fix:** Use HAVING after GROUP BY.
```sql
GROUP BY gender
HAVING AVG(age) > 30;
```

---

### 3 — Filtering JOIN result with WHERE vs ON

```sql
-- ON filters during the JOIN (Step 1)
INNER JOIN employee_salary es ON ed.emp_id = es.emp_id AND es.salary > 40000

-- WHERE filters after JOIN (Step 2)
INNER JOIN employee_salary es ON ed.emp_id = es.emp_id
WHERE es.salary > 40000
```

> For INNER JOIN both give the same result.
> For LEFT JOIN they differ — `ON` keeps all left rows, `WHERE` removes unmatched ones.

---

## Note

> - Execution order is fixed — writing order does not change it
> - `FROM + JOIN` happens first — all table loading and joining before any filtering
> - `WHERE` filters rows before grouping, `HAVING` filters groups after grouping
> - Aliases defined in `SELECT` are only available in `ORDER BY`
> - `LIMIT` always runs last — sorting happens before the cut
> - For multiple JOINs, each JOIN is processed left to right in Step 1
