# ESPSomfy Vertical Blinds Blueprint (Event-Based)

A simple, event-based Home Assistant automation blueprint for controlling vertical blinds with ESPSomfy RTS. One automation per blind plus a simple helper script!

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/Shiu/ESPSomfy-IQ2-vertical-blinds-blueprint/blob/main/vertical_blinds_controller.yaml)

## Features

- **One automation per blind** - Simple and organized
- **Event-based control** - Fire events to move blinds to any position
- **Single helper script** - Shared by all blinds for firing events
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

### 3. Create ONE Automation Per Blind

1. Go to Settings → Automations & Scenes → Automations
2. Create Automation → Use Blueprint → "ESPSomfy Vertical Blinds Controller (Event-Based)"
3. Configure:
   - **Blind Entity**: `cover.dining_blinds`
   - **Blind ID**: `dining` (unique identifier for this blind)
   - **Position Tracker**: `input_select.dining_blinds_position`
   - **Position Values**: Closed=`0`, Middle=`33`, Open=`100`
   - **Timings**: Enter your measured times in milliseconds
   - Save as: `Dining Blinds Controller`

### 4. Create a Helper Script to Fire Events

Since dashboard buttons can't directly fire events, create this simple helper script:

```yaml
# In scripts.yaml or via UI
move_vertical_blinds:
  alias: Move Vertical Blinds
  fields:
    blind_id:
      description: The blind identifier
    position:
      description: Target position (0-100)
  sequence:
    - event: vertical_blinds_move
      event_data:
        blind_id: "{{ blind_id }}"
        position: "{{ position }}"
```

### 5. Add Dashboard Buttons

Now add buttons that call the helper script:

```yaml
type: horizontal-stack
cards:
  - type: button
    name: Open
    icon: mdi:blinds-open
    tap_action:
      action: call-service
      service: script.move_vertical_blinds
      data:
        blind_id: dining
        position: 100
  
  - type: button
    name: 33%
    icon: mdi:blinds
    tap_action:
      action: call-service
      service: script.move_vertical_blinds
      data:
        blind_id: dining
        position: 33
  
  - type: button
    name: Close
    icon: mdi:blinds-closed
    tap_action:
      action: call-service
      service: script.move_vertical_blinds
      data:
        blind_id: dining
        position: 0
```

## How It Works

The automation listens for `vertical_blinds_move` events with:
- `blind_id`: Which blind to control (matches the ID you configured)
- `position`: Target position (0, 33, 50, 100, etc.)

When an event is fired, the automation:
1. Checks current position from the tracker
2. Calculates the movement path needed
3. Sends appropriate RTS commands with precise timing
4. Updates the position tracker

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
Each blind gets its own automation with individual timings. Perfect for blinds with different motor speeds or travel distances.

## Developer API

You can also control blinds programmatically:

```yaml
# In automations or scripts
- event: vertical_blinds_move
  event_data:
    blind_id: dining
    position: 50

# From Developer Tools → Events
Event Type: vertical_blinds_move
Event Data:
  blind_id: dining
  position: 33
```

## Troubleshooting

**Position drift over time?**
- Enable "Always Reset Position First" option
- Increases movement time but ensures accuracy

**Blinds overshoot position?**
- Reduce timing values by 500-1000ms
- Motor soft stop adds extra travel

**Different motor speeds?**
- Each automation has independent timings
- Measure and configure each blind separately

## Example Setup

For a home with two rooms:

**Dining Room** (33% middle position):
- Automation: `Dining Blinds Controller` with blind_id: `dining`
- Position tracker: `input_select.dining_blinds_position` with options: 0, 33, 100

**Lounge** (50% middle position):
- Automation: `Lounge Blinds Controller` with blind_id: `lounge`
- Position tracker: `input_select.lounge_blinds_position` with options: 0, 50, 100

## License

MIT

## Support

Created for [ESPSomfy-RTS](https://github.com/rstrouse/ESPSomfy-RTS) with Benthin IQ2 motors.

Found an issue? [Report it here](https://github.com/Shiu/ESPSomfy-IQ2-vertical-blinds-blueprint/issues)