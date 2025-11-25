# motion-traces

## Trace Analysis: Ensuite Sensor Light Automation

This repository contains a Home Assistant automation trace for the "Ensuite Sensor Light" automation, which uses the **Blackshome Sensor Light Blueprint** (`Blackshome/sensor-light.yaml`).

### What This Automation Does

The automation controls `light.area_en_suite` based on motion detection from `binary_sensor.en_suite_motion_occupancy`.

**Configuration Summary:**
- **Trigger**: Motion sensor occupancy (`binary_sensor.en_suite_motion_occupancy`)
- **Controlled Light**: `light.area_en_suite`
- **Mode**: `restart` (restarts if triggered again while running)
- **Time Delay**: 10 seconds (configured motion timeout)
- **Dim Before Off**: Enabled (15 second warning dim)

### Trace Timeline

| Time | Event |
|------|-------|
| 15:17:49.749 | Automation triggered by motion sensor state change |
| 15:17:49.756 | All 7 conditions evaluated (all passed ✓) |
| 15:17:49.764 | Action started - chose "default" path |
| 15:17:49.819 | Light service called: `light.turn_on` with brightness_pct=0 (see issue below) |
| 15:17:49.836 | Parallel sequences started for light control and dynamic lighting |
| 15:18:49.824 | Repeat loop completed (light off check) |
| 15:18:56.974 | Internal delay started (600s = 10min, blueprint internal timing) |

### Current State

**Status: `running`** - The automation is still executing, waiting in a 600-second (10-minute) delay at step `action/0/default/3/parallel/1/sequence/7`.

### Conditions Evaluated

All initial conditions passed:
1. ✓ **Condition 0**: Main condition block (entity state check)
2. ✓ **Condition 1**: Empty entity check (bypass related)
3. ✓ **Condition 2**: Empty entity check (bypass related)  
4. ✓ **Condition 3**: Ambient light check - `sensor.en_suite_motion_illuminance` was evaluated
5. ✓ **Condition 4-6**: Additional bypass/mode conditions

### Key Features Detected in This Run

Based on the blueprint inputs, this automation has these features enabled:

| Feature | Status | Configured Value | Actual in Trace |
|---------|--------|------------------|-----------------|
| **Light Control** | Brightness + Transition | 100% brightness, 1s on, 2s off | **0% brightness** (see issue below) |
| **Dim Before Off** | Enabled | 50% dim, 15 second delay | - |
| **Dynamic Lighting** | Time-controlled brightness | Min 22% | - |
| **Ambient Light** | Enabled | Threshold: 40 lux | Passed |
| **Night Lights** | Enabled | 23:50-07:00, 1% brightness | Not active (time was 15:17) |

### What Happened in This Trace

1. **Motion Detected** at 15:17:49 - The occupancy sensor triggered the automation
2. **Conditions Checked** - All conditions passed including ambient light level check
3. **Parallel Execution Started** - Two parallel sequences began:
   - **Sequence 0**: Light off monitoring loop (checks if light is off)
   - **Sequence 1**: Light turn-on and dynamic brightness control
4. **Light Service Called** - `light.turn_on` was called with `brightness_pct: 0` and `transition: 1` 
   - ⚠️ **Note**: The brightness is set to 0%, which would effectively turn the light off or very dim
5. **Delay Active** - Currently waiting in a 600-second delay before completing

### ⚠️ Potential Issue: brightness_pct=0

The trace shows the light being turned on with `brightness_pct: 0`, which contradicts the configured 100% brightness. This is likely a bug or unexpected behavior.

**Possible causes:**
- The automation was in a **dim-before-off phase** (dimming down before turn-off)
- A **dynamic lighting calculation error** resulted in 0%
- The light was already **being turned off** when this service call was made
- **Blueprint internal logic** may have overridden the configured brightness

**Investigation suggestion:** Check if the light actually responded to this call, or if subsequent calls corrected the brightness.

### Trace File

`trace automation.ensuite_sensor_light 2025-11-25T15_17_49.749178+00_00.json` (1.07 MB, 21,136 lines)