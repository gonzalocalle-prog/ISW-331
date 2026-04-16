# Week 9 — Relational Database Design and Implementation

> **Block 3: Backend and Databases**
> This week covers how to design a relational database from scratch: modeling entities, defining relationships, normalizing your schema, and implementing it in SQL. The concepts follow the structure of *Grokking Relational Database Design* (Hao & Tsikerdekis, Manning 2025).

---

## Part 1 — Why Database Design Matters

### The Spreadsheet Trap

Imagine a startup that stores everything — customers, products, orders — in a single spreadsheet. At first it works. Then:

- A customer appears in 50 rows (one per order). You update their email in one row but forget the other 49. **Update anomaly.**
- You delete the only order for a customer. The customer's information disappears with it. **Delete anomaly.**
- You want to add a new product that nobody has ordered yet, but the sheet requires a customer for every row. **Insert anomaly.**

These are not hypothetical problems. They are the direct consequence of **bad database design**, and they happen in production systems every day.

> A relational database solves these problems — **but only if you design it correctly.** A poorly designed database inside PostgreSQL can suffer the same anomalies as a spreadsheet.

### The Five Goals of Database Design

Before you write a single `CREATE TABLE`, understand what you're aiming for:

| Goal | What It Means | What Happens Without It |
|------|--------------|------------------------|
| **Data consistency & integrity** | Every piece of data is accurate, non-contradictory, and follows defined rules | Orphan records, impossible states, corrupted data |
| **Maintainability & ease of use** | The schema is understandable; queries are straightforward | Developers waste hours deciphering table structures |
| **Performance & optimization** | Queries run fast even as data grows | Pages that take 30 seconds to load, timeouts |
| **Data security** | Sensitive data is protected; access is controlled | Data breaches, compliance violations |
| **Scalability & flexibility** | The design accommodates growth and change | Painful migrations, rewriting half the application |

> *Reference: Grokking Relational Database Design, Chapter 3 — Goals of database design*

---

## Part 2 — Relational Database Fundamentals

### What Is a Relational Database?

A relational database is a collection of **tables** (also called relations) that store data in **rows** and **columns**. Each table represents one type of thing (an **entity**), and tables are connected to each other through **relationships**.

```
┌─────────────────────────────┐     ┌──────────────────────────────┐
│         customer            │     │          product             │
├────────┬────────┬───────────┤     ├────────┬───────┬─────────────┤
│ id (PK)│ name   │ email     │     │ id (PK)│ name  │ price       │
├────────┼────────┼───────────┤     ├────────┼───────┼─────────────┤
│ 1      │ Alice  │ a@mail.com│     │ 101    │ Widget│ 9.99        │
│ 2      │ Bob    │ b@mail.com│     │ 102    │ Gadget│ 24.99       │
└────────┴────────┴───────────┘     └────────┴───────┴─────────────┘
              │                                  │
              └──────────┐    ┌──────────────────┘
                         ▼    ▼
                  ┌──────────────────────┐
                  │       order          │
                  ├────────┬──────┬──────┤
                  │ id (PK)│cust  │prod  │
                  │        │(FK)  │(FK)  │
                  ├────────┼──────┼──────┤
                  │ 1      │ 1    │ 101  │
                  │ 2      │ 1    │ 102  │
                  │ 3      │ 2    │ 101  │
                  └────────┴──────┴──────┘
```

### Key Terminology

| Term | Definition | Analogy |
|------|-----------|---------|
| **Table (Relation)** | A structured collection of rows and columns representing one entity type | A spreadsheet sheet — but with rules |
| **Row (Record/Tuple)** | A single data entry in a table | One line in the spreadsheet |
| **Column (Attribute/Field)** | A named property with a specific data type | A column header in the spreadsheet |
| **Primary Key (PK)** | A column (or set of columns) that uniquely identifies each row | A student ID — no two students share one |
| **Foreign Key (FK)** | A column that references the primary key of another table | A pointer that says "this row is related to that row over there" |
| **Schema** | The overall structure: all tables, columns, types, and relationships | The blueprint of the database |
| **RDBMS** | The software that manages the database (PostgreSQL, MySQL, SQLite, SQL Server) | The engine that runs the blueprint |

### SQL Data Types That Matter

You don't need to memorize every data type. Focus on the categories:

