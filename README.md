# ESPSomfy IQ2 Vertical Blinds Blueprint

A Home Assistant blueprint for controlling vertical blinds with Benthin IQ2 motors via ESPSomfy RTS.

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/Shiu/ESPSomfy-IQ2-vertical-blinds-blueprint/blob/main/vertical_blinds_controller.yaml)

## Features

- **Customizable positions**: Define up to 4 positions per blind (e.g., 0%, 33%, 50%, 100%)
- **Precise timing control**: Uses timed motor commands instead of ESPSomfy's position calculations
- **Soft start compensation**: Works with IQ2 motor's soft start/stop feature
- **Multiple blind support**: Each blind can have different positions and timings
- **UI-adjustable timings**: All timing values stored in helpers for easy adjustment

## Problem This Solves

The Benthin IQ2 motor for vertical blinds has:
- Soft start/stop that makes timing non-linear
- 180° slat rotation after reaching closed position
- Complex sequencing that doesn't match ESPSomfy's assumptions

This blueprint bypasses ESPSomfy's position calculations and uses direct motor control with precise timing.

## Setup Instructions

### 1. Create Input Helpers

Create these helpers in Home Assistant (Settings → Devices & Services → Helpers):

#### Position Tracker (Input Select)
For each blind, create an input select with your desired positions:

**Dining Room Example** (3 positions):
- Name: `Dining Blinds Position`
- Options: `0`, `33`, `100`

**Lounge Example** (3 positions):
- Name: `Lounge Blinds Position`  
- Options: `0`, `50`, `100`

#### Timing Helpers (Input Numbers)
For each blind, create input_number helpers for movement timings:

**Naming Convention**: `[prefix]_[from]_to_[to]`

Example for dining room with positions 0, 33, 100:
- `dining_blinds_100_to_0` - Time from open to closed
- `dining_blinds_100_to_33` - Time from open to 33%
- `dining_blinds_33_to_0` - Time from 33% to closed
- `dining_blinds_0_to_33` - Time from closed to 33%
- `dining_blinds_full_open` - Time to fully open from any position

Settings for each helper:
- Min: 0
- Max: 60000
- Step: 100
- Unit: ms
- Mode: box

### 2. Import the Blueprint

Click the import button above or manually import from:
```
https://github.com/Shiu/ESPSomfy-IQ2-vertical-blinds-blueprint/blob/main/vertical_blinds_controller.yaml
```

### 3. Create Scripts Using the Blueprint

For each position of each blind, create a script:

1. Go to Settings → Automations & Scenes → Scripts
2. Click "Add Script" → "Use Blueprint"
3. Select "ESPSomfy Vertical Blinds Position Controller"
4. Configure:
   - **Blind Entity**: Your ESPSomfy cover entity
   - **Position Tracker**: Your position input_select
   - **Target Position**: The position for this script (e.g., "100", "33", "0")
   - **Position 1-4**: Your position values (use same value for unused positions)
   - **Timing Prefix**: Your helper prefix (e.g., "dining_blinds")
   - **Full Open Helper**: Your full_open timing helper

Example scripts to create:
- `dining_blinds_open` (target: "100")
- `dining_blinds_33` (target: "33")
- `dining_blinds_close` (target: "0")

### 4. Add Dashboard Controls

```yaml
type: horizontal-stack
cards:
  - type: button
    name: Open
    icon: mdi:blinds-open
    tap_action:
      action: call-service
      service: script.dining_blinds_open
      
  - type: button
    name: 33%
    icon: mdi:blinds
    tap_action:
      action: call-service
      service: script.dining_blinds_33
      
  - type: button
    name: Close
    icon: mdi:blinds-closed
    tap_action:
      action: call-service
      service: script.dining_blinds_close
```

## Measuring Timings

1. Start with blinds fully open (100%)
2. Time each movement path:
   - 100% → 0%: Full close time (stop before slat rotation)
   - 100% → 33%: Partial close time
   - 0% → 33%: Partial open time
   - 33% → 0%: Close from partial
3. Add 500-1000ms buffer to full open time
4. Enter times in milliseconds into helpers

## Tips

- **Never use actual 0%** if motor auto-rotates slats at 0%
- **Reset position** by going to 100% if timing drifts
- **Adjust timings** through UI helpers without reloading
- **Test each path** after setting initial timings

## Example Configuration

See [examples/configuration_example.yaml](examples/configuration_example.yaml) for complete helper configuration.

## License

MIT

## Credits

Created for use with [ESPSomfy-RTS](https://github.com/rstrouse/ESPSomfy-RTS) and Benthin IQ2 motors.