# Cafeteria POS — Data Modeling “Hello World” (Beginner Friendly)

## Scenario (simple words)
A small cafeteria sells food and drinks. A cashier creates an order when a customer buys something.  
Each order can contain multiple items (e.g., coffee + sandwich) and the customer pays (cash or card).  
The café wants to store this data to answer questions like: **“How much did we sell today?”** and **“What items are most popular?”**

---

## Step 1 — Identify entities (tables) and attributes (columns)
To store the business data in a database, we first identify the “things” we need to store:

- **Customer** (who buys)
- **Order** (the purchase event)
- **Order Line** (the items inside an order)
- **Payment** (how the order was paid)
- **Menu Item** (the products sold)

**Entity = Table** (a “thing”)  
**Attribute = Column** (a detail about the thing)

---

## Step 2 — A typical beginner model (with intentional mistakes)
Beginners often try to store everything inside one table.

### ❌ Naive table: `orders`
| Column | What it means |
|---|---|
| `order_id` (PK) | unique order number |
| `order_time` | when the order happened |
| `customer_id` | customer identifier |
| `customer_name` | customer name |
| `customer_phone` | customer phone |
| `cashier_name` | cashier who processed the order |
| `items` | text like `"Coffee x2, Sandwich x1"` |
| `total_amount` | total money for the order |
| `payment_method` | cash / card |

---

## Step 3 — Why this model is wrong (linking to normalization)

### 1NF problem: “lists inside a cell”
The column `items = "Coffee x2, Sandwich x1"` violates **1NF** because one cell contains multiple values.
- Hard to query: “How many coffees were sold?”
- Hard to validate: you must parse text

### 3NF problem: attributes that depend on non-key attributes (transitive dependency)
Assume the primary key is `order_id`.

In this table:
- `order_id → customer_id` (each order belongs to one customer)
- `customer_id → customer_name, customer_phone` (customer details are determined by the customer)

So:
`order_id → customer_id → customer_name`

This means `customer_name` and `customer_phone` do **not** depend directly on the primary key (`order_id`).  
They depend on another non-key attribute (`customer_id`) → this is a **3NF violation** (transitive dependency).

**Practical consequence:** customer details repeat in many orders and can become inconsistent when data changes.

---

## Step 4 — Fix direction (what normalization leads us to)
To fix the issues we separate data into tables where each table stores facts about one entity.

### ✅ Better entity split (high level)
- `customer(customer_id, name, phone, ...)`
- `sales_order(order_id, order_time, customer_id, cashier_id, ...)`
- `sales_order_line(order_id, line_nbr, item_id, qty, unit_price, ...)`
- `payment(payment_id, order_id, method, amount, ...)`
- `menu_item(item_id, item_name, current_price, ...)`

This structure:
- fixes **1NF** by putting each item on its own row in `sales_order_line`
- moves customer details to the `customer` table to support **3NF**
- makes analytics and reporting much easier

---

## Key takeaways
- **1NF**: don’t store lists inside a single column (no repeating groups).
- **3NF**: avoid transitive dependency (non-key columns should not depend on other non-key columns).
- Good models reduce duplication, prevent update anomalies, and make querying easier.
