# Complete Database Schema - Airline Management System

## 📋 **Table Definitions with Attributes**

### **1. User Table** (Strong Entity)
```sql
CREATE TABLE users (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    phone_number VARCHAR(15) NOT NULL,
    role VARCHAR(20) DEFAULT 'Customer' CHECK (role IN ('Customer', 'Admin', 'Staff')),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

**Attributes:**
- **user_id** (PK): Unique identifier for each user
- **full_name**: Complete name of the user
- **email**: Email address for login and communication (UNIQUE)
- **password_hash**: Encrypted password for security
- **phone_number**: Contact phone number
- **role**: User type (Customer, Admin, Staff)
- **created_at**: Account creation timestamp
- **updated_at**: Last profile update timestamp

---

### **2. Airport Table** (Strong Entity)
```sql
CREATE TABLE airports (
    airport_id INT PRIMARY KEY AUTO_INCREMENT,
    airport_code VARCHAR(10) UNIQUE NOT NULL,
    airport_name VARCHAR(255) NOT NULL,
    city VARCHAR(100) NOT NULL,
    country VARCHAR(100) NOT NULL,
    timezone VARCHAR(50) NOT NULL
);
```

**Attributes:**
- **airport_id** (PK): Unique identifier for each airport
- **airport_code**: IATA/ICAO code (e.g., LAX, JFK) - UNIQUE
- **airport_name**: Full name of the airport
- **city**: City where airport is located
- **country**: Country where airport is located
- **timezone**: Airport timezone (e.g., UTC-5, UTC+1)

---

### **3. Airplane Table** (Strong Entity)
```sql
CREATE TABLE airplanes (
    airplane_id INT PRIMARY KEY AUTO_INCREMENT,
    model VARCHAR(50) NOT NULL,
    manufacturer VARCHAR(50) NOT NULL,
    capacity INT NOT NULL CHECK (capacity > 0),
    status VARCHAR(20) DEFAULT 'Active' CHECK (status IN ('Active', 'Maintenance', 'Retired'))
);
```

**Attributes:**
- **airplane_id** (PK): Unique identifier for each aircraft
- **model**: Aircraft model (e.g., Boeing 737, Airbus A320)
- **manufacturer**: Aircraft manufacturer (Boeing, Airbus, etc.)
- **capacity**: Total number of seats in aircraft
- **status**: Current aircraft status (Active, Maintenance, Retired)

---

### **4. Service Table** (Strong Entity)
```sql
CREATE TABLE services (
    service_id INT PRIMARY KEY AUTO_INCREMENT,
    service_name VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL CHECK (price >= 0),
    service_type VARCHAR(50) NOT NULL CHECK (service_type IN ('Food', 'Entertainment', 'Comfort', 'WiFi', 'Amenity'))
);
```

**Attributes:**
- **service_id** (PK): Unique identifier for each service
- **service_name**: Name of the service (e.g., In-flight Meal)
- **description**: Detailed description of the service
- **price**: Cost of the service (>= 0)
- **service_type**: Category (Food, Entertainment, Comfort, WiFi, Amenity)

---

### **5. Flight Table** (Strong Entity)
```sql
CREATE TABLE flights (
    flight_id INT PRIMARY KEY AUTO_INCREMENT,
    flight_number VARCHAR(10) UNIQUE NOT NULL,
    departure_time TIMESTAMP NOT NULL,
    arrival_time TIMESTAMP NOT NULL,
    price DECIMAL(10,2) NOT NULL CHECK (price > 0),
    status VARCHAR(20) DEFAULT 'Scheduled' CHECK (status IN ('Scheduled', 'Delayed', 'Cancelled', 'Completed')),
    airplane_id INT NOT NULL,
    source_airport_id INT NOT NULL,
    destination_airport_id INT NOT NULL,
    
    FOREIGN KEY (airplane_id) REFERENCES airplanes(airplane_id),
    FOREIGN KEY (source_airport_id) REFERENCES airports(airport_id),
    FOREIGN KEY (destination_airport_id) REFERENCES airports(airport_id),
    CHECK (departure_time < arrival_time),
    CHECK (source_airport_id != destination_airport_id)
);
```

**Attributes:**
- **flight_id** (PK): Unique identifier for each flight
- **flight_number**: Flight code (e.g., AA123, UA456) - UNIQUE
- **departure_time**: Scheduled departure date and time
- **arrival_time**: Scheduled arrival date and time
- **price**: Base ticket price (> 0)
- **status**: Flight status (Scheduled, Delayed, Cancelled, Completed)
- **airplane_id** (FK): Reference to assigned aircraft
- **source_airport_id** (FK): Departure airport reference
- **destination_airport_id** (FK): Arrival airport reference

---

### **6. Booking Table** (Strong Entity)
```sql
CREATE TABLE bookings (
    booking_id INT PRIMARY KEY AUTO_INCREMENT,
    booking_reference VARCHAR(10) UNIQUE NOT NULL,
    booking_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    total_amount DECIMAL(10,2) NOT NULL CHECK (total_amount >= 0),
    status VARCHAR(20) DEFAULT 'Confirmed' CHECK (status IN ('Confirmed', 'Cancelled', 'Pending')),
    user_id INT NOT NULL,
    flight_id INT NOT NULL,
    
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (flight_id) REFERENCES flights(flight_id)
);
```

**Attributes:**
- **booking_id** (PK): Unique identifier for each booking
- **booking_reference**: Human-readable booking code (e.g., ABC123) - UNIQUE
- **booking_date**: When the booking was made
- **total_amount**: Total cost including all services (>= 0)
- **status**: Booking status (Confirmed, Cancelled, Pending)
- **user_id** (FK): Reference to customer who made booking
- **flight_id** (FK): Reference to booked flight

---

### **7. Payment Table** (Strong Entity)
```sql
CREATE TABLE payments (
    payment_id INT PRIMARY KEY AUTO_INCREMENT,
    amount DECIMAL(10,2) NOT NULL CHECK (amount > 0),
    payment_method VARCHAR(50) NOT NULL CHECK (payment_method IN ('Credit Card', 'Debit Card', 'PayPal', 'Bank Transfer')),
    transaction_id VARCHAR(100) UNIQUE,
    payment_status VARCHAR(20) DEFAULT 'Pending' CHECK (payment_status IN ('Pending', 'Completed', 'Failed', 'Refunded')),
    payment_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    booking_id INT UNIQUE NOT NULL,
    
    FOREIGN KEY (booking_id) REFERENCES bookings(booking_id)
);
```

**Attributes:**
- **payment_id** (PK): Unique identifier for each payment
- **amount**: Payment amount (> 0)
- **payment_method**: How payment was made
- **transaction_id**: External payment gateway transaction ID - UNIQUE
- **payment_status**: Current payment status
- **payment_date**: When payment was processed
- **booking_id** (FK): Reference to associated booking - UNIQUE (1:1 relationship)

---

### **8. Review Table** (Strong Entity)
```sql
CREATE TABLE reviews (
    review_id INT PRIMARY KEY AUTO_INCREMENT,
    rating INT NOT NULL CHECK (rating >= 1 AND rating <= 5),
    comment TEXT,
    review_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    user_id INT NOT NULL,
    flight_id INT NOT NULL,
    
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (flight_id) REFERENCES flights(flight_id),
    UNIQUE(user_id, flight_id) -- One review per user per flight
);
```

**Attributes:**
- **review_id** (PK): Unique identifier for each review
- **rating**: Star rating from 1 to 5
- **comment**: Written review text (optional)
- **review_date**: When review was submitted
- **user_id** (FK): Reference to user who wrote review
- **flight_id** (FK): Reference to reviewed flight
- **UNIQUE constraint**: One review per user per flight

---

### **9. Passenger Table** (Weak Entity)
```sql
CREATE TABLE passengers (
    booking_id INT NOT NULL,
    seat_number VARCHAR(5) NOT NULL,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    date_of_birth DATE NOT NULL,
    passport_number VARCHAR(20) NOT NULL,
    nationality VARCHAR(50) NOT NULL,
    
    PRIMARY KEY (booking_id, seat_number),
    FOREIGN KEY (booking_id) REFERENCES bookings(booking_id) ON DELETE CASCADE
);
```

**Attributes:**
- **booking_id** (PK1, FK): Reference to parent booking
- **seat_number** (PK2): Assigned seat (e.g., 12A, 15C)
- **first_name**: Passenger's first name
- **last_name**: Passenger's last name
- **date_of_birth**: Passenger's birth date
- **passport_number**: Passport or ID number
- **nationality**: Passenger's nationality
- **Composite PK**: (booking_id, seat_number)

---

### **10. Flight_Services Table** (Junction/Bridge Table for N:M)
```sql
CREATE TABLE flight_services (
    flight_id INT NOT NULL,
    service_id INT NOT NULL,
    availability_status VARCHAR(20) DEFAULT 'Available' CHECK (availability_status IN ('Available', 'Unavailable', 'Limited')),
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    PRIMARY KEY (flight_id, service_id),
    FOREIGN KEY (flight_id) REFERENCES flights(flight_id) ON DELETE CASCADE,
    FOREIGN KEY (service_id) REFERENCES services(service_id) ON DELETE CASCADE
);
```

**Attributes:**
- **flight_id** (PK1, FK): Reference to flight
- **service_id** (PK2, FK): Reference to service
- **availability_status**: Service availability on this flight
- **last_updated**: When availability was last updated
- **Composite PK**: (flight_id, service_id)

---

## 🔗 **Entity-Relationship (ER) Diagram**

```
                    ┌─────────────┐
                    │    USER     │
                    │             │
                    │ user_id (PK)│
                    │ full_name   │
                    │ email       │
                    │ password    │
                    │ phone       │
                    │ role        │
                    └──────┬──────┘
                           │
                           │ 1:N (makes)
                           ▼
                    ┌─────────────┐
                    │   BOOKING   │
                    │             │
                    │booking_id PK│
                    │booking_ref  │
                    │total_amount │
                    │status       │
                    │user_id (FK) │
                    │flight_id FK │
                    └──────┬──────┘
                           │
                    ┌──────┴──────┐
                    │             │
                    ▼ 1:N         ▼ 1:1
            ┌─────────────┐ ┌─────────────┐
            │  PASSENGER  │ │   PAYMENT   │
            │   (WEAK)    │ │             │
            │booking_id PK│ │payment_id PK│
            │seat_num PK  │ │amount       │
            │first_name   │ │method       │
            │last_name    │ │status       │
            │passport     │ │booking_id FK│
            └─────────────┘ └─────────────┘

