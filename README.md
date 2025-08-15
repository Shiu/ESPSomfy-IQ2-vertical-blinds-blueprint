# ESPSomfy Vertical Blinds Blueprint (Smart Timing)

A Home Assistant automation blueprint for controlling vertical blinds with ESPSomfy RTS. Features automatic timing calculation from a single measurement - just measure the full travel time once!

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/Shiu/ESPSomfy-IQ2-vertical-blinds-blueprint/blob/main/vertical_blinds_controller.yaml)

## Features

- **Smart Timing** - Only measure ONE time (full travel), everything else is calculated!
- **3 preset positions** - Closed, middle, and fully open
- **Position index system** - Uses simple 1,2,3 instead of confusing percentages
- **Dynamic calculation** - Automatically calculates movement times based on position percentages
- **Event-based control** - Fire events to move blinds to any position
- **Single helper script** - Shared by all blinds for firing events
- **Soft start compensation** - Works perfectly with Benthin IQ2 motors
- **Position reset option** - Can auto-reset to position 3 (open) before movements

## Position System

Instead of using percentages directly, this blueprint uses position indices:
- **Position 1**: Closed (you define what % this is, e.g., 1%)
- **Position 2**: Middle position (e.g., 50%)
- **Position 3**: Fully open (usually 100%)

This makes it clear you're selecting presets, not actual percentages.

## Quick Start

### 1. Create Position Tracker Helper

Create an Input Select helper to track position:

**Go to Settings → Devices & Services → Helpers → Create Helper → Dropdown**
- Name: `Dining Blinds Position`
- Options: 
  ```
  1
  2
  3
  ```
- Set initial value to match your blind's current position

### 2. Import the Blueprint

Click the import button above or use:
```
https://github.com/Shiu/ESPSomfy-IQ2-vertical-blinds-blueprint/blob/main/vertical_blinds_controller.yaml
```

### 3. Create ONE Automation Per Blind

1. Go to Settings → Automations & Scenes → Automations
2. Create Automation → Use Blueprint → "ESPSomfy Vertical Blinds Controller (Smart Timing)"
3. Configure:
   - **Blind Entity**: `cover.dining_blinds`
   - **Blind ID**: `dining` (unique identifier)
   - **Position Tracker**: `input_select.dining_blinds_position`
   - **Position Percentages**: 
     - Position 1 = 1% (closed)
     - Position 2 = 50% (middle)
     - Position 3 = 100% (open)
   - **Full Travel Time**: Measure time from fully open to closed (e.g., 30000ms)
   - **Soft Start Compensation**: Start with 1000ms, adjust if needed
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
      description: Target position (1,2,3)
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
  
  - type: horizontal-stack
    cards:
      - type: button
        name: Closed
        icon: mdi:blinds-closed
        tap_action:
          action: call-service
          service: script.move_vertical_blinds
          data:
            blind_id: dining
            position: 1
      
      - type: button
        name: Middle
        icon: mdi:blinds
        tap_action:
          action: call-service
          service: script.move_vertical_blinds
          data:
            blind_id: dining
            position: 2
      
      - type: button
        name: Open
        icon: mdi:blinds-open
        tap_action:
          action: call-service
          service: script.move_vertical_blinds
          data:
            blind_id: dining
            position: 3
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

### Position Percentages
- **Position 1**: Use 1% instead of 0% if motor auto-rotates slats at 0%
- **Position 2**: Set to your preferred middle position (33%, 50%, 66%, etc.)
- **Position 3**: Usually 100% for fully open

### Timing Accuracy
- Measure the full travel time 2-3 times and average
- Soft Start Compensation: Start with 1000ms, increase if blinds don't reach target
- The blueprint handles all calculations and directions automatically

### Multiple Blinds
Each blind gets its own automation with individual timings and position definitions.

## Troubleshooting

**Automation not triggering?**
- Check that blind_id matches exactly
- Ensure position tracker has options "1", "2", "3" as strings
- Reload automations after updating blueprint

**Position drift?**
- Enable "Always Reset Position First" option
- This makes blinds go to position 3 first, then to target
- Increase Soft Start Compensation if positions are consistently short

**Wrong position percentages?**
- Adjust the position_X_percent values in automation configuration
- These define what actual percentage each position represents

## Example Setup

**Dining Room:**
- Automation: `Dining Blinds Controller` with blind_id: `dining`
- Position tracker: `input_select.dining_blinds_position`
- Positions: 1=1%, 2=50%, 3=100%
- Full Travel Time: 30000ms (30 seconds)

**Lounge:**
- Automation: `Lounge Blinds Controller` with blind_id: `lounge`
- Position tracker: `input_select.lounge_blinds_position`  
- Positions: 1=0%, 2=33%, 3=100%
- Full Travel Time: 25000ms (25 seconds)

## License

MIT

## Support

Created for [ESPSomfy-RTS](https://github.com/rstrouse/ESPSomfy-RTS) with Benthin IQ2 motors.

Found an issue? [Report it here](https://github.com/Shiu/ESPSomfy-IQ2-vertical-blinds-blueprint/issues)