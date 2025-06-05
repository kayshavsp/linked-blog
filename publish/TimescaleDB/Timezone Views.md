
### Timezone views using 30min aggregate to get data 
```sql

-- Asia/Kuala_Lumpur Hourly
CREATE VIEW iot_data.tz_hour_asia_kuala_lumpur AS
SELECT 
    date_trunc('hour', start_time AT TIME ZONE 'Asia/Kuala_Lumpur') AS local_start_time,
    date_trunc('hour', start_time AT TIME ZONE 'Asia/Kuala_Lumpur') + INTERVAL '1 hour' AS local_end_time,
    date_trunc('hour', start_time) AS utc_start_time,
    date_trunc('hour', start_time) + INTERVAL '1 hour' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Kuala_Lumpur' AS timezone,
    AVG(avg_30min_value) AS avg_hour_value,
    MIN(min_30min_value) AS min_hour_value,
    MAX(max_30min_value) AS max_hour_value,
    SUM(sum_30min_value) AS sum_hour_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    date_trunc('hour', start_time AT TIME ZONE 'Asia/Kuala_Lumpur'),
    date_trunc('hour', start_time),
    device_id,
    parameter_id;

-- Asia/Kolkata Hourly
CREATE VIEW iot_data.tz_hour_asia_kolkata AS
SELECT 
    date_trunc('hour', start_time AT TIME ZONE 'Asia/Kolkata') AS local_start_time,
    date_trunc('hour', start_time AT TIME ZONE 'Asia/Kolkata') + INTERVAL '1 hour' AS local_end_time,
    date_trunc('hour', start_time) AS utc_start_time,
    date_trunc('hour', start_time) + INTERVAL '1 hour' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Kolkata' AS timezone,
    AVG(avg_30min_value) AS avg_hour_value,
    MIN(min_30min_value) AS min_hour_value,
    MAX(max_30min_value) AS max_hour_value,
    SUM(sum_30min_value) AS sum_hour_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    date_trunc('hour', start_time AT TIME ZONE 'Asia/Kolkata'),
    date_trunc('hour', start_time),
    device_id,
    parameter_id;

-- Asia/Qatar Hourly
CREATE VIEW iot_data.tz_hour_asia_qatar AS
SELECT 
    date_trunc('hour', start_time AT TIME ZONE 'Asia/Qatar') AS local_start_time,
    date_trunc('hour', start_time AT TIME ZONE 'Asia/Qatar') + INTERVAL '1 hour' AS local_end_time,
    date_trunc('hour', start_time) AS utc_start_time,
    date_trunc('hour', start_time) + INTERVAL '1 hour' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Qatar' AS timezone,
    AVG(avg_30min_value) AS avg_hour_value,
    MIN(min_30min_value) AS min_hour_value,
    MAX(max_30min_value) AS max_hour_value,
    SUM(sum_30min_value) AS sum_hour_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    date_trunc('hour', start_time AT TIME ZONE 'Asia/Qatar'),
    date_trunc('hour', start_time),
    device_id,
    parameter_id;

-- Daily Views

-- Asia/Kuala_Lumpur Daily
CREATE VIEW iot_data.tz_day_asia_kuala_lumpur AS
SELECT 
    date_trunc('day', start_time AT TIME ZONE 'Asia/Kuala_Lumpur') AS local_start_time,
    date_trunc('day', start_time AT TIME ZONE 'Asia/Kuala_Lumpur') + INTERVAL '1 day' AS local_end_time,
    date_trunc('day', start_time) AS utc_start_time,
    date_trunc('day', start_time) + INTERVAL '1 day' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Kuala_Lumpur' AS timezone,
    AVG(avg_30min_value) AS avg_day_value,
    MIN(min_30min_value) AS min_day_value,
    MAX(max_30min_value) AS max_day_value,
    SUM(sum_30min_value) AS sum_day_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    date_trunc('day', start_time AT TIME ZONE 'Asia/Kuala_Lumpur'),
    date_trunc('day', start_time),
    device_id,
    parameter_id;

-- Asia/Kolkata Daily
CREATE VIEW iot_data.tz_day_asia_kolkata AS
SELECT 
    date_trunc('day', start_time AT TIME ZONE 'Asia/Kolkata') AS local_start_time,
    date_trunc('day', start_time AT TIME ZONE 'Asia/Kolkata') + INTERVAL '1 day' AS local_end_time,
    date_trunc('day', start_time) AS utc_start_time,
    date_trunc('day', start_time) + INTERVAL '1 day' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Kolkata' AS timezone,
    AVG(avg_30min_value) AS avg_day_value,
    MIN(min_30min_value) AS min_day_value,
    MAX(max_30min_value) AS max_day_value,
    SUM(sum_30min_value) AS sum_day_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    date_trunc('day', start_time AT TIME ZONE 'Asia/Kolkata'),
    date_trunc('day', start_time),
    device_id,
    parameter_id;

-- Asia/Qatar Daily
CREATE VIEW iot_data.tz_day_asia_qatar AS
SELECT 
    date_trunc('day', start_time AT TIME ZONE 'Asia/Qatar') AS local_start_time,
    date_trunc('day', start_time AT TIME ZONE 'Asia/Qatar') + INTERVAL '1 day' AS local_end_time,
    date_trunc('day', start_time) AS utc_start_time,
    date_trunc('day', start_time) + INTERVAL '1 day' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Qatar' AS timezone,
    AVG(avg_30min_value) AS avg_day_value,
    MIN(min_30min_value) AS min_day_value,
    MAX(max_30min_value) AS max_day_value,
    SUM(sum_30min_value) AS sum_day_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    date_trunc('day', start_time AT TIME ZONE 'Asia/Qatar'),
    date_trunc('day', start_time),
    device_id,
    parameter_id;

-- Monthly Views

-- Asia/Kuala_Lumpur Monthly
CREATE VIEW iot_data.tz_month_asia_kuala_lumpur AS
SELECT 
    date_trunc('month', start_time AT TIME ZONE 'Asia/Kuala_Lumpur') AS local_start_time,
    date_trunc('month', start_time AT TIME ZONE 'Asia/Kuala_Lumpur') + INTERVAL '1 month' AS local_end_time,
    date_trunc('month', start_time) AS utc_start_time,
    date_trunc('month', start_time) + INTERVAL '1 month' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Kuala_Lumpur' AS timezone,
    AVG(avg_30min_value) AS avg_month_value,
    MIN(min_30min_value) AS min_month_value,
    MAX(max_30min_value) AS max_month_value,
    SUM(sum_30min_value) AS sum_month_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    date_trunc('month', start_time AT TIME ZONE 'Asia/Kuala_Lumpur'),
    date_trunc('month', start_time),
    device_id,
    parameter_id;

-- Asia/Kolkata Monthly
CREATE VIEW iot_data.tz_month_asia_kolkata AS
SELECT 
    date_trunc('month', start_time AT TIME ZONE 'Asia/Kolkata') AS local_start_time,
    date_trunc('month', start_time AT TIME ZONE 'Asia/Kolkata') + INTERVAL '1 month' AS local_end_time,
    date_trunc('month', start_time) AS utc_start_time,
    date_trunc('month', start_time) + INTERVAL '1 month' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Kolkata' AS timezone,
    AVG(avg_30min_value) AS avg_month_value,
    MIN(min_30min_value) AS min_month_value,
    MAX(max_30min_value) AS max_month_value,
    SUM(sum_30min_value) AS sum_month_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    date_trunc('month', start_time AT TIME ZONE 'Asia/Kolkata'),
    date_trunc('month', start_time),
    device_id,
    parameter_id;

-- Asia/Qatar Monthly
CREATE VIEW iot_data.tz_month_asia_qatar AS
SELECT 
    date_trunc('month', start_time AT TIME ZONE 'Asia/Qatar') AS local_start_time,
    date_trunc('month', start_time AT TIME ZONE 'Asia/Qatar') + INTERVAL '1 month' AS local_end_time,
    date_trunc('month', start_time) AS utc_start_time,
    date_trunc('month', start_time) + INTERVAL '1 month' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Qatar' AS timezone,
    AVG(avg_30min_value) AS avg_month_value,
    MIN(min_30min_value) AS min_month_value,
    MAX(max_30min_value) AS max_month_value,
    SUM(sum_30min_value) AS sum_month_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    date_trunc('month', start_time AT TIME ZONE 'Asia/Qatar'),
    date_trunc('month', start_time),
    device_id,
    parameter_id;
```