┌─────────────┐                    ┌─────────────┐
│   AIRPORT   │                    │  AIRPLANE   │
│             │                    │             │
│airport_id PK│                    │airplane_id  │
│airport_code │                    │model        │
│airport_name │                    │manufacturer │
│city         │                    │capacity     │
│country      │                    │status       │
└──────┬──────┘                    └──────┬──────┘
       │                                  │
       │ N:1 (source/dest)                │ N:1 (uses)
       ▼                                  ▼
┌─────────────────────────────────────────────────────┐
│                   FLIGHT                            │
│                                                     │
│ flight_id (PK)                                     │
│ flight_number                                      │
│ departure_time                                     │
│ arrival_time                                       │
│ price                                              │
│ status                                             │
│ airplane_id (FK)                                   │
│ source_airport_id (FK)                             │
│ destination_airport_id (FK)                        │
└──────┬───────────────────┬───────────────────┬─────┘
       │                   │                   │
       │ 1:N (has reviews) │ N:M (services)    │ 1:N (has bookings)
       ▼                   ▼                   ▼
  ┌─────────────┐    ┌─────────────────┐   ┌─────────────┐
  │   REVIEW    │    │ FLIGHT_SERVICES │   │  BOOKING    │
  │             │    │   (JUNCTION)    │   │             │
  │ review_id PK│    │ flight_id (PK1) │   │booking_id PK│
  │ rating      │    │ service_id (PK2)│   │...          │
  │ comment     │    │ availability    │   │flight_id FK │
  │ user_id (FK)│    │ last_updated    │   └─────────────┘
  │flight_id FK │    └─────────┬───────┘
  └─────────────┘              │
                               │ N:M
                               ▼
                        ┌─────────────┐
                        │   SERVICE   │
                        │             │
                        │ service_id  │
                        │ service_name│
                        │ description │
                        │ price       │
                        │ service_type│
                        └─────────────┘
