# Database Design: Asset-Device Hierarchical Mapping System for React Flow

## 1. New Tables Design

### 1.1 Asset-Device Junction Table

```sql
-- Junction table for many-to-many relationship between assets and devices
CREATE TABLE iot_data.asset_devices (
    asset_device_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    asset_id UUID NOT NULL REFERENCES client_data.assets(asset_id) ON DELETE CASCADE,
    device_id TEXT NOT NULL REFERENCES iot_data.redshift_devices(device_id) ON DELETE CASCADE,
    is_primary BOOLEAN DEFAULT false, -- Indicates if this is the primary asset for the device
    role VARCHAR(100), -- Role of device in this asset (e.g., 'main_incomer', 'backup_feeder', 'load')
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    UNIQUE(asset_id, device_id)
);

-- Indexes for performance
CREATE INDEX idx_asset_devices_asset_id ON iot_data.asset_devices(asset_id);
CREATE INDEX idx_asset_devices_device_id ON iot_data.asset_devices(device_id);
CREATE INDEX idx_asset_devices_primary ON iot_data.asset_devices(asset_id, is_primary) WHERE is_primary = true;
```

### 1.2 Device Connections Table

```sql
-- Device-to-device connections with hierarchical information
CREATE TABLE iot_data.device_connections (
    connection_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    parent_device_id TEXT NOT NULL REFERENCES iot_data.redshift_devices(device_id) ON DELETE CASCADE,
    child_device_id TEXT NOT NULL REFERENCES iot_data.redshift_devices(device_id) ON DELETE CASCADE,
    asset_id UUID NOT NULL REFERENCES client_data.assets(asset_id) ON DELETE CASCADE,
    connection_type VARCHAR(50) DEFAULT 'electrical', -- 'electrical', 'data', 'control'
    flow_direction VARCHAR(20) DEFAULT 'downstream', -- 'downstream', 'upstream', 'bidirectional'
    hierarchy_level_diff INTEGER DEFAULT 1, -- Level difference (usually 1 for adjacent levels)
    connection_metadata JSONB, -- Additional connection properties (voltage, current rating, etc.)
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    -- Prevent self-connections
    CONSTRAINT no_self_connection CHECK (parent_device_id != child_device_id),
    -- Unique connection per asset
    UNIQUE(parent_device_id, child_device_id, asset_id)
);

-- Indexes for performance
CREATE INDEX idx_device_connections_parent ON iot_data.device_connections(parent_device_id, asset_id);
CREATE INDEX idx_device_connections_child ON iot_data.device_connections(child_device_id, asset_id);
CREATE INDEX idx_device_connections_asset ON iot_data.device_connections(asset_id);
CREATE INDEX idx_device_connections_active ON iot_data.device_connections(asset_id, is_active) WHERE is_active = true;
```

### 1.3 React Flow Layout Data

```sql
-- React Flow specific positioning and layout data
CREATE TABLE iot_data.device_flow_layouts (
    layout_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    asset_id UUID NOT NULL REFERENCES client_data.assets(asset_id) ON DELETE CASCADE,
    device_id TEXT NOT NULL REFERENCES iot_data.redshift_devices(device_id) ON DELETE CASCADE,
    position_x NUMERIC(10,2) DEFAULT 0,
    position_y NUMERIC(10,2) DEFAULT 0,
    node_width NUMERIC(8,2) DEFAULT 150,
    node_height NUMERIC(8,2) DEFAULT 60,
    node_type VARCHAR(50) DEFAULT 'default', -- 'input', 'output', 'default', 'custom'
    node_style JSONB, -- Custom styling for React Flow
    is_hidden BOOLEAN DEFAULT false,
    z_index INTEGER DEFAULT 1,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    UNIQUE(asset_id, device_id)
);

-- Indexes
CREATE INDEX idx_device_flow_layouts_asset ON iot_data.device_flow_layouts(asset_id);
CREATE INDEX idx_device_flow_layouts_visible ON iot_data.device_flow_layouts(asset_id, is_hidden) WHERE is_hidden = false;
```
### 1.4 Computed Hierarchy Levels Cache