### Using Materialized Views
Replace the regular views with materialized views that pre-compute the results:

```sql
-- Drop existing views first (if they exist)
DROP VIEW IF EXISTS iot_data.tz_hour_asia_kuala_lumpur CASCADE;
DROP VIEW IF EXISTS iot_data.tz_hour_asia_kolkata CASCADE;
DROP VIEW IF EXISTS iot_data.tz_hour_asia_qatar CASCADE;
DROP VIEW IF EXISTS iot_data.tz_day_asia_kuala_lumpur CASCADE;
DROP VIEW IF EXISTS iot_data.tz_day_asia_kolkata CASCADE;
DROP VIEW IF EXISTS iot_data.tz_day_asia_qatar CASCADE;
DROP VIEW IF EXISTS iot_data.tz_month_asia_kuala_lumpur CASCADE;
DROP VIEW IF EXISTS iot_data.tz_month_asia_kolkata CASCADE;
DROP VIEW IF EXISTS iot_data.tz_month_asia_qatar CASCADE;

-- Create Materialized Views for Hourly Aggregates
CREATE MATERIALIZED VIEW iot_data.tz_hour_asia_kuala_lumpur AS
SELECT 
    date_trunc('hour', start_time AT TIME ZONE 'Asia/Kuala_Lumpur') AS local_start_time,
    date_trunc('hour', start_time AT TIME ZONE 'Asia/Kuala_Lumpur') + INTERVAL '1 hour' AS local_end_time,
    date_trunc('hour', start_time) AS utc_start_time,
    date_trunc('hour', start_time) + INTERVAL '1 hour' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Kuala_Lumpur' AS timezone,
    AVG(avg_30min_value) AS avg_hour_value,
    MIN(min_30min_value) AS min_hour_value,
    MAX(max_30min_value) AS max_hour_value,
    SUM(sum_30min_value) AS sum_hour_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    date_trunc('hour', start_time AT TIME ZONE 'Asia/Kuala_Lumpur'),
    date_trunc('hour', start_time),
    device_id,
    parameter_id;

CREATE MATERIALIZED VIEW iot_data.tz_hour_asia_kolkata AS
SELECT 
    date_trunc('hour', start_time AT TIME ZONE 'Asia/Kolkata') AS local_start_time,
    date_trunc('hour', start_time AT TIME ZONE 'Asia/Kolkata') + INTERVAL '1 hour' AS local_end_time,
    date_trunc('hour', start_time) AS utc_start_time,
    date_trunc('hour', start_time) + INTERVAL '1 hour' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Kolkata' AS timezone,
    AVG(avg_30min_value) AS avg_hour_value,
    MIN(min_30min_value) AS min_hour_value,
    MAX(max_30min_value) AS max_hour_value,
    SUM(sum_30min_value) AS sum_hour_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    date_trunc('hour', start_time AT TIME ZONE 'Asia/Kolkata'),
    date_trunc('hour', start_time),
    device_id,
    parameter_id;

CREATE MATERIALIZED VIEW iot_data.tz_hour_asia_qatar AS
SELECT 
    date_trunc('hour', start_time AT TIME ZONE 'Asia/Qatar') AS local_start_time,
    date_trunc('hour', start_time AT TIME ZONE 'Asia/Qatar') + INTERVAL '1 hour' AS local_end_time,
    date_trunc('hour', start_time) AS utc_start_time,
    date_trunc('hour', start_time) + INTERVAL '1 hour' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Qatar' AS timezone,
    AVG(avg_30min_value) AS avg_hour_value,
    MIN(min_30min_value) AS min_hour_value,
    MAX(max_30min_value) AS max_hour_value,
    SUM(sum_30min_value) AS sum_hour_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    date_trunc('hour', start_time AT TIME ZONE 'Asia/Qatar'),
    date_trunc('hour', start_time),
    device_id,
    parameter_id;

-- Create Materialized Views for Daily Aggregates
CREATE MATERIALIZED VIEW iot_data.tz_day_asia_kuala_lumpur AS
SELECT 
    date_trunc('day', start_time AT TIME ZONE 'Asia/Kuala_Lumpur') AS local_start_time,
    date_trunc('day', start_time AT TIME ZONE 'Asia/Kuala_Lumpur') + INTERVAL '1 day' AS local_end_time,
    date_trunc('day', start_time) AS utc_start_time,
    date_trunc('day', start_time) + INTERVAL '1 day' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Kuala_Lumpur' AS timezone,
    AVG(avg_30min_value) AS avg_day_value,
    MIN(min_30min_value) AS min_day_value,
    MAX(max_30min_value) AS max_day_value,
    SUM(sum_30min_value) AS sum_day_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    date_trunc('day', start_time AT TIME ZONE 'Asia/Kuala_Lumpur'),
    date_trunc('day', start_time),
    device_id,
    parameter_id;

CREATE MATERIALIZED VIEW iot_data.tz_day_asia_kolkata AS
SELECT 
    date_trunc('day', start_time AT TIME ZONE 'Asia/Kolkata') AS local_start_time,
    date_trunc('day', start_time AT TIME ZONE 'Asia/Kolkata') + INTERVAL '1 day' AS local_end_time,
    date_trunc('day', start_time) AS utc_start_time,
    date_trunc('day', start_time) + INTERVAL '1 day' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Kolkata' AS timezone,
    AVG(avg_30min_value) AS avg_day_value,
    MIN(min_30min_value) AS min_day_value,
    MAX(max_30min_value) AS max_day_value,
    SUM(sum_30min_value) AS sum_day_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    date_trunc('day', start_time AT TIME ZONE 'Asia/Kolkata'),
    date_trunc('day', start_time),
    device_id,
    parameter_id;

CREATE MATERIALIZED VIEW iot_data.tz_day_asia_qatar AS
SELECT 
    date_trunc('day', start_time AT TIME ZONE 'Asia/Qatar') AS local_start_time,
    date_trunc('day', start_time AT TIME ZONE 'Asia/Qatar') + INTERVAL '1 day' AS local_end_time,
    date_trunc('day', start_time) AS utc_start_time,
    date_trunc('day', start_time) + INTERVAL '1 day' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Qatar' AS timezone,
    AVG(avg_30min_value) AS avg_day_value,
    MIN(min_30min_value) AS min_day_value,
    MAX(max_30min_value) AS max_day_value,
    SUM(sum_30min_value) AS sum_day_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    date_trunc('day', start_time AT TIME ZONE 'Asia/Qatar'),
    date_trunc('day', start_time),
    device_id,
    parameter_id;

-- Create Materialized Views for Monthly Aggregates
CREATE MATERIALIZED VIEW iot_data.tz_month_asia_kuala_lumpur AS
SELECT 
    date_trunc('month', start_time AT TIME ZONE 'Asia/Kuala_Lumpur') AS local_start_time,
    date_trunc('month', start_time AT TIME ZONE 'Asia/Kuala_Lumpur') + INTERVAL '1 month' AS local_end_time,
    date_trunc('month', start_time) AS utc_start_time,
    date_trunc('month', start_time) + INTERVAL '1 month' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Kuala_Lumpur' AS timezone,
    AVG(avg_30min_value) AS avg_month_value,
    MIN(min_30min_value) AS min_month_value,
    MAX(max_30min_value) AS max_month_value,
    SUM(sum_30min_value) AS sum_month_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    date_trunc('month', start_time AT TIME ZONE 'Asia/Kuala_Lumpur'),
    date_trunc('month', start_time),
    device_id,
    parameter_id;

CREATE MATERIALIZED VIEW iot_data.tz_month_asia_kolkata AS
SELECT 
    date_trunc('month', start_time AT TIME ZONE 'Asia/Kolkata') AS local_start_time,
    date_trunc('month', start_time AT TIME ZONE 'Asia/Kolkata') + INTERVAL '1 month' AS local_end_time,
    date_trunc('month', start_time) AS utc_start_time,
    date_trunc('month', start_time) + INTERVAL '1 month' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Kolkata' AS timezone,
    AVG(avg_30min_value) AS avg_month_value,
    MIN(min_30min_value) AS min_month_value,
    MAX(max_30min_value) AS max_month_value,
    SUM(sum_30min_value) AS sum_month_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    date_trunc('month', start_time AT TIME ZONE 'Asia/Kolkata'),
    date_trunc('month', start_time),
    device_id,
    parameter_id;

CREATE MATERIALIZED VIEW iot_data.tz_month_asia_qatar AS
SELECT 
    date_trunc('month', start_time AT TIME ZONE 'Asia/Qatar') AS local_start_time,
    date_trunc('month', start_time AT TIME ZONE 'Asia/Qatar') + INTERVAL '1 month' AS local_end_time,
    date_trunc('month', start_time) AS utc_start_time,
    date_trunc('month', start_time) + INTERVAL '1 month' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Qatar' AS timezone,
    AVG(avg_30min_value) AS avg_month_value,
    MIN(min_30min_value) AS min_month_value,
    MAX(max_30min_value) AS max_month_value,
    SUM(sum_30min_value) AS sum_month_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    date_trunc('month', start_time AT TIME ZONE 'Asia/Qatar'),
    date_trunc('month', start_time),
    device_id,
    parameter_id;
```

