# JOINs in MySQL

A JOIN combines rows from two or more tables based on a related column.

---

## Tables Used in Examples

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

**department**

| dept_id | emp_id | dept_name   |
|---------|--------|-------------|
| 101     | 1      | Analytics   |
| 102     | 2      | HR          |
| 103     | 3      | Engineering |
| 104     | 5      | Engineering |

---

## INNER JOIN

Returns only rows that have a **match in both tables**.

```sql
SELECT ed.first_name, ed.age, es.job_title, es.salary
FROM employee_demographics ed
INNER JOIN employee_salary es
    ON ed.emp_id = es.emp_id;
```

| first_name | age | job_title        | salary |
|------------|-----|------------------|--------|
| John       | 25  | Data Analyst     | 50000  |
| Alice      | 28  | HR Manager       | 60000  |
| Mark       | 52  | Senior Developer | 90000  |

> `Anna` (emp_id 4) and `Robert` (emp_id 5) are excluded — no match in salary table.
> `emp_id 6` (Intern) is excluded — no match in demographics table.

---

## LEFT JOIN

Returns **all rows from the left table** and matched rows from the right.
Unmatched right side shows `NULL`.

```sql
SELECT ed.first_name, ed.age, es.job_title, es.salary
FROM employee_demographics ed
LEFT JOIN employee_salary es
    ON ed.emp_id = es.emp_id;
```

| first_name | age | job_title        | salary |
|------------|-----|------------------|--------|
| John       | 25  | Data Analyst     | 50000  |
| Alice      | 28  | HR Manager       | 60000  |
| Mark       | 52  | Senior Developer | 90000  |
| Anna       | 22  | NULL             | NULL   |
| Robert     | 45  | NULL             | NULL   |

> All 5 employees appear. Anna and Robert have no salary record — shown as NULL.

---

## RIGHT JOIN

Returns **all rows from the right table** and matched rows from the left.
Unmatched left side shows `NULL`.

```sql
SELECT ed.first_name, ed.age, es.job_title, es.salary
FROM employee_demographics ed
RIGHT JOIN employee_salary es
    ON ed.emp_id = es.emp_id;
```

| first_name | age  | job_title        | salary |
|------------|------|------------------|--------|
| John       | 25   | Data Analyst     | 50000  |
| Alice      | 28   | HR Manager       | 60000  |
| Mark       | 52   | Senior Developer | 90000  |
| NULL       | NULL | Intern           | 20000  |

> All salary records appear. emp_id 6 (Intern) has no demographics — shown as NULL.

---

## FULL OUTER JOIN

Returns **all rows from both tables**.
MySQL does not support `FULL OUTER JOIN` directly — simulate with `UNION`.

```sql
SELECT ed.first_name, es.job_title, es.salary
FROM employee_demographics ed
LEFT JOIN employee_salary es ON ed.emp_id = es.emp_id

UNION

SELECT ed.first_name, es.job_title, es.salary
FROM employee_demographics ed
RIGHT JOIN employee_salary es ON ed.emp_id = es.emp_id;
```

| first_name | job_title        | salary |
|------------|------------------|--------|
| John       | Data Analyst     | 50000  |
| Alice      | HR Manager       | 60000  |
| Mark       | Senior Developer | 90000  |
| Anna       | NULL             | NULL   |
| Robert     | NULL             | NULL   |
| NULL       | Intern           | 20000  |

---

## SELF JOIN

A table joined **with itself**.
Useful when rows in the same table are related to each other.

**employee_demographics** (with manager column added)

| emp_id | first_name | age | manager_id |
|--------|------------|-----|------------|
| 1      | John       | 25  | 3          |
| 2      | Alice      | 28  | 3          |
| 3      | Mark       | 52  | NULL       |
| 4      | Anna       | 22  | 2          |
| 5      | Robert     | 45  | 3          |

```sql
-- Show each employee with their manager's name
SELECT
    emp.first_name AS employee,
    mgr.first_name AS manager
FROM employee_demographics emp
LEFT JOIN employee_demographics mgr
    ON emp.manager_id = mgr.emp_id;
```

| employee | manager |
|----------|---------|
| John     | Mark    |
| Alice    | Mark    |
| Mark     | NULL    |
| Anna     | Alice   |
| Robert   | Mark    |

