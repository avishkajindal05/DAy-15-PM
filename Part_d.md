# Ride-Sharing App Database Design (Ola/Uber-like)

## 1. ER Diagram Description

### Entities and Attributes

**1. User**
- user_id (PK)
- name
- phone
- email
- created_at

**2. Driver**
- driver_id (PK)
- name
- phone
- license_number
- rating
- status (active/offline)

**3. Vehicle**
- vehicle_id (PK)
- driver_id (FK → Driver)
- vehicle_type
- plate_number
- capacity

**4. Ride**
- ride_id (PK)
- user_id (FK → User)
- driver_id (FK → Driver)
- vehicle_id (FK → Vehicle)
- pickup_location
- drop_location
- request_time
- start_time
- end_time
- fare
- status

**5. Payment**
- payment_id (PK)
- ride_id (FK → Ride)
- payment_method
- amount
- payment_time
- payment_status

**6. Rating**
- rating_id (PK)
- ride_id (FK → Ride)
- user_rating
- driver_rating
- feedback

---

## Relationships

| Relationship | Description | Cardinality |
|---|---|---|
| User → Ride | A user can request many rides | 1 : N |
| Driver → Ride | A driver can complete many rides | 1 : N |
| Driver → Vehicle | One driver can have one or more vehicles | 1 : N |
| Ride → Payment | Each ride has one payment | 1 : 1 |
| Ride → Rating | Each ride may have a rating | 1 : 1 |

---

## ER Diagram (Conceptual Representation)
User (user_id PK)
|
| 1
|------< requests >------ N
Ride (ride_id PK)
|
| N
Driver (driver_id PK) ---------|
|
Vehicle (vehicle_id PK) -------|

Ride ----- 1 : 1 ----- Payment
Ride ----- 1 : 1 ----- Rating


---

# 2. Normalized Tables (3NF)

