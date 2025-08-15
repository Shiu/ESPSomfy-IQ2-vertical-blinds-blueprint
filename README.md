# ESPSomfy Vertical Blinds Blueprint (4-Position System)

A flexible Home Assistant automation blueprint for controlling vertical blinds with ESPSomfy RTS. Supports up to 4 preset positions using a simple index system.

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/Shiu/ESPSomfy-IQ2-vertical-blinds-blueprint/blob/main/vertical_blinds_controller.yaml)

## Features

- **4 preset positions** - Closed, two middle positions, and fully open
- **Position index system** - Uses simple 1,2,3,4 instead of confusing percentages
- **Flexible configuration** - Define what percentage each position represents
- **Event-based control** - Fire events to move blinds to any position
- **Single helper script** - Shared by all blinds for firing events
- **Soft start compensation** - Works perfectly with Benthin IQ2 motors
- **Position reset option** - Can auto-reset to position 4 (open) before movements

## Position System

Instead of using percentages directly, this blueprint uses position indices:
- **Position 1**: Closed (you define what % this is, e.g., 1%)
- **Position 2**: First middle position (e.g., 33%)
- **Position 3**: Second middle position (e.g., 66%)
- **Position 4**: Fully open (usually 100%)

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
  4
  ```
- Set initial value to match your blind's current position

### 2. Import the Blueprint

Click the import button above or use:
```
https://github.com/Shiu/ESPSomfy-IQ2-vertical-blinds-blueprint/blob/main/vertical_blinds_controller.yaml
```

### 3. Create ONE Automation Per Blind

1. Go to Settings → Automations & Scenes → Automations
2. Create Automation → Use Blueprint → "ESPSomfy Vertical Blinds Controller (4-Position)"
3. Configure:
   - **Blind Entity**: `cover.dining_blinds`
   - **Blind ID**: `dining` (unique identifier)
   - **Position Tracker**: `input_select.dining_blinds_position`
   - **Position Percentages**: 
     - Position 1 = 1% (closed)
     - Position 2 = 33% (first middle)
     - Position 3 = 66% (second middle)
     - Position 4 = 100% (open)
   - **Timings**: Measure and enter times for each movement path
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
      description: Target position (1,2,3,4)
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
        name: 33%
        icon: mdi:blinds
        tap_action:
          action: call-service
          service: script.move_vertical_blinds
          data:
            blind_id: dining
            position: 2
      
      - type: button
        name: 66%
        icon: mdi:blinds
        tap_action:
          action: call-service
          service: script.move_vertical_blinds
          data:
            blind_id: dining
            position: 3
      
      - type: button
        name: Open
        icon: mdi:blinds-open
        tap_action:
          action: call-service
          service: script.move_vertical_blinds
          data:
            blind_id: dining
            position: 4
```

## Measuring Timings

You need to measure times between ALL position combinations:

**From Position 4 (Open) going down:**
- Position 4 → 3: Time to second middle
- Position 4 → 2: Time to first middle
- Position 4 → 1: Time to closed

**Between middle positions:**
- Position 3 → 2: Time between middles (down)
- Position 2 → 3: Time between middles (up)
- Position 3 → 1: Second middle to closed
- Position 2 → 1: First middle to closed

**From Position 1 (Closed) going up:**
- Position 1 → 2: Time to first middle
- Position 1 → 3: Time to second middle

Enter all times in milliseconds (seconds × 1000).

## Configuration Tips

### Position Percentages
- **Position 1**: Use 1% instead of 0% if motor auto-rotates slats at 0%
- **Position 2-3**: Set to your preferred middle positions
- **Position 4**: Usually 100% for fully open

### Timing Accuracy
- Measure each path 2-3 times and average
- Add 500ms buffer for soft stop
- The blueprint handles both directions (up/down) automatically

### Multiple Blinds
Each blind gets its own automation with individual timings and position definitions.

## Troubleshooting

**Automation not triggering?**
- Check that blind_id matches exactly
- Ensure position tracker has options "1", "2", "3", "4" as strings
- Reload automations after updating blueprint

**Position drift?**
- Enable "Always Reset Position First" option
- This makes blinds go to position 4 first, then to target

**Wrong position percentages?**
- Adjust the position_X_percent values in automation configuration
- These define what actual percentage each position represents

## Example Setup

**Dining Room:**
- Automation: `Dining Blinds Controller` with blind_id: `dining`
- Position tracker: `input_select.dining_blinds_position`
- Positions: 1=1%, 2=33%, 3=66%, 4=100%

**Lounge:**
- Automation: `Lounge Blinds Controller` with blind_id: `lounge`
- Position tracker: `input_select.lounge_blinds_position`  
- Positions: 1=0%, 2=25%, 3=75%, 4=100%

## License

MIT

## Support

Created for [ESPSomfy-RTS](https://github.com/rstrouse/ESPSomfy-RTS) with Benthin IQ2 motors.

Found an issue? [Report it here](https://github.com/Shiu/ESPSomfy-IQ2-vertical-blinds-blueprint/issues)