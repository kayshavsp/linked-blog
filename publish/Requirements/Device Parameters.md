# Requirement: Device Parameters Synchronization Lambda

Create an AWS Chalice application that runs on AWS Lambda to synchronize device parameters based on IoT measurement data. The application should replicate the functionality of the `iot_data.device_parameters` view by maintaining an up-to-date table of device-parameter relationships.

### Database Schema Context

**Tables and Views:**

1. **`iot_data.device_parameters`** (View) - Target view to replicate
    
    - `device_id` (text)
    - `device_name` (text)
    - `parameter_id` (text)
    - `parameter_name` (text)
2. **`iot_data.redshift_devices`** (Table) - Device master data
    
    - `device_id` (text, primary key)
    - `device_name` (text)
    - `status` (boolean) - indicates if device is active
    - `is_active` (boolean) - additional active status flag
    - Other fields: device_type_id, location_id, created_at, etc.
3. **`iot_data.timescale_measurements`** (Table) - Time-series measurement data
    
    - `time` (timestamp with time zone, primary key)
    - `device_id` (text, primary key)
    - `parameter_id` (text, primary key)
    - `value` (double precision)
    - `quality` (smallint)
    - `metadata` (jsonb)
4. **`iot_data.timescale_parameters`** (Table) - Parameter definitions
    
    - `parameter_id` (text, primary key)
    - `parameter_name` (text)
    - `attribute` (text)
    - `device_type_id` (text)
    - `data_type` (text)
    - `unit` (text)
    - `description` (text)

### Requirements

**Core Functionality:**

1. **Active Device Discovery**: Query `iot_data.redshift_devices` to get all devices where `status = true` (active devices)
    
2. **Measurement Analysis**: For each active device, query `iot_data.timescale_measurements` within a configurable time interval to find all unique `parameter_id` values that have recent readings
    
3. **Parameter Resolution**: Join the discovered parameter IDs with `iot_data.timescale_parameters` to get parameter names
    
4. **Delta Detection**: Compare current device-parameter relationships from measurements against existing records in `iot_data.device_parameters` table/view
    
5. **Incremental Updates**: Only perform necessary changes:
    
    - **ADD**: Insert new device-parameter combinations found in recent measurements
    - **REMOVE**: Delete device-parameter combinations no longer present in recent measurements

**Configuration:**

- **Check Interval**: Configurable via environment variable (e.g., `CHECK_INTERVAL_MINUTES`)
- **Database Connection**: Use environment variables for database credentials and connection details

**Technical Specifications:**

- Use AWS Chalice framework
- Include proper error handling and logging
- Use connection pooling for database efficiency
- Implement retry logic for database operations
- Add CloudWatch metrics for monitoring
- Use environment variables for configuration

**Code Structure Requirements:**
```text
app.py (main Chalice application)
├── Lambda handler function
├── Database connection management
├── Device discovery logic
├── Measurement analysis logic
├── Delta calculation logic
└── Database update operations

requirements.txt
├── chalice
├── psycopg2-binary (for PostgreSQL/TimescaleDB)
├── boto3
└── other dependencies

.chalice/config.json (Chalice configuration)
```

**Environment Variables:**

- `TIMESCALE_DB_HOST`
- `TIMESCALE_DB_PORT`
- `TIMESCALE_DB_NAME`
- `TIMESCALE_DB_USER`
- `TIMESCALE_DB_PASSWORD`
- `CHECK_INTERVAL_MINUTES` (default: 60)

**Sample SQL Logic to Implement:**
```sql
-- 1. Get active devices
SELECT device_id, device_name FROM iot_data.redshift_devices WHERE status = true;

-- 2. Get recent measurements for a device (within interval)
SELECT DISTINCT device_id, parameter_id 
FROM iot_data.timescale_measurements 
WHERE device_id = ? AND time >= NOW() - INTERVAL '? minutes';

-- 3. Get parameter names
SELECT parameter_id, parameter_name 
FROM iot_data.timescale_parameters 
WHERE parameter_id IN (?);

-- 4. Compare with existing device_parameters
-- 5. Perform INSERT/DELETE operations as needed
```

**Additional Features:**

- Scheduled execution (can be triggered by CloudWatch Events Cron job)
- Comprehensive logging for debugging
- Performance optimization for large datasets
- Graceful handling of database connection issues
- Don't perform changes if database connection failed
- Error handling 