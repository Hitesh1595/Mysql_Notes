# LIKE and _ (Underscore) in MySQL

`LIKE` is used for pattern matching in string comparisons.
It uses two wildcards:

| Wildcard | Meaning                        |
|----------|--------------------------------|
| `%`      | Match zero or more characters  |
| `_`      | Match exactly one character    |

---

## % — Matches Any Number of Characters

```sql
-- Names starting with 'A'
SELECT * FROM employee_demographics WHERE first_name LIKE 'A%';
```

| first_name |
|------------|
| Alice      |
| Andrew     |
| Anna       |

```sql
-- Names ending with 'n'
SELECT * FROM employee_demographics WHERE first_name LIKE '%n';
```

| first_name |
|------------|
| John       |
| Nathan     |
| Kevin      |

```sql
-- Names containing 'an' anywhere
SELECT * FROM employee_demographics WHERE first_name LIKE '%an%';
```

| first_name |
|------------|
| Nathan     |
| Anna       |
| Daniel     |

---

## _ — Matches Exactly One Character

```sql
-- Names where second character is 'o'
SELECT * FROM employee_demographics WHERE first_name LIKE '_o%';
```

| first_name |
|------------|
| John       |
| Robert     |

```sql
-- Names that are exactly 4 characters long
SELECT * FROM employee_demographics WHERE first_name LIKE '____';
```

| first_name |
|------------|
| John       |
| Anna       |
| Mark       |

> Each `_` represents one character. Four underscores = exactly 4 characters.

```sql
-- Names starting with 'J' and exactly 4 characters
SELECT * FROM employee_demographics WHERE first_name LIKE 'J___';
```

| first_name |
|------------|
| John       |

---

## Combining % and _

```sql
-- Names where third character is 'h'
SELECT * FROM employee_demographics WHERE first_name LIKE '__h%';
```

| first_name |
|------------|
| John       |
| Nathan     |

---

## LIKE with Dates

In MySQL, dates are stored as `YYYY-MM-DD`. Since it's a string-like format, `LIKE` works on date columns too.

```sql
-- All employees born in the year 1990
SELECT * FROM employee_demographics WHERE birth_date LIKE '1990%';
```

| first_name | birth_date |
|------------|------------|
| John       | 1990-03-15 |
| Anna       | 1990-11-02 |

```sql
-- All employees born in March (any year)
SELECT * FROM employee_demographics WHERE birth_date LIKE '____-03-%';
```

| first_name | birth_date |
|------------|------------|
| John       | 1990-03-15 |
| Mark       | 1985-03-22 |

> `____` matches exactly 4 digits of year, then `-03-` matches March exactly.

```sql
-- All employees born on the 1st of any month any year
SELECT * FROM employee_demographics WHERE birth_date LIKE '____-__-01';
```

| first_name | birth_date |
|------------|------------|
| Alice      | 1992-07-01 |
| Kevin      | 1988-01-01 |

```sql
-- All employees born in the 1990s
SELECT * FROM employee_demographics WHERE birth_date LIKE '199%';
```

| first_name | birth_date |
|------------|------------|
| John       | 1990-03-15 |
| Anna       | 1991-11-02 |
| Daniel     | 1995-06-18 |

---

## Note

> - `LIKE` is **case-insensitive** by default in MySQL (same as regular string comparison)
> - `%` can match **zero** characters too — `'A%'` matches `'A'` as well
> - `_` is strict — it must match **exactly one** character, no more no less
> - To search for a literal `%` or `_` in data, escape it: `LIKE '50\%'` or `LIKE '\_name'`
> - Prefer `LIKE` only for pattern matching; for exact match use `=` (faster)
