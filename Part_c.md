# Q1 — Update Anomalies in a Denormalised Database

A **denormalised database** stores redundant data in a single table. While this can simplify queries, it often leads to **update anomalies**, which cause data inconsistency.

## Types of Update Anomalies

### 1. Update Anomaly
Occurs when the same information appears in multiple rows and must be updated everywhere.

**Example (E-commerce Orders Table)**

| order_id | customer_name | product_name | product_price |
|----------|---------------|--------------|---------------|
| 101 | Ravi | Laptop | 60000 |
| 102 | Meera | Laptop | 60000 |
| 103 | Amit | Laptop | 60000 |

If the laptop price changes to **65000**, every row must be updated.  
If one row is missed, the database becomes **inconsistent**.

---

### 2. Insert Anomaly
Occurs when certain data cannot be inserted without other unrelated data.

**Example**

If a **new product "Tablet"** is added but **no order has been placed yet**, we cannot insert it because the table requires an `order_id`.

---

### 3. Delete Anomaly
Occurs when deleting a row removes additional useful information.

**Example**

| order_id | customer_name | product_name |
|----------|---------------|--------------|
| 104 | Neha | Headphones |

If order **104** is deleted, information about the **Headphones product** might be lost entirely if this was the only row containing it.

---

## Solution
Normalization separates data into related tables such as:

- **Customers**
- **Products**
- **Orders**
- **OrderItems**

This removes redundancy and prevents anomalies.

---

# Q2 — Schema Design with Price History (3NF)

We design a schema that:
1. Maintains the **current product price**
2. Stores **historical prices with timestamps**
3. Satisfies **Third Normal Form (3NF)**

## Tables

### 1. Products
Stores product details and the **current price**.

| Column | Description |
|------|-------------|
| product_id (PK) | Unique product ID |
| product_name | Product name |
| current_price | Current price |

---

### 2. PriceHistory
Stores every price change with timestamps.

| Column | Description |
|------|-------------|
| history_id (PK) | Unique record |
| product_id (FK) | References Products |
| price | Historical price |
| start_time | When price became active |
| end_time | When price ended |

---

### 3. Orders

| Column | Description |
|------|-------------|
| order_id (PK) | Order identifier |
| customer_id | Customer placing order |
| order_date | Date of purchase |

---

### 4. OrderItems

| Column | Description |
|------|-------------|
| order_item_id (PK) | Item record |
| order_id (FK) | References Orders |
| product_id (FK) | References Products |
| quantity | Number of items |
| purchase_price | Price at time of purchase |

---

## Why This Schema is in 3NF

- **1NF:** All attributes are atomic.
- **2NF:** No partial dependency on composite keys.
- **3NF:** No transitive dependencies (product price history is separated).

This design also preserves **historical pricing information** for analytics.

---

# Q3 — ACID Violation Scenario (Double Booking)

## Scenario
Two users try to **book the last hotel room simultaneously**.

### ACID Property at Risk
**Isolation**

Isolation ensures that **concurrent transactions do not interfere with each other**.

If isolation is weak, both users might see the room as available and both bookings may succeed.

---

## How Databases Prevent Double Booking

### 1. Transaction Isolation Levels
Databases enforce isolation using levels such as:

- Read Committed
- Repeatable Read
- Serializable (strongest)

**Serializable** ensures transactions behave as if executed sequentially.

---

### 2. Row-Level Locking

Example workflow:

1. Transaction A reads room availability.
2. Database places a **lock on the row**.
3. Transaction B must wait.
4. Transaction A commits booking.
5. Transaction B re-checks availability → finds room unavailable.

---

### 3. Constraint Enforcement

A **unique constraint or check constraint** can also prevent duplicate bookings.

Example:
UNIQUE(room_id, booking_date)


This ensures only **one booking per room per date**.

---

## Final Result

Using **transaction isolation + locking + constraints**, the database ensures:

- Only **one transaction succeeds**
- The other **fails or retries**

Thus maintaining **ACID consistency**.
