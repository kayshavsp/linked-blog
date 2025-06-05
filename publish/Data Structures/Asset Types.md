
SQL Queries
```sql
-- =====================================================
-- DYNAMIC EXTENSIBLE SCHEMA FOR LSTM FORECASTING
-- =====================================================
-- =================
-- 1. ASSET TYPES TABLE (Updated to use parameter_id)
-- =================
-- This table defines different types of assets and their associated parameters

CREATE TABLE iot_data.asset_types (
	asset_type_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	 asset_type_name VARCHAR(255) NOT NULL UNIQUE,
	 description TEXT,
	 category VARCHAR(100), -- For grouping similar asset types (e.g., 'energy', 'hvac', 'industrial')
	 parameters TEXT[], -- TEXT array for storing parameter_ids from timescale_parameters
	 metadata JSONB, -- For extensibility to store additional properties
	 created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
	 updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Add indexes for performance
CREATE INDEX idx_asset_types_name ON iot_data.asset_types(asset_type_name);
CREATE INDEX idx_asset_types_category ON iot_data.asset_types(category);
CREATE INDEX idx_asset_types_parameters ON iot_data.asset_types USING GIN(parameters);

-- =================
-- 2. DEVICE-ASSET TYPES JUNCTION TABLE (No changes needed)
-- =================
-- Many-to-many relationship between devices and asset types

CREATE TABLE iot_data.device_asset_types (
	 device_asset_type_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
	 device_id TEXT NOT NULL REFERENCES iot_data.redshift_devices(device_id),
	 asset_type_id UUID NOT NULL REFERENCES iot_data.asset_types(asset_type_id),
	 priority INTEGER DEFAULT 1, -- For cases where order matters
	 is_primary BOOLEAN DEFAULT false, -- Mark primary asset type for the device
	 created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
	 updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
	 metadata JSONB, -- For additional context per assignment
	 -- Constraints
	 UNIQUE(device_id, asset_type_id),
	 CHECK (priority > 0)
);

-- Add indexes for performance
CREATE INDEX idx_device_asset_types_device ON iot_data.device_asset_types(device_id);
CREATE INDEX idx_device_asset_types_asset ON iot_data.device_asset_types(asset_type_id);
CREATE INDEX idx_device_asset_types_primary ON iot_data.device_asset_types(is_primary);
CREATE INDEX idx_device_asset_types_priority ON iot_data.device_asset_types(priority);

-- =================
-- 3. SAMPLE DATA INSERTION (Updated to use parameter_ids)
-- =================
-- Insert sample asset types with their associated parameter_ids
-- Note: Replace these example parameter_ids with actual IDs from your timescale_parameters table

INSERT INTO iot_data.asset_types (asset_type_name, description, category, parameters, metadata) VALUES
(
	 'Energy Incomers',
	 'Primary energy measurement devices for building distribution panels',
	 'energy',
	 ARRAY[
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'energy'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'total_current'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'voltage_12'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'voltage_23'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'voltage_31'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'current_phase_1'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'current_phase_2'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'current_phase_3')
	 ],
	 '{"forecasting_horizon": "24h", "sampling_frequency": "15min", "asset_features": ["energy", "total_current", "voltage_12", "voltage_23", "voltage_31", "current_phase_1", "current_phase_2", "current_phase_3"]}'
),
(
	 'Chiller',
	 'Heating, Ventilation, and Air Conditioning control and monitoring',
	 'hvac',
	 ARRAY[
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'energy'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'total_current'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'voltage_12'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'voltage_23'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'voltage_31'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'current_phase_1'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'current_phase_2'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'current_phase_3')
	 ],
	 '{"forecasting_horizon": "12h", "sampling_frequency": "5min", "asset_features": ["energy", "total_current", "voltage_12", "voltage_23", "voltage_31", "current_phase_1", "current_phase_2", "current_phase_3"]}'
),
(
	 'Pump',
	 'Water Pump, Fire Pump control and monitoring',
	 'flow',
	 ARRAY[
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'energy'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'total_current'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'voltage_12'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'voltage_23'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'voltage_31'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'current_phase_1'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'current_phase_2'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'current_phase_3'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'water_flowrate'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'water_totalizer')
	 ],
	 '{"forecasting_horizon": "12h", "sampling_frequency": "5min", "asset_features": ["energy", "total_current", "voltage_12", "voltage_23", "voltage_31", "current_phase_1", "current_phase_2", "current_phase_3", "water_flowrate", "water_totalizer"]}'
),
(
	 'Air Compressor',
	 'Heating, Ventilation, and Air Conditioning control and monitoring',
	 'hvac',
	 ARRAY[
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'energy'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'total_current')
	 ],
	 '{"forecasting_horizon": "12h", "sampling_frequency": "5min", "asset_features": ["energy", "total_current"]}'
),
(
	 'Generic Machine',
	 'Manufacturing machines control and monitoring',
	 'industrial',
	 ARRAY[
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'energy'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'total_current')
	 ],
	 '{"forecasting_horizon": "12h", "sampling_frequency": "5min", "asset_features": ["energy", "total_current"]}'
),
(
	 'Motor',
	 'Variable frequency drives and motor control systems',
	 'industrial',
	 ARRAY[
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'total_current'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'current_phase_1'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'current_phase_2'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'current_phase_3')
	 ],
	 '{"forecasting_horizon": "8h", "sampling_frequency": "1min", "asset_features": ["total_current", "current_phase_1", "current_phase_2", "current_phase_3"]}'
),
(
	 'Solar',
	 'Solar photovoltaic energy generation systems',
	 'renewable',
	 ARRAY[
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'energy_AIE'),
	 	 (SELECT parameter_id FROM iot_data.timescale_parameters WHERE parameter_name = 'energy_AEE')
	 ],
	 '{"forecasting_horizon": "48h", "sampling_frequency": "15min", "asset_features": ["energy_AIE", "energy_AEE"]}'
);

-- Link sample devices to asset types (No changes needed)
INSERT INTO iot_data.device_asset_types (device_id, asset_type_id, priority, is_primary, metadata)
SELECT
	 rd.device_id,
	 at.asset_type_id,
	 1 as priority,
	 true as is_primary,
	 jsonb_build_object(
	 	 'assignment_method', 'manual',
	 	 'assigned_by', 'system_admin',
	 	 'device_location', rd.location_id
	 ) as metadata
FROM iot_data.redshift_devices rd
CROSS JOIN iot_data.asset_types at
WHERE rd.device_id IN (
	 SELECT device_id FROM iot_data.redshift_devices
	 WHERE meter_name LIKE '%Chiller%'
	 ORDER BY created_at DESC
	 LIMIT 10
)
AND at.asset_type_name = 'Chiller';

-- =================
-- 4. CORE VIEWS FOR LSTM DATA PIPELINE
-- =================

-- View 1: Get all parameters for a specific device with metadata
CREATE OR REPLACE VIEW iot_data.device_asset_parameters AS
WITH device_parameters AS (
	 SELECT
	 	 d.device_id,
	 	 d.device_name,
	 	 d.location_id,
	 	 l.client_id,
	 	 l.location_name,
	 	 dat.asset_type_id,
	 	 at.asset_type_name,
	 	 at.category as asset_category,
	 	 dat.is_primary,
	 	 dat.priority,
	 	 param.parameter_id,
	 	 at.metadata as asset_type_metadata,
	 	 dat.metadata as assignment_metadata
	 FROM iot_data.redshift_devices d
	 JOIN client_data.locations l ON d.location_id = l.location_id
	 JOIN iot_data.device_asset_types dat ON d.device_id = dat.device_id
	 JOIN iot_data.asset_types at ON dat.asset_type_id = at.asset_type_id
	 CROSS JOIN LATERAL UNNEST(at.parameters) AS param(parameter_id)
	 WHERE d.is_active = true
)
SELECT
	 dp.device_id,
	 dp.device_name,
	 dp.location_id,
	 dp.client_id,
	 dp.location_name,
	 dp.asset_type_id,
	 dp.asset_type_name,
	 dp.asset_category,
	 dp.is_primary,
	 dp.priority,
	 dp.parameter_id,
	 tp.parameter_name,
	 tp.attribute,
	 tp.data_type,
	 tp.unit,
	 tp.description as parameter_description,
	 dp.asset_type_metadata,
	 dp.assignment_metadata
FROM device_parameters dp
JOIN iot_data.timescale_parameters tp ON dp.parameter_id = tp.parameter_id;

-- View 2: Device summary with primary asset type and parameter count
CREATE OR REPLACE VIEW iot_data.device_asset_summary AS
WITH device_asset_stats AS (
	 SELECT
	 	 d.device_id,
	 	 d.device_name,
	 	 d.location_id,
	 	 l.client_id,
	 	 l.location_name,
	 	 dat.asset_type_id,
	 	 at.asset_type_name,
	 	 at.category,
	 	 dat.is_primary,
	 	 array_length(at.parameters, 1) as parameter_count,
	 	 d.updated_at
	 FROM iot_data.redshift_devices d
	 JOIN client_data.locations l ON d.location_id = l.location_id
	 JOIN iot_data.device_asset_types dat ON d.device_id = dat.device_id
	 JOIN iot_data.asset_types at ON dat.asset_type_id = at.asset_type_id
	 WHERE d.is_active = true
)
SELECT
	 device_id,
	 device_name,
	 location_id,
	 client_id,
	 location_name,
	 COUNT(DISTINCT asset_type_id) as asset_type_count,
	 MAX(CASE WHEN is_primary THEN asset_type_name END) as primary_asset_type,
	 MAX(CASE WHEN is_primary THEN category END) as primary_category,
	 STRING_AGG(DISTINCT asset_type_name, ', ' ORDER BY asset_type_name) as all_asset_types,
	 SUM(parameter_count) as total_parameters,
	 MAX(CASE WHEN is_primary THEN parameter_count END) as primary_parameters_count,
	 MAX(updated_at) as last_updated
FROM device_asset_stats
GROUP BY device_id, device_name, location_id, client_id, location_name;

-- View 3: LSTM-specific parameters by client
CREATE OR REPLACE VIEW iot_data.asset_parameters_by_client AS
WITH client_device_parameters AS (
	 SELECT
	 	 l.client_id,
	 	 l.location_name,
	 	 at.category as asset_category,
	 	 at.asset_type_name,
	 	 d.device_id,
	 	 tp.parameter_name,
	 	 tp.data_type,
	 	 tp.unit
	 FROM client_data.locations l
	 JOIN iot_data.redshift_devices d ON l.location_id = d.location_id
	 JOIN iot_data.device_asset_types dat ON d.device_id = dat.device_id
	 JOIN iot_data.asset_types at ON dat.asset_type_id = at.asset_type_id
	 CROSS JOIN LATERAL UNNEST(at.parameters) AS param(parameter_id)
	 JOIN iot_data.timescale_parameters tp ON param.parameter_id = tp.parameter_id
	 WHERE d.is_active = true
)
SELECT
	 client_id,
	 location_name,
	 asset_category,
	 asset_type_name,
	 COUNT(DISTINCT device_id) as device_count,
	 ARRAY_AGG(DISTINCT parameter_name ORDER BY parameter_name) as all_parameters,
	 jsonb_object_agg(
	 	 parameter_name,
	 	 jsonb_build_object(
	 	 	 'data_type', data_type,
	 	 	 'unit', unit
	 	 )
	 ) as parameter_details
FROM client_device_parameters
GROUP BY client_id, location_name, asset_category, asset_type_name;

-- View 4: Get device-parameter relationships without LSTM complexity
CREATE OR REPLACE VIEW iot_data.device_parameters_simple AS
WITH device_params AS (
	 SELECT
	 	 d.device_id,
	 	 d.device_name,
	 	 l.client_id,
	 	 at.asset_type_name,
	 	 dat.is_primary,
	 	 param.parameter_id
	 FROM iot_data.redshift_devices d
	 JOIN client_data.locations l ON d.location_id = l.location_id
	 JOIN iot_data.device_asset_types dat ON d.device_id = dat.device_id
	 JOIN iot_data.asset_types at ON dat.asset_type_id = at.asset_type_id
	 CROSS JOIN LATERAL UNNEST(at.parameters) AS param(parameter_id)
	 WHERE d.is_active = true
)
SELECT
	 dp.device_id,
	 dp.device_name,
	 dp.client_id,
	 dp.asset_type_name,
	 dp.is_primary,
	 dp.parameter_id,
	 tp.parameter_name,
	 tp.data_type,
	 tp.unit
FROM device_params dp
JOIN iot_data.timescale_parameters tp ON dp.parameter_id = tp.parameter_id;

-- View 5: Combines asset types, devices, parameters, and measurements for LSTM dataset. 
CREATE OR REPLACE VIEW iot_data.asset_measurements_view AS
WITH asset_parameter_expanded AS (
    -- Unnest the parameters array to get individual parameter_ids for each asset type
    SELECT 
        at.asset_type_id,
        at.asset_type_name,
        at.description AS asset_description,
        at.category,
        unnest(at.parameters) AS parameter_id,
        at.metadata AS asset_metadata,
        at.created_at AS asset_created_at,
        at.updated_at AS asset_updated_at
    FROM iot_data.asset_types at
    WHERE at.parameters IS NOT NULL 
      AND array_length(at.parameters, 1) > 0
),
asset_device_parameters AS (
    -- Join asset types with their associated devices and parameters
    SELECT 
        ape.asset_type_id,
        ape.asset_type_name,
        ape.asset_description,
        ape.category,
        ape.parameter_id,
        ape.asset_metadata,
        dat.device_id,
        dat.priority,
        dat.is_primary,
        dat.created_at AS device_asset_created_at,
        dat.metadata AS device_asset_metadata
    FROM asset_parameter_expanded ape
    INNER JOIN iot_data.device_asset_types dat 
        ON ape.asset_type_id = dat.asset_type_id
)
-- Final selection with measurement data
SELECT 
    adp.asset_type_id,
    adp.asset_type_name,
    adp.asset_description,
    adp.category,
    adp.device_id,
    adp.parameter_id,
    adp.priority,
    adp.is_primary,
    adp.asset_metadata,
    adp.device_asset_metadata,
    -- Measurement data
    tm.time AS measurement_time,
    tm.value AS measurement_value,
    tm.quality AS measurement_quality,
    tm.metadata AS measurement_metadata,
    -- Parameter details
    tp.parameter_name,
    tp.attribute AS parameter_attribute,
    tp.device_type_id,
    tp.data_type AS parameter_data_type,
    tp.unit AS parameter_unit,
    tp.description AS parameter_description
FROM asset_device_parameters adp
INNER JOIN iot_data.timescale_measurements tm 
    ON adp.device_id = tm.device_id 
    AND adp.parameter_id = tm.parameter_id
LEFT JOIN iot_data.timescale_parameters tp 
    ON adp.parameter_id = tp.parameter_id;

-- Add a comment to describe the view
COMMENT ON VIEW iot_data.asset_measurements_view IS 
'Comprehensive view that combines asset types, their parameters, associated devices, and measurement readings. 
This view unnests the parameters array from asset_types, joins with device_asset_types to get device associations, 
and retrieves actual measurements from timescale_measurements for each asset-device-parameter combination.';

-- =================
-- 5. MAINTENANCE AND MONITORING

-- =================
-- View for monitoring asset type assignments (No changes needed)
CREATE OR REPLACE VIEW iot_data.asset_type_monitoring AS
SELECT
	 at.asset_type_name,
	 at.category,
	 COUNT(dat.device_id) as assigned_devices,
	 COUNT(*) FILTER (WHERE dat.is_primary) as primary_assignments,
	 array_length(at.parameters, 1) as parameter_count,
	 at.created_at,
	 MAX(dat.updated_at) as last_assignment
FROM iot_data.asset_types at
LEFT JOIN iot_data.device_asset_types dat ON at.asset_type_id = dat.asset_type_id
GROUP BY at.asset_type_id, at.asset_type_name, at.category, at.parameters, at.created_at
ORDER BY assigned_devices DESC;

-- View for asset type parameter summary
CREATE OR REPLACE VIEW iot_data.asset_type_parameters AS
WITH asset_params AS (
	 SELECT
	 	 at.asset_type_id,
	 	 at.asset_type_name,
	 	 at.category,
	 	 param.parameter_id
	 FROM iot_data.asset_types at
	 CROSS JOIN LATERAL UNNEST(at.parameters) AS param(parameter_id)
)
SELECT
	 ap.asset_type_id,
	 ap.asset_type_name,
	 ap.category,
	 COUNT(*) as parameter_count,
	 ARRAY_AGG(tp.parameter_name ORDER BY tp.parameter_name) as parameter_names,
	 ARRAY_AGG(tp.parameter_id ORDER BY tp.parameter_name) as parameter_ids
FROM asset_params ap
JOIN iot_data.timescale_parameters tp ON ap.parameter_id = tp.parameter_id
GROUP BY ap.asset_type_id, ap.asset_type_name, ap.category;

-- =================
-- 6. EXAMPLE USAGE SCENARIOS
-- =================
-- Example 1: Add a new asset type for Water Management

-- SELECT iot_data.add_asset_type(
-- 	 'Water Management System',
-- 	 'Smart water meters and flow control systems',
-- 	 'utilities',
-- 	 ARRAY['flow_rate', 'pressure', 'temperature', 'water_quality'],
-- 	 '{"forecasting_horizon": "24h", "sampling_frequency": "30min", "asset_features": ["flow_rate", "pressure"]}'::jsonb
-- );

-- Example 2: Assign device to multiple asset types
-- INSERT INTO iot_data.device_asset_types (device_id, asset_type_id, priority, is_primary)
-- SELECT 'device_123', asset_type_id, 1, true
-- FROM iot_data.asset_types
-- WHERE asset_type_name = 'Energy Meter - Main Distribution';

-- =================
-- 7. COMMENTS AND DOCUMENTATION
-- =================
COMMENT ON TABLE iot_data.asset_types IS 'Defines asset types and their associated parameters for LSTM forecasting';
COMMENT ON TABLE iot_data.device_asset_types IS 'Many-to-many relationship between devices and asset types';
COMMENT ON COLUMN iot_data.asset_types.parameters IS 'Array of parameter id values from timescale_parameters table';
COMMENT ON VIEW iot_data.device_asset_parameters IS 'Complete view of devices, their asset types, and LSTM parameters';
COMMENT ON VIEW iot_data.device_asset_parameters IS 'Returns LSTM parameters for a specific device';
```


