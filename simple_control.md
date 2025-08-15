# Simple ESPSomfy RTS Control for Vertical Blinds

A straightforward control setup using only ESPSomfy RTS commands with state tracking for automations.

## Setup

1. **Program your My position**: Set blinds to desired height with slats closed
2. **Configure ESPSomfy**: Ensure your blind entity is set up (e.g., `cover.dining_blinds`)
3. **Create state helper**: See below

## State Tracking Helper

Create an input_select to track blind position:

**Go to Settings → Devices & Services → Helpers → Create Helper → Dropdown**
- Name: `Dining Blinds State`
- Options:
  ```
  open
  closed
  my_position
  unknown
  ```
- Icon: `mdi:blinds`

## ESPSomfy RTS Commands with State Tracking

### Scripts with State Updates

```yaml
# In scripts.yaml
script:
  dining_blinds_open:
    alias: "Open Dining Blinds"
    sequence:
      - service: espsomfy_rts.send_command
        data:
          entity_id: cover.dining_blinds
          command: "Up"
      - service: input_select.select_option
        data:
          entity_id: input_select.dining_blinds_state
          option: "open"
  
  dining_blinds_close:
    alias: "Close Dining Blinds"
    sequence:
      - service: espsomfy_rts.send_command
        data:
          entity_id: cover.dining_blinds
          command: "Down"
      - service: input_select.select_option
        data:
          entity_id: input_select.dining_blinds_state
          option: "closed"
  
  dining_blinds_my:
    alias: "Dining Blinds to My Position"
    sequence:
      - service: espsomfy_rts.send_command
        data:
          entity_id: cover.dining_blinds
          command: "My"
      - service: input_select.select_option
        data:
          entity_id: input_select.dining_blinds_state
          option: "my_position"
  
  dining_blinds_stop:
    alias: "Stop Dining Blinds"
    sequence:
      - service: espsomfy_rts.send_command
        data:
          entity_id: cover.dining_blinds
          command: "My"  # My acts as stop during movement
      - service: input_select.select_option
        data:
          entity_id: input_select.dining_blinds_state
          option: "unknown"  # Position uncertain after manual stop
  
  dining_slats_open:
    alias: "Open Dining Blind Slats"
    sequence:
      - service: espsomfy_rts.send_step_command
        data:
          entity_id: cover.dining_blinds
          direction: "StepUp"
          step_size: 100
      # Don't change state - only slats moved
  
  dining_slats_close:
    alias: "Close Dining Blind Slats"
    sequence:
      - service: espsomfy_rts.send_step_command
        data:
          entity_id: cover.dining_blinds
          direction: "StepDown"
          step_size: 100
      # Don't change state - only slats moved
```

## Dashboard Configuration

```yaml
type: vertical-stack
cards:
  - type: markdown
    content: "## Dining Blinds"
  
  # State display
  - type: entities
    entities:
      - entity: input_select.dining_blinds_state
        name: Current State
  
  # Main position controls
  - type: horizontal-stack
    cards:
      - type: button
        name: Open
        icon: mdi:arrow-up
        icon_height: 40px
        tap_action:
          action: call-service
          service: script.dining_blinds_open
      
      - type: button
        name: My
        icon: mdi:star
        icon_height: 40px
        tap_action:
          action: call-service
          service: script.dining_blinds_my
      
      - type: button
        name: Close
        icon: mdi:arrow-down
        icon_height: 40px
        tap_action:
          action: call-service
          service: script.dining_blinds_close
  
  # Slat tilt controls
  - type: horizontal-stack
    cards:
      - type: button
        name: Open Slats
        icon: mdi:angle-right
        tap_action:
          action: call-service
          service: script.dining_slats_open
      
      - type: button
        name: Close Slats
        icon: mdi:angle-acute
        tap_action:
          action: call-service
          service: script.dining_slats_close
  
  # Stop button
  - type: button
    name: STOP
    icon: mdi:stop
    icon_height: 30px
    tap_action:
      action: call-service
      service: script.dining_blinds_stop
```

## Wall Button Automations with State Tracking

```yaml
automation:
  # Button 1 - Short press opens blinds
  - alias: "Dining Blinds - Wall Button 1 Press"
    trigger:
      - platform: event
        event_type: zha_event  # Adjust based on your button type
        event_data:
          device_id: YOUR_BUTTON_1_DEVICE_ID
          command: "press"
    action:
      - service: script.dining_blinds_open

  # Button 1 - Long press goes to My position
  - alias: "Dining Blinds - Wall Button 1 Hold"
    trigger:
      - platform: event
        event_type: zha_event
        event_data:
          device_id: YOUR_BUTTON_1_DEVICE_ID
          command: "hold"
    action:
      - service: script.dining_blinds_my

  # Button 2 - Short press closes blinds
  - alias: "Dining Blinds - Wall Button 2 Press"
    trigger:
      - platform: event
        event_type: zha_event
        event_data:
          device_id: YOUR_BUTTON_2_DEVICE_ID
          command: "press"
    action:
      - service: script.dining_blinds_close

  # Button 2 - Long press also goes to My position
  - alias: "Dining Blinds - Wall Button 2 Hold"
    trigger:
      - platform: event
        event_type: zha_event
        event_data:
          device_id: YOUR_BUTTON_2_DEVICE_ID
          command: "hold"
    action:
      - service: script.dining_blinds_my
```

## Presence-Based Lighting Automation

Now you can use the state tracker in your lighting automation:

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
        before: sunrise
      - condition: state
        entity_id: input_select.dining_blinds_state
        state: "closed"  # Only turn on lights if blinds are closed
    action:
      - service: light.turn_on
        entity_id: light.dining_room
```

## Alternative: Binary Sensor for Closed State

If you prefer a simple binary sensor for "are blinds closed":

```yaml
# In configuration.yaml or templates.yaml
template:
  - binary_sensor:
      - name: "Dining Blinds Closed"
        state: "{{ states('input_select.dining_blinds_state') == 'closed' }}"
        device_class: window
        icon: >
          {% if states('input_select.dining_blinds_state') == 'closed' %}
            mdi:blinds-closed
          {% else %}
            mdi:blinds-open
          {% endif %}
```

Then in your automation:
```yaml
condition:
  - condition: state
    entity_id: binary_sensor.dining_blinds_closed
    state: "on"
```

## Advanced: State with Confidence

For more sophisticated tracking, you could also track how confident you are about the state:

```yaml
input_datetime:
  dining_blinds_last_command:
    name: Last Blind Command Time
    has_date: true
    has_time: true

# Update this in each script:
- service: input_datetime.set_datetime
  data:
    entity_id: input_datetime.dining_blinds_last_command
    datetime: "{{ now() }}"
```

Then create a template sensor that marks state as "unknown" if no command sent in last 24 hours (in case someone manually moved them).

## Notes

- **State Persistence**: The input_select survives restarts
- **Manual Override**: If someone uses the physical remote, state will be wrong until next command
- **My Position**: You might want to treat "my_position" as "closed" for lighting purposes
- **Multiple Blinds**: Create separate state trackers for each blind

This gives you reliable state tracking for your automations while keeping the control simple!