# MySQL Notes

Personal notes on MySQL concepts with examples.
Follow the sequence below for the best learning flow — each topic builds on the previous one.

---

## Sequence

| # | File | What you will learn |
|---|------|---------------------|
| 1 | [distinct.md](./distinct.md) | DISTINCT on single and multiple fields, DISTINCT vs GROUP BY |
| 2 | [case_sensitivity.md](./case_sensitivity.md) | How MySQL handles case in string comparison, collation, BINARY keyword |
| 3 | [like_and_underscore.md](./like_and_underscore.md) | Pattern matching with `%` and `_`, usage with strings and dates |
| 4 | [group_by_order_by.md](./group_by_order_by.md) | Grouping rows with GROUP BY, sorting results with ORDER BY |
| 5 | [having.md](./having.md) | Filtering groups with HAVING, difference between WHERE and HAVING |
| 6 | [limit_and_aliasing.md](./limit_and_aliasing.md) | Limit rows with LIMIT, OFFSET for pagination, column and table aliases |
| 7 | [joins.md](./joins.md) | INNER, LEFT, RIGHT, FULL OUTER, SELF JOIN, 3-table JOIN, UNION and UNION ALL |
| 8 | [string_functions.md](./string_functions.md) | Most used string functions — UPPER, TRIM, SUBSTRING, CONCAT, REPLACE, etc. |
| 9 | [case_statement.md](./case_statement.md) | CASE for conditional logic, age groups, salary bands, pivot counting |
| 10 | [subquery.md](./subquery.md) | Subquery in SELECT, WHERE IN / NOT IN, FROM as derived table, subquery vs JOIN |
| 11 | [window_functions.md](./window_functions.md) | OVER, PARTITION BY, SUM rolling total, ROW_NUMBER, RANK, DENSE_RANK |
| 12 | [query_execution_order.md](./query_execution_order.md) | How MySQL internally executes a query — FROM to LIMIT, common mistakes |

---

## Tip

> Read `query_execution_order.md` again after finishing all files.
> It will make much more sense once you have seen all the clauses in action.
