## Key Features

### Core Functionality

1. **Active Device Discovery**: Queries `iot_data.redshift_devices` for devices with `status = true`
2. **Measurement Analysis**: Analyzes recent measurements within configurable time intervals
3. **Parameter Resolution**: Joins parameter IDs with `iot_data.timescale_parameters` for names
4. **Delta Detection**: Compares current vs existing device-parameter relationships
5. **Incremental Updates**: Only performs necessary INSERT/DELETE operations

### Technical Features

- **Connection Pooling**: Efficient database connection management with retry logic
- **Error Handling**: Comprehensive error handling and logging
- **CloudWatch Metrics**: Monitoring and alerting capabilities
- **Health Checks**: Built-in health check endpoint
- **Manual Triggers**: API endpoint for on-demand synchronization
- **Scheduled Execution**: Cron-based automatic synchronization

### Configuration

- **Environment Variables**: All configuration via environment variables
- **Configurable Intervals**: Adjustable check intervals and timeouts
- **Multiple Stages**: Support for dev/prod deployments

### Deployment Commands

```
# Install Chalice
pip install chalice

# Deploy to development
chalice deploy --stage dev

# Deploy to production  
chalice deploy --stage prod

# Update function only
chalice deploy --stage dev --no-autogen-policy
```

This implementation provides a robust, scalable solution for synchronizing device parameters based on IoT measurement data, with comprehensive monitoring and error handling capabilities.

## Database Table for Device Parameters

```sql
CREATE TABLE IF NOT EXISTS iot_data.device_parameters (
	device_id TEXT NOT NULL,
	device_name TEXT NOT NULL,
	parameter_id TEXT NOT NULL,
	parameter_name TEXT NOT NULL,
	created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
	updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
	PRIMARY KEY (device_id, parameter_id)
);
 
CREATE INDEX IF NOT EXISTS idx_device_parameters_device_id 
ON iot_data.device_parameters(device_id);

CREATE INDEX IF NOT EXISTS idx_device_parameters_parameter_id 
ON iot_data.device_parameters(parameter_id);
```
