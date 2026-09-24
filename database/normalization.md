# Normalization (1NF-5NF)

The process of organizing columns and tables to reduce redundancy and avoid insertion, update, and deletion anomalies.

## The three anomalies normalization avoids

- **Insert anomaly**: you can't add a piece of data because it depends on something that doesn't exist yet (e.g. you can't add a course without an enrolled student, if student and course are in the same table).
- **Update anomaly**: the same data repeated across several rows — if it changes, every row has to be updated or they end up inconsistent.
- **Delete anomaly**: deleting a row deletes information that shouldn't be lost (e.g. deleting a course's only student deletes the course too).

## 1NF — First normal form

- Every column has atomic values (no lists, no arrays, no CSV in a field).
- No repeating groups (columns `phone1`, `phone2`, `phone3`).

```
❌ users(id, name, phones="123,456,789")
✅ users(id, name)  +  phones(id, user_id, phone)
```

## 2NF — Second normal form

- Satisfies 1NF **and** every non-key attribute depends on the **entire primary key** (only relevant with composite keys).

```
❌ order_items(order_id, product_id, product_name, quantity)
   -- product_name depends only on product_id, not on the full key (order_id, product_id)
✅ order_items(order_id, product_id, quantity)  +  products(product_id, product_name)
```

## 3NF — Third normal form

- Satisfies 2NF **and** has no transitive dependencies (a non-key attribute that depends on another non-key attribute, instead of depending directly on the key).

```
❌ employees(id, name, dept_id, dept_name)
   -- dept_name depends on dept_id, not directly on id
✅ employees(id, name, dept_id)  +  departments(dept_id, dept_name)
```

## BCNF — Boyce-Codd (3.5NF)

- A stricter version of 3NF: every functional dependency `X → Y` requires `X` to be a superkey. Resolves rare cases where 3NF still allows anomalies with overlapping candidate keys.

## 4NF — Fourth normal form

- Satisfies BCNF **and** has no independent multivalued dependencies in the same table.

```
❌ employee_skills_languages(emp_id, skill, language)
   -- skills and languages are independent of each other, mixing them generates redundant rows (cartesian product)
✅ employee_skills(emp_id, skill)  +  employee_languages(emp_id, language)
```

## 5NF — Fifth normal form (Project-Join Normal Form)

- Satisfies 4NF **and** the table can't be decomposed into smaller tables without losing information when joined back together (lossless join). Applies to complex ternary relationship cases — uncommon in everyday practical design.

## Summary table

| Form | Key requirement |
|---|---|
| 1NF | Atomic values, no repeating groups |
| 2NF | 1NF + no partial dependency on a composite key |
| 3NF | 2NF + no transitive dependencies |
| BCNF | 3NF + every determinant is a superkey |
| 4NF | BCNF + no multivalued dependencies |
| 5NF | 4NF + no join dependencies |

## Denormalization: the trade-off

In practice, 3NF is usually the sweet spot. Denormalizing (duplicating data on purpose) is valid for:

- Optimizing reads in systems with much more read traffic than write traffic (e.g. storing `product_name` in `order_items` as a historical snapshot of the price/name at purchase time).
- Avoiding expensive joins in high-frequency queries (the classic trade-off: **consistency vs performance**).

See also [Sharding vs partitioning](sharding-vs-partitioning.md) for larger-scale strategies.