## Add Indexes for Better Performance

```sql
-- Create indexes on materialized views for faster querying
CREATE INDEX idx_tz_hour_kl_local_time ON iot_data.tz_hour_asia_kuala_lumpur (local_start_time, device_id, parameter_id);
CREATE INDEX idx_tz_hour_kolkata_local_time ON iot_data.tz_hour_asia_kolkata (local_start_time, device_id, parameter_id);
CREATE INDEX idx_tz_hour_qatar_local_time ON iot_data.tz_hour_asia_qatar (local_start_time, device_id, parameter_id);

CREATE INDEX idx_tz_day_kl_local_time ON iot_data.tz_day_asia_kuala_lumpur (local_start_time, device_id, parameter_id);
CREATE INDEX idx_tz_day_kolkata_local_time ON iot_data.tz_day_asia_kolkata (local_start_time, device_id, parameter_id);
CREATE INDEX idx_tz_day_qatar_local_time ON iot_data.tz_day_asia_qatar (local_start_time, device_id, parameter_id);

CREATE INDEX idx_tz_month_kl_local_time ON iot_data.tz_month_asia_kuala_lumpur (local_start_time, device_id, parameter_id);
CREATE INDEX idx_tz_month_kolkata_local_time ON iot_data.tz_month_asia_kolkata (local_start_time, device_id, parameter_id);
CREATE INDEX idx_tz_month_qatar_local_time ON iot_data.tz_month_asia_qatar (local_start_time, device_id, parameter_id);
```


