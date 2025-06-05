# Complete Database Design Requirements: Asset-Device Hierarchical Mapping System for React Flow

## Context
You are designing a database schema for an IoT energy management system that needs to support React Flow visualizations of device hierarchies within assets. The system currently has:

### Existing Tables:
- **assets**: Contains asset information with `asset_id`, `asset_name`, `location_id`, `client_id`, `asset_type_id`
- **asset_types**: Defines asset types with parameters for LSTM forecasting (`asset_type_id`, `asset_type_name`, `parameters`, `metadata`)
- **redshift_devices**: Device information with `device_id`, `device_name`, `asset_id`, `level`, `source`, `is_incomer`, etc.
- **device_types**: Device type definitions

## Requirements

### 1. Many-to-Many Relationships
- **Asset ↔ Device**: One asset can contain multiple devices, and devices can potentially belong to multiple assets
- **Device ↔ Device**: Devices within an asset can connect to multiple other devices (creating a network topology)

### 2. Hierarchical Levels
- Devices must have a hierarchy level (1, 2, 3, etc.)
- Level 1: Source devices (entry points)
- Level 2: Devices directly connected to Level 1
- Level 3: Devices connected to Level 2, and so on
- Support for calculating levels dynamically based on connections

### 3. React Flow Integration
- Data structure must support React Flow nodes and edges
- Need to easily generate:
  - Nodes: Device information with positioning data
  - Edges: Connections between devices with flow direction

### 4. Views Required
- **Complete Device Map**: All devices and connections within an asset
- **Adjacent Level View**: Only devices and connections between two adjacent levels (e.g., Level 2-3)

### 5. Asset Type Integration
- Maintain relationship between assets and asset_types
- Asset types determine relevant parameters for the asset
- Allow modifications to existing asset_types table structure if needed

## Design Tasks

### 1. Table Design
Create new tables and modify existing ones to support:
- Asset-Device many-to-many relationships
- Device-Device connections with directional flow
- Hierarchical level management
- React Flow positioning and metadata

### 2. Junction Tables
Design junction tables for:
- `asset_devices`: Asset to device relationships
- `device_connections`: Device to device connections with hierarchy information

### 3. Enhanced Tables
Modify or enhance existing tables:
- Add React Flow specific columns (position_x, position_y, node_type, etc.)
- Enhance asset_types for better parameter management
- Add computed hierarchy level support

### 4. Views
Create SQL views for:
- **`v_asset_device_map`**: Complete device topology for an asset
- **`v_adjacent_level_map`**: Filtered view for adjacent levels
- **`v_react_flow_data`**: Formatted data ready for React Flow consumption

### 5. Additional Considerations
- Include proper foreign key constraints
- Add indexes for performance
- Consider recursive CTE support for level calculation
- Include audit fields (created_at, updated_at)
- Support for soft deletes if needed
- Handle circular dependency prevention in device connections

## Expected Deliverables

1. **DDL Statements**: Complete CREATE TABLE statements for new tables
2. **ALTER Statements**: Modifications to existing tables
3. **View Definitions**: SQL for all required views
4. **Sample Data**: Example INSERT statements showing the relationships
5. **Query Examples**: Sample queries demonstrating:
   - Getting complete device map for an asset
   - Getting adjacent level connections
   - Calculating device hierarchy levels
   - Formatting data for React Flow

## Technical Constraints
- Database: PostgreSQL/TimescaleDB
- Must maintain backward compatibility with existing asset and device references
- Performance considerations for large device networks (1000+ devices per asset)
- Support for real-time updates to device connections

Please provide a comprehensive database design that addresses all these requirements with clear explanations for each design decision.

# Initial Prompt
Create for me a prompt to give an llm like Claude that will create a new data structure for asset to devices relationship (many:many) as well as devices to devices relationship within the asset (many:many). The data structure will be used with React Flow to create the device map of an asset and how the devices link with each other.

On top of the edges connection between the devices as nodes, the devices will have levels in the hierarchy where the initial source device is in level 1, then the following connected devices from the source is level 2, then level 3 devices have level 2 as its source and so on. Also, create a view where we can get the devices and the links to populate the device map easily and another view to only get the map within adjecent two levels (such as level 2 and 3 or level 3 and 4).

Furthermore, there is also the link between each asset and the asset_type. The asset_type helps to determine the relevant parameters that is used by that specific asset and should also be incorporated in this data structure. You can make changes to the current asset_types table as well in creating this new data structure.