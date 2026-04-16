# Database Normalization Exercises

## Decoupling Ugly Tables

Each exercise presents a single, brittle table that violates one or more normal forms. Your job:

1. **Identify** which normal form(s) the table violates and why.
2. **Describe** the anomalies (insert, update, delete) that can occur.
3. **Redesign** the table into a properly normalized schema (at least 3NF).
4. **Write** the `CREATE TABLE` statements for your new design.

---

## Exercise 1 — The "Everything Store" Orders Table

A small online store keeps all order data in one table:

| order_id | customer_name | customer_email     | customer_city | product_name | product_price | category      | quantity | order_date |
|----------|---------------|--------------------|---------------|--------------|---------------|---------------|----------|------------|
| 1        | Alice Rivera  | alice@mail.com     | Santo Domingo | Wireless Mouse | 12.99       | Peripherals   | 2        | 2025-09-01 |
| 1        | Alice Rivera  | alice@mail.com     | Santo Domingo | USB Keyboard   | 24.99       | Peripherals   | 1        | 2025-09-01 |
| 2        | Bob Santos    | bob@mail.com       | Santiago      | Wireless Mouse | 12.99       | Peripherals   | 1        | 2025-09-03 |
| 3        | Alice Rivera  | alice@mail.com     | Santo Domingo | HDMI Cable     | 8.50        | Cables        | 3        | 2025-09-05 |
| 4        | Carlos Peña   | carlos@mail.com    | La Vega       | USB Keyboard   | 24.99       | Peripherals   | 2        | 2025-09-06 |
| 4        | Carlos Peña   | carlos@mail.com    | La Vega       | HDMI Cable     | 8.50        | Cables        | 1        | 2025-09-06 |

> **Starting point for dbdiagram.io** — paste the DBML below to visualize the brittle table, then redesign it.

```dbml
Table flat_order {
  order_id       int          [note: "Not a PK alone — same order_id repeats for multiple products"]
  customer_name  varchar(100)
  customer_email varchar(320)
  customer_city  varchar(100)
  product_name   varchar(200)
  product_price  decimal(10,2)
  category       varchar(100)
  quantity       int
  order_date     date
}
```

### Questions

**a)** List every normal-form violation you can find (1NF, 2NF, 3NF). For each one, explain what depends on what.

**b)** Describe one concrete example of each anomaly type (insert, update, delete) that this table allows.

**c)** Decompose the table into a set of normalized tables. Draw the relationships (or describe them in text/DBML).

---

## Exercise 2 — The University Course Registration Table

The university registrar stores enrollments like this:

| student_id | student_name  | student_phone          | course_code | course_title               | instructor      | instructor_office | semester   | grade |
|------------|---------------|------------------------|-------------|-----------------------------|-----------------|-------------------|------------|-------|
| 2021-0045  | María López   | 809-555-1234           | ISW-311     | Intro to Software Eng.      | Prof. Ramírez   | B-204             | 2025-1     | A     |
| 2021-0045  | María López   | 809-555-1234           | MAT-201     | Linear Algebra              | Prof. Cruz      | A-110             | 2025-1     | B+    |
| 2021-0112  | José Herrera  | 829-555-5678           | ISW-311     | Intro to Software Eng.      | Prof. Ramírez   | B-204             | 2025-1     | B     |
| 2021-0045  | María López   | 809-555-1234, 849-555-9999 | ISW-331 | Web Programming II          | Prof. Ramírez   | B-204             | 2025-2     | NULL  |
| 2021-0200  | Ana Reyes     | 809-555-0000           | MAT-201     | Linear Algebra              | Prof. Cruz      | A-110             | 2025-1     | A-    |

> **Starting point for dbdiagram.io** — paste the DBML below to visualize the brittle table, then redesign it.

```dbml
Table flat_enrollment {
  student_id       varchar(10)  [note: "Part of composite PK: (student_id, course_code, semester)"]
  student_name     varchar(100)
  student_phone    varchar(100) [note: "Violates 1NF — may contain comma-separated numbers"]
  course_code      varchar(10)
  course_title     varchar(200)
  instructor       varchar(100)
  instructor_office varchar(10)
  semester         varchar(10)
  grade            varchar(5)   [note: "NULL when course not yet completed"]
}
```

### Questions

**a)** This table violates 1NF in at least one row. Find it and explain why it breaks atomicity.