## Refresh Strategy
Set up a refresh strategy for the materialized views:
```sql
-- Manual refresh (run after new data is loaded)
REFRESH MATERIALIZED VIEW iot_data.tz_hour_asia_kuala_lumpur;
REFRESH MATERIALIZED VIEW iot_data.tz_hour_asia_kolkata;
REFRESH MATERIALIZED VIEW iot_data.tz_hour_asia_qatar;
-- Repeat for all views...

-- Or refresh all at once using a function
CREATE OR REPLACE FUNCTION refresh_timezone_views()
RETURNS void AS $$
BEGIN
    REFRESH MATERIALIZED VIEW iot_data.tz_hour_asia_kuala_lumpur;
    REFRESH MATERIALIZED VIEW iot_data.tz_hour_asia_kolkata;
    REFRESH MATERIALIZED VIEW iot_data.tz_hour_asia_qatar;
    REFRESH MATERIALIZED VIEW iot_data.tz_day_asia_kuala_lumpur;
    REFRESH MATERIALIZED VIEW iot_data.tz_day_asia_kolkata;
    REFRESH MATERIALIZED VIEW iot_data.tz_day_asia_qatar;
    REFRESH MATERIALIZED VIEW iot_data.tz_month_asia_kuala_lumpur;
    REFRESH MATERIALIZED VIEW iot_data.tz_month_asia_kolkata;
    REFRESH MATERIALIZED VIEW iot_data.tz_month_asia_qatar;
END;
$$ LANGUAGE plpgsql;

-- Call the function to refresh all views
SELECT refresh_timezone_views();
```


