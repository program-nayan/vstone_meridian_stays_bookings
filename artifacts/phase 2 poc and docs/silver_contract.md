**Silver Contract**


* **Naming**: All tables and columns use lowercase snake_case.


* **Data Types**: Dates are cast to `TIMESTAMP` or `DATE`; financial amounts to `DECIMAL(10,2)`.


* **Validity Rules (Expectations)**: `booking_id` and `payment_id` must not be null; `payment_amount` must be greater than zero; `check_out_date` must be after `check_in_date`.


* **Uniqueness**: The Silver layer guarantees exactly one row per business key for all entities.


* **History**: The `properties` dimension retains historical attribute changes using SCD Type 2.