```sql
-- Cache table for computed hierarchy levels to improve performance
CREATE TABLE iot_data.device_hierarchy_cache (
    cache_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    asset_id UUID NOT NULL REFERENCES client_data.assets(asset_id) ON DELETE CASCADE,
    device_id TEXT NOT NULL REFERENCES iot_data.redshift_devices(device_id) ON DELETE CASCADE,
    computed_level INTEGER NOT NULL,
    path_to_root TEXT[], -- Array of device_ids showing path to root
    is_root BOOLEAN DEFAULT false,
    is_leaf BOOLEAN DEFAULT false,
    last_computed TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    UNIQUE(asset_id, device_id)
);

-- Indexes
CREATE INDEX idx_hierarchy_cache_asset_level ON iot_data.device_hierarchy_cache(asset_id, computed_level);
CREATE INDEX idx_hierarchy_cache_roots ON iot_data.device_hierarchy_cache(asset_id, is_root) WHERE is_root = true;
```
## 2. Enhanced Existing Tables

### 2.1 Enhance Asset Types for Better React Flow Integration

```sql
-- Add React Flow specific metadata to asset_types
ALTER TABLE iot_data.asset_types 
ADD COLUMN default_layout_config JSONB,
ADD COLUMN flow_direction VARCHAR(20) DEFAULT 'top-bottom', -- 'top-bottom', 'left-right', 'bottom-top', 'right-left'
ADD COLUMN auto_layout_enabled BOOLEAN DEFAULT true;

-- Update existing records with default values
UPDATE iot_data.asset_types 
SET default_layout_config = '{
    "nodeSpacing": {"x": 200, "y": 150},
    "levelSpacing": 200,
    "nodeDefaults": {
        "width": 150,
        "height": 60,
        "style": {"border": "1px solid #ddd", "borderRadius": "8px"}
    }
}'::jsonb
WHERE default_layout_config IS NULL;
```
## 3. Views for React Flow Integration

### 3.1 Complete Device Map View

```sql
-- Complete device topology for an asset
CREATE OR REPLACE VIEW iot_data.v_asset_device_map AS
SELECT 
    a.asset_id,
    a.asset_name,
    d.device_id,
    d.device_name,
    d.device_type_id,
    dt.type_name as device_type_name,
    d.level as original_level,
    dhc.computed_level,
    dhc.is_root,
    dhc.is_leaf,
    dhc.path_to_root,
    ad.role as device_role,
    ad.is_primary,
    dfl.position_x,
    dfl.position_y,
    dfl.node_width,
    dfl.node_height,
    dfl.node_type,
    dfl.node_style,
    d.is_incomer,
    d.transformer,
    d.phase_type,
    d.is_active,
    d.metadata as device_metadata
FROM client_data.assets a
JOIN iot_data.asset_devices ad ON a.asset_id = ad.asset_id
JOIN iot_data.redshift_devices d ON ad.device_id = d.device_id
LEFT JOIN iot_data.device_types dt ON d.device_type_id::uuid = dt.device_type_id
LEFT JOIN iot_data.device_hierarchy_cache dhc ON a.asset_id = dhc.asset_id AND d.device_id = dhc.device_id
LEFT JOIN iot_data.device_flow_layouts dfl ON a.asset_id = dfl.asset_id AND d.device_id = dfl.device_id
WHERE d.is_active = true;
```
### 3.2 Device Connections View

```sql
-- Device connections with hierarchy information
CREATE OR REPLACE VIEW iot_data.v_device_connections AS
SELECT 
    dc.connection_id,
    dc.asset_id,
    dc.parent_device_id,
    pd.device_name as parent_device_name,
    dc.child_device_id,
    cd.device_name as child_device_name,
    dc.connection_type,
    dc.flow_direction,
    dc.hierarchy_level_diff,
    dc.connection_metadata,
    phc.computed_level as parent_level,
    chc.computed_level as child_level,
    dc.is_active
FROM iot_data.device_connections dc
JOIN iot_data.redshift_devices pd ON dc.parent_device_id = pd.device_id
JOIN iot_data.redshift_devices cd ON dc.child_device_id = cd.device_id
LEFT JOIN iot_data.device_hierarchy_cache phc ON dc.asset_id = phc.asset_id AND dc.parent_device_id = phc.device_id
LEFT JOIN iot_data.device_hierarchy_cache chc ON dc.asset_id = chc.asset_id AND dc.child_device_id = chc.device_id
WHERE dc.is_active = true;
```
### 3.3 Adjacent Level View

