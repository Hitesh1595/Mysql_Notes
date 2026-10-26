# 12 Essential SQL / Data Engineering Patterns

Patterns that come up again and again in interviews and real pipelines. Each one is a reusable shape, not a one-off query.

---

## 1. Deduplication

Keep only the latest record per group (e.g. latest order per customer).

```sql
SELECT *
FROM (
    SELECT *,
        ROW_NUMBER() OVER(PARTITION BY email ORDER BY created_at DESC) AS rn
    FROM orders
) t
WHERE rn = 1;
```

> Use `ROW_NUMBER()`, not `RANK()` — `RANK()` can return multiple `1`s on a tie, so you'd keep more than one row per group.

---

## 2. Running Totals

Cumulative sum row by row.

```sql
SELECT order_date, amount,
    SUM(amount) OVER(
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM orders;
```

> Explicit frame `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` matters when there are ties in `ORDER BY` — without it some engines default to `RANGE`, which lumps tied rows together.

---

## 3. Growth Rate (MoM / YoY)

Compare current value to a prior period using `LAG()`.

```sql
SELECT month, revenue,
    LAG(revenue) OVER(ORDER BY month) AS prev_revenue,
    (revenue - LAG(revenue) OVER(ORDER BY month))
        / NULLIF(LAG(revenue) OVER(ORDER BY month), 0) AS growth_rate
FROM monthly_revenue;
```

> `NULLIF(prev_value, 0)` guards against division-by-zero when prior period is `0`.

---

## 4. Gaps and Islands

Group consecutive rows (e.g. consecutive active days) into "islands," and detect breaks ("gaps").

```sql
SELECT user_id, activity_date,
    DATE_SUB(activity_date, INTERVAL ROW_NUMBER() OVER(
        PARTITION BY user_id ORDER BY activity_date
    ) DAY) AS island_id
FROM user_activity;
```

> When dates are consecutive, `activity_date - row_number` stays constant → same island.
> A gap shifts that difference → new island_id → effectively a new group.

---

## 5. Sessionization

Split a user's events into sessions based on a time-gap threshold.

```sql
SELECT user_id, event_time,
    SUM(new_session) OVER(PARTITION BY user_id ORDER BY event_time) AS session_id
FROM (
    SELECT user_id, event_time,
        CASE
            WHEN TIMESTAMPDIFF(MINUTE,
                LAG(event_time) OVER(PARTITION BY user_id ORDER BY event_time),
                event_time) > 30
            OR LAG(event_time) OVER(PARTITION BY user_id ORDER BY event_time) IS NULL
            THEN 1 ELSE 0
        END AS new_session
    FROM events
) t;
```

> `LAG()` finds the gap to the previous event; gap > threshold (30 min) flags a new session.
> Running `SUM()` of that flag turns flags into incrementing session IDs.

---

## 6. Funnel Analysis

Measure how many users progress through sequential stages (signup → activation → purchase).

```sql
SELECT
    COUNT(CASE WHEN stage = 'signup' THEN user_id END) AS signups,
    COUNT(CASE WHEN stage = 'activated' THEN user_id END) AS activated,
    COUNT(CASE WHEN stage = 'purchased' THEN user_id END) AS purchased
FROM user_funnel;
```

> Use `LEFT JOIN` (not `INNER JOIN`) between stage tables so users who drop out early are still counted in earlier stages instead of disappearing from the whole result.

---

## 7. Retention Cohorts

Group users by their first-activity month ("cohort"), then track how many return in later months.

```sql
SELECT
    DATE_FORMAT(cohort_month, '%Y-%m') AS cohort,
    TIMESTAMPDIFF(MONTH, cohort_month, activity_month) AS month_number,
    COUNT(DISTINCT user_id) AS active_users
FROM (
    SELECT user_id, activity_month,
        MIN(activity_month) OVER(PARTITION BY user_id) AS cohort_month
    FROM user_monthly_activity
) t
GROUP BY cohort, month_number;
```

> Cohort = `MIN()` activity date per user.
> `month_number` = tenure (activity month minus cohort month) → `COUNT(DISTINCT user_id)` per tenure bucket gives the retention curve.

