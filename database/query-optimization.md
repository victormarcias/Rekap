# Query Optimization

A checklist of what to check before calling a query production-ready.

## Correct joins — avoiding cartesian products

A `JOIN` with no condition (or with an incomplete/incorrect condition) doesn't relate the tables — it **multiplies** them: every row on one side combines with every matching row on the other, silently, with no error. The result is far more rows than expected, sometimes by orders of magnitude.

```sql
-- ❌ no ON condition: every orders row combines with EVERY order_items row
SELECT * FROM orders, order_items;
-- with 1,000 orders and 5,000 items, this returns 5,000,000 rows, not 5,000

-- ❌ join condition on a column that doesn't identify the real relationship (fan-out)
SELECT * FROM orders o
JOIN order_items oi ON o.user_id = oi.user_id  -- should be o.id = oi.order_id
-- each order of a user combines with EVERY item of ALL their orders, not just its own

-- ✅ correct join condition, on the key that actually relates the rows
SELECT * FROM orders o
JOIN order_items oi ON o.id = oi.order_id;
```

**How to detect it**: the result has far more rows than the business would expect for that filter, or `EXPLAIN ANALYZE` shows an absurdly high row estimate at some step of the plan — see [Database Diagnostics](../diagnostics/database.md#execution-plan).

## Subqueries — when to avoid them

The problem isn't subqueries in general, it's the **correlated subquery**: one that references a column from the outer query, and so gets re-executed **once per row** of the outer result — basically the same underlying problem as [N+1](../diagnostics/backend.md#n1-problem), but within a single SQL statement instead of across requests.

```sql
-- ❌ correlated subquery: runs once PER ROW of orders
SELECT o.id, o.total,
  (SELECT COUNT(*) FROM order_items oi WHERE oi.order_id = o.id) AS item_count
FROM orders o;

-- ✅ JOIN + GROUP BY: the planner resolves it in a single pass (hash join or merge join)
SELECT o.id, o.total, COUNT(oi.id) AS item_count
FROM orders o
LEFT JOIN order_items oi ON oi.order_id = o.id
GROUP BY o.id, o.total;
```

Not every subquery is bad — a subquery in `WHERE ... EXISTS (...)` to check existence can be as good as or better than a `JOIN`, because the planner can short-circuit as soon as it finds the first match without pulling extra rows. The real rule is to avoid **row-by-row correlated** ones and deep nesting (subquery within subquery within subquery), not subqueries as a category.

## The rest of the checklist — already covered elsewhere

- **Fetch only the columns you need** (not `SELECT *`) → [CRUD](crud.md#read--select)
- **Filter correctly with `WHERE` vs `HAVING`** (rows vs groups) → [CRUD](crud.md#read--select)
- **Avoid unnecessary `LIKE`** (especially with a leading wildcard, which breaks index usage) → [Non-sargable queries](queries-non-sargable.md)

---
Related: [Indexes](indexes.md), [Database Diagnostics](../diagnostics/database.md), [N+1](../diagnostics/backend.md#n1-problem).