```sql
-- Filtered view for adjacent levels
CREATE OR REPLACE VIEW iot_data.v_adjacent_level_map AS
SELECT 
    vdc.*,
    CASE 
        WHEN vdc.parent_level IS NOT NULL AND vdc.child_level IS NOT NULL 
        THEN ABS(vdc.parent_level - vdc.child_level) 
        ELSE NULL 
    END as actual_level_diff
FROM iot_data.v_device_connections vdc
WHERE vdc.parent_level IS NOT NULL 
  AND vdc.child_level IS NOT NULL
  AND ABS(vdc.parent_level - vdc.child_level) <= 1; -- Adjacent levels only
```
### 3.4 React Flow Data View

```sql
-- Formatted data ready for React Flow consumption
CREATE OR REPLACE VIEW iot_data.v_react_flow_data AS
WITH nodes AS (
    SELECT 
        vadm.asset_id,
        jsonb_build_object(
            'id', vadm.device_id,
            'type', COALESCE(vadm.node_type, 'default'),
            'position', jsonb_build_object(
                'x', COALESCE(vadm.position_x, 0),
                'y', COALESCE(vadm.position_y, 0)
            ),
            'data', jsonb_build_object(
                'label', vadm.device_name,
                'deviceType', vadm.device_type_name,
                'level', vadm.computed_level,
                'isRoot', vadm.is_root,
                'isLeaf', vadm.is_leaf,
                'isIncomer', vadm.is_incomer,
                'isTransformer', vadm.transformer,
                'phaseType', vadm.phase_type,
                'role', vadm.device_role,
                'isPrimary', vadm.is_primary,
                'metadata', vadm.device_metadata
            ),
            'style', COALESCE(vadm.node_style, '{}'::jsonb),
            'width', COALESCE(vadm.node_width, 150),
            'height', COALESCE(vadm.node_height, 60)
        ) as node_data
    FROM iot_data.v_asset_device_map vadm
),
edges AS (
    SELECT 
        vdc.asset_id,
        jsonb_build_object(
            'id', vdc.connection_id::text,
            'source', vdc.parent_device_id,
            'target', vdc.child_device_id,
            'type', CASE 
                WHEN vdc.connection_type = 'electrical' THEN 'smoothstep'
                ELSE 'default'
            END,
            'animated', CASE WHEN vdc.flow_direction = 'bidirectional' THEN true ELSE false END,
            'data', jsonb_build_object(
                'connectionType', vdc.connection_type,
                'flowDirection', vdc.flow_direction,
                'levelDiff', vdc.hierarchy_level_diff,
                'metadata', vdc.connection_metadata
            ),
            'style', jsonb_build_object(
                'stroke', CASE 
                    WHEN vdc.connection_type = 'electrical' THEN '#ff6b6b'
                    WHEN vdc.connection_type = 'data' THEN '#4ecdc4'
                    ELSE '#95a5a6'
                END,
                'strokeWidth', 2
            )
        ) as edge_data
    FROM iot_data.v_device_connections vdc
)
SELECT 
    asset_id,
    'nodes' as data_type,
    jsonb_agg(node_data) as flow_data
FROM nodes
GROUP BY asset_id

UNION ALL

SELECT 
    asset_id,
    'edges' as data_type,
    jsonb_agg(edge_data) as flow_data
FROM edges
GROUP BY asset_id;
```

## 4. Functions for Hierarchy Management

### 4.1 Function to Compute Device Hierarchy Levels