```

## 📊 **Relationship Summary**

### **1:1 (One-to-One) Relationships**
- **Booking ↔ Payment**: Each booking has exactly one payment
  - `payments.booking_id` has UNIQUE constraint

### **1:N (One-to-Many) Relationships**
- **User → Booking**: One user can have multiple bookings
- **User → Review**: One user can write multiple reviews  
- **Booking → Passenger**: One booking can have multiple passengers
- **Flight → Booking**: One flight can have multiple bookings
- **Flight → Review**: One flight can have multiple reviews
- **Airplane → Flight**: One airplane can operate multiple flights
- **Airport → Flight**: One airport can be source/destination for multiple flights

### **N:1 (Many-to-One) Relationships**
- **Booking → User**: Many bookings belong to one user
- **Booking → Flight**: Many bookings can be for one flight
- **Review → User**: Many reviews can be written by one user
- **Review → Flight**: Many reviews can be for one flight
- **Flight → Airplane**: Many flights can use one airplane
- **Flight → Airport**: Many flights can depart from/arrive at one airport

### **N:M (Many-to-Many) Relationships**
- **Flight ↔ Service**: Many flights can offer many services
  - Resolved through `flight_services` junction table
  - Examples: WiFi, meals, entertainment, extra legroom

### **Weak Entity Relationship**
- **Passenger** depends on **Booking** for existence
  - Composite primary key: `(booking_id, seat_number)`
  - Cannot exist without parent booking

## 🎯 **Key Design Features**

### **Integrity Constraints**
- **Primary Keys**: All tables have unique identifiers
- **Foreign Keys**: All relationships properly referenced
- **Check Constraints**: Data validation (ratings 1-5, positive prices)
- **Unique Constraints**: Prevent duplicates (emails, booking references)
- **NOT NULL**: Critical fields cannot be empty

### **Business Rules Enforced**
- Users must have unique emails
- Flights must have valid departure/arrival times
- Payments must have positive amounts
- Reviews must have ratings between 1-5
- Aircraft must have positive capacity
- Source and destination airports must be different
- One review per user per flight

### **Normalization Level**
- **1NF**: All attributes are atomic
- **2NF**: No partial dependencies
- **3NF**: No transitive dependencies
- **BCNF**: All functional dependencies are proper

This comprehensive schema demonstrates **all DBMS concepts** required for academic projects while being a practical, real-world airline management system! ✈️

---

## ⚠️ BCNF and the total_amount Attribute in Bookings

### Why total_amount Can Break BCNF
- If total_amount is derived from other attributes (flight price, services, passenger count, etc.), then there is a functional dependency:
  - {flight_id, selected services, passenger count, ...} → total_amount
- But {flight_id, selected services, passenger count, ...} is not a superkey for bookings.
- This means total_amount is transitively dependent on the primary key, not directly, and is thus a derived/calculated value.

### BCNF Solution
- **Best Practice:** Remove total_amount from the bookings table for full BCNF compliance. Always calculate it on the fly using related tables (flight price, services, passengers, etc.).
- **If you must store it** (for audit/history): Document that it is a denormalized, cached value and accept that this is a controlled BCNF violation for business reasons.

### Example Query to Calculate total_amount
```sql
SELECT
  b.booking_id,
  f.price AS flight_price,
  SUM(fs.price) AS services_price,
  COUNT(p.seat_number) AS passenger_count,
  (f.price + COALESCE(SUM(fs.price),0)) * COUNT(p.seat_number) AS total_amount
FROM bookings b
JOIN flights f ON b.flight_id = f.flight_id
LEFT JOIN flight_services fs ON fs.flight_id = b.flight_id
LEFT JOIN passengers p ON p.booking_id = b.booking_id
WHERE b.booking_id = :booking_id
GROUP BY b.booking_id, f.price;
```

### Summary
- Storing total_amount in bookings can break BCNF if it is derived from other attributes.
- Remove it for full BCNF compliance, or document/justify if you keep it for business reasons.
