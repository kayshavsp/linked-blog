# Database Design: Asset-Device Hierarchical Mapping System for React Flow

```sql
-- =====================================================
-- COMPREHENSIVE IOT ENERGY MANAGEMENT DATABASE SCHEMA
-- WITH REACT FLOW INTEGRATION
-- =====================================================

-- 1. JUNCTION TABLES FOR MANY-TO-MANY RELATIONSHIPS
-- =====================================================

-- Asset-Device Many-to-Many Relationship
CREATE TABLE client_data.asset_devices (
    asset_device_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    asset_id UUID NOT NULL,
    device_id TEXT NOT NULL,
    role VARCHAR(100), -- 'primary', 'secondary', 'backup', etc.
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
    
    CONSTRAINT fk_asset_devices_asset 
        FOREIGN KEY (asset_id) REFERENCES client_data.assets(asset_id) ON DELETE CASCADE,
    CONSTRAINT fk_asset_devices_device 
        FOREIGN KEY (device_id) REFERENCES iot_data.redshift_devices(device_id) ON DELETE CASCADE,
    
    -- Ensure unique asset-device combinations
    CONSTRAINT uk_asset_devices_asset_device UNIQUE (asset_id, device_id)
);

-- Device-to-Device Connection Network
CREATE TABLE client_data.device_connections (
    connection_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    source_device_id TEXT NOT NULL,
    target_device_id TEXT NOT NULL,
    connection_type VARCHAR(50) DEFAULT 'electrical', -- 'electrical', 'data', 'control'
    hierarchy_level_source INTEGER NOT NULL,
    hierarchy_level_target INTEGER NOT NULL,
    flow_direction VARCHAR(20) DEFAULT 'bidirectional', -- 'upstream', 'downstream', 'bidirectional'
    connection_strength DECIMAL(5,2) DEFAULT 1.0, -- For weighted connections
    is_active BOOLEAN DEFAULT true,
    metadata JSONB, -- Additional connection properties
    created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
    
    CONSTRAINT fk_device_connections_source 
        FOREIGN KEY (source_device_id) REFERENCES iot_data.redshift_devices(device_id) ON DELETE CASCADE,
    CONSTRAINT fk_device_connections_target 
        FOREIGN KEY (target_device_id) REFERENCES iot_data.redshift_devices(device_id) ON DELETE CASCADE,
    
    -- Prevent self-connections
    CONSTRAINT chk_device_connections_no_self 
        CHECK (source_device_id != target_device_id),
    
    -- Ensure unique directional connections
    CONSTRAINT uk_device_connections_source_target 
        UNIQUE (source_device_id, target_device_id),
    
    -- Validate hierarchy levels
    CONSTRAINT chk_device_connections_hierarchy 
        CHECK (hierarchy_level_source > 0 AND hierarchy_level_target > 0)
);

-- 2. REACT FLOW SPECIFIC TABLES
-- =====================================================

-- React Flow Node Positions and Styling
CREATE TABLE client_data.asset_flow_nodes (
    node_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    device_id TEXT NOT NULL,
    asset_id UUID NOT NULL,
    position_x DECIMAL(10,2) NOT NULL DEFAULT 0,
    position_y DECIMAL(10,2) NOT NULL DEFAULT 0,
    width DECIMAL(8,2) DEFAULT 150,
    height DECIMAL(8,2) DEFAULT 50,
    node_type VARCHAR(50) DEFAULT 'default', -- 'input', 'output', 'default', 'custom'
    style JSONB, -- React Flow node styling
    data JSONB, -- Custom node data
    is_draggable BOOLEAN DEFAULT true,
    is_selectable BOOLEAN DEFAULT true,
    is_connectable BOOLEAN DEFAULT true,
    z_index INTEGER DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
    
    CONSTRAINT fk_asset_flow_nodes_device 
        FOREIGN KEY (device_id) REFERENCES iot_data.redshift_devices(device_id) ON DELETE CASCADE,
    CONSTRAINT fk_asset_flow_nodes_asset 
        FOREIGN KEY (asset_id) REFERENCES client_data.assets(asset_id) ON DELETE CASCADE,
    
    -- Ensure unique device per asset for React Flow
    CONSTRAINT uk_asset_flow_nodes_device_asset 
        UNIQUE (device_id, asset_id)
);

-- React Flow Edge Styling and Behavior
CREATE TABLE client_data.asset_flow_edges (
    edge_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    connection_id UUID NOT NULL,
    asset_id UUID NOT NULL,
    edge_type VARCHAR(50) DEFAULT 'default', -- 'default', 'straight', 'step', 'smoothstep'
    animated BOOLEAN DEFAULT false,
    style JSONB, -- Edge styling (color, width, etc.)
    label VARCHAR(255),
    label_style JSONB,
    marker_end JSONB, -- Arrow markers
    marker_start JSONB,
    z_index INTEGER DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
    
    CONSTRAINT fk_asset_flow_edges_connection 
        FOREIGN KEY (connection_id) REFERENCES client_data.device_connections(connection_id) ON DELETE CASCADE,
    CONSTRAINT fk_asset_flow_edges_asset 
        FOREIGN KEY (asset_id) REFERENCES client_data.assets(asset_id) ON DELETE CASCADE,
    
    -- Ensure unique connection per asset for React Flow
    CONSTRAINT uk_asset_flow_edges_connection_asset 
        UNIQUE (connection_id, asset_id)
);

-- React Flow Node Positions and Styling
CREATE TABLE client_data.location_flow_nodes (
    node_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    device_id TEXT NOT NULL,
    location_id UUID NOT NULL,
    position_x DECIMAL(10,2) NOT NULL DEFAULT 0,
    position_y DECIMAL(10,2) NOT NULL DEFAULT 0,
    width DECIMAL(8,2) DEFAULT 150,
    height DECIMAL(8,2) DEFAULT 50,
    node_type VARCHAR(50) DEFAULT 'default', -- 'input', 'output', 'default', 'custom'
    style JSONB, -- React Flow node styling
    data JSONB, -- Custom node data
    is_draggable BOOLEAN DEFAULT true,
    is_selectable BOOLEAN DEFAULT true,
    is_connectable BOOLEAN DEFAULT true,
    z_index INTEGER DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
    
    CONSTRAINT fk_location_flow_nodes_device 
        FOREIGN KEY (device_id) REFERENCES iot_data.redshift_devices(device_id) ON DELETE CASCADE,
    CONSTRAINT fk_location_flow_nodes_asset 
        FOREIGN KEY (location_id) REFERENCES client_data.locations(location_id) ON DELETE CASCADE,
    
    -- Ensure unique device per asset for React Flow
    CONSTRAINT uk_location_flow_nodes_device_asset 
        UNIQUE (device_id, location_id)
);

-- React Flow Edge Styling and Behavior
CREATE TABLE client_data.location_flow_edges (
    edge_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    connection_id UUID NOT NULL,
    location_id UUID NOT NULL,
    edge_type VARCHAR(50) DEFAULT 'default', -- 'default', 'straight', 'step', 'smoothstep'
    animated BOOLEAN DEFAULT false,
    style JSONB, -- Edge styling (color, width, etc.)
    label VARCHAR(255),
    label_style JSONB,
    marker_end JSONB, -- Arrow markers
    marker_start JSONB,
    z_index INTEGER DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
    
    CONSTRAINT fk_location_flow_edges_connection 
        FOREIGN KEY (connection_id) REFERENCES client_data.device_connections(connection_id) ON DELETE CASCADE,
    CONSTRAINT fk_location_flow_edges_asset 
        FOREIGN KEY (location_id) REFERENCES client_data.locations(location_id) ON DELETE CASCADE,
    
    -- Ensure unique connection per asset for React Flow
    CONSTRAINT uk_location_flow_edges_connection_asset 
        UNIQUE (connection_id, location_id)
);

-- 3. ENHANCED EXISTING TABLES
-- =====================================================

-- Add React Flow and hierarchy columns to redshift_devices
ALTER TABLE iot_data.redshift_devices 
ADD COLUMN IF NOT EXISTS hierarchy_level INTEGER DEFAULT 1,
ADD COLUMN IF NOT EXISTS is_source_device BOOLEAN DEFAULT false,
ADD COLUMN IF NOT EXISTS device_category VARCHAR(100), -- 'meter', 'transformer', 'switch', 'sensor'
ADD COLUMN IF NOT EXISTS react_flow_metadata JSONB; -- Additional React Flow specific data

-- Add computed level validation
ALTER TABLE iot_data.redshift_devices 
ADD CONSTRAINT chk_redshift_devices_hierarchy_level 
    CHECK (hierarchy_level > 0 AND hierarchy_level <= 100);

-- Update level column to be integer instead of varchar
-- (This requires careful migration in production)
-- ALTER TABLE iot_data.redshift_devices ALTER COLUMN level TYPE INTEGER USING level::INTEGER;

-- 4. INDEXES FOR PERFORMANCE
-- =====================================================

-- Asset-Device relationships
CREATE INDEX idx_asset_devices_asset_id ON client_data.asset_devices(asset_id);
CREATE INDEX idx_asset_devices_device_id ON client_data.asset_devices(device_id);
CREATE INDEX idx_asset_devices_active ON client_data.asset_devices(is_active) WHERE is_active = true;

-- Device connections
CREATE INDEX idx_device_connections_source ON client_data.device_connections(source_device_id);
CREATE INDEX idx_device_connections_target ON client_data.device_connections(target_device_id);
CREATE INDEX idx_device_connections_hierarchy_source ON client_data.device_connections(hierarchy_level_source);
CREATE INDEX idx_device_connections_hierarchy_target ON client_data.device_connections(hierarchy_level_target);
CREATE INDEX idx_device_connections_active ON client_data.device_connections(is_active) WHERE is_active = true;

-- React Flow nodes
CREATE INDEX idx_asset_flow_nodes_device_asset ON client_data.asset_flow_nodes(device_id, asset_id);
CREATE INDEX idx_asset_flow_nodes_position ON client_data.asset_flow_nodes(position_x, position_y);

-- React Flow edges
CREATE INDEX idx_asset_flow_edges_connection ON client_data.asset_flow_edges(connection_id);
CREATE INDEX idx_asset_flow_edges_asset ON client_data.asset_flow_edges(asset_id);

-- React Flow nodes
CREATE INDEX idx_location_flow_nodes_device_asset ON client_data.location_flow_nodes(device_id, location_id);
CREATE INDEX idx_location_flow_nodes_position ON client_data.location_flow_nodes(position_x, position_y);

-- React Flow edges
CREATE INDEX idx_location_flow_edges_connection ON client_data.location_flow_edges(connection_id);
CREATE INDEX idx_location_flow_edges_asset ON client_data.location_flow_edges(location_id);


-- Enhanced device indexes
CREATE INDEX idx_redshift_devices_hierarchy_level ON iot_data.redshift_devices(hierarchy_level);
CREATE INDEX idx_redshift_devices_source ON iot_data.redshift_devices(is_source_device) WHERE is_source_device = true;
CREATE INDEX idx_redshift_devices_asset_level ON iot_data.redshift_devices(asset_id, hierarchy_level);

-- 5. VIEWS FOR DATA ACCESS
-- =====================================================

-- Complete Device Map for an Asset
CREATE OR REPLACE VIEW client_data.asset_flow_device_map AS
SELECT 
    a.asset_id,
    a.asset_name,
    d.device_id,
    d.device_name,
    d.hierarchy_level,
    d.is_source_device,
    d.device_category,
    d.is_incomer,
    d.transformer,
    d.phase_type,
    ad.role as asset_role,
    -- React Flow node data
    afn.position_x,
    afn.position_y,
    afn.width,
    afn.height,
    afn.node_type,
    afn.style as node_style,
    afn.data as node_data,
    -- Connection count
    (SELECT COUNT(*) FROM client_data.device_connections dc 
     WHERE dc.source_device_id = d.device_id OR dc.target_device_id = d.device_id) as connection_count
FROM client_data.assets a
JOIN client_data.asset_devices ad ON a.asset_id = ad.asset_id
JOIN iot_data.redshift_devices d ON ad.device_id = d.device_id
LEFT JOIN client_data.asset_flow_nodes afn ON d.device_id = afn.device_id AND a.asset_id = afn.asset_id
WHERE ad.is_active = true AND d.is_active = true;

-- Complete Device Map for a location
CREATE OR REPLACE VIEW client_data.location_flow_device_map AS
SELECT 
    l.location_id,
    l.location_name,
    d.device_id,
    d.device_name,
    d.hierarchy_level,
    d.is_source_device,
    d.device_category,
    d.is_incomer,
    d.transformer,
    d.phase_type,
    ad.role as asset_role,
    -- React Flow node data
    lfn.position_x,
    lfn.position_y,
    lfn.width,
    lfn.height,
    lfn.node_type,
    lfn.style as node_style,
    lfn.data as node_data,
    -- Connection count
    (SELECT COUNT(*) FROM client_data.device_connections dc 
     WHERE dc.source_device_id = d.device_id OR dc.target_device_id = d.device_id) as connection_count
FROM client_data.locations l
JOIN iot_data.redshift_devices d ON l.location_id = d.location_id
LEFT JOIN client_data.location_flow_nodes lfn ON d.device_id = lfn.device_id AND l.location_id = lfn.location_id
WHERE d.is_active = true;

-- Device Connections with React Flow Edge Data
CREATE OR REPLACE VIEW client_data.v_device_connections_map AS
SELECT 
    dc.connection_id,
    dc.source_device_id,
    dc.target_device_id,
    dc.connection_type,
    dc.hierarchy_level_source,
    dc.hierarchy_level_target,
    dc.flow_direction,
    dc.connection_strength,
    -- Source device info
    ds.device_name as source_device_name,
    ds.device_category as source_device_category,
    -- React Flow edge data
    rfe.edge_type,
    rfe.animated,
    rfe.style as edge_style,
    rfe.label as edge_label,
    rfe.label_style,
    rfe.marker_end,
    rfe.marker_start,
    -- Asset context
    rfe.asset_id
FROM client_data.device_connections dc
JOIN iot_data.redshift_devices ds ON dc.source_device_id = ds.device_id
LEFT JOIN client_data.react_flow_edges rfe ON dc.connection_id = rfe.connection_id
WHERE dc.is_active = true;

-- Adjacent Level View (for filtering specific hierarchy levels)
CREATE OR REPLACE VIEW client_data.v_adjacent_level_map AS
SELECT 
    asset_id,
    connection_id,
    source_device_id,
    target_device_id,
    source_device_name,
    target_device_name,
    hierarchy_level_source,
    hierarchy_level_target,
    connection_type,
    flow_direction,
    edge_type,
    animated,
    edge_style,
    edge_label
FROM client_data.v_device_connections_map
WHERE ABS(hierarchy_level_source - hierarchy_level_target) = 1; -- Adjacent levels only

-- React Flow Complete Data View
CREATE OR REPLACE VIEW client_data.v_react_flow_data AS
SELECT 
    a.asset_id,
    a.asset_name,
    -- Nodes array
    jsonb_agg(DISTINCT jsonb_build_object(
        'id', d.device_id,
        'type', COALESCE(rfn.node_type, 'default'),
        'position', jsonb_build_object('x', COALESCE(rfn.position_x, 0), 'y', COALESCE(rfn.position_y, 0)),
        'data', jsonb_build_object(
            'label', d.device_name,
            'deviceId', d.device_id,
            'hierarchyLevel', d.hierarchy_level,
            'isSource', d.is_source_device,
            'category', d.device_category,
            'isIncomer', d.is_incomer,
            'transformer', d.transformer,
            'phaseType', d.phase_type,
            'customData', COALESCE(rfn.data, '{}'::jsonb)
        ),
        'style', COALESCE(rfn.style, '{}'::jsonb),
        'draggable', COALESCE(rfn.is_draggable, true),
        'selectable', COALESCE(rfn.is_selectable, true),
        'connectable', COALESCE(rfn.is_connectable, true)
    )) FILTER (WHERE d.device_id IS NOT NULL) as nodes,
    
    -- Edges array  
    jsonb_agg(DISTINCT jsonb_build_object(
        'id', dc.connection_id::text,
        'source', dc.source_device_id,
        'target', dc.target_device_id,
        'type', COALESCE(rfe.edge_type, 'default'),
        'animated', COALESCE(rfe.animated, false),
        'style', COALESCE(rfe.style, '{}'::jsonb),
        'label', rfe.label,
        'labelStyle', COALESCE(rfe.label_style, '{}'::jsonb),
        'markerEnd', COALESCE(rfe.marker_end, jsonb_build_object('type', 'arrowclosed')),
        'data', jsonb_build_object(
            'connectionType', dc.connection_type,
            'flowDirection', dc.flow_direction,
            'connectionStrength', dc.connection_strength,
            'hierarchyLevels', jsonb_build_object(
                'source', dc.hierarchy_level_source,
                'target', dc.hierarchy_level_target
            )
        )
    )) FILTER (WHERE dc.connection_id IS NOT NULL) as edges,
    
    -- Viewport data
    jsonb_build_object(
        'zoom', COALESCE(rfv.zoom, 1.0),
        'x', COALESCE(rfv.pan_x, 0),
        'y', COALESCE(rfv.pan_y, 0)
    ) as viewport
    
FROM client_data.assets a
LEFT JOIN client_data.asset_devices ad ON a.asset_id = ad.asset_id AND ad.is_active = true
LEFT JOIN iot_data.redshift_devices d ON ad.device_id = d.device_id AND d.is_active = true
LEFT JOIN client_data.react_flow_nodes rfn ON d.device_id = rfn.device_id AND a.asset_id = rfn.asset_id
LEFT JOIN client_data.device_connections dc ON (dc.source_device_id = d.device_id OR dc.target_device_id = d.device_id) 
    AND dc.is_active = true
LEFT JOIN client_data.react_flow_edges rfe ON dc.connection_id = rfe.connection_id AND a.asset_id = rfe.asset_id
LEFT JOIN client_data.react_flow_viewports rfv ON a.asset_id = rfv.asset_id AND rfv.is_default = true
GROUP BY a.asset_id, a.asset_name, rfv.zoom, rfv.pan_x, rfv.pan_y;

-- 6. FUNCTIONS FOR HIERARCHY MANAGEMENT
-- =====================================================

-- Function to calculate and update hierarchy levels based on connections
CREATE OR REPLACE FUNCTION client_data.calculate_device_hierarchy_levels(p_asset_id UUID)
RETURNS TABLE(device_id TEXT, calculated_level INTEGER) AS $$
BEGIN
    -- Reset all levels to NULL for recalculation
    UPDATE iot_data.redshift_devices 
    SET hierarchy_level = NULL 
    WHERE asset_id = p_asset_id;
    
    -- Set source devices to level 1
    UPDATE iot_data.redshift_devices 
    SET hierarchy_level = 1 
    WHERE asset_id = p_asset_id AND is_source_device = true;
    
    -- Iteratively calculate levels based on connections
    FOR i IN 2..100 LOOP -- Prevent infinite loops
        UPDATE iot_data.redshift_devices d
        SET hierarchy_level = i
        WHERE d.asset_id = p_asset_id 
        AND d.hierarchy_level IS NULL
        AND EXISTS (
            SELECT 1 FROM client_data.device_connections dc
            JOIN iot_data.redshift_devices ds ON dc.source_device_id = ds.device_id
            WHERE dc.target_device_id = d.device_id
            AND ds.hierarchy_level = i - 1
            AND dc.is_active = true
        );
        
        -- Exit if no more devices were updated
        EXIT WHEN NOT FOUND;
    END LOOP;
    
    -- Return the calculated levels
    RETURN QUERY
    SELECT d.device_id, d.hierarchy_level
    FROM iot_data.redshift_devices d
    WHERE d.asset_id = p_asset_id
    ORDER BY d.hierarchy_level, d.device_name;
END;
$$ LANGUAGE plpgsql;

-- Function to prevent circular dependencies
CREATE OR REPLACE FUNCTION client_data.check_circular_dependency()
RETURNS TRIGGER AS $$
BEGIN
    -- Check if adding this connection would create a cycle
    IF EXISTS (
        WITH RECURSIVE connection_path AS (
            -- Start from the new target device
            SELECT NEW.target_device_id as device_id, 1 as depth
            UNION ALL
            SELECT dc.target_device_id, cp.depth + 1
            FROM client_data.device_connections dc
            JOIN connection_path cp ON dc.source_device_id = cp.device_id
            WHERE cp.depth < 100 -- Prevent infinite recursion
        )
        SELECT 1 FROM connection_path 
        WHERE device_id = NEW.source_device_id
    ) THEN
        RAISE EXCEPTION 'Connection would create a circular dependency';
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create trigger to prevent circular dependencies
CREATE TRIGGER trg_device_connections_check_circular
    BEFORE INSERT OR UPDATE ON client_data.device_connections
    FOR EACH ROW EXECUTE FUNCTION client_data.check_circular_dependency();

-- 7. SAMPLE DATA INSERTION
-- =====================================================

-- Sample Asset Type (if not exists)
INSERT INTO client_data.asset_types (asset_type_name, description, category, parameters, metadata)
VALUES (
    'Industrial Building',
    'Standard industrial building with electrical distribution',
    'Building',
    ARRAY['power_consumption', 'voltage_levels', 'load_distribution'],
    '{"max_devices": 1000, "supports_hierarchy": true, "react_flow_enabled": true}'::jsonb
) ON CONFLICT (asset_type_name) DO NOTHING;

-- Sample Asset
INSERT INTO client_data.assets (asset_name, location_id, client_id, asset_type_id)
SELECT 
    'Demo Industrial Facility',
    '00000000-0000-0000-0000-000000000001'::uuid, -- Replace with actual location_id
    '00000000-0000-0000-0000-000000000001'::uuid, -- Replace with actual client_id
    asset_type_id
FROM client_data.asset_types 
WHERE asset_type_name = 'Industrial Building'
ON CONFLICT DO NOTHING;

-- Sample Devices with hierarchy
INSERT INTO iot_data.redshift_devices (
    device_name, device_type_id, topic_id, location_id, level, asset_id, 
    hierarchy_level, is_source_device, device_category, is_incomer, metadata
) VALUES 
    ('Main Incomer', 'device_type_1', 'topic_1', '00000000-0000-0000-0000-000000000001'::uuid, '1', 
     (SELECT asset_id FROM client_data.assets WHERE asset_name = 'Demo Industrial Facility' LIMIT 1),
     1, true, 'meter', true, '{"capacity": "1000A", "voltage": "400V"}'::jsonb),
    ('Distribution Panel A', 'device_type_2', 'topic_2', '00000000-0000-0000-0000-000000000001'::uuid, '2',
     (SELECT asset_id FROM client_data.assets WHERE asset_name = 'Demo Industrial Facility' LIMIT 1),
     2, false, 'meter', false, '{"capacity": "400A"}'::jsonb),
    ('Production Line 1', 'device_type_3', 'topic_3', '00000000-0000-0000-0000-000000000001'::uuid, '3',
     (SELECT asset_id FROM client_data.assets WHERE asset_name = 'Demo Industrial Facility' LIMIT 1),
     3, false, 'meter', false, '{"load_type": "motor_load"}'::jsonb)
ON CONFLICT (device_id) DO NOTHING;

-- 8. UTILITY QUERIES
-- =====================================================

-- Query to get complete device map for an asset
/*
SELECT * FROM client_data.v_asset_device_map 
WHERE asset_id = 'your-asset-id'
ORDER BY hierarchy_level, device_name;
*/

-- Query to get React Flow data for an asset
/*
SELECT * FROM client_data.v_react_flow_data 
WHERE asset_id = 'your-asset-id';
*/

-- Query to get adjacent level connections
/*
SELECT * FROM client_data.v_adjacent_level_map 
WHERE asset_id = 'your-asset-id' 
AND hierarchy_level_source = 1 AND hierarchy_level_target = 2;
*/

-- Query to calculate hierarchy levels
/*
SELECT * FROM client_data.calculate_device_hierarchy_levels('your-asset-id');
*/
```

