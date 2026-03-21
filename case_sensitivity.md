# Case Sensitivity in MySQL String Comparison

By default, MySQL string comparisons are **case-insensitive** due to the **collation** setting.

## What is Collation?

Collation defines how strings are compared and sorted.
The default collation in MySQL ends with `_ci` — meaning **Case Insensitive**.

```sql
-- Default collation example
utf8mb4_general_ci
latin1_swedish_ci
```

---

## Example — Case-Insensitive (default behavior)

```sql
SELECT * FROM employee_demographics WHERE gender = 'male';
```

This will match all three:

| gender |
|--------|
| male   |
| Male   |
| MALE   |

> MySQL treats them as equal because of `_ci` collation.

---

## Example — Case-Sensitive using BINARY

```sql
SELECT * FROM employee_demographics WHERE BINARY gender = 'male';
```

This will **only** match exact lowercase `'male'`:

| gender |
|--------|
| male   |

`Male` and `MALE` will **not** be returned.

---

## Example — Case-Sensitive using COLLATE

```sql
SELECT * FROM employee_demographics
WHERE gender COLLATE utf8mb4_bin = 'male';
```

Same result as BINARY — only exact case match.

---

## Check Collation of a Table

```sql
SHOW CREATE TABLE employee_demographics;
```

Look for the collation on the `gender` column:

```
`gender` varchar(10) COLLATE utf8mb4_general_ci
```

---

## Note

> - `_ci` at the end of a collation = **Case Insensitive** (default in MySQL)
> - `_cs` or `_bin` = **Case Sensitive**
> - Use `BINARY` or `COLLATE utf8mb4_bin` only when you need strict case matching
> - In most real-world apps, data is stored consistently (`Male`/`Female`), so case-insensitive default works fine
> - If you are filtering with `WHERE gender = 'male'` and data is stored as `'Male'`, it will still match — no issue