```sql
-- Function to compute and cache device hierarchy levels
CREATE OR REPLACE FUNCTION iot_data.compute_device_hierarchy(p_asset_id UUID)
RETURNS VOID AS $$
DECLARE
    rec RECORD;
BEGIN
    -- Clear existing cache for this asset
    DELETE FROM iot_data.device_hierarchy_cache WHERE asset_id = p_asset_id;
    
    -- Insert computed hierarchy using recursive CTE
    WITH RECURSIVE device_hierarchy AS (
        -- Base case: root devices (incomers or devices with no parents)
        SELECT 
            ad.device_id,
            1 as level,
            ARRAY[ad.device_id] as path,
            true as is_root,
            false as is_leaf
        FROM iot_data.asset_devices ad
        JOIN iot_data.redshift_devices d ON ad.device_id = d.device_id
        WHERE ad.asset_id = p_asset_id
          AND d.is_active = true
          AND (d.is_incomer = true OR NOT EXISTS (
              SELECT 1 FROM iot_data.device_connections dc 
              WHERE dc.child_device_id = ad.device_id 
                AND dc.asset_id = p_asset_id 
                AND dc.is_active = true
          ))
        
        UNION ALL
        
        -- Recursive case: child devices
        SELECT 
            dc.child_device_id,
            dh.level + 1,
            dh.path || dc.child_device_id,
            false as is_root,
            false as is_leaf
        FROM device_hierarchy dh
        JOIN iot_data.device_connections dc ON dh.device_id = dc.parent_device_id
        WHERE dc.asset_id = p_asset_id 
          AND dc.is_active = true
          AND NOT (dc.child_device_id = ANY(dh.path)) -- Prevent cycles
    )
    INSERT INTO iot_data.device_hierarchy_cache (
        asset_id, device_id, computed_level, path_to_root, is_root, is_leaf
    )
    SELECT 
        p_asset_id,
        dh.device_id,
        dh.level,
        dh.path,
        dh.is_root,
        NOT EXISTS (
            SELECT 1 FROM iot_data.device_connections dc 
            WHERE dc.parent_device_id = dh.device_id 
              AND dc.asset_id = p_asset_id 
              AND dc.is_active = true
        ) as is_leaf
    FROM device_hierarchy dh;
    
    -- Update the cache timestamp
    UPDATE iot_data.device_hierarchy_cache 
    SET last_computed = NOW() 
    WHERE asset_id = p_asset_id;
END;
$$ LANGUAGE plpgsql;
```
### 4.2 Function for Auto-Layout Generation

```sql
-- Function to generate automatic layout for React Flow
CREATE OR REPLACE FUNCTION iot_data.generate_auto_layout(p_asset_id UUID)
RETURNS VOID AS $$
DECLARE
    layout_config JSONB;
    node_spacing_x NUMERIC := 200;
    node_spacing_y NUMERIC := 150;
    level_spacing NUMERIC := 200;
    rec RECORD;
    level_counts JSONB := '{}';
    level_positions JSONB := '{}';
BEGIN
    -- Get layout configuration from asset type
    SELECT at.default_layout_config INTO layout_config
    FROM client_data.assets a
    JOIN iot_data.asset_types at ON a.asset_type_id = at.asset_type_id
    WHERE a.asset_id = p_asset_id;
    
    -- Extract spacing values from config
    IF layout_config IS NOT NULL THEN
        node_spacing_x := COALESCE((layout_config->'nodeSpacing'->>'x')::numeric, 200);
        node_spacing_y := COALESCE((layout_config->'nodeSpacing'->>'y')::numeric, 150);
        level_spacing := COALESCE((layout_config->>'levelSpacing')::numeric, 200);
    END IF;
    
    -- Count devices per level
    FOR rec IN 
        SELECT computed_level, COUNT(*) as device_count
        FROM iot_data.device_hierarchy_cache 
        WHERE asset_id = p_asset_id
        GROUP BY computed_level
    LOOP
        level_counts := jsonb_set(level_counts, ARRAY[rec.computed_level::text], to_jsonb(rec.device_count));
        level_positions := jsonb_set(level_positions, ARRAY[rec.computed_level::text], to_jsonb(0));
    END LOOP;
    
    -- Generate positions for each device
    FOR rec IN
        SELECT device_id, computed_level
        FROM iot_data.device_hierarchy_cache
        WHERE asset_id = p_asset_id
        ORDER BY computed_level, device_id
    LOOP
        DECLARE
            current_pos INTEGER := (level_positions->>rec.computed_level::text)::integer;
            total_devices INTEGER := (level_counts->>rec.computed_level::text)::integer;
            start_x NUMERIC := -(total_devices - 1) * node_spacing_x / 2;
            pos_x NUMERIC := start_x + current_pos * node_spacing_x;
            pos_y NUMERIC := (rec.computed_level - 1) * level_spacing;
        BEGIN
            -- Insert or update layout position
            INSERT INTO iot_data.device_flow_layouts (
                asset_id, device_id, position_x, position_y
            ) VALUES (
                p_asset_id, rec.device_id, pos_x, pos_y
            )
            ON CONFLICT (asset_id, device_id) 
            DO UPDATE SET 
                position_x = EXCLUDED.position_x,
                position_y = EXCLUDED.position_y,
                updated_at = NOW();
            
            -- Update position counter
            level_positions := jsonb_set(level_positions, ARRAY[rec.computed_level::text], to_jsonb(current_pos + 1));
        END;
    END LOOP;
END;
$$ LANGUAGE plpgsql;
```

