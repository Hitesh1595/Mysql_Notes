# String Functions in MySQL

Most commonly used string functions for working with text data.

---

## UPPER and LOWER

Convert string to uppercase or lowercase.

```sql
SELECT first_name, UPPER(first_name) AS upper_name, LOWER(first_name) AS lower_name
FROM employee_demographics;
```

| first_name | upper_name | lower_name |
|------------|------------|------------|
| John       | JOHN       | john       |
| Alice      | ALICE      | alice      |

---

## LENGTH

Returns the number of characters in a string.

```sql
SELECT first_name, LENGTH(first_name) AS name_length
FROM employee_demographics;
```

| first_name | name_length |
|------------|-------------|
| John       | 4           |
| Alice      | 5           |
| Mark       | 4           |

```sql
-- Find employees with first name longer than 4 characters
SELECT first_name FROM employee_demographics
WHERE LENGTH(first_name) > 4;
```

| first_name |
|------------|
| Alice      |
| Robert     |
| Daniel     |

---

## TRIM, LTRIM, RTRIM

Remove spaces from a string.

| Function  | Removes                  |
|-----------|--------------------------|
| `TRIM`    | Spaces from both sides   |
| `LTRIM`   | Spaces from left only    |
| `RTRIM`   | Spaces from right only   |

```sql
SELECT TRIM('   John   ') AS trimmed;       -- 'John'
SELECT LTRIM('   John   ') AS left_trim;    -- 'John   '
SELECT RTRIM('   John   ') AS right_trim;   -- '   John'
```

> Useful when data has accidental spaces from user input or imports.

---

## SUBSTRING

Extracts part of a string.

```sql
SUBSTRING(string, start, length)
```

```sql
SELECT first_name, SUBSTRING(first_name, 1, 3) AS short_name
FROM employee_demographics;
```

| first_name | short_name |
|------------|------------|
| John       | Joh        |
| Alice      | Ali        |
| Robert     | Rob        |

```sql
-- Extract year from birth_date (stored as '1990-03-15')
SELECT first_name, SUBSTRING(birth_date, 1, 4) AS birth_year
FROM employee_demographics;
```

| first_name | birth_year |
|------------|------------|
| John       | 1990       |
| Alice      | 1992       |

> Start position begins at **1** in MySQL (not 0).

---

## REPLACE

Replaces all occurrences of a substring with another.

```sql
REPLACE(string, old, new)
```

```sql
SELECT REPLACE('Data Analyst', 'Analyst', 'Engineer') AS new_title;
```

| new_title      |
|----------------|
| Data Engineer  |

```sql
SELECT job_title, REPLACE(job_title, 'Senior', 'Lead') AS updated_title
FROM employee_salary;
```

| job_title        | updated_title    |
|------------------|------------------|
| Senior Developer | Lead Developer   |
| HR Manager       | HR Manager       |

---

## CONCAT

Joins two or more strings together.

```sql
SELECT CONCAT(first_name, ' ', last_name) AS full_name
FROM employee_demographics;
```

| full_name    |
|--------------|
| John Smith   |
| Alice Brown  |
| Mark Johnson |

```sql
-- Add label text
SELECT CONCAT(first_name, ' is ', age, ' years old') AS description
FROM employee_demographics;
```

| description            |
|------------------------|
| John is 25 years old   |
| Alice is 28 years old  |

---

## CONCAT_WS

`CONCAT` **With Separator** — joins strings with a fixed separator.

```sql
SELECT CONCAT_WS(', ', first_name, gender, age) AS info
FROM employee_demographics;
```

| info               |
|--------------------|
| John, Male, 25     |
| Alice, Female, 28  |

> Cleaner than adding separator manually in `CONCAT`.

---

## LOCATE

Returns the position of a substring inside a string. Returns `0` if not found.

```sql
SELECT job_title, LOCATE('Manager', job_title) AS position
FROM employee_salary;
```

| job_title    | position |
|--------------|----------|
| HR Manager   | 4        |
| Data Analyst | 0        |

```sql
-- Find employees whose job title contains 'Manager'
SELECT first_name, job_title
FROM employee_demographics ed
INNER JOIN employee_salary es ON ed.emp_id = es.emp_id
WHERE LOCATE('Manager', es.job_title) > 0;
```

| first_name | job_title  |
|------------|------------|
| Alice      | HR Manager |

---

## LEFT and RIGHT

Extract N characters from the left or right of a string.

```sql
SELECT
    first_name,
    LEFT(first_name, 2)  AS first_two,
    RIGHT(first_name, 2) AS last_two
FROM employee_demographics;
```

| first_name | first_two | last_two |
|------------|-----------|----------|
| John       | Jo        | hn       |
| Alice      | Al        | ce       |
| Robert     | Ro        | rt       |

---

## REVERSE

Reverses a string.

```sql
SELECT first_name, REVERSE(first_name) AS reversed
FROM employee_demographics;
```

| first_name | reversed |
|------------|----------|
| John       | nhoJ     |
| Alice      | ecilA    |

---

## CHAR_LENGTH vs LENGTH

| Function      | Counts                         |
|---------------|--------------------------------|
| `LENGTH`      | Bytes (differs for multi-byte characters) |
| `CHAR_LENGTH` | Actual characters (safe for UTF-8) |

```sql
SELECT LENGTH('John') AS bytes, CHAR_LENGTH('John') AS chars;
-- Both return 4 for standard ASCII
```

> For names with accented characters (é, ü, etc.) use `CHAR_LENGTH` for accurate count.

---

## Quick Reference

| Function        | Purpose                              |
|-----------------|--------------------------------------|
| `UPPER()`       | Convert to uppercase                 |
| `LOWER()`       | Convert to lowercase                 |
| `LENGTH()`      | Count bytes in string                |
| `CHAR_LENGTH()` | Count characters in string           |
| `TRIM()`        | Remove spaces both sides             |
| `LTRIM()`       | Remove spaces left side              |
| `RTRIM()`       | Remove spaces right side             |
| `SUBSTRING()`   | Extract part of string               |
| `REPLACE()`     | Replace substring with another       |
| `CONCAT()`      | Join strings together                |
| `CONCAT_WS()`   | Join strings with separator          |
| `LOCATE()`      | Find position of substring           |
| `LEFT()`        | Extract N chars from left            |
| `RIGHT()`       | Extract N chars from right           |
| `REVERSE()`     | Reverse a string                     |

---

## Note

> - String positions in MySQL start at **1**, not 0
> - `CONCAT` with a `NULL` value returns `NULL` — use `CONCAT_WS` or `COALESCE` to handle NULLs
> - `REPLACE` is case-sensitive by default
> - `LOCATE` returns `0` (not NULL) when substring is not found
> - `TRIM` only removes spaces by default — use `TRIM(LEADING 'x' FROM col)` to trim specific characters
