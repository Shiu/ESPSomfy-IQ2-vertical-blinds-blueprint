# ESPSomfy Vertical Blinds Blueprint (My Position Enhanced)

A Home Assistant automation blueprint for controlling vertical blinds with ESPSomfy RTS. Uses the motor's My position for perfect closed accuracy, with timed movements for other positions. Only requires measuring ONE timing!

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/Shiu/ESPSomfy-IQ2-vertical-blinds-blueprint/blob/main/vertical_blinds_controller.yaml)

## Features

- **Perfect Closed Position** - Uses motor's My preset for positions 1-10% (no gaps!)
- **Smart Timing** - Only measure ONE time (full travel), everything else is calculated!
- **Latency-Proof Closing** - My command eliminates radio latency issues when closing
- **Percentage Control** - Send values from 1-100% to position blinds
- **Event-based control** - Fire events to move blinds to any percentage
- **Single helper script** - Shared by all blinds for firing events
- **Soft start compensation** - Works perfectly with Benthin IQ2 motors

## Position System

This blueprint uses a hybrid approach for accuracy:
- **100%**: Fully open (timed movement)
- **11-99%**: Intermediate positions (timed movements)
- **1-10%**: Closed position (uses My preset for perfect accuracy)

**Note:** All positions from 1-10% go to the same My preset position. This ensures no gaps when closing, regardless of radio latency.

## Prerequisites

**IMPORTANT:** You must program your motor's My position to the closed position (1%):
1. Use your remote to move blinds to the desired closed position (about 1% open)
2. Press and hold the My button on your remote until the motor jogs
3. The My position is now set to your closed position

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
  
  # Row 1: Common positions
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
        name: 70%
        icon: mdi:blinds
        tap_action:
          action: call-service
          service: script.move_vertical_blinds
          data:
            blind_id: dining
            position: 70
      
      - type: button
        name: 50%
        icon: mdi:blinds
        tap_action:
          action: call-service
          service: script.move_vertical_blinds
          data:
            blind_id: dining
            position: 50
  
  # Row 2: Lower positions
  - type: horizontal-stack
    cards:
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

1. Start with blinds fully open (100%)
2. Send a "Down" command and time how long it takes to reach your My position (closed)
3. Enter this time as "Full Travel Time" in milliseconds (seconds × 1000)

That's it! The blueprint:
- Uses My command for perfect closed position (1-10%)
- Calculates all other movements proportionally
- Accounts for soft start characteristics

## Configuration Tips

### My Position Setup
- **Critical:** Program your motor's My position to the closed position first
- Positions 1-10% all go to this My preset (perfect accuracy)
- This eliminates radio latency issues when closing

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
- Only affects positions 11-99% (timed movements)
- Positions 1-10% use My preset (always accurate)
- Adjust Soft Start Distance % if intermediate positions are off
- Enable "Always Reset Position First" for better accuracy

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