## Performance Benefits:

1. **Materialized Views**: 10-100x faster queries since data is pre-computed
2. **Indexes**: Additional 2-10x speedup for filtered queries
3. **Storage Trade-off**: Uses more disk space but dramatically improves query performance
4. **Refresh Control**: You control when expensive computations happen (e.g., nightly)

The materialized view approach will give you the best performance improvement. You'll need to refresh them periodically (daily/hourly) depending on how fresh you need the data to be.
## Option 2: TimescaleDB Continuous Aggregates (Alternative)

If you're using TimescaleDB's advanced features, you could also consider continuous aggregates which automatically maintain the materialized data:

```sql
-- Drop existing views/materialized views first (if they exist)
DROP MATERIALIZED VIEW IF EXISTS iot_data.tz_hour_asia_kuala_lumpur CASCADE;
DROP MATERIALIZED VIEW IF EXISTS iot_data.tz_hour_asia_kolkata CASCADE;
DROP MATERIALIZED VIEW IF EXISTS iot_data.tz_hour_asia_qatar CASCADE;
DROP MATERIALIZED VIEW IF EXISTS iot_data.tz_day_asia_kuala_lumpur CASCADE;
DROP MATERIALIZED VIEW IF EXISTS iot_data.tz_day_asia_kolkata CASCADE;
DROP MATERIALIZED VIEW IF EXISTS iot_data.tz_day_asia_qatar CASCADE;
DROP MATERIALIZED VIEW IF EXISTS iot_data.tz_month_asia_kuala_lumpur CASCADE;
DROP MATERIALIZED VIEW IF EXISTS iot_data.tz_month_asia_kolkata CASCADE;
DROP MATERIALIZED VIEW IF EXISTS iot_data.tz_month_asia_qatar CASCADE;

-- HOURLY CONTINUOUS AGGREGATES

-- Asia/Kuala_Lumpur Hourly Continuous Aggregate
CREATE MATERIALIZED VIEW iot_data.tz_hour_asia_kuala_lumpur
WITH (timescaledb.continuous) AS
SELECT 
    time_bucket('1 hour', start_time AT TIME ZONE 'Asia/Kuala_Lumpur') AS local_start_time,
    time_bucket('1 hour', start_time AT TIME ZONE 'Asia/Kuala_Lumpur') + INTERVAL '1 hour' AS local_end_time,
    time_bucket('1 hour', start_time) AS utc_start_time,
    time_bucket('1 hour', start_time) + INTERVAL '1 hour' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Kuala_Lumpur' AS timezone,
    AVG(avg_30min_value) AS avg_hour_value,
    MIN(min_30min_value) AS min_hour_value,
    MAX(max_30min_value) AS max_hour_value,
    SUM(sum_30min_value) AS sum_hour_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    time_bucket('1 hour', start_time AT TIME ZONE 'Asia/Kuala_Lumpur'),
    time_bucket('1 hour', start_time),
    device_id,
    parameter_id;

-- Asia/Kolkata Hourly Continuous Aggregate
CREATE MATERIALIZED VIEW iot_data.tz_hour_asia_kolkata
WITH (timescaledb.continuous) AS
SELECT 
    time_bucket('1 hour', start_time AT TIME ZONE 'Asia/Kolkata') AS local_start_time,
    time_bucket('1 hour', start_time AT TIME ZONE 'Asia/Kolkata') + INTERVAL '1 hour' AS local_end_time,
    time_bucket('1 hour', start_time) AS utc_start_time,
    time_bucket('1 hour', start_time) + INTERVAL '1 hour' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Kolkata' AS timezone,
    AVG(avg_30min_value) AS avg_hour_value,
    MIN(min_30min_value) AS min_hour_value,
    MAX(max_30min_value) AS max_hour_value,
    SUM(sum_30min_value) AS sum_hour_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    time_bucket('1 hour', start_time AT TIME ZONE 'Asia/Kolkata'),
    time_bucket('1 hour', start_time),
    device_id,
    parameter_id;

-- Asia/Qatar Hourly Continuous Aggregate
CREATE MATERIALIZED VIEW iot_data.tz_hour_asia_qatar
WITH (timescaledb.continuous) AS
SELECT 
    time_bucket('1 hour', start_time AT TIME ZONE 'Asia/Qatar') AS local_start_time,
    time_bucket('1 hour', start_time AT TIME ZONE 'Asia/Qatar') + INTERVAL '1 hour' AS local_end_time,
    time_bucket('1 hour', start_time) AS utc_start_time,
    time_bucket('1 hour', start_time) + INTERVAL '1 hour' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Qatar' AS timezone,
    AVG(avg_30min_value) AS avg_hour_value,
    MIN(min_30min_value) AS min_hour_value,
    MAX(max_30min_value) AS max_hour_value,
    SUM(sum_30min_value) AS sum_hour_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    time_bucket('1 hour', start_time AT TIME ZONE 'Asia/Qatar'),
    time_bucket('1 hour', start_time),
    device_id,
    parameter_id;

-- DAILY CONTINUOUS AGGREGATES

-- Asia/Kuala_Lumpur Daily Continuous Aggregate
CREATE MATERIALIZED VIEW iot_data.tz_day_asia_kuala_lumpur
WITH (timescaledb.continuous) AS
SELECT 
    time_bucket('1 day', start_time AT TIME ZONE 'Asia/Kuala_Lumpur') AS local_start_time,
    time_bucket('1 day', start_time AT TIME ZONE 'Asia/Kuala_Lumpur') + INTERVAL '1 day' AS local_end_time,
    time_bucket('1 day', start_time) AS utc_start_time,
    time_bucket('1 day', start_time) + INTERVAL '1 day' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Kuala_Lumpur' AS timezone,
    AVG(avg_30min_value) AS avg_day_value,
    MIN(min_30min_value) AS min_day_value,
    MAX(max_30min_value) AS max_day_value,
    SUM(sum_30min_value) AS sum_day_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    time_bucket('1 day', start_time AT TIME ZONE 'Asia/Kuala_Lumpur'),
    time_bucket('1 day', start_time),
    device_id,
    parameter_id;

-- Asia/Kolkata Daily Continuous Aggregate
CREATE MATERIALIZED VIEW iot_data.tz_day_asia_kolkata
WITH (timescaledb.continuous) AS
SELECT 
    time_bucket('1 day', start_time AT TIME ZONE 'Asia/Kolkata') AS local_start_time,
    time_bucket('1 day', start_time AT TIME ZONE 'Asia/Kolkata') + INTERVAL '1 day' AS local_end_time,
    time_bucket('1 day', start_time) AS utc_start_time,
    time_bucket('1 day', start_time) + INTERVAL '1 day' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Kolkata' AS timezone,
    AVG(avg_30min_value) AS avg_day_value,
    MIN(min_30min_value) AS min_day_value,
    MAX(max_30min_value) AS max_day_value,
    SUM(sum_30min_value) AS sum_day_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    time_bucket('1 day', start_time AT TIME ZONE 'Asia/Kolkata'),
    time_bucket('1 day', start_time),
    device_id,
    parameter_id;

-- Asia/Qatar Daily Continuous Aggregate
CREATE MATERIALIZED VIEW iot_data.tz_day_asia_qatar
WITH (timescaledb.continuous) AS
SELECT 
    time_bucket('1 day', start_time AT TIME ZONE 'Asia/Qatar') AS local_start_time,
    time_bucket('1 day', start_time AT TIME ZONE 'Asia/Qatar') + INTERVAL '1 day' AS local_end_time,
    time_bucket('1 day', start_time) AS utc_start_time,
    time_bucket('1 day', start_time) + INTERVAL '1 day' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Qatar' AS timezone,
    AVG(avg_30min_value) AS avg_day_value,
    MIN(min_30min_value) AS min_day_value,
    MAX(max_30min_value) AS max_day_value,
    SUM(sum_30min_value) AS sum_day_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    time_bucket('1 day', start_time AT TIME ZONE 'Asia/Qatar'),
    time_bucket('1 day', start_time),
    device_id,
    parameter_id;

-- MONTHLY CONTINUOUS AGGREGATES

-- Asia/Kuala_Lumpur Monthly Continuous Aggregate
CREATE MATERIALIZED VIEW iot_data.tz_month_asia_kuala_lumpur
WITH (timescaledb.continuous) AS
SELECT 
    time_bucket('1 month', start_time AT TIME ZONE 'Asia/Kuala_Lumpur') AS local_start_time,
    time_bucket('1 month', start_time AT TIME ZONE 'Asia/Kuala_Lumpur') + INTERVAL '1 month' AS local_end_time,
    time_bucket('1 month', start_time) AS utc_start_time,
    time_bucket('1 month', start_time) + INTERVAL '1 month' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Kuala_Lumpur' AS timezone,
    AVG(avg_30min_value) AS avg_month_value,
    MIN(min_30min_value) AS min_month_value,
    MAX(max_30min_value) AS max_month_value,
    SUM(sum_30min_value) AS sum_month_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    time_bucket('1 month', start_time AT TIME ZONE 'Asia/Kuala_Lumpur'),
    time_bucket('1 month', start_time),
    device_id,
    parameter_id;

-- Asia/Kolkata Monthly Continuous Aggregate
CREATE MATERIALIZED VIEW iot_data.tz_month_asia_kolkata
WITH (timescaledb.continuous) AS
SELECT 
    time_bucket('1 month', start_time AT TIME ZONE 'Asia/Kolkata') AS local_start_time,
    time_bucket('1 month', start_time AT TIME ZONE 'Asia/Kolkata') + INTERVAL '1 month' AS local_end_time,
    time_bucket('1 month', start_time) AS utc_start_time,
    time_bucket('1 month', start_time) + INTERVAL '1 month' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Kolkata' AS timezone,
    AVG(avg_30min_value) AS avg_month_value,
    MIN(min_30min_value) AS min_month_value,
    MAX(max_30min_value) AS max_month_value,
    SUM(sum_30min_value) AS sum_month_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    time_bucket('1 month', start_time AT TIME ZONE 'Asia/Kolkata'),
    time_bucket('1 month', start_time),
    device_id,
    parameter_id;

-- Asia/Qatar Monthly Continuous Aggregate
CREATE MATERIALIZED VIEW iot_data.tz_month_asia_qatar
WITH (timescaledb.continuous) AS
SELECT 
    time_bucket('1 month', start_time AT TIME ZONE 'Asia/Qatar') AS local_start_time,
    time_bucket('1 month', start_time AT TIME ZONE 'Asia/Qatar') + INTERVAL '1 month' AS local_end_time,
    time_bucket('1 month', start_time) AS utc_start_time,
    time_bucket('1 month', start_time) + INTERVAL '1 month' AS utc_end_time,
    device_id,
    parameter_id,
    'Asia/Qatar' AS timezone,
    AVG(avg_30min_value) AS avg_month_value,
    MIN(min_30min_value) AS min_month_value,
    MAX(max_30min_value) AS max_month_value,
    SUM(sum_30min_value) AS sum_month_value,
    SUM(reading_count) AS reading_count
FROM iot_data.timescale_30min_aggregate_utc_complete
GROUP BY 
    time_bucket('1 month', start_time AT TIME ZONE 'Asia/Qatar'),
    time_bucket('1 month', start_time),
    device_id,
    parameter_id;
```

