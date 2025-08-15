# ESPSomfy Vertical Blinds Simple Blueprint

A simplified Home Assistant blueprint for ESPSomfy vertical blinds with just 3 positions and state tracking.

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/Shiu/ESPSomfy-IQ2-vertical-blinds-blueprint/blob/main/vertical_blinds_simple.yaml)

## Features

- **3 Simple Positions**: Open, Closed, My (your preset)
- **State Tracking**: Always know if blinds are closed for automations
- **Wall Button Support**: Built-in support for physical buttons
- **No Timing Calculations**: Uses direct motor commands only
- **Slat Control**: Optional slat tilt commands

## Why This Blueprint?

- **Reliable**: No timing calculations means no drift or latency issues
- **State Aware**: Tracks blind position for presence-based lighting
- **Simple**: Just 3 positions - all you really need
- **Flexible**: Works with wall buttons and dashboard controls

## Setup

### 1. Program Your My Position

Use your remote to set the motor's My position to your preferred setting (e.g., 50% with slats tilted).

### 2. Create State Tracker

**Go to Settings → Devices & Services → Helpers → Create Helper → Dropdown**
- Name: `Dining Blinds State`
- Options:
  ```
  open
  closed
  my
  unknown
  ```

### 3. Import Blueprint

Click the import button above or manually import from:
```
https://github.com/Shiu/ESPSomfy-IQ2-vertical-blinds-blueprint/blob/main/vertical_blinds_simple.yaml
```

### 4. Create Automation

1. Go to Settings → Automations & Scenes
2. Create Automation → Use Blueprint → "ESPSomfy Vertical Blinds Simple Controller"
3. Configure:
   - **Blind Entity**: `cover.dining_blinds`
   - **Blind ID**: `dining` (unique identifier)
   - **State Tracker**: `input_select.dining_blinds_state`
4. Save as: `Dining Blinds Controller`

### 5. Add Helper Script

```yaml
script:
  control_vertical_blinds:
    alias: Control Vertical Blinds
    fields:
      blind_id:
        description: The blind identifier
      command:
        description: Command to send
    sequence:
      - event: vertical_blinds_control
        event_data:
          blind_id: "{{ blind_id }}"
          command: "{{ command }}"
```

### 6. Add Dashboard

```yaml
type: vertical-stack
cards:
  - type: markdown
    content: "## Dining Blinds"
  
  - type: entity
    entity: input_select.dining_blinds_state
    name: Current State
  
  - type: horizontal-stack
    cards:
      - type: button
        name: Open
        icon: mdi:arrow-up
        tap_action:
          action: call-service
          service: script.control_vertical_blinds
          data:
            blind_id: dining
            command: open
      
      - type: button
        name: My
        icon: mdi:star
        tap_action:
          action: call-service
          service: script.control_vertical_blinds
          data:
            blind_id: dining
            command: my
      
      - type: button
        name: Close
        icon: mdi:arrow-down
        tap_action:
          action: call-service
          service: script.control_vertical_blinds
          data:
            blind_id: dining
            command: close
```

## Wall Button Configuration

Edit the automation after creating it from the blueprint. Replace `YOUR_BUTTON_1_ID` and `YOUR_BUTTON_2_ID` with your actual button device IDs:

1. Find your button device IDs in Developer Tools → Events
2. Listen for `zha_event` (or your button's event type)
3. Press your button and note the `device_id`
4. Edit the automation and replace the placeholder IDs

## Using State for Automations

### Presence-Based Lighting

Only turn on lights when motion detected AND blinds are closed:

```yaml
automation:
  - alias: "Dining Room Presence Lighting"
    trigger:
      - platform: state
        entity_id: binary_sensor.dining_room_presence
        to: "on"
    condition:
      - condition: sun
        after: sunset
      - condition: state
        entity_id: input_select.dining_blinds_state
        state: "closed"  # Only when blinds are closed
    action:
      - service: light.turn_on
        entity_id: light.dining_room
```

### Binary Sensor (Optional)

Create a simple on/off sensor for "blinds closed":

```yaml
template:
  - binary_sensor:
      - name: "Dining Blinds Closed"
        state: "{{ states('input_select.dining_blinds_state') == 'closed' }}"
        device_class: window
```

## Commands

The blueprint responds to these commands via the `vertical_blinds_control` event:

- `open` - Move blinds up
- `close` - Move blinds down
- `my` - Go to My position
- `stop` - Stop movement
- `slat_open` - Tilt slats open
- `slat_close` - Tilt slats closed

## Troubleshooting

**State not updating?**
- Check that blind_id matches in automation and scripts
- Ensure state tracker helper exists

**Wall buttons not working?**
- Replace placeholder device IDs with your actual button IDs
- Check event type matches your buttons (zha_event, deconz_event, etc.)

**My position includes slat angle?**
- Yes, the My position saves both height and slat angle
- Program it to your most common setting

## Notes

- State tracking assumes commands are successful
- Manual remote use won't update state
- Consider "my" position as "closed" for lighting if that's your preference

## License

MIT