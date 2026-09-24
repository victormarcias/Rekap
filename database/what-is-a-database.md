# What's a database

A system specifically designed to store, organize, and retrieve data in a **persistent**, **concurrent** (multiple processes or users accessing it at once without stepping on each other), and **reliable** way — unlike storing data in a text file or JSON on disk.

## Why a file isn't enough

```python
# ❌ a JSON file solves none of what follows:
import json

def save_user(user):
    with open('users.json') as f:
        data = json.load(f)
    data.append(user)
    with open('users.json', 'w') as f:
        json.dump(data, f)

# if two processes call this at the same time, one can overwrite the other's
# change — there's no concurrency handling, and if the process crashes right
# in the middle of writing, the file can end up half-corrupted
```

A database solves, out of the box, several problems that hand-rolling something with files doesn't solve for free:

- **Concurrency**: multiple clients reading/writing at the same time, without stepping on each other — see [Locks](locks.md).
- **Durability**: if the process crashes mid-write, the data isn't left corrupted — see [Write-Ahead Log](rdbms.md#write-ahead-log-wal--how-nothing-gets-lost-if-the-server-crashes).
- **Efficient queries**: searching, filtering, and sorting without having to read and parse the entire file into memory every time — see [Indexes](indexes.md).
- **Integrity**: rules the database itself enforces, so data never ends up in a half-applied or invalid state — see [ACID](acid.md).

## A brief history of databases

Video: [A brief history of databases](https://www.youtube.com/watch?v=KG-mqHoXOXY)

Before the relational model took over, other data models existed (and in some niches, still do):

- **Hierarchical** (e.g. IBM IMS, 1966; the Windows registry): data is organized in a tree, each record has a single parent. Simple, but rigid — modeling a many-to-many relationship is forced.
- **Network** (CODASYL): like hierarchical, but a record can have several parents — more flexible, at the cost of navigating the data requiring you to know the physical structure ahead of time. It's, conceptually, the direct ancestor of what's now a graph database.
- **Relational**: the one that won — tables independent of how they're accessed afterward, queried with declarative SQL instead of manual navigation between records. See [RDBMS](rdbms.md).
- **Object-oriented** (e.g. db4o, ObjectDB): stores objects exactly as they exist in memory in an OOP language, without translating them into tables. Never took off outside specific niches — an ORM solves the same problem (persisting objects) but by translating them into relational tables underneath, instead of avoiding that translation.
- **Flat file**: a single file, no relationships — a CSV is, in essence, this.
- **Semi-structured**: JSON/XML with no fixed schema — this is what document-type NoSQL databases cover today. See [NoSQL](nosql.md).

**Entity-Relationship (E-R) is not a storage model** like the ones above — it's a **design** technique (E-R diagrams) for modeling entities and relationships before translating them into tables in a relational model. An ORM doesn't work "on top of E-R" directly: it works on the result of that design (the relational tables), the E-R diagram is a prior step in the head of whoever designs the schema.

## The two major types

- **Relational (RDBMS)**: data organized into tables with strict relationships between them, SQL as the query language — see [RDBMS](rdbms.md).
- **NoSQL**: steps away from the table model to prioritize horizontal scalability or flexible schemas — see [NoSQL](nosql.md).

---
Related: [RDBMS](rdbms.md), [NoSQL](nosql.md).