## Set Up Automatic Refresh Policies

Now add refresh policies so TimescaleDB automatically maintains these continuous aggregates:

```sql
-- Add refresh policies for hourly aggregates (refresh every 30 minutes)
SELECT add_continuous_aggregate_policy('iot_data.tz_hour_asia_kuala_lumpur',
    start_offset => INTERVAL '3 hours',
    end_offset => INTERVAL '30 minutes',
    schedule_interval => INTERVAL '30 minutes');

SELECT add_continuous_aggregate_policy('iot_data.tz_hour_asia_kolkata',
    start_offset => INTERVAL '3 hours',
    end_offset => INTERVAL '30 minutes',
    schedule_interval => INTERVAL '30 minutes');

SELECT add_continuous_aggregate_policy('iot_data.tz_hour_asia_qatar',
    start_offset => INTERVAL '3 hours',
    end_offset => INTERVAL '30 minutes',
    schedule_interval => INTERVAL '30 minutes');

-- Add refresh policies for daily aggregates (refresh every hour)
SELECT add_continuous_aggregate_policy('iot_data.tz_day_asia_kuala_lumpur',
    start_offset => INTERVAL '3 days',
    end_offset => INTERVAL '1 hour',
    schedule_interval => INTERVAL '1 hour');

SELECT add_continuous_aggregate_policy('iot_data.tz_day_asia_kolkata',
    start_offset => INTERVAL '3 days',
    end_offset => INTERVAL '1 hour',
    schedule_interval => INTERVAL '1 hour');

SELECT add_continuous_aggregate_policy('iot_data.tz_day_asia_qatar',
    start_offset => INTERVAL '3 days',
    end_offset => INTERVAL '1 hour',
    schedule_interval => INTERVAL '1 hour');

-- Add refresh policies for monthly aggregates (refresh every 6 hours)
SELECT add_continuous_aggregate_policy('iot_data.tz_month_asia_kuala_lumpur',
    start_offset => INTERVAL '3 months',
    end_offset => INTERVAL '6 hours',
    schedule_interval => INTERVAL '6 hours');

SELECT add_continuous_aggregate_policy('iot_data.tz_month_asia_kolkata',
    start_offset => INTERVAL '3 months',
    end_offset => INTERVAL '6 hours',
    schedule_interval => INTERVAL '6 hours');

SELECT add_continuous_aggregate_policy('iot_data.tz_month_asia_qatar',
    start_offset => INTERVAL '3 months',
    end_offset => INTERVAL '6 hours',
    schedule_interval => INTERVAL '6 hours');
```

