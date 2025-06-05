
# Requirements: Update MQTT-Kinesis Bridge for Multiple Broker Support

## Task Description

Update the provided MQTT-Kinesis bridge Python code to support connections to multiple MQTT brokers simultaneously, using broker configuration data retrieved from a TimescaleDB database.

## Current Code Context

The existing code connects to a single hardcoded MQTT broker (`broker.client.io`) and bridges messages to AWS Kinesis. It needs to be enhanced to:

1. Connect to multiple MQTT brokers dynamically
2. Retrieve broker configurations from the database
3. Maintain all existing functionality (topic updates, Kinesis publishing, etc.)

## Reference Implementation

Use the second code file as a reference for the multi-broker pattern. Key elements to adapt:

### Database Schema for Brokers

The `iot_data.mqtt_broker` table contains:

- `mqtt_id`: Unique broker identifier
- `mqtt_host`: Broker hostname/IP
- `mqtt_port`: Broker port number
- `mqtt_username`: Authentication username
- `mqtt_password`: Authentication password
- `status`: Boolean indicating if broker is active

### Multi-Broker Connection Pattern

From the reference code, implement:

```sql
# Get broker list from database
mqtt_broker_list = self.get_mqtt_broker_list()

# Create connections for each broker
bridge_list = []
for mqtt_broker in mqtt_broker_list:
    bridge = MQTTConnection(device_check, mqtt_broker)
    bridge.connect()
    bridge_list.append(bridge)

# Start all MQTT loops
for bridge in bridge_list:
    bridge.client.loop_start()
```
## Specific Requirements

### 1. Add Broker Configuration Method

Create a `get_mqtt_broker_list()` method that:

- Queries the `iot_data.mqtt_broker` table
- Returns active brokers (where `status=TRUE`)
- Returns a list of dictionaries with broker connection details

### 2. Modify MQTTKinesisBridge Class

Update the class to:

- Accept broker configuration instead of hardcoded values
- Remove hardcoded `mqtt_host = "broker.client.io"` and credentials
- Use broker-specific connection parameters

### 3. Update Main Execution Logic

Modify the `if __name__ == '__main__':` section to:

- Retrieve broker list from database
- Create multiple `MQTTKinesisBridge` instances (one per broker)
- Start all bridges concurrently
- Maintain a list of active bridges

### 4. Handle Multiple Client Loops

Ensure proper handling of:

- Multiple `client.loop_forever()` calls (use threading or async)
- Graceful shutdown of all connections
- Error handling for individual broker failures

### 5. Preserve Existing Functionality

Maintain all current features:

- Dynamic topic subscription updates
- Kinesis record publishing
- Topic periodic updates (60-second interval)
- Connection error handling
- Message processing and routing

## Technical Considerations

### Database Connection

- Reuse the existing TimescaleDB connection parameters
- Add proper error handling for broker list retrieval
- Handle cases where no active brokers are found

### Threading/Concurrency

- Each broker connection should run in its own thread
- Use `client.loop_start()` instead of `client.loop_forever()` for concurrent operation
- Consider using daemon threads for broker connections

### Error Resilience

- Individual broker failures should not affect other connections
- Log broker-specific connection status and errors
- Implement reconnection logic per broker

### Resource Management

- Properly close all database connections
- Clean shutdown of all MQTT client connections
- Handle KeyboardInterrupt for graceful exit

## Expected Output Structure

The updated code should:

1. Query active brokers from the database on startup
2. Create separate `MQTTKinesisBridge` instances for each broker
3. Start all broker connections concurrently
4. Continue processing messages from all brokers simultaneously
5. Maintain existing Kinesis publishing functionality
6. Preserve dynamic topic subscription updates across all brokers

## Code Quality Requirements

- Add appropriate logging for multi-broker operations
- Include error handling for database queries and broker connections
- Maintain the existing code structure and patterns where possible
- Add comments explaining the multi-broker implementation
- Ensure proper resource cleanup and connection management

Please update the first code file to implement these multi-broker capabilities while preserving all existing functionality.