## Key Design Decisions Explained:

### 1. **Separation of Concerns**

- **Business Logic**: Asset-device relationships in `client_data` schema
- **Technical Implementation**: Device connections and React Flow data in `iot_data` schema
- **Clear boundaries** between core IoT functionality and visualization

### 2. **Many-to-Many Relationships**

- **`asset_devices`**: Handles asset ↔ device relationships with roles
- **`device_connections`**: Manages device ↔ device network topology
- **Flexible associations** without breaking existing references

### 3. **React Flow Integration**

- **Dedicated tables** for nodes, edges, and viewport state
- **JSONB storage** for flexible styling and custom data
- **Optimized views** that generate React Flow-compatible JSON

### 4. **Hierarchy Management**

- **Computed levels** based on source devices and connections
- **Circular dependency prevention** through triggers
- **Dynamic recalculation** function for topology changes

### 5. **Performance Considerations**

- **Strategic indexing** on frequently queried columns
- **Materialized views** can be added for large datasets
- **Partitioning options** for time-series device data

### 6. **Backward Compatibility**

- **Existing tables preserved** with minimal changes
- **New columns added** with defaults to avoid breaking changes
- **Migration-friendly** design for production deployment

This schema provides a robust foundation for your IoT energy management system with full React Flow integration while maintaining flexibility for future enhancements.

## Backlink
[[Asset Types]]