**b)** Assuming a composite primary key of `(student_id, course_code, semester)`, identify the partial dependencies (2NF violations) and the transitive dependencies (3NF violations).

**c)** Decompose the table into normalized tables that reach 3NF. Be specific about primary keys and foreign keys.

**d)** In your new schema, how would you handle the fact that a student can have multiple phone numbers without violating 1NF?

**e)** Write an `INSERT` statement that enrolls student `2021-0112` into `ISW-331` for semester `2025-2` using your normalized tables. How many tables do you need to touch?

---

## Exercise 3 — The Event Venue Booking Table

An event management company tracks all bookings in this single table:

| booking_id | event_name          | event_type  | client_name     | client_email        | venue_name       | venue_address              | venue_capacity | booking_date | start_time | end_time | total_price | catering_package | catering_price_per_head | guest_count |
|------------|---------------------|-------------|-----------------|---------------------|------------------|---------------------------|----------------|--------------|------------|----------|-------------|------------------|------------------------|-------------|
| 1001       | Tech Meetup RD      | Conference  | Pedro Martínez  | pedro@techrd.com    | Salón Diamante   | Av. Lincoln 500, SD       | 200            | 2025-10-15   | 09:00      | 17:00    | 45000.00    | Premium          | 350.00                 | 80          |
| 1002       | Wedding Gómez-Lara  | Wedding     | Luisa Gómez     | luisa@mail.com      | Jardín Imperial  | Carr. Santiago Km 5       | 300            | 2025-11-22   | 16:00      | 23:00    | 120000.00   | Deluxe           | 500.00                 | 150         |
| 1003       | Birthday Bash       | Party       | Pedro Martínez  | pedro@techrd.com    | Salón Diamante   | Av. Lincoln 500, SD       | 200            | 2025-12-01   | 19:00      | 01:00    | 28000.00    | Standard         | 250.00                 | 60          |
| 1004       | Tech Meetup RD v2   | Conference  | Pedro Martínez  | pedro.m@newmail.com | Salón Diamante   | Av. Lincoln 500, SD       | 200            | 2026-01-20   | 09:00      | 17:00    | 52000.00    | Premium          | 350.00                 | 100         |
| 1005       | Company Retreat     | Corporate   | Ana Castillo    | ana@corp.com        | Jardín Imperial  | Carr. Santiago Km 5       | 300            | 2026-02-14   | 08:00      | 18:00    | 75000.00    | Deluxe           | 500.00                 | 120         |

> **Starting point for dbdiagram.io** — paste the DBML below to visualize the brittle table, then redesign it.

```dbml
Table flat_booking {
  booking_id               int           [pk]
  event_name               varchar(200)
  event_type               varchar(50)
  client_name              varchar(100)  [note: "Same client can appear with different emails"]
  client_email             varchar(320)
  venue_name               varchar(200)  [note: "Repeated across many bookings"]
  venue_address            varchar(300)
  venue_capacity           int
  booking_date             date
  start_time               time
  end_time                 time
  total_price              decimal(10,2) [note: "Is this calculated or independent?"]
  catering_package         varchar(50)
  catering_price_per_head  decimal(10,2) [note: "Depends on catering_package, not on booking_id"]
  guest_count              int
}
```

### Questions

**a)** Look at rows 1001 and 1004. Pedro Martínez appears with two different emails. What anomaly does this represent, and why did the current design allow it?

**b)** Suppose Salón Diamante changes its address. How many rows need to be updated? What is the risk?

**c)** You want to add a new venue, "Centro de Convenciones del Cibao", that nobody has booked yet. Can you do it in this table? Why or why not?

**d)** Identify all the entities hidden inside this single table. For each entity, list its attributes.

**e)** Decompose the table into 3NF. Pay special attention to:
   - Where `total_price` comes from (is it `catering_price_per_head × guest_count`, or is it independent?)
   - Whether `catering_package` and `catering_price_per_head` should live in their own table

**f)** The `total_price` column is a candidate for **intentional denormalization**. Under what conditions would you keep it as a stored column on the booking table instead of calculating it on the fly? What mechanism would you use to keep it in sync?

---

## Submission Guidelines

For each exercise, submit:

1. A list of violations and anomalies (plain text or a table).
2. A brief paragraph justifying any design decisions (e.g., "I kept `total_price` on the booking table because…").
