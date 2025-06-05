# Performance Optimization Requirements

## Objective

Refactor the provided AWS Lambda code to implement Database Optimization, Parallel Processing, and Excel Optimization while maintaining complete functional correctness and existing business logic.

## Critical Requirements

1. **MAINTAIN FUNCTIONAL CORRECTNESS**: All existing functionality must work exactly as before
2. **PRESERVE BUSINESS LOGIC**: Do not alter calculation methods, data processing logic, or report content
3. **MAINTAIN API COMPATIBILITY**: All endpoints and response formats must remain unchanged
4. **ERROR HANDLING**: Preserve existing error handling and add appropriate error handling for new concurrent operations
5. **LOGGING**: Maintain existing logging and add performance monitoring logs

## Optimization Methods to Implement

### 1. Database Connection Optimization

**Requirements:**

- Implement connection pooling using `psycopg2.pool.ThreadedConnectionPool`
- Replace single connection pattern with connection pool management
- Ensure thread-safe database operations
- Maintain existing query logic and results

**Implementation Guidelines:**

# Replace the current connection pattern with:

```python
# Replace the current connection pattern with:
from psycopg2 import pool

class WattReport:
    def __init__(self):
        # Create connection pool (min=2, max=10 connections)
        self.connection_pool = psycopg2.pool.ThreadedConnectionPool(
            2, 10,  # Adjust based on Lambda concurrency needs
            host=self.timescale_host,
            port=self.timescale_port,
            dbname=self.timescale_dbname,
            user=self.timescale_user,
            password=self.timescale_password,
        )
    
    def get_timescale_cursor(self):
        # Return both cursor and connection for proper cleanup
        # Handle connection pool exhaustion gracefully
    
    def return_connection(self, conn):
        # Properly return connections to pool
```

**Critical Points:**

- Update ALL database methods to use connection pooling
- Ensure connections are always returned to the pool (use try/finally blocks)
- Handle connection pool exhaustion scenarios
- Maintain autocommit behavior where currently used

### 2. Parallel Processing Implementation

**Requirements:**

- Implement parallel processing for multiple devices in report generation methods
- Use `ThreadPoolExecutor` with appropriate worker limits (max 5-8 workers for Lambda)
- Maintain data integrity and proper error handling per device
- Preserve existing data aggregation and formatting logic

**Implementation Guidelines:**

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import threading

# Refactor these methods to process devices in parallel:
# - activity_report()
# - daily_consumption_report() 
# - monthly_consumption_report()
# - max_consumption_report()
# - daily_energy_report()
# - monthly_energy_report()

def process_devices_parallel(self, device_list, processing_function, *args, max_workers=5):
    """
    Generic parallel processing function for device data
    Must handle individual device failures gracefully
    Must maintain original data structure and format
    """
```

Copy

**Critical Points:**

- Each device should be processed independently
- If one device fails, others should continue processing
- Maintain the exact same data structure in `device_data_dict`
- Use thread-safe operations for shared resources
- Add performance timing logs for parallel vs sequential comparison

### 3. Excel Generation Optimization

**Requirements:**

- Implement write-only mode for openpyxl Workbook creation
- Optimize image handling and resizing operations
- Use streaming/generator patterns for large datasets
- Maintain exact Excel formatting, styling, and layout

**Implementation Guidelines:**

```python
from openpyxl import Workbook
from openpyxl.writer.excel import save_virtual_workbook

# Optimize these Excel creation methods:
# - create_activity_report()
# - create_daily_consumption_report()
# - create_monthly_consumption_report()
# - create_max_consumption_report()
# - create_daily_energy_report()
# - create_monthly_energy_report()

def create_optimized_excel_report(self, data, report_type):
    """
    Use write_only=True for Workbook
    Optimize image loading and processing
    Use generators for row data where possible
    Maintain all existing formatting and styling
    """
```

**Critical Points:**

- Preserve all existing Excel formatting (fonts, borders, alignment, etc.)
- Maintain image positioning and sizing
- Keep all chart generation functionality
- Ensure S3 upload process remains unchanged
- Use `BytesIO` efficiently for memory management

## Specific Refactoring Instructions

### Phase 1: Database Connection Pool

1. Modify `__init__` method to create connection pool
2. Update `get_timescale_cursor()` to return cursor and connection
3. Add `return_connection()` method
4. Update ALL methods that use database connections:
    - `get_device_data()`
    - `get_demand_data()`
    - `get_topic_ids_by_location()`
    - `get_redshift_client_ids_from_client_id()`
    - `get_meter_details()`
    - `get_peak_off_peak_data()`
    - `get_tariff_config()`
    - `get_device_details()`
    - `get_run_hour_config()`
    - `get_run_hour_data()`
    - `get_run_hour_data_optimized()`
    - `get_client_id_from_redshift_id()`
    - `get_recipients()`
    - `insert_report_history()`
    - `get_report_schedules()`

### Phase 2: Parallel Processing

1. Create generic parallel processing function
2. Refactor main report methods to use parallel processing:
    
```python
# Instead of:
for i in range(len(device_list)):
    data = self.get_activity_data(...)
    device_data_dict[incname] = {...}

# Use:
device_data_dict = self.process_devices_parallel(
    device_list, self.get_activity_data, start_date_obj, end_date_obj, timezone, on_peak_time
)
```

### Phase 3: Excel Optimization

1. Add write-only mode to all Workbook creations
2. Optimize image loading with caching or streaming
3. Use generators for repetitive row operations
4. Implement memory-efficient chart generation

## Validation Requirements

### Functional Testing

- All existing API endpoints must return identical results
- Excel files must have identical content, formatting, and structure
- S3 uploads must work without changes
- Email notifications must continue functioning
- Scheduled reports must work correctly

### Performance Testing

- Measure execution time before and after optimization
- Monitor memory usage patterns
- Test with various device list sizes (1, 5, 10, 20+ devices)
- Verify no timeout issues with optimized code

### Error Handling Testing

- Test database connection failures
- Test individual device processing failures
- Test memory constraints
- Test concurrent access scenarios

## Code Quality Standards

1. **Maintain existing code style and patterns**
2. **Add comprehensive error handling for new concurrent operations**
3. **Include performance monitoring logs**:
```python
start_time = time.time()
# ... processing ...
app.log.info(f"Parallel processing completed in {time.time() - start_time:.2f} seconds")
```
1. **Add docstrings for new methods**
2. **Preserve all existing comments and documentation**

## Expected Performance Improvements

- Database operations: 40-60% improvement
- Multi-device processing: 70-80% improvement
- Excel generation: 25-40% improvement
- Overall Lambda execution: 50-70% improvement for multi-device reports

## Deliverables

1. **Fully refactored Lambda code** with all three optimizations implemented
2. **Detailed change summary** highlighting what was modified
3. **Performance comparison notes** indicating expected improvements
4. **Migration notes** for any configuration changes needed

**IMPORTANT**: The refactored code must be a drop-in replacement that maintains 100% functional compatibility while delivering significant performance improvements.