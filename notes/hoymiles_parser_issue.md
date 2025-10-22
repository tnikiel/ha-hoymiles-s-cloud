# Type Comparison Error in decode_data_point Method

## Issue Observed

The Hoymiles integration fails to create individual solar module sensors, only creating basic station sensors. The integration throws a type comparison error during setup.

### Error Logs

```
2025-10-22 21:53:41.590 ERROR (MainThread) [homeassistant.components.sensor] Error while setting up hoymiles_nimbus platform for sensor: '<' not supported between instances of 'str' and 'int'
  File "/config/custom_components/ha-hoymiles-s-cloud/sensor.py", line 63, in async_setup_entry
  File "/config/custom_components/ha-hoymiles-s-cloud/hoymiles_client.py", line 334, in fill_system_data
  File "/config/custom_components/ha-hoymiles-s-cloud/classes/station.py", line 35, in set_data
  File "/config/custom_components/ha-hoymiles-s-cloud/classes/micro_inverter.py", line 32, in set_data
  File "/config/custom_components/ha-hoymiles-s-cloud/classes/solar_module.py", line 29, in set_data
  File "/config/custom_components/ha-hoymiles-s-cloud/classes/data_point.py", line 12, in __init__
  File "/config/custom_components/ha-hoymiles-s-cloud/parsers.py", line 277, in decode_data_point
```

## Proposed Fix

Add type checking in the `decode_data_point` method in `parsers.py`:

```python
@staticmethod
def decode_data_point(data_point: list):
    decoded = []
    for i, val in enumerate(data_point):
        if isinstance(val, (int, float)) and val < 10:
            decoded.append(val)
        elif isinstance(val, (int, float)):
            decoded.append(struct.unpack('<f', struct.pack('<I', val))[0])
        else:
            # Skip non-numeric values
            continue
    return decoded
```

## Explanation

The error occurs because the `decode_data_point` method assumes all values in the `data_point` list are numeric, but the API can return mixed data types including strings. The original code attempts to compare strings with integers (`val < 10`), causing a TypeError.

The fix adds `isinstance()` checks to ensure only numeric values are processed:
1. Check if value is numeric before comparison
2. Process numeric values as before
3. Skip non-numeric values to prevent crashes

This allows `fill_system_data` to complete successfully, enabling creation of all module sensors instead of just basic station sensors.