### Users Table
```sql
CREATE TABLE Users (
    user_id INT PRIMARY KEY,
    name VARCHAR(100),
    phone VARCHAR(20),
    email VARCHAR(100),
    created_at TIMESTAMP
);

Drivers Table
CREATE TABLE Drivers (
    driver_id INT PRIMARY KEY,
    name VARCHAR(100),
    phone VARCHAR(20),
    license_number VARCHAR(50),
    rating DECIMAL(3,2),
    status VARCHAR(20)
);
Vehicles Table
CREATE TABLE Vehicles (
    vehicle_id INT PRIMARY KEY,
    driver_id INT,
    vehicle_type VARCHAR(50),
    plate_number VARCHAR(20),
    capacity INT,
    FOREIGN KEY (driver_id) REFERENCES Drivers(driver_id)
);
Rides Table
CREATE TABLE Rides (
    ride_id INT PRIMARY KEY,
    user_id INT,
    driver_id INT,
    vehicle_id INT,
    pickup_location VARCHAR(255),
    drop_location VARCHAR(255),
    request_time TIMESTAMP,
    start_time TIMESTAMP,
    end_time TIMESTAMP,
    fare DECIMAL(10,2),
    status VARCHAR(20),
    FOREIGN KEY (user_id) REFERENCES Users(user_id),
    FOREIGN KEY (driver_id) REFERENCES Drivers(driver_id),
    FOREIGN KEY (vehicle_id) REFERENCES Vehicles(vehicle_id)
);
Payments Table
CREATE TABLE Payments (
    payment_id INT PRIMARY KEY,
    ride_id INT UNIQUE,
    payment_method VARCHAR(50),
    amount DECIMAL(10,2),
    payment_time TIMESTAMP,
    payment_status VARCHAR(20),
    FOREIGN KEY (ride_id) REFERENCES Rides(ride_id)
);
Ratings Table
CREATE TABLE Ratings (
    rating_id INT PRIMARY KEY,
    ride_id INT UNIQUE,
    user_rating INT,
    driver_rating INT,
    feedback TEXT,
    FOREIGN KEY (ride_id) REFERENCES Rides(ride_id)
);
3. SQL Queries Using Window Functions
1. Rank Drivers by Total Earnings
SELECT
    driver_id,
    SUM(fare) AS total_earnings,
    RANK() OVER (ORDER BY SUM(fare) DESC) AS driver_rank
FROM Rides
GROUP BY driver_id;
2. Running Total of Daily Ride Revenue
SELECT
    DATE(request_time) AS ride_date,
    SUM(fare) AS daily_revenue,
    SUM(SUM(fare)) OVER (ORDER BY DATE(request_time)) AS running_total
FROM Rides
GROUP BY DATE(request_time);
3. Top 3 Drivers per City (Assuming pickup_location is city)
SELECT *
FROM (
    SELECT
        driver_id,
        pickup_location,
        COUNT(*) AS total_rides,
        ROW_NUMBER() OVER (
            PARTITION BY pickup_location
            ORDER BY COUNT(*) DESC
        ) AS rank_in_city
    FROM Rides
    GROUP BY driver_id, pickup_location
) t
WHERE rank_in_city <= 3;
4. Previous Ride Fare of Each Driver (Using LAG)
SELECT
    driver_id,
    ride_id,
    fare,
    LAG(fare) OVER (
        PARTITION BY driver_id
        ORDER BY start_time
    ) AS previous_ride_fare
FROM Rides;
5. Average Fare Compared with Driver Average
SELECT
    ride_id,
    driver_id,
    fare,
    AVG(fare) OVER (PARTITION BY driver_id) AS driver_avg_fare
FROM Rides;

4. Summary

The schema follows 3NF normalization because:
Each table represents one entity
No partial dependencies
No transitive dependencies
All attributes depend only on the primary key

Main components:

Users
Drivers
Vehicles
Rides
Payments
Rating

Window functions enable advanced analytics like:

driver ranking
revenue tracking
ride comparison
city-level leaderboards



# Evaluation of the Ride-Sharing Schema

## 1. Is the Schema in 3NF?

A relation is in **Third Normal Form (3NF)** if:

1. The table is in **2NF**.
2. **No transitive dependencies** exist.
3. Every non-key attribute depends **only on the primary key**.

### Table Analysis

#### Users
| Column | Dependency |
|---|---|
| user_id (PK) | determines name, phone, email, created_at |

All attributes depend only on `user_id`.

✅ **3NF satisfied**

---

#### Drivers
| Column | Dependency |
|---|---|
| driver_id (PK) | determines name, phone, license_number, rating, status |

No transitive dependency.

✅ **3NF satisfied**

---

#### Vehicles
| Column | Dependency |
|---|---|
| vehicle_id (PK) | determines driver_id, vehicle_type, plate_number, capacity |

Each attribute depends on `vehicle_id`.

✅ **3NF satisfied**

---

#### Rides
| Column | Dependency |
|---|---|
| ride_id (PK) | determines user_id, driver_id, vehicle_id, pickup_location, drop_location, times, fare, status |

No attribute depends on another non-key attribute.

✅ **3NF satisfied**

---

#### Payments
| Column | Dependency |
|---|---|
| payment_id (PK) | determines ride_id, payment_method, amount, payment_time, payment_status |

`ride_id` is also **UNIQUE**, ensuring one payment per ride.

✅ **3NF satisfied**

---

#### Ratings
| Column | Dependency |
|---|---|
| rating_id (PK) | determines ride_id, user_rating, driver_rating, feedback |

`ride_id` unique → one rating per ride.

✅ **3NF satisfied**

---

## Conclusion

The schema **is in 3NF** because:

- Each table has a **primary key**
- All non-key attributes depend **fully on the key**
- No **transitive dependencies** exist

---

# 2. Possible Missing Relationships

Although the schema is normalized, a few **practical relationships are missing**.

### 1. Location Table (Recommended)

Currently locations are stored as text:

---