# ESPSomfy Vertical Blinds Blueprint (Self-Contained)

A simple, self-contained Home Assistant blueprint for controlling vertical blinds with ESPSomfy RTS. All timing configurations are built into each script - minimal setup required!

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/Shiu/ESPSomfy-IQ2-vertical-blinds-blueprint/blob/main/vertical_blinds_controller.yaml)

## Features

- **Self-contained scripts** - All timings configured within each script
- **Minimal helpers** - Only need one position tracker per blind
- **Customizable positions** - Set your own closed/middle/open values
- **Soft start compensation** - Works perfectly with Benthin IQ2 motors
- **Position reset option** - Can auto-reset to 100% before movements to prevent drift

## Quick Start

### 1. Create ONE Helper Per Blind

Create an Input Select helper to track position:

**For Dining Room (with 33% middle):**
- Name: `Dining Blinds Position`
- Options: `0`, `33`, `100`

**For Lounge (with 50% middle):**
- Name: `Lounge Blinds Position`
- Options: `0`, `50`, `100`

### 2. Import the Blueprint

Click the import button above or use:
```
https://github.com/Shiu/ESPSomfy-IQ2-vertical-blinds-blueprint/blob/main/vertical_blinds_controller.yaml
```

### 3. Create Scripts (One Per Position)

Create 3 scripts for each blind using the blueprint:

#### Example: Dining Room Open Script
1. Go to Settings → Automations & Scenes → Scripts
2. Add Script → Use Blueprint → "ESPSomfy Vertical Blinds Controller"
3. Configure:
   - **Blind Entity**: `cover.dining_blinds`
   - **Target Position**: `100`
   - **Position Tracker**: `input_select.dining_blinds_position`
   - **Position Values**: Closed=`0`, Middle=`33`, Open=`100`
   - **Timings**: Enter your measured times
   - Save as: `dining_blinds_open`

Repeat for:
- `dining_blinds_33` (Target: `33`)
- `dining_blinds_close` (Target: `0`)

### 4. Add Dashboard Buttons

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

Time these movements with a stopwatch:
1. **Open → Closed**: Full travel time (stop at 1% to avoid auto-rotation)
2. **Open → Middle**: Time to your middle position
3. **Middle → Closed**: Time from middle to closed
4. **Closed → Middle**: Time from closed to middle
5. **Full Open**: Add 3-5 seconds buffer to ensure full open

Enter times in milliseconds (seconds × 1000).

## Configuration Tips

### Position Values
- **Closed**: Use `1` instead of `0` if your motor auto-rotates slats at 0%
- **Middle**: Any value between 20-80 (33%, 50%, etc.)
- **Open**: Usually `100`

### Timing Accuracy
- Measure each path 2-3 times and average
- Add 500ms buffer to "close" movements to ensure full travel
- Enable "Always Reset" if you notice position drift

### Multiple Blinds
Each blind gets its own set of scripts with individual timings. Perfect for blinds with different motor speeds or travel distances.

## Troubleshooting

**Position drift over time?**
- Enable "Always Reset Position First" option
- Increases movement time but ensures accuracy

**Blinds overshoot position?**
- Reduce timing values by 500-1000ms
- Motor soft stop adds extra travel

**Different motor speeds?**
- Each script has independent timings
- Measure and configure each blind separately

## Example Setup

For a home with two rooms:

**Dining Room** (33% middle position):
- Scripts: `dining_blinds_open`, `dining_blinds_33`, `dining_blinds_close`
- Position tracker: `input_select.dining_blinds_position` with options: 0, 33, 100

**Lounge** (50% middle position):
- Scripts: `lounge_blinds_open`, `lounge_blinds_50`, `lounge_blinds_close`
- Position tracker: `input_select.lounge_blinds_position` with options: 0, 50, 100

## License

MIT

## Support

Created for [ESPSomfy-RTS](https://github.com/rstrouse/ESPSomfy-RTS) with Benthin IQ2 motors.

Found an issue? [Report it here](https://github.com/Shiu/ESPSomfy-IQ2-vertical-blinds-blueprint/issues)