> Same table used twice with different aliases — `emp` for employee, `mgr` for manager.
> Mark has no manager so it shows NULL.

---

## 3 Table JOIN

Join all three tables — demographics, salary, and department together.

```sql
SELECT
    ed.first_name,
    ed.gender,
    es.job_title,
    es.salary,
    d.dept_name
FROM employee_demographics ed
INNER JOIN employee_salary es
    ON ed.emp_id = es.emp_id
INNER JOIN department d
    ON ed.emp_id = d.emp_id;
```

| first_name | gender | job_title        | salary | dept_name   |
|------------|--------|------------------|--------|-------------|
| John       | Male   | Data Analyst     | 50000  | Analytics   |
| Alice      | Female | HR Manager       | 60000  | HR          |
| Mark       | Male   | Senior Developer | 90000  | Engineering |

> Each JOIN adds one more table to the result.
> Employee must exist in all 3 tables to appear (INNER JOIN).
> Robert (emp_id 5) is in department but not in salary — excluded.

---

## UNION and UNION ALL

`UNION` stacks results of two `SELECT` queries **vertically** (adds rows, not columns).

### Rules
- Both queries must have the **same number of columns**
- Columns must have **compatible data types**
- Column names come from the **first** SELECT

---

### UNION — removes duplicates

```sql
SELECT first_name, gender FROM employee_demographics WHERE gender = 'Male'
UNION
SELECT first_name, gender FROM employee_demographics WHERE age < 30;
```

| first_name | gender |
|------------|--------|
| John       | Male   |
| Mark       | Male   |
| Robert     | Male   |
| Alice      | Female |
| Anna       | Female |

> John appears only once even though he matches both conditions (Male AND age < 30).
> `UNION` automatically removes duplicate rows.

---

### UNION ALL — keeps duplicates

```sql
SELECT first_name, gender FROM employee_demographics WHERE gender = 'Male'
UNION ALL
SELECT first_name, gender FROM employee_demographics WHERE age < 30;
```

| first_name | gender |
|------------|--------|
| John       | Male   |
| Mark       | Male   |
| Robert     | Male   |
| John       | Male   |
| Alice      | Female |
| Anna       | Female |

> John appears **twice** — once from each SELECT. `UNION ALL` keeps all rows including duplicates.

---

### UNION across different tables

```sql
SELECT first_name, 'Demographics' AS source FROM employee_demographics
UNION
SELECT job_title, 'Salary' AS source FROM employee_salary;
```

| first_name       | source       |
|------------------|--------------|
| John             | Demographics |
| Alice            | Demographics |
| Mark             | Demographics |
| Data Analyst     | Salary       |
| HR Manager       | Salary       |
| Senior Developer | Salary       |
| Intern           | Salary       |

> Useful for combining data from different tables into one result set.

---

### UNION vs JOIN

| Feature      | JOIN                              | UNION                        |
|--------------|-----------------------------------|------------------------------|
| Direction    | Horizontal (adds columns)         | Vertical (adds rows)         |
| Purpose      | Combine related columns           | Stack similar result sets    |
| Condition    | Needs `ON` matching condition     | Needs same column count/type |

---

## JOIN Types Summary

| JOIN Type       | Returns                                              |
|-----------------|------------------------------------------------------|
| `INNER JOIN`    | Only matching rows from both tables                  |
| `LEFT JOIN`     | All rows from left + matched from right (NULL if no match) |
| `RIGHT JOIN`    | All rows from right + matched from left (NULL if no match) |
| `FULL OUTER`    | All rows from both (simulated via UNION in MySQL)    |
| `SELF JOIN`     | Table joined with itself using aliases               |
| `UNION`         | Stacks rows from two queries, removes duplicates     |
| `UNION ALL`     | Stacks rows from two queries, keeps duplicates       |

---

## Note

> - Always use table aliases when joining — makes queries readable
> - `INNER JOIN` and `JOIN` are the same — `INNER` is optional
> - For 3+ table joins, chain each `JOIN ... ON` one after another
> - `LEFT JOIN` is the most commonly used join in real-world queries
> - In a SELF JOIN, both aliases point to the same physical table
> - Column names that exist in multiple tables must be prefixed with table alias to avoid ambiguity
> - `UNION` is slower than `UNION ALL` because it does duplicate removal — use `UNION ALL` when you know there are no duplicates
> - `UNION` column names are taken from the first SELECT statement