| Category | Common Types | When To Use |
|----------|-------------|-------------|
| **Integer** | `INT`, `BIGINT`, `SMALLINT`, `SERIAL` | IDs, counts, quantities |
| **Decimal** | `DECIMAL(10,2)`, `NUMERIC` | Money, precise calculations (never use `FLOAT` for money!) |
| **Text** | `VARCHAR(n)`, `TEXT`, `CHAR(n)` | Names, emails, descriptions |
| **Boolean** | `BOOLEAN` | True/false flags (is_active, is_verified) |
| **Date/Time** | `DATE`, `TIMESTAMP`, `TIMESTAMPTZ` | Dates, creation times, deadlines |
| **UUID** | `UUID` | Globally unique identifiers (alternative to auto-increment) |
| **JSON** | `JSON`, `JSONB` | Semi-structured data (PostgreSQL feature, use sparingly) |

> **Common mistake:** Using `VARCHAR(255)` for everything. Choose the type that matches the data. An email column should be `VARCHAR(320)` (the maximum email length per RFC). A boolean flag should be `BOOLEAN`, not `INT`.

> *Reference: Grokking Relational Database Design, Chapter 1 — SQL data types*

---

## Part 3 — The Database Design Process

Designing a database is not about immediately writing `CREATE TABLE`. There is a process:

```
Requirements ──▶ Identify Entities ──▶ Define Attributes ──▶ Establish Relationships
                                                                        │
                 Implementation ◀── Normalization ◀── E-R Diagram ◀───┘
```

### Step 1: Gather Requirements

Before opening any tool, answer these questions:

- **What data does the application need to store?** (users, products, orders, reviews…)
- **What operations will the application perform?** (search products, place orders, generate reports…)
- **What are the business rules?** (a user can have many orders, an order must have at least one item, a product belongs to one category…)
- **What are the access patterns?** (will we query by user? by date? by product category?)

> **Practical tip:** Walk through your application's user stories. Each user story implies data that needs to exist somewhere.

### Step 2: Identify Entities

An **entity** is any object or concept that your system needs to track. Each entity will become a table.

How to find entities:
- Look at the **nouns** in your requirements: "Users can browse **products**, add them to a **cart**, and place **orders**"
- Ask: "Does this thing have its own identity? Do I need to store multiple instances of it?"

| Noun from Requirements | Is It an Entity? | Why / Why Not |
|----------------------|-----------------|--------------|
| User | Yes | Has identity (ID), multiple instances, own attributes |
| Product | Yes | Has identity, own attributes (name, price, description) |
| Order | Yes | Has identity, tracks a transaction over time |
| Email | No | It's an **attribute** of User, not a standalone thing |
| Price | No | It's an **attribute** of Product |
| Cart | Maybe | Could be an entity (if persisted) or just frontend state |

> *Reference: Grokking Relational Database Design, Chapter 4 — Entities and Attributes*

### Step 3: Define Attributes for Each Entity

For each entity, list its attributes and determine:

| Decision | Question | Example |
|----------|---------|---------|
| **Name** | What do we call this attribute? | `first_name`, not `fn` or `FirstName` |
| **Data type** | What kind of data is it? | `VARCHAR(100)`, `INT`, `TIMESTAMP` |
| **Required?** | Can this be NULL or must it always have a value? | Email is required; middle name is optional |
| **Unique?** | Must every value be different? | Email: yes. First name: no |
| **Default?** | Does it have a default value? | `created_at DEFAULT CURRENT_TIMESTAMP` |
| **Primary key** | Which attribute(s) uniquely identify a row? | Usually an auto-incrementing `id` or a `UUID` |

**Naming conventions for columns:**

- Use `snake_case`: `first_name`, not `firstName` or `FirstName`
- Be descriptive: `created_at`, not `date1`
- Use prefixes for foreign keys: `customer_id`, `product_id`
- Boolean columns: `is_active`, `has_verified_email`

### Step 4: Establish Relationships

Entities don't live in isolation. A customer **places** orders. An order **contains** products. These connections are **relationships**.

---

## Part 4 — Relationships and Cardinality

### The Three Types of Relationships

| Relationship | Meaning | Example | Implementation |
|-------------|---------|---------|----------------|
| **One-to-One (1:1)** | Each row in table A relates to exactly one row in table B | User ↔ UserProfile | FK in either table (usually the dependent one) |
| **One-to-Many (1:N)** | One row in table A relates to many rows in table B | Customer → Orders | FK in the "many" table pointing to the "one" table |
| **Many-to-Many (M:N)** | Many rows in A relate to many rows in B | Students ↔ Courses | **Junction table** (also called bridge/join/pivot table) |

### One-to-Many: The Most Common Relationship

A customer can place many orders, but each order belongs to one customer:

```sql
CREATE TABLE customer (
    id       SERIAL PRIMARY KEY,
    name     VARCHAR(100) NOT NULL,
    email    VARCHAR(320) NOT NULL UNIQUE
);

CREATE TABLE "order" (
    id          SERIAL PRIMARY KEY,
    customer_id INT NOT NULL,
    total       DECIMAL(10,2) NOT NULL,
    created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    FOREIGN KEY (customer_id) REFERENCES customer(id)
);
```

