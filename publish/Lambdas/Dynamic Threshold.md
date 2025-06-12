## Key Features:

### 1. **API Endpoint** (`/threshold-recommendation`)

- Accepts POST requests with analytics pattern, device IDs, percentile, timezone, and calculation period
- Returns threshold recommendations with device breakdown and metadata
- Supports all 5 analytics patterns: tariff_ratio, demand_profile, load_profile, run_hour, enpi

### 2. **Automated Monthly Updates**

- **Master Lambda**: Runs on 1st of every month, identifies applets with `dynamic_threshold.enabled = true`
- **Child Lambda**: Processes individual applet threshold updates
- Supports all update conditions: `lower_only`, `higher_only`, `always`
- Applies offset percentages and validates minimum data points

### 3. **Analytics Pattern Calculations**

#### Tariff Ratio

- Calculates average daily on-peak/off-peak ratios
- Uses 30-min aggregate data with tariff configuration

#### Demand Profile

- Calculates percentile thresholds for peak/off-peak periods
- Returns separate thresholds for on_peak and off_peak

#### Load Profile

- Calculates percentile thresholds for active_power, reactive_power, apparent_power
- Uses raw timescale_measurements data

#### Run Hour

- Calculates load distribution percentages (Off Load, No Load, Partial Load, Full Load)
- Supports both single-phase and three-phase devices

#### EnPI

- Calculates operational energy performance indicators
- Combines current and energy measurements

### 4. **Database Integration**

- Robust connection management with reconnection logic
- Proper transaction handling and error recovery
- Support for multiple timezone views

### 5. **Error Handling & Logging**

- Comprehensive error handling throughout
- Detailed logging for debugging and monitoring
- Graceful degradation when data is unavailable

### 6. **Security & Performance**

- Environment variable configuration
- Connection pooling and autocommit for performance
- Input validation and sanitization

### 7. **Testing Support**

- Manual test functions for both API and update functionality
- Dry-run capabilities for debugging

The implementation follows the exact mathematical formulas and database schema described in your requirements, using the DEES Incidents lambda as reference for calculation patterns and database interactions.

## Usage Examples:
**API Call:**

```JSON
// POST /recommend-threshold
{
  "analytics_pattern": "load_profile",
  "device_ids": ["dda3677b-4756-4544-ada4-094af298c79b"],
  "percentile": 1.0,
  "power_type": "active_power",
  "timezone": "Asia/Kuala_Lumpur"
}
```

**Dynamic Threshold Configuration:**

```JSON
{
  "timezone": "Asia/Kuala_Lumpur",
  "dynamic_threshold": {
    "enabled": true,
    "offset_percentage": 10.0,
    "update_condition": "lower_only",
    "min_data_points": 100,
    "percentile": 1.0
  }
}
```

The system handles all requirements including master-child architecture, monthly updates, audit trails, and comprehensive error handling while maintaining optimal database performance.

**Test Data:**

```Python
# Test threshold recommendation
device_ids = ["93bb50ac-e605-4f4d-8968-076cb9a9a432","dda3677b-4756-4544-ada4-094af298c79b"]
result = test_threshold_recommendation(
	"tariff_ratio", 
	device_ids, 
	timezone="Asia/Kuala_Lumpur",
	client_id="b3a92112-c60f-427a-9862-179adbace390"
)
print(json.dumps(result, indent=2))

# Test dynamic threshold update
result = test_dynamic_threshold_update("35ab88ca-2e32-4270-98f6-a2f5fa69e97a")
print(json.dumps(result, indent=2))
```

The system handles all requirements including master-child architecture, monthly updates, audit trails, and comprehensive error handling while maintaining optimal database performance.