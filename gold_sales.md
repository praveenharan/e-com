### Creating a date table

```sql
-- 1. Creating the Date table structure 
CREATE TABLE dim_date (
    date_key INT NOT NULL, -- Keep it NOT NULL for best practice
    full_date DATE,
    day_of_week_name VARCHAR(10),
    day_of_month INT,
    month_name VARCHAR(10),
    quarter INT,
    year INT,
    is_weekend BIT
);

-- 2. Logic to populate 
DECLARE @StartDate DATE = '2024-01-01', @EndDate DATE = '2026-12-31';

WHILE @StartDate <= @EndDate
BEGIN
    INSERT INTO dim_date (date_key, full_date, day_of_week_name, day_of_month, month_name, quarter, year, is_weekend)
    VALUES (
        CAST(FORMAT(@StartDate, 'yyyyMMdd') AS INT), 
        @StartDate,
        DATENAME(WEEKDAY, @StartDate),
        DAY(@StartDate),
        DATENAME(MONTH, @StartDate),
        DATEPART(QUARTER, @StartDate),
        YEAR(@StartDate),
        CASE WHEN DATEPART(WEEKDAY, @StartDate) IN (1, 7) THEN 1 ELSE 0 END
    );
    SET @StartDate = DATEADD(DAY, 1, @StartDate);
END;

SELECT *
FROM dim_date
```
