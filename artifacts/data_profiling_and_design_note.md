# Phase 0: Data Profiling and Design Note

**Project:** Project Meridian Capstone  
**Business Unit:** Booking revenue and occupancy (Package 12: Meridian Stays Bookings)  
**Source Dataset:** `samples.wanderbricks`

## 1. Overview and Verification
Before designing the architecture, I verified the existence of the required source tables in the `samples.wanderbricks` catalog. Specifically, the `destinations` table presence was confirmed via SQL `SHOW TABLES`, fulfilling the project prerequisite.

## 2. Table Profiles

### Facts

**1. `bookings`**
* **Type:** Fact Table
* **Row Count:** 72,247
* **Grain:** One row per booking (`booking_id`). A uniqueness check confirmed no duplicate `booking_id`s currently exist in the raw data.
* **Key Columns:** `booking_id` (PK), `user_id` (FK), `property_id` (FK)
* **Date Range:** `check_in` dates start from **2022-12-29**, and `check_out` dates extend to **2025-07-31**. This range is critical for the Calendar Explosion challenge.

**2. `payments`**
* **Type:** Fact Table (Financial)
* **Row Count:** 49,638
* **Grain:** One row per payment transaction (`payment_id`).
* **Key Columns:** `payment_id` (PK), `booking_id` (FK)
* **Security Note:** Contains sensitive financial data (`amount`).

### Dimensions

**3. `properties`**
* **Type:** Dimension Table
* **Row Count:** 18,163
* **Grain:** One row per property (`property_id`).
* **Key Columns:** `property_id` (PK), `host_id` (FK), `destination_id` (FK)
* **Notes:** Contains `base_price` which will be tracked using SCD Type 2.

**4. `destinations`**
* **Type:** Dimension Table
* **Row Count:** 42
* **Grain:** One row per destination (`destination_id`).
* **Key Columns:** `destination_id` (PK)
* **Security Note:** Contains the `country` column, which serves as the top level of the destination hierarchy for row-level security.

**5. `users` (Guests)**
* **Type:** Dimension Table
* **Row Count:** 124,509
* **Grain:** One row per user/guest (`user_id`).
* **Key Columns:** `user_id` (PK)
* **Security Note:** Contains Personally Identifiable Information (PII).

**6. `hosts`**
* **Type:** Dimension Table
* **Row Count:** 19,384
* **Grain:** One row per host (`host_id`).
* **Key Columns:** `host_id` (PK)
* **Security Note:** Contains Personally Identifiable Information (PII).

---

## 3. Design Note & Architecture Planning

Based on the profiling above, here is the architectural plan for the pipeline and governance:

**Entity Classification**
* **Facts:** Bookings, Payments. (Requires payment reconciliation logic to handle 0, partial, multiple, or excess payments per booking).
* **Dimensions:** Properties, Destinations, Users, Hosts.

**Data Governance Plan (Phase 4)**
* **Row-Level Security (Row Filter):** Applied to the Destination Country hierarchy (`destinations.country`). Users will only see rows for countries they are entitled to.
* **Column-Level Security (Masking):** 
    1. Financial: `payments.amount`
    2. Guest Identity: `users.email`, `users.name`
    3. Host Identity: `hosts.email`, `hosts.name`
    *(Note: Guests and Hosts represent two separate identity sets with different entitlements, satisfying the Advanced Challenge).*

**Transformation Challenges**
* **Calendar Explosion:** The `bookings` fact will need to be expanded in the Silver/Gold layer to generate one row per night between `check_in` and `check_out` to accurately report occupancy by date.
* **SCD Type 2:** Changes to `properties.base_price` will be captured historically across batches using Lakeflow declarative pipelines.