The **foreign key** (`customer_id` in the `order` table) enforces referential integrity: you cannot create an order for a customer that doesn't exist.

### Many-to-Many: The Junction Table Pattern

A product can appear in many orders, and an order can contain many products. You cannot represent this with a single foreign key — you need a **junction table**:

```sql
CREATE TABLE order_item (
    id         SERIAL PRIMARY KEY,
    order_id   INT NOT NULL,
    product_id INT NOT NULL,
    quantity   INT NOT NULL DEFAULT 1,
    unit_price DECIMAL(10,2) NOT NULL,

    FOREIGN KEY (order_id)   REFERENCES "order"(id),
    FOREIGN KEY (product_id) REFERENCES product(id)
);
```

```
┌──────────┐       ┌──────────────┐       ┌──────────┐
│  order   │ 1───N │  order_item  │ N───1 │ product  │
│          │       │              │       │          │
│ id (PK)  │       │ order_id(FK) │       │ id (PK)  │
│ total    │       │ product_id(FK│       │ name     │
│ ...      │       │ quantity     │       │ price    │
└──────────┘       │ unit_price   │       └──────────┘
                   └──────────────┘
```

> **Key insight from the book:** A many-to-many relationship is always decomposed into two one-to-many relationships through a junction table. The junction table often carries its own attributes (quantity, unit_price).

### One-to-One: Use Sparingly

One-to-one relationships are rare. Common use cases:

- **Splitting a wide table:** If a user table has 30 columns but most queries only need 5, split the rarely-used columns into a `user_profile` table
- **Optional extension:** Not every user has billing info; keep it in a separate `billing_info` table
- **Security isolation:** Separate sensitive data (SSN, payment details) into a restricted table

```sql
CREATE TABLE user_profile (
    id      INT PRIMARY KEY,
    bio     TEXT,
    avatar  VARCHAR(500),

    FOREIGN KEY (id) REFERENCES "user"(id)
);
```

> *Reference: Grokking Relational Database Design, Chapter 5 — Relationships, cardinality, and dependency*

---

## Part 5 — Entity-Relationship (E-R) Diagrams

### What Is an E-R Diagram?

An E-R diagram is a visual representation of your database design. It shows entities (tables), their attributes, and the relationships between them. It's the **blueprint** you create before writing any SQL.

### Notation Styles

There are several notation styles. The two most common:

| Style | Entities | Relationships | Used In |
|-------|---------|--------------|---------|
| **Chen notation** | Rectangles | Diamonds connecting rectangles | Academic textbooks |
| **Crow's foot notation** | Rectangles | Lines with symbols at the ends | Industry tools (dbdiagram.io, MySQL Workbench, pgAdmin) |

### Crow's Foot Symbols

```
──────────    Exactly one
──────||──    One (and only one)
──────|<──    One to many
──────>|──    Many to one
──────><──    Many to many (rare — decompose into junction table)
──○───────    Zero (optional)
──○──|<──     Zero or many
```

### Tools for Creating E-R Diagrams

