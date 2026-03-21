# GROUP BY and ORDER BY in MySQL

---

## GROUP BY

Groups rows that have the same value in a column.
Usually used with aggregate functions like `COUNT`, `AVG`, `MAX`, `MIN`, `SUM`.

### Example 1 — Count by Gender

```sql
SELECT gender, COUNT(*) AS total
FROM employee_demographics
GROUP BY gender;
```

| gender | total |
|--------|-------|
| Male   | 6     |
| Female | 4     |

### Example 2 — Average Age by Gender

```sql
SELECT gender, AVG(age) AS avg_age
FROM employee_demographics
GROUP BY gender;
```

| gender | avg_age |
|--------|---------|
| Male   | 34.5    |
| Female | 30.2    |

### Example 3 — Max and Min Age by Gender

```sql
SELECT gender, MAX(age) AS oldest, MIN(age) AS youngest
FROM employee_demographics
GROUP BY gender;
```

| gender | oldest | youngest |
|--------|--------|----------|
| Male   | 52     | 24       |
| Female | 45     | 22       |

### Example 4 — Group by Multiple Columns

```sql
SELECT gender, age, COUNT(*) AS total
FROM employee_demographics
GROUP BY gender, age;
```

| gender | age | total |
|--------|-----|-------|
| Male   | 30  | 2     |
| Male   | 45  | 1     |
| Female | 28  | 2     |

> Each unique combination of `gender + age` is treated as one group.

---

## ORDER BY

Sorts the result set by one or more columns.
Default is **ASC** (ascending). Use **DESC** for descending.

### Example 1 — Order by Age Ascending (default)

```sql
SELECT first_name, age
FROM employee_demographics
ORDER BY age;
```

| first_name | age |
|------------|-----|
| Anna       | 22  |
| John       | 25  |
| Alice      | 28  |
| Mark       | 34  |

### Example 2 — Order by Age Descending

```sql
SELECT first_name, age
FROM employee_demographics
ORDER BY age DESC;
```

| first_name | age |
|------------|-----|
| Mark       | 52  |
| Robert     | 45  |
| Alice      | 28  |
| John       | 25  |

### Example 3 — Order by Gender then Age

```sql
SELECT first_name, gender, age
FROM employee_demographics
ORDER BY gender, age;
```

| first_name | gender | age |
|------------|--------|-----|
| Anna       | Female | 22  |
| Alice      | Female | 28  |
| Karen      | Female | 45  |
| John       | Male   | 25  |
| Daniel     | Male   | 30  |
| Mark       | Male   | 52  |

> First sorted by `gender` A→Z, then within each gender sorted by `age` low→high.

### Example 4 — Order by Gender ASC, Age DESC

```sql
SELECT first_name, gender, age
FROM employee_demographics
ORDER BY gender ASC, age DESC;
```

| first_name | gender | age |
|------------|--------|-----|
| Karen      | Female | 45  |
| Alice      | Female | 28  |
| Anna       | Female | 22  |
| Mark       | Male   | 52  |
| Daniel     | Male   | 30  |
| John       | Male   | 25  |

---

## GROUP BY + ORDER BY Together

```sql
SELECT gender, AVG(age) AS avg_age
FROM employee_demographics
GROUP BY gender
ORDER BY avg_age DESC;
```

| gender | avg_age |
|--------|---------|
| Male   | 34.5    |
| Female | 30.2    |

> `GROUP BY` first groups the data, then `ORDER BY` sorts the grouped result.

---

## Note

> - `GROUP BY` **collapses** rows into groups — you can only SELECT the grouped column or an aggregate function
> - `ORDER BY` does **not** change the data, only the display order
> - `ORDER BY` always comes **after** `GROUP BY` in a query
> - Default sort is `ASC` — you only need to write it explicitly for clarity
> - You can `ORDER BY` a column even if it is not in the `SELECT` list
> - With multiple columns in `ORDER BY`, sorting is applied left to right