## 5. Triggers for Automatic Updates

### 5.1 Trigger to Update Hierarchy Cache

```sql
-- Function to trigger hierarchy recalculation
CREATE OR REPLACE FUNCTION iot_data.trigger_hierarchy_update()
RETURNS TRIGGER AS $$
BEGIN
    -- Schedule hierarchy recalculation for affected asset(s)
    IF TG_OP = 'DELETE' THEN
        PERFORM iot_data.compute_device_hierarchy(OLD.asset_id);
        RETURN OLD;
    ELSE
        PERFORM iot_data.compute_device_hierarchy(NEW.asset_id);
        RETURN NEW;
    END IF;
END;
$$ LANGUAGE plpgsql;

-- Create triggers
CREATE TRIGGER trg_device_connections_hierarchy_update
    AFTER INSERT OR UPDATE OR DELETE ON iot_data.device_connections
    FOR EACH ROW EXECUTE FUNCTION iot_data.trigger_hierarchy_update();

CREATE TRIGGER trg_asset_devices_hierarchy_update
    AFTER INSERT OR UPDATE OR DELETE ON iot_data.asset_devices
    FOR EACH ROW EXECUTE FUNCTION iot_data.trigger_hierarchy_update();
```

### 5.2 Trigger for Updated Timestamps

```sql
-- Function to update timestamps
CREATE OR REPLACE FUNCTION iot_data.update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Apply to relevant tables
CREATE TRIGGER trg_asset_devices_updated_at
    BEFORE UPDATE ON iot_data.asset_devices
    FOR EACH ROW EXECUTE FUNCTION iot_data.update_updated_at();

CREATE TRIGGER trg_device_connections_updated_at
    BEFORE UPDATE ON iot_data.device_connections
    FOR EACH ROW EXECUTE FUNCTION iot_data.update_updated_at();

CREATE TRIGGER trg_device_flow_layouts_updated_at
    BEFORE UPDATE ON iot_data.device_flow_layouts
    FOR EACH ROW EXECUTE FUNCTION iot_data.update_updated_at();
```

## 6. Sample Data and Usage Examples

### 6.1 Sample Data Insertion

```sql
-- Sample asset type with React Flow configuration
INSERT INTO iot_data.asset_types (asset_type_name, description, category, default_layout_config) VALUES
('Industrial Facility', 'Large industrial facility with complex electrical hierarchy', 'Industrial', 
'{
    "nodeSpacing": {"x": 250, "y": 180},
    "levelSpacing": 220,
    "nodeDefaults": {
        "width": 180,
        "height": 80,
        "style": {"border": "2px solid #3498db", "borderRadius": "12px", "backgroundColor": "#ecf0f1"}
    }
}'::jsonb);

-- Sample devices and connections
-- (Assuming you have existing asset and device data)

-- Create asset-device relationships
INSERT INTO iot_data.asset_devices (asset_id, device_id, is_primary, role) VALUES
('your-asset-id', 'main-incomer-device-id', true, 'main_incomer'),
('your-asset-id', 'transformer-device-id', false, 'step_down_transformer'),
('your-asset-id', 'panel-device-id', false, 'distribution_panel');

-- Create device connections
INSERT INTO iot_data.device_connections (parent_device_id, child_device_id, asset_id, connection_type, flow_direction) VALUES
('main-incomer-device-id', 'transformer-device-id', 'your-asset-id', 'electrical', 'downstream'),
('transformer-device-id', 'panel-device-id', 'your-asset-id', 'electrical', 'downstream');

-- Compute hierarchy and generate layout
SELECT iot_data.compute_device_hierarchy('your-asset-id');
SELECT iot_data.generate_auto_layout('your-asset-id');
```

