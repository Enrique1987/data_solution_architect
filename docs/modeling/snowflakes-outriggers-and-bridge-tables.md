# Snowflakes, Outriggers, and Bridge Tables in Dimensional Modeling

## Abstract / Conclusion

All three concepts extend a **star schema**, but they solve different problems:

- **Snowflake**: splits one dimension into several normalized tables. It is generally best avoided.
- **Outrigger**: connects one dimension to another dimension as a targeted exception. It can be useful, but sparingly.
- **Bridge table**: an intermediate table needed to represent complex relationships, usually **many-to-many** relationships or variable-depth hierarchies.

Simplicity is especially valuable in dimensional modeling. Kimball therefore recommends keeping dimensions as denormalized as possible, using snowflakes, outriggers, and bridges only when there is a clear reason.

```text
SNOWFLAKE
"Let's split this dimension."

OUTRIGGER
"This dimension needs another dimension."

BRIDGE
"One key is not enough to represent this relationship."
```

---

## 1. Starting with a Standard Star Schema

Imagine an online store with a fact table:

```text
fact_sales

sale_id
product_key
customer_key
date_key
quantity
revenue
```

And these dimensions:

```text
dim_product
dim_customer
dim_date
```

```text
              dim_product
                   |
dim_customer -- fact_sales -- dim_date
```

This is a normal **star schema**. The product dimension could contain:

```text
dim_product

product_key
product_name
brand
category
subcategory
manufacturer
color
size
```

For example:

```text
101 | iPhone 17     | Apple   | Electronics | Smartphones | Apple Inc | Black
102 | iPhone 17 Pro | Apple   | Electronics | Smartphones | Apple Inc | Silver
103 | Galaxy S26    | Samsung | Electronics | Smartphones | Samsung   | Black
```

Values such as `Apple`, `Electronics`, and `Smartphones` repeat. In a transactional OLTP system, that can look undesirable. In a dimensional data warehouse, it is normally acceptable: Kimball generally favors this simple structure over normalizing every dimension.

---

## 2. Snowflake Schemas

A **snowflake schema** appears when we normalize a dimension. Instead of this:

```text
dim_product

product_key
product_name
brand
category
```

we might create:

```text
dim_product

product_key
product_name
brand_key
```

Then:

```text
dim_brand

brand_key
brand_name
category_key
```

And then:

```text
dim_category

category_key
category_name
```

The model becomes:

```text
fact_sales
    |
dim_product
    |
dim_brand
    |
dim_category
```

That is a **snowflake**: one dimension has been split into several normalized tables.

### 2.1 Why would someone do this?

An OLTP-oriented developer might ask: “Why store the text `Apple` 50,000 times when it can be stored once?”

```text
dim_brand

1 | Apple
2 | Samsung
3 | Google
```

```text
dim_product

101 | iPhone 17     | 1
102 | iPhone 17 Pro | 1
103 | Galaxy S26    | 2
```

This is clean from a relational-normalization perspective, but it can add unnecessary complexity to a data warehouse.

### 2.2 Why Kimball Usually Discourages Snowflakes

#### More joins

With a denormalized dimension, this may be enough:

```text
fact_sales
JOIN dim_product
```

With a snowflake, a query may require:

```text
fact_sales
JOIN dim_product
JOIN dim_brand
JOIN dim_category
JOIN dim_department
```

#### Harder for users

A Power BI user may see separate tables for product, brand, category, subcategory, manufacturer, department, supplier, and supplier country. A single `dim_product` table containing the relevant attributes is easier to understand and use.

#### More complex ETL

For a hierarchy such as `Product → Brand → Category → Department`, ETL must maintain all keys and relationships. Historical changes make that maintenance considerably harder.

#### Storage savings rarely justify it

Avoiding repeated text is a common justification, but Kimball considers the added complexity usually not worth the storage saved. Modern formats and engines such as Parquet, Delta Lake, columnar compression, and dictionary encoding compress repeated values very effectively.

### 2.3 Remembering Snowflakes

```text
Product
   ↓
Brand
   ↓
Category
   ↓
Department
```

Kimball would usually prefer:

```text
dim_product

product
brand
category
department
```

In short:

```text
SNOWFLAKE = splitting one dimension into multiple normalized tables
```

---

## 3. Outriggers

An **outrigger** also introduces a `Dimension → Dimension` relationship, but it is more limited than a snowflake. It is used when a dimension needs exceptional access to another existing dimension.

### 3.1 Example: Employee and Hire Date

```text
dim_employee

employee_key
employee_name
department
hire_date_key
```

The hire date could be stored directly as `2021-05-17`. But suppose users need calendar attributes such as fiscal year, fiscal quarter, fiscal month, business week, and holiday indicator. Those attributes already exist in `dim_date`:

```text
fact_salary
     |
dim_employee
     |
dim_date
```

```text
dim_employee.hire_date_key
        ↓
dim_date.date_key
```

Here, `dim_date` acts as an **outrigger**.

### 3.2 Another Example: Customer Signup Date

```text
dim_customer

customer_key
customer_name
city
signup_date_key
```

To analyze customers by signup fiscal year, quarter, or month, connect `dim_customer` to `dim_date`. The date dimension is again an outrigger.

### 3.3 Is an Outrigger Just a Snowflake?

The concepts are related because both introduce `Dimension → Dimension` relationships. The difference is scope:

```text
Snowflake = a systematic normalization strategy
Outrigger = a targeted exception
```

A snowflake might be `fact_sales → dim_product → dim_brand → dim_category → dim_department`. An outrigger might simply be `fact_salary → dim_employee → dim_date`.

Kimball accepts outriggers in moderation. If one dimension points to department, location, manager, date, job, and company dimensions, the model has probably become a snowflake and should be simplified.