## Optional: Add Retention Policies

You can also add retention policies to automatically clean up old data:

```sql
-- Example: Keep hourly data for 1 year, daily for 3 years, monthly for 10 years
SELECT add_retention_policy('iot_data.tz_hour_asia_kuala_lumpur', INTERVAL '1 year');
SELECT add_retention_policy('iot_data.tz_hour_asia_kolkata', INTERVAL '1 year');
SELECT add_retention_policy('iot_data.tz_hour_asia_qatar', INTERVAL '1 year');

SELECT add_retention_policy('iot_data.tz_day_asia_kuala_lumpur', INTERVAL '3 years');
SELECT add_retention_policy('iot_data.tz_day_asia_kolkata', INTERVAL '3 years');
SELECT add_retention_policy('iot_data.tz_day_asia_qatar', INTERVAL '3 years');

SELECT add_retention_policy('iot_data.tz_month_asia_kuala_lumpur', INTERVAL '10 years');
SELECT add_retention_policy('iot_data.tz_month_asia_kolkata', INTERVAL '10 years');
SELECT add_retention_policy('iot_data.tz_month_asia_qatar', INTERVAL '10 years');
```

## Key Features of TimescaleDB Continuous Aggregates:

1. **Automatic Maintenance**: TimescaleDB automatically keeps the aggregates up-to-date based on the refresh policies
2. **Incremental Updates**: Only processes new/changed data, not the entire dataset
3. **Real-time Queries**: Can combine pre-computed data with real-time data for the most recent time periods
4. **High Performance**: Dramatically faster than regular views, similar to materialized views but auto-maintained
5. **Policy-based Management**: Automatic refresh and retention policies reduce manual maintenance

**Performance Benefits:**

- **Query Speed**: 10-1000x faster than regular views
- **Automatic Updates**: No manual refresh needed
- **Resource Efficiency**: Incremental processing reduces compute load
- **Storage Optimization**: Built-in compression and retention policies

These continuous aggregates will provide excellent performance while automatically staying current with your source data.