# ESPSomfy Vertical Blinds State Tracker

A simple Home Assistant blueprint that tracks blind state for automations.

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/Shiu/ESPSomfy-IQ2-vertical-blinds-blueprint/blob/main/vertical_blinds_simple.yaml)

## What It Does

- Tracks if your blinds are open, closed, or at My position
- Updates state when you control blinds (from wall buttons, dashboard, etc.)
- Use the state for automations (like presence-based lighting)

## Setup

### 1. Create State Helper

**Settings → Devices & Services → Helpers → Create Helper → Dropdown**
- Name: `Dining Blinds State`
- Options:
  ```
  open
  closed
  my
  unknown
  ```

### 2. Import Blueprint

Click the import button above.

### 3. Create Automation

1. Settings → Automations & Scenes → Create Automation
2. Use Blueprint → "ESPSomfy Vertical Blinds State Tracker"
3. Configure:
   - **Blind ID**: `dining`
   - **State Tracker**: `input_select.dining_blinds_state`
4. Save

### 4. Control Your Blinds

Fire `vertical_blinds_control` events with:
- `blind_id`: dining
- `command`: open, close, or my

The blueprint will track the state.

## Use State in Automations

```yaml
automation:
  - alias: "Presence Lighting - Only When Blinds Closed"
    trigger:
      - platform: state
        entity_id: binary_sensor.dining_room_presence
        to: "on"
    condition:
      - condition: sun
        after: sunset
      - condition: state
        entity_id: input_select.dining_blinds_state
        state: "closed"
    action:
      - service: light.turn_on
        entity_id: light.dining_room
```

That's it. Simple state tracking.