### 6.2 Query Examples

```sql
-- Get complete device map for an asset
SELECT * FROM iot_data.v_asset_device_map 
WHERE asset_id = 'your-asset-id'
ORDER BY computed_level, device_name;

-- Get React Flow formatted data
SELECT 
    data_type,
    flow_data
FROM iot_data.v_react_flow_data 
WHERE asset_id = 'your-asset-id';

-- Get devices at specific level
SELECT * FROM iot_data.v_asset_device_map 
WHERE asset_id = 'your-asset-id' AND computed_level = 2;

-- Get adjacent level connections (Level 2-3)
SELECT * FROM iot_data.v_adjacent_level_map 
WHERE asset_id = 'your-asset-id' 
  AND parent_level = 2 AND child_level = 3;

-- Find root devices (incomers)
SELECT * FROM iot_data.v_asset_device_map 
WHERE asset_id = 'your-asset-id' AND is_root = true;

-- Get device path to root
SELECT 
    device_name,
    computed_level,
    path_to_root
FROM iot_data.v_asset_device_map 
WHERE asset_id = 'your-asset-id' 
  AND device_id = 'specific-device-id';
```

## 7. Performance Considerations

### 7.1 Additional Indexes

```sql
-- Composite indexes for common query patterns
CREATE INDEX idx_asset_devices_composite ON iot_data.asset_devices(asset_id, is_primary, device_id);
CREATE INDEX idx_device_connections_hierarchy ON iot_data.device_connections(asset_id, parent_device_id, child_device_id);
CREATE INDEX idx_hierarchy_cache_composite ON iot_data.device_hierarchy_cache(asset_id, computed_level, is_root, is_leaf);

-- Partial indexes for active records
CREATE INDEX idx_device_connections_active_asset ON iot_data.device_connections(asset_id, parent_device_id) WHERE is_active = true;
CREATE INDEX idx_redshift_devices_active_asset ON iot_data.redshift_devices(asset_id) WHERE is_active = true;
```
### 7.2 Materialized Views for Large Datasets

```sql
-- Materialized view for frequently accessed React Flow data
CREATE MATERIALIZED VIEW iot_data.mv_react_flow_data AS
SELECT * FROM iot_data.v_react_flow_data;

-- Create index on materialized view
CREATE INDEX idx_mv_react_flow_data_asset ON iot_data.mv_react_flow_data(asset_id);

-- Function to refresh materialized view
CREATE OR REPLACE FUNCTION iot_data.refresh_react_flow_data(p_asset_id UUID DEFAULT NULL)
RETURNS VOID AS $$
BEGIN
    IF p_asset_id IS NOT NULL THEN
        -- Refresh specific asset data (requires PostgreSQL 14+)
        -- For now, refresh entire view
        REFRESH MATERIALIZED VIEW iot_data.mv_react_flow_data;
    ELSE
        REFRESH MATERIALIZED VIEW iot_data.mv_react_flow_data;
    END IF;
END;
$$ LANGUAGE plpgsql;
```

This comprehensive database design provides:

1. **Flexible many-to-many relationships** between assets and devices
2. **Hierarchical device connections** with cycle prevention
3. **React Flow integration** with positioning and styling data
4. **Performance optimization** through caching and indexing
5. **Automatic layout generation** based on asset type configurations
6. **Real-time updates** through triggers
7. **Backward compatibility** with existing schema

The design supports complex device topologies, efficient querying for React Flow visualization, and maintains data integrity while providing the flexibility needed for your IoT energy management system.

## Backlink
[[Asset Types]]