---

## 8. Top-N per Group

Get the top N rows within each partition (e.g. top 3 earners per department).

```sql
SELECT *
FROM (
    SELECT *,
        DENSE_RANK() OVER(PARTITION BY department ORDER BY salary DESC) AS rnk
    FROM employees
) t
WHERE rnk <= 3;
```

> `ROW_NUMBER()` → exactly N rows per group, ties broken arbitrarily.
> `DENSE_RANK()` → ties share a rank, no skipped ranks, so N "positions" can return more than N rows.
> Ranking filter (`WHERE rnk <= 3`) must sit in an outer query/CTE — window functions can't be used directly in `WHERE`.

---

## 9. Pivoting

Turn long (row-per-category) data into wide (column-per-category) data.

```sql
SELECT
    user_id,
    COALESCE(SUM(CASE WHEN product = 'A' THEN amount END), 0) AS product_a,
    COALESCE(SUM(CASE WHEN product = 'B' THEN amount END), 0) AS product_b,
    COALESCE(SUM(CASE WHEN product = 'C' THEN amount END), 0) AS product_c
FROM sales
GROUP BY user_id;
```

> `COALESCE(..., 0)` fills in `0` for users with no rows for a given category, instead of `NULL`.

---

## 10. Self-Joins

Join a table to itself to compare rows within the same dataset (e.g. employee vs. manager).

```sql
SELECT e.first_name AS employee, m.first_name AS manager
FROM employees e
JOIN employees m ON e.manager_id = m.employee_id
WHERE e.salary > m.salary;
```

> Two aliases (`e`, `m`) of the same table act as two logically separate tables.
> Classic use: hierarchical comparisons — "employees earning more than their manager."

---

## 11. Hierarchy Traversal

Walk an org chart / tree of unknown depth using a recursive CTE.

```sql
WITH RECURSIVE org_chart AS (
    -- anchor: top-level manager
    SELECT employee_id, manager_id, first_name, 1 AS level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- recursive: join chart back to employees
    SELECT e.employee_id, e.manager_id, e.first_name, oc.level + 1
    FROM employees e
    JOIN org_chart oc ON e.manager_id = oc.employee_id
)
SELECT * FROM org_chart;
```

> Anchor member = starting point (top of hierarchy).
> Recursive member = self-join back to the CTE, `UNION ALL`'d with the anchor, repeats until no more matches.

---

## 12. Moving Averages

Smooth a metric using a rolling window of N preceding rows.

```sql
SELECT order_date, revenue,
    AVG(revenue) OVER(
        ORDER BY order_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS moving_avg_7d
FROM daily_revenue;
```

> `ROWS BETWEEN 6 PRECEDING AND CURRENT ROW` counts **rows**, not calendar days.
> If some days are missing from the data, the "7-day" average silently becomes a "7-row" average spanning more than 7 days.
> Fix: join against a generated "date spine" (one row per calendar day) before computing the average.

---

## Quick Reference

| # | Pattern              | Core tool                              |
|---|-----------------------|-----------------------------------------|
| 1 | Deduplication          | `ROW_NUMBER()` + filter                |
| 2 | Running Totals         | `SUM() OVER` with explicit frame        |
| 3 | Growth Rate            | `LAG()` + `NULLIF`                      |
| 4 | Gaps and Islands        | `ROW_NUMBER()` vs. date difference      |
| 5 | Sessionization          | `LAG()` + running `SUM()` of flags      |
| 6 | Funnel Analysis         | `CASE WHEN` / `LEFT JOIN`               |
| 7 | Retention Cohorts       | `MIN()` cohort date + `COUNT(DISTINCT)` |
| 8 | Top-N per Group         | `ROW_NUMBER()` / `DENSE_RANK()`         |
| 9 | Pivoting                | `CASE WHEN` + `COALESCE`                |
| 10 | Self-Joins             | table joined to itself via aliases      |
| 11 | Hierarchy Traversal    | `WITH RECURSIVE`                        |
| 12 | Moving Averages        | `AVG() OVER` with row-bound frame       |
