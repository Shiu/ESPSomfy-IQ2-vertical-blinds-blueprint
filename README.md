# ESPSomfy Vertical Blinds Blueprint (Percentage-Based)

A Home Assistant automation blueprint for controlling vertical blinds with ESPSomfy RTS. Send any percentage value (1-100) to move blinds to that exact position. Only requires measuring ONE timing!

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/Shiu/ESPSomfy-IQ2-vertical-blinds-blueprint/blob/main/vertical_blinds_controller.yaml)

## Features

- **Percentage Control** - Send any value from 1-100% to position blinds exactly
- **Smart Timing** - Only measure ONE time (full travel), everything else is calculated!
- **Dynamic calculation** - Automatically calculates movement times for any position
- **Event-based control** - Fire events to move blinds to any percentage
- **Single helper script** - Shared by all blinds for firing events
- **Soft start compensation** - Works perfectly with Benthin IQ2 motors
- **Position reset option** - Can auto-reset to fully open before movements

## Position System

This blueprint uses direct percentage control:
- **100%**: Fully open
- **1-99%**: Any position you want
- **1%**: Closed (use 1% instead of 0% to avoid auto-rotation on some motors)

You can send any percentage value and the blinds will move to that exact position.

## Quick Start

### 1. Create Position Tracker Helper

Create an Input Number helper to track position:

**Go to Settings → Devices & Services → Helpers → Create Helper → Number**
- Name: `Dining Blinds Position`
- Minimum: 1
- Maximum: 100
- Step size: 1
- Unit of measurement: %
- Initial value: 100

### 2. Import the Blueprint

Click the import button above or use:
```
https://github.com/Shiu/ESPSomfy-IQ2-vertical-blinds-blueprint/blob/main/vertical_blinds_controller.yaml
```

### 3. Create ONE Automation Per Blind

1. Go to Settings → Automations & Scenes → Automations
2. Create Automation → Use Blueprint → "ESPSomfy Vertical Blinds Controller (Percentage-Based)"
3. Configure:
   - **Blind Entity**: `cover.dining_blinds`
   - **Blind ID**: `dining` (unique identifier)
   - **Position Tracker**: `input_number.dining_blinds_position`
   - **Full Travel Time**: Measure time from fully open to closed (e.g., 30000ms)
   - **Soft Start Compensation**: Start with 1000ms, adjust if needed
   - **Reset Buffer**: 3000ms (extra time when resetting)
   - Save as: `Dining Blinds Controller`

### 4. Create Helper Script

Create this helper script to fire events from dashboard:

```yaml
# In scripts.yaml or via UI
move_vertical_blinds:
  alias: Move Vertical Blinds
  fields:
    blind_id:
      description: The blind identifier
    position:
      description: Target position percentage (1-100)
  sequence:
    - event: vertical_blinds_move
      event_data:
        blind_id: "{{ blind_id }}"
        position: "{{ position }}"
```

### 5. Add Dashboard Buttons

```yaml
type: vertical-stack
cards:
  - type: markdown
    content: "## Dining Blinds"
  
  # Current position display
  - type: entities
    entities:
      - entity: input_number.dining_blinds_position
        name: Current Position
  
  # Row 1: 100%, 90%, 80%
  - type: horizontal-stack
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
        name: 90%
        icon: mdi:blinds
        tap_action:
          action: call-service
          service: script.move_vertical_blinds
          data:
            blind_id: dining
            position: 90
      
      - type: button
        name: 80%
        icon: mdi:blinds
        tap_action:
          action: call-service
          service: script.move_vertical_blinds
          data:
            blind_id: dining
            position: 80
  
  # Row 2: 70%, 60%, 50%
  - type: horizontal-stack
    cards:
      - type: button
        name: 70%
        icon: mdi:blinds
        tap_action:
          action: call-service
          service: script.move_vertical_blinds
          data:
            blind_id: dining
            position: 70
      
      - type: button
        name: 60%
        icon: mdi:blinds
        tap_action:
          action: call-service
          service: script.move_vertical_blinds
          data:
            blind_id: dining
            position: 60
      
      - type: button
        name: 50%
        icon: mdi:blinds
        tap_action:
          action: call-service
          service: script.move_vertical_blinds
          data:
            blind_id: dining
            position: 50
  
  # Row 3: 40%, 30%, 20%
  - type: horizontal-stack
    cards:
      - type: button
        name: 40%
        icon: mdi:blinds
        tap_action:
          action: call-service
          service: script.move_vertical_blinds
          data:
            blind_id: dining
            position: 40
      
      - type: button
        name: 30%
        icon: mdi:blinds
        tap_action:
          action: call-service
          service: script.move_vertical_blinds
          data:
            blind_id: dining
            position: 30
      
      - type: button
        name: 20%
        icon: mdi:blinds
        tap_action:
          action: call-service
          service: script.move_vertical_blinds
          data:
            blind_id: dining
            position: 20
  
  # Row 4: 10%, Closed (1%)
  - type: horizontal-stack
    cards:
      - type: button
        name: 10%
        icon: mdi:blinds
        tap_action:
          action: call-service
          service: script.move_vertical_blinds
          data:
            blind_id: dining
            position: 10
      
      - type: button
        name: Closed
        icon: mdi:blinds-closed
        tap_action:
          action: call-service
          service: script.move_vertical_blinds
          data:
            blind_id: dining
            position: 1
```

## Measuring Timing

**You only need to measure ONE timing!**

1. Start with blinds fully open (Position 3)
2. Send a "Down" command and time how long it takes to reach fully closed (Position 1)
3. Enter this time as "Full Travel Time" in milliseconds (seconds × 1000)

That's it! The blueprint automatically calculates all other movements using:
- Your position percentages (what % each position represents)
- The soft start compensation value
- Smart proportional calculations

## Configuration Tips

### Closed Position
- Use 1% instead of 0% if your motor auto-rotates slats at 0%
- You can send any percentage value from 1-100

### Timing Accuracy
- Measure the full travel time 2-3 times and average
- Soft Start Compensation: Start with 1000ms, increase if blinds don't reach target
- The blueprint handles all calculations and directions automatically

### Multiple Blinds
Each blind gets its own automation with individual timings and position definitions.

## Troubleshooting

**Automation not triggering?**
- Check that blind_id matches exactly
- Ensure position tracker is an input_number with min: 1, max: 100
- Reload automations after updating blueprint

**Position drift?**
- Enable "Always Reset Position First" option
- This makes blinds go to 100% first, then to target
- Increase Soft Start Compensation if positions are consistently short

**Wrong position percentages?**
- Adjust the position_X_percent values in automation configuration
- These define what actual percentage each position represents

## Example Setup

**Dining Room:**
- Automation: `Dining Blinds Controller` with blind_id: `dining`
- Position tracker: `input_number.dining_blinds_position`
- Full Travel Time: 30000ms (30 seconds)
- Soft Start Compensation: 1000ms

**Lounge:**
- Automation: `Lounge Blinds Controller` with blind_id: `lounge`
- Position tracker: `input_number.lounge_blinds_position`  
- Full Travel Time: 25000ms (25 seconds)
- Soft Start Compensation: 1000ms

## License

MIT

## Support

Created for [ESPSomfy-RTS](https://github.com/rstrouse/ESPSomfy-RTS) with Benthin IQ2 motors.

Found an issue? [Report it here](https://github.com/Shiu/ESPSomfy-IQ2-vertical-blinds-blueprint/issues)