| Tool | Free? | Best For |
|------|-------|---------|
| [dbdiagram.io](https://dbdiagram.io) | Yes | Quick, text-based diagrams using DBML syntax |
| [draw.io / diagrams.net](https://app.diagrams.net) | Yes | General purpose diagrams with E-R templates |
| [Excalidraw](https://excalidraw.com) | Yes | Quick hand-drawn style sketches |
| MySQL Workbench | Yes | Full visual E-R modeling with forward/reverse engineering |
| pgAdmin (ERD tool) | Yes | PostgreSQL-specific E-R diagrams |

### Example: E-R Diagram in DBML (for dbdiagram.io)

```dbml
Table customer {
  id        int [pk, increment]
  name      varchar(100) [not null]
  email     varchar(320) [not null, unique]
  created_at timestamp [default: `now()`]
}

Table product {
  id          int [pk, increment]
  name        varchar(200) [not null]
  description text
  price       decimal(10,2) [not null]
  category_id int [ref: > category.id]
}

Table category {
  id   int [pk, increment]
  name varchar(100) [not null, unique]
}

Table order {
  id          int [pk, increment]
  customer_id int [not null, ref: > customer.id]
  status      varchar(20) [not null, default: 'pending']
  created_at  timestamp [default: `now()`]
}

Table order_item {
  id         int [pk, increment]
  order_id   int [not null, ref: > order.id]
  product_id int [not null, ref: > product.id]
  quantity   int [not null, default: 1]
  unit_price decimal(10,2) [not null]
}
```

Paste this into [dbdiagram.io](https://dbdiagram.io) || https://www.drawdb.app/ and you'll get a visual E-R diagram instantly.

> *Reference: Grokking Relational Database Design, Chapters 4 & 5 — Building an E-R diagram step by step*

---

## Part 6 — Normalization: Eliminating Redundancy

### What Is Normalization?

Normalization is the process of organizing your tables to **minimize redundancy** and **prevent anomalies**. It's a series of rules (called "normal forms") that you apply progressively.

### The Anomalies You're Preventing

| Anomaly | What Happens | Example |
|---------|-------------|---------|
| **Update anomaly** | Change data in one place but not others → inconsistency | Customer changes email; updated in 1 of 50 order rows |
| **Insert anomaly** | Can't add data because unrelated required data is missing | Can't add a product if no customer has ordered it yet |
| **Delete anomaly** | Deleting data removes unrelated data you wanted to keep | Deleting the last order for a customer deletes the customer's info too |

### The Normal Forms — A Progressive Checklist

Think of normal forms as levels. Each level builds on the previous:

#### First Normal Form (1NF) — No Repeating Groups

A table is in 1NF if:
- Every column contains **atomic** (indivisible) values
- There are no repeating groups or arrays in a single cell

❌ **Violates 1NF:**

| order_id | customer | products |
|----------|----------|----------|
| 1 | Alice | Widget, Gadget, Gizmo |
| 2 | Bob | Widget |

The `products` column contains a comma-separated list — not atomic.

✅ **In 1NF:**

| order_id | customer | product |
|----------|----------|---------|
| 1 | Alice | Widget |
| 1 | Alice | Gadget |
| 1 | Alice | Gizmo |
| 2 | Bob | Widget |

Each cell contains exactly one value.

#### Second Normal Form (2NF) — No Partial Dependencies

A table is in 2NF if:
- It's already in 1NF
- Every non-key column depends on the **entire** primary key, not just part of it

This applies when the primary key is **composite** (made of multiple columns).

❌ **Violates 2NF:**

| order_id | product_id | product_name | quantity |
|----------|-----------|--------------|----------|
| 1 | 101 | Widget | 3 |
| 1 | 102 | Gadget | 1 |

PK = (order_id, product_id). But `product_name` depends only on `product_id`, not on the full key.

✅ **In 2NF:** Split into two tables:

**order_item:** (order_id, product_id, quantity)
**product:** (product_id, product_name)

*Functional dependency* -> If you know the value of column A, you can figure out the value of column B

`employee_id -> functional determines -> employee_name`
`employee_name -> functional depends -> employee_id`

*A proper subset* is a smaller group completely contained within a larger group, but isn't the same as the large group

#### Third Normal Form (3NF) — No Transitive Dependencies

A table is in 3NF if:
- It's already in 2NF
- No non-key column depends on **another non-key column**

❌ **Violates 3NF:**

| customer_id | name | city | state | zip_code |
|-------------|------|------|-------|----------|
| 1 | Alice | Springfield | IL | 62704 |

Here, `city` and `state` depend on `zip_code`, not directly on `customer_id`. That's a **transitive dependency**: customer_id → zip_code → city/state.

✅ **In 3NF:** Split:

**customer:** (customer_id, name, zip_code)
**zip_location:** (zip_code, city, state)

> **Practical note:** For most web applications, reaching **3NF is sufficient**. Higher normal forms (BCNF, 4NF, 5NF) exist but are rarely needed outside academic contexts.

### The Normalization Decision Flowchart

```
Does every cell contain a single, atomic value?
  ├── No  → Fix it → Now you're in 1NF
  └── Yes
       │
       Does every non-key column depend on the ENTIRE primary key?
         ├── No  → Split the table → Now you're in 2NF
         └── Yes
              │
              Does every non-key column depend ONLY on the primary key
              (no transitive dependencies)?
                ├── No  → Split the table → Now you're in 3NF
                └── Yes → You're in 3NF ✓
```

> *Reference: Grokking Relational Database Design, Chapter 6 — Normalization*

---

## Part 7 — SQL Essentials for Implementation

### Creating Tables

```sql
CREATE TABLE customer (
    id         SERIAL PRIMARY KEY,
    name       VARCHAR(100) NOT NULL,
    email      VARCHAR(320) NOT NULL UNIQUE,
    is_active  BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### Constraints — The Rules Your Database Enforces

| Constraint | Purpose | SQL Syntax |
|-----------|---------|------------|
| `PRIMARY KEY` | Uniquely identifies each row | `id SERIAL PRIMARY KEY` |
| `NOT NULL` | Column cannot be empty | `name VARCHAR(100) NOT NULL` |
| `UNIQUE` | No duplicate values allowed | `email VARCHAR(320) UNIQUE` |
| `FOREIGN KEY` | References a row in another table | `FOREIGN KEY (customer_id) REFERENCES customer(id)` |
| `CHECK` | Custom validation rule | `CHECK (price > 0)` |
| `DEFAULT` | Value when none is provided | `DEFAULT CURRENT_TIMESTAMP` |

> **Key insight:** Constraints are your **first line of defense** for data integrity. They run inside the database engine and cannot be bypassed by application bugs.

### Foreign Keys and Cascade Actions

When you delete a customer, what happens to their orders? Foreign keys let you define this behavior:

```sql
CREATE TABLE "order" (
    id          SERIAL PRIMARY KEY,
    customer_id INT NOT NULL,
    total       DECIMAL(10,2) NOT NULL,

    FOREIGN KEY (customer_id) REFERENCES customer(id)
        ON DELETE CASCADE        -- Delete customer → delete their orders
        ON UPDATE CASCADE        -- Update customer ID → update in orders too
);
```

| Cascade Action | Behavior | Use Case |
|---------------|----------|----------|
| `CASCADE` | Automatically delete/update related rows | Order items when an order is deleted |
| `SET NULL` | Set the FK column to NULL | Assign orders to NULL when a salesperson leaves |
| `SET DEFAULT` | Set the FK column to its default value | Reassign to a default category |
| `RESTRICT` | Prevent the delete/update if related rows exist | Don't allow deleting a customer who has orders |
| `NO ACTION` | Similar to RESTRICT (default in most RDBMS) | Same as RESTRICT, checked at the end of the statement |

> **Best practice:** Be intentional about cascade behavior. `CASCADE` delete is dangerous on important data — prefer `RESTRICT` and handle deletion in application logic where business rules apply.

> *Reference: Grokking Relational Database Design, Chapter 6 — Constraints and cascades*

### Querying Related Tables — JOINs

Once your tables are related through foreign keys, you query across them with JOINs:

```sql
-- Get all orders with customer names
SELECT o.id AS order_id, c.name AS customer_name, o.total
FROM "order" o
INNER JOIN customer c ON o.customer_id = c.id;

-- Get all customers, even those without orders
SELECT c.name, o.id AS order_id
FROM customer c
LEFT JOIN "order" o ON c.id = o.customer_id;

-- Get order details with product information
SELECT o.id AS order_id, p.name AS product, oi.quantity, oi.unit_price
FROM "order" o
INNER JOIN order_item oi ON o.id = oi.order_id
INNER JOIN product p ON oi.product_id = p.id
WHERE o.id = 1;
```

### The Four JOINs You Need to Know

| JOIN Type | Returns | When To Use |
|----------|---------|-------------|
| `INNER JOIN` | Only rows that have a match in **both** tables | Most queries — "show me orders with their customers" |
| `LEFT JOIN` | All rows from the left table + matching rows from the right (NULL if no match) | "Show me all customers, even without orders" |
| `RIGHT JOIN` | All rows from the right table + matching rows from the left | Rarely used — rewrite as a LEFT JOIN with tables swapped |
| `FULL OUTER JOIN` | All rows from both tables, NULLs where there's no match | Comparing two datasets for differences |

> *Reference: Grokking Relational Database Design, Chapter 2 — Related tables, foreign keys, and JOINs*

---

## Part 8 — Indexing and Performance

### What Is an Index?

An index is a data structure that makes queries faster — like the index at the back of a book. Without an index, the database must scan **every row** in the table to find what you're looking for (a "full table scan"). With an index, it can jump directly to the relevant rows.

### When to Create Indexes

| Create an Index On... | Because... |
|----------------------|-----------|
| Primary keys | Automatically indexed by the RDBMS |
| Foreign keys | JOINs and lookups on FKs are extremely common |
| Columns in `WHERE` clauses | Speeds up filtering |
| Columns in `ORDER BY` | Speeds up sorting |
| Columns in `JOIN ON` | Speeds up join operations |
| Columns used in `UNIQUE` constraints | Automatically indexed |

### Creating Indexes

```sql
-- Index on a frequently filtered column
CREATE INDEX idx_product_category ON product(category_id);

-- Index on a column used for sorting
CREATE INDEX idx_order_created ON "order"(created_at);

-- Composite index for queries that filter on both columns
CREATE INDEX idx_order_customer_date ON "order"(customer_id, created_at);

-- Unique index (enforces uniqueness + speeds up lookups)
CREATE UNIQUE INDEX idx_customer_email ON customer(email);
```

### The Cost of Indexes

Indexes are not free:

| Benefit | Cost |
|---------|------|
| Faster `SELECT` queries | Slower `INSERT`, `UPDATE`, `DELETE` (the index must be updated too) |
| Faster `WHERE` filtering | More disk space |
| Faster `JOIN` operations | More memory usage |

> **Rule of thumb:** Index columns you **read frequently** and **write infrequently**. Don't index every column — measure first, then optimize.

### When to Consider Denormalization

Normalization reduces redundancy. But sometimes, fully normalized schemas require many JOINs, and those JOINs hurt performance.

**Denormalization** means intentionally introducing redundancy into a normalized schema to improve read performance at the cost of write complexity.

> **The rule:** Normalize first. Measure the real bottleneck. Denormalize only when you have evidence — not speculation — that a normalized design is causing a problem. Premature denormalization creates data integrity bugs that are hard to find and expensive to fix.

#### The Fundamental Trade-off

| Normalized | Denormalized |
|------------|-------------|
| One place to update a value | Multiple places to update the same value |
| Risk of anomalies is low | Risk of stale / inconsistent data |
| Reads may require JOINs | Reads are simpler and faster |
| Suitable for write-heavy workloads | Suitable for read-heavy workloads |

#### Legitimate Reasons to Denormalize

**1. Pre-computing aggregated values**

When a value would require recalculating an aggregate over many rows on every read, storing the result is justified — as long as you keep it updated.

```sql
-- Denormalized: store the total on the order row
-- instead of summing order_items on every request
ALTER TABLE "order" ADD COLUMN total DECIMAL(10,2) NOT NULL DEFAULT 0;

-- You must update it whenever items are inserted, updated, or deleted
UPDATE "order" SET total = (
    SELECT SUM(quantity * unit_price) FROM order_item WHERE order_id = 1
) WHERE id = 1;
```

| Scenario | Aggregated Column | Must Update When... |
|----------|------------------|---------------------|
| Order total on dashboard | `order.total` | Item added, quantity changed, item removed |
| Products per category | `category.product_count` | Product created or reassigned |
| Average product rating | `product.avg_rating` | Review added, edited, or deleted |
| User's order count | `user.order_count` | Order placed or cancelled |

**2. Snapshotting data at the time of a transaction**

This is perhaps the most important and most overlooked reason to denormalize. When a transaction is recorded, the values that were true *at that moment* should be stored directly — not referenced from a table that can change later.

```sql
-- order_item stores unit_price at the time of the order
-- NOT a reference to product.price (which may change tomorrow)
CREATE TABLE order_item (
    id         SERIAL PRIMARY KEY,
    order_id   INT NOT NULL REFERENCES "order"(id),
    product_id INT NOT NULL REFERENCES product(id),
    quantity   INT NOT NULL DEFAULT 1,
    unit_price DECIMAL(10,2) NOT NULL   -- snapshot of price at purchase time
);
```

Same pattern applies to:
- Storing `customer_name` / `shipping_address` on an order (the customer may update their profile later)
- Storing `tax_rate` on an invoice (tax rates change)
- Storing `discount_amount` on an order line (promotions expire)

**3. Avoiding expensive JOINs on hot read paths**

If a specific query is executed thousands of times per second and its `EXPLAIN ANALYZE` shows a costly JOIN even with proper indexes, copying the joined column into the main table is reasonable.

```sql
-- Normalized: every dashboard load JOINs customer to get the name
SELECT o.id, c.name FROM "order" o JOIN customer c ON o.customer_id = c.id;

-- Denormalized: store customer_name directly on the order
-- (acceptable here because order history doesn't need to reflect name changes)
ALTER TABLE "order" ADD COLUMN customer_name VARCHAR(100);
```

Use this pattern when:
- The source value is stable (unlikely to change, or changes don't need to propagate)
- The query is a known hot path with a measured performance problem
- Adding an index on the join column did not solve the problem

**4. Read-optimized reporting and analytics tables**

For aggregate reports and analytics dashboards that are never used to write data back, maintaining a separate denormalized table (or a materialized view) avoids degrading the transactional schema:

```sql
-- A denormalized summary table refreshed nightly or via triggers
CREATE TABLE daily_sales_summary (
    report_date  DATE PRIMARY KEY,
    total_orders INT NOT NULL,
    total_revenue DECIMAL(12,2) NOT NULL,
    top_category VARCHAR(100)
);
```

This keeps analytical queries fast without impacting `INSERT`/`UPDATE` throughput on the transactional tables.

#### When NOT to Denormalize

| Situation | Why Denormalization Is Wrong Here |
|-----------|----------------------------------|
| You haven't measured a performance problem | You're optimizing for a problem that may not exist |
| The column changes frequently | Every change must propagate; risk of corruption is high |
| The data is small (< a few thousand rows) | A full table scan is fast; an index solves it |
| A proper index would fix the query | Add the index first — it's much safer |
| You're doing it to avoid writing a JOIN | That's a code simplicity shortcut, not a performance reason |
| The data models a current fact (address, price) | Use a reference; snapshotting is only for history |

#### Keeping Denormalized Data Consistent

Once you denormalize, the burden of consistency moves from the database (constraints) to your application or database triggers. Your options:

| Strategy | How It Works | Tradeoff |
|----------|-------------|----------|
| **Application logic** | The code that writes to the source table also updates the denormalized column | Simple to understand; can be missed by direct SQL operations |
| **Database trigger** | A trigger automatically updates the denormalized value on insert/update/delete | Consistent regardless of how data is changed; harder to test and debug |
| **Scheduled job** | A background job recalculates the denormalized value periodically | Fine for analytics; introduces staleness for transactional data |
| **Materialized view** | The database maintains a pre-computed query result | Declarative and clean; must be refreshed; PostgreSQL supports `REFRESH MATERIALIZED VIEW` |

```sql
-- Example: trigger to keep order.total in sync
CREATE OR REPLACE FUNCTION update_order_total()
RETURNS TRIGGER AS $$
BEGIN
    UPDATE "order"
    SET total = (
        SELECT COALESCE(SUM(quantity * unit_price), 0)
        FROM order_item
        WHERE order_id = COALESCE(NEW.order_id, OLD.order_id)
    )
    WHERE id = COALESCE(NEW.order_id, OLD.order_id);
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_order_total
AFTER INSERT OR UPDATE OR DELETE ON order_item
FOR EACH ROW EXECUTE FUNCTION update_order_total();
```

> *Reference: Grokking Relational Database Design, Chapter 7 — Security and Optimization (indexing, denormalization)*

---

## Part 9 — SQL vs NoSQL: When to Use Each

### The Two Paradigms

| Aspect | Relational (SQL) | Document (NoSQL) |
|--------|-----------------|------------------|
| **Data model** | Tables with rows and columns, strict schema | Collections of JSON-like documents, flexible schema |
| **Schema** | Defined upfront (`CREATE TABLE`) | Schema-on-read (structure can vary per document) |
| **Relationships** | First-class: foreign keys, JOINs | Embedded documents or manual references (no JOINs) |
| **Query language** | SQL (standardized) | Database-specific (MongoDB Query Language, etc.) |
| **Transactions** | Full ACID support across tables | Limited (per-document in MongoDB; multi-document since v4.0) |
| **Scaling** | Vertical (bigger server) + read replicas | Horizontal (add more servers / sharding) |
| **Examples** | PostgreSQL, MySQL, SQL Server, SQLite | MongoDB, DynamoDB, CouchDB, Firestore |

### When to Choose SQL (Relational)

Choose a relational database when:

- Your data has **clear structure and relationships** (users, orders, products, invoices)
- You need **referential integrity** (foreign keys, constraints)
- You need **complex queries** with JOINs, aggregations, GROUP BY
- **Data consistency** is critical (financial transactions, inventory)
- The schema is relatively **stable** (you know the structure upfront)

### When to Choose NoSQL (Document)

Choose a document database when:

- Your data is **highly variable** or **semi-structured** (CMS content, user-generated data, IoT events)
- You need **horizontal scaling** to handle massive write throughput
- Your access pattern is **read-one-document** (fetch a user profile with all nested data)
- Schema changes are **frequent** and unpredictable
- You're prototyping and need to move fast (no migrations)

### Honest Answer for Most Web Apps

For the vast majority of web applications (CRUD apps with users, products, orders):

> **Start with PostgreSQL.** It handles relational data, has JSON support (`JSONB`), full-text search, and scales well beyond what most apps need. Add a document database only when you have a specific use case that demands it.

| Project Type | Recommended Database | Why |
|-------------|---------------------|-----|
| E-commerce, SaaS, CRM | PostgreSQL / MySQL | Strong relationships, transactions, data integrity |
| Blog / CMS with flexible content | PostgreSQL (with JSONB) or MongoDB | Flexible content shapes |
| Real-time analytics / IoT | MongoDB, ClickHouse, TimescaleDB | High write throughput, time-series data |
| Chat / messaging | MongoDB or PostgreSQL | Depends on scale and query patterns |
| Your course project | **PostgreSQL** | Industry standard, teaches relational concepts, free tier on most cloud providers |

---

## Part 10 — Connecting Your Backend to a Database

### The Architecture

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│   Frontend   │ ──▶  │   Backend    │ ──▶  │   Database   │
│  (React,     │      │  (Express,   │      │ (PostgreSQL, │
│   Vue, etc.) │      │   Flask,     │      │  MongoDB,    │
│              │      │   .NET, etc.)│      │  MySQL)      │
└──────────────┘      └──────────────┘      └──────────────┘
                       │            │
                       │  ORM/      │
                       │  Driver    │
                       │            │
```

### Two Approaches: Raw Driver vs ORM

| Approach | What It Is | Pros | Cons |
|----------|-----------|------|------|
| **Raw driver** | Use the database driver directly and write SQL strings | Full control, no abstractions, great for learning | More boilerplate, manual SQL injection prevention |
| **ORM** | Object-Relational Mapper — define models in code, ORM generates SQL | Less boilerplate, safer by default, migrations built-in | Abstraction can hide performance issues, learning curve |

**Common ORMs by stack:**

| Stack | ORM / Query Builder | Raw Driver |
|-------|-------------------|------------|
| **Node.js** | Prisma, Drizzle, Sequelize, TypeORM | `pg` (PostgreSQL), `mysql2` |
| **Python** | SQLAlchemy, Django ORM | `psycopg2`, `PyMySQL` |
| **.NET** | Entity Framework Core | `Npgsql`, `SqlClient` |

---

## Part 11 — Designing Your Project's Database (Guided Exercise)

### The Exercise

Take your personal project and design its database following this process:

#### 1. List Your Entities

Write down every entity your application needs. Start with user stories:

> "As a **user**, I can browse **products** by **category**, add them to my **cart**, and place an **order**."

Entities: `User`, `Product`, `Category`, `Cart`, `Order`, `OrderItem`

#### 2. Draw the E-R Diagram

Use [dbdiagram.io](https://dbdiagram.io) or [drawdb.app](https://www.drawdb.app/) or Excalidraw to create a visual E-R diagram showing all entities, attributes, and relationships.

#### 3. Normalize to 3NF

Walk through each table and check:
- 1NF: Are all values atomic?
- 2NF: Do all non-key columns depend on the entire PK?
- 3NF: Are there any transitive dependencies?

#### 4. Implement and Connect

Using your chosen ORM or driver, implement the schema in your project and verify you can create, read, update, and delete data.

---

## Part 12 — Common Database Design Mistakes

| Mistake | Why It's a Problem | Fix |
|---------|-------------------|-----|
| Storing everything in one table | Anomalies, redundancy, slow queries | Normalize into separate entities |
| Using `VARCHAR(255)` for everything | Wastes space, unclear intent | Choose appropriate types and lengths |
| No foreign keys | Data integrity depends entirely on application code (which has bugs) | Always define FK constraints |
| No indexes on foreign keys | JOINs become full table scans as data grows | Index every FK column |
| Storing prices/money as `FLOAT` | Floating-point rounding errors (0.1 + 0.2 ≠ 0.3) | Use `DECIMAL(10,2)` |
| Not using `NOT NULL` | Queries must handle NULL everywhere; bugs hide in NULL values | Default to NOT NULL, allow NULL only when semantically meaningful |
| Storing comma-separated values | Violates 1NF, impossible to query efficiently | Create a separate table and use a relationship |
| No timestamps | Can't debug, audit, or sort chronologically | Add `created_at` and `updated_at` to every table |
| Hardcoded connection strings | Security risk, can't deploy to different environments | Use environment variables |

---

## Deliverable

By the end of this week, you should have:

- [ ] **E-R diagram** of your project's database (created with dbdiagram.io, Excalidraw, or similar)
- [ ] **Normalization check** — evidence that your schema is at least in 3NF (or documented reasons for intentional denormalization)
- [ ] **Working connection** — your backend can create, read, update, and delete records in the database
- [ ] **Seed data** — a script or migration that populates the database with sample data for development

---

## Recommended Reading

- [Grokking Relational Database Design](https://www.manning.com/books/grokking-relational-database-design) (Hao & Tsikerdekis, Manning 2025) — The primary reference for this week. Chapters 1–6 cover everything from SQL basics to normalization.
- [GitHub: grokking-relational-database-design](https://github.com/Neo-Hao/grokking-relational-database-design) — Companion code repository with SQL scripts for every chapter
- [SQLite Online](https://sqliteonline.com/) — Practice SQL in the browser without installing anything
- [dbdiagram.io](https://dbdiagram.io) — Create E-R diagrams from a simple text-based DSL
- [PostgreSQL Official Tutorial](https://www.postgresql.org/docs/current/tutorial.html) — The definitive PostgreSQL getting started guide
- [Prisma Documentation](https://www.prisma.io/docs) — If using Node.js, a modern ORM with great docs
- [Use The Index, Luke](https://use-the-index-luke.com/) — Deep dive into how database indexes work
- [YouTube: Grokking Relational Database Design playlist](https://www.youtube.com/playlist?list=PL3fg3zQpW0k4UO9eBDLdroADnB18ZAOgj) — Video lectures from the book's authors