---

## 4. Bridge Tables

**Bridge tables** solve a different problem: they are not primarily about normalization. Use one when a relationship cannot be represented correctly by a single foreign key. The most common case is **many-to-many**.

### 4.1 Example: Bank Accounts

Consider an account with a balance of €10,000 that belongs to both John and Mary.

```text
fact_account_balance

account_key
customer_key
date_key
balance
```

Which `customer_key` should the fact contain? Keeping only John drops Mary; keeping only Mary drops John. Storing two fact rows duplicates the balance and could lead BI to report €20,000. Splitting it into €5,000 each imposes a business meaning that may not exist.

The correct model keeps the balance once:

```text
fact_account_balance

account_key
date_key
balance

A123 | 20260828 | €10,000
```

And represents the relationship separately:

```text
bridge_account_customer

account_key
customer_key

A123 | John
A123 | Mary
```

Conceptually:

```text
fact_account_balance
        |
     Account
        |
bridge_account_customer
     /       \
   John      Mary
```

The bridge says that the account relates to John and Mary, while the balance exists only once.

### 4.2 Example: Movies and Actors

A movie has many actors and an actor can appear in many movies, so `Movie ↔ Actor` is many-to-many.

```text
bridge_movie_actor

movie_key
actor_key

Matrix    | Keanu
Matrix    | Carrie-Anne
Matrix    | Laurence
John Wick | Keanu
```

That is a bridge table.

### 4.3 Other Common Cases

```text
Customer ↔ Account
Patient ↔ Diagnosis
Student ↔ Course
Employee ↔ Skill
Movie ↔ Actor
Article ↔ Tag
Order ↔ Promotion
```

Adding columns such as `customer_1_key`, `customer_2_key`, and so on is not a solution: the number of related entities is not known in advance. A bridge represents any number of relationships without inventing a fixed limit.

### 4.4 Variable-Depth Hierarchies

Bridge tables also help with ragged or variable-depth hierarchies. One organization might have `CEO → Manager → Employee`, while another branch might include vice president, director, senior manager, manager, team lead, and employee.

Instead of fixed `level_1`, `level_2`, and `level_3` columns, use a hierarchy bridge:

```text
bridge_employee_hierarchy

employee
ancestor
distance

Alice | Bob   | 1
Alice | Sarah | 2
Alice | John  | 3
```

For the hierarchy below, this supports questions such as “Show me the sales of every employee below John,” without knowing how many levels separate John from each employee.

```text
John
 ↓
Sarah
 ↓
Bob
 ↓
Alice
```

---

## 5. Comparing the Three Concepts

| Concept | Problem it solves | Shape |
| --- | --- | --- |
| **Snowflake** | Normalizing a dimension's attributes | Fact → Dim → Dim → Dim |
| **Outrigger** | A targeted relationship between dimensions | Fact → Dim → Dim |
| **Bridge** | Many-to-many relationships or complex hierarchies | Fact/Dim → Bridge → Dim |

```text
SNOWFLAKE
"I want to split this dimension."

OUTRIGGER
"This dimension needs access to another dimension."

BRIDGE
"A single foreign key cannot represent this relationship."
```

---

## 6. Complete Airline Example

Start with:

```text
fact_booking

booking_key
passenger_key
flight_key
date_key
revenue
```

And dimensions `dim_passenger`, `dim_flight`, and `dim_date`.

### 6.1 Snowflake

An initial flight dimension might contain flight number, origin and destination airport and country, aircraft type, and manufacturer. Splitting it into `dim_flight → dim_airport → dim_country`, or into `dim_flight → dim_aircraft → dim_aircraft_manufacturer`, creates a snowflake. Kimball would generally prefer to retain many of those attributes in denormalized dimensions.

### 6.2 Outrigger

```text
dim_passenger

passenger_key
name
frequent_flyer_status
registration_date_key
```

Connecting `registration_date_key` to `dim_date` enables analysis by registration fiscal year, quarter, and month. The date dimension is an outrigger.

### 6.3 Bridge

One booking can contain multiple passengers, and a passenger can have many bookings. The relationship is represented by:

```text
bridge_booking_passenger

booking_key
passenger_key
```

This is a bridge table.

---

## 7. Kimball's Recommendation

### Snowflake

**Usually avoid it.** It is not technically wrong, but it normally adds joins, modeling complexity, BI difficulty, and ETL complexity. Prefer a dimension containing `product`, `brand`, `category`, `department`, and `manufacturer` over a chain of normalized dimension tables.

### Outrigger

**Acceptable in moderation.** `Employee → Hire Date` can be a good reason for one. A long chain of dimension-to-dimension links is usually a signal to simplify the model.

### Bridge

**Use it when the business requires it.** Many-to-many relationships are real business relationships. Forcing one into a many-to-one shape produces an incorrect model. Bridges add query complexity, so they should still be introduced deliberately.

### A Practical Compromise

Sometimes a bridge is the complete, correct model, while a simplified attribute serves frequent reports. If 90% of reports only need the primary account holder, `dim_account` can include `primary_customer_key`; queries needing every related customer still use `bridge_account_customer`.

```text
Complete model for the full relationship = bridge table
Simplified model for frequent queries = primary customer
```

---

## Final Mental Rule

Ask these questions when choosing a pattern:

```text
Am I separating one dimension's attributes into multiple tables?
→ Snowflake

Does a dimension exceptionally need to query another dimension?
→ Outrigger

Can one entity relate to multiple others such that one foreign key is insufficient?
→ Bridge
```

From a Kimball design perspective:

```text
Snowflake → usually avoid
Outrigger → acceptable exception
Bridge → use when the business genuinely requires it
```
