# ESPSomfy Vertical Blinds Controller - Project Status

## Current Implementation (Simplified State Tracker)

### Overview
Simplified to a state-tracking blueprint that listens for events and updates position helpers. No timing calculations or complex positioning - just Open, Close, and My positions.

### Architecture
1. **Blueprint** (`vertical_blinds_simple.yaml`): Listens for `vertical_blinds_control` events and updates state
2. **Scripts**: Send ESPSomfy commands AND fire events to update state
3. **State Tracking**: `input_select` helpers track position (open/closed/my/unknown)
4. **Dashboard**: Buttons call scripts that handle both control and state

### Working Components

#### Scripts (per blind)
```yaml
dining_blinds_open:
  sequence:
    - service: espsomfy_rts.send_command
      data:
        entity_id: cover.dining_blinds
        command: "Up"
    - event: vertical_blinds_control
      event_data:
        blind_id: "dining"
        command: "open"

dining_blinds_close:
  sequence:
    - service: espsomfy_rts.send_command
      data:
        entity_id: cover.dining_blinds
        command: "Down"
    - delay:
        seconds: 30  # ONLY on close - for lighting automation
    - event: vertical_blinds_control
      event_data:
        blind_id: "dining"
        command: "close"

dining_blinds_my:
  sequence:
    - service: espsomfy_rts.send_command
      data:
        entity_id: cover.dining_blinds
        command: "My"
    - event: vertical_blinds_control
      event_data:
        blind_id: "dining"
        command: "my"

dining_slats_open:
  sequence:
    - service: espsomfy_rts.send_step_command
      data:
        entity_id: cover.dining_blinds
        direction: "Up"  # NOT "StepUp"
        step_size: 100

dining_slats_close:
  sequence:
    - service: espsomfy_rts.send_step_command
      data:
        entity_id: cover.dining_blinds
        direction: "Down"  # NOT "StepDown"
        step_size: 100
```

#### State Helpers
- `input_select.dining_blinds_position` - Options: open, closed, my, unknown
- `input_select.lounge_blinds_position` - Options: open, closed, my, unknown

#### Automations Created
1. **Dynamic Presence Night Control** - Toggles `switch.dynamic_presence_dining_night_manual_on` based on blind state after 23:00
2. **Southwest Sun Protection** - Moves blinds to My position when sun azimuth is 225-270°
3. **Morning Open** - Opens blinds at 8:00 AM

### Key Lessons Learned

#### What Works
- Simple state tracking via events
- Scripts that handle both control and state update
- Delay ONLY on close command (for lighting)
- Using My position for frequently used setting

#### What Doesn't Work (RTS Limitations)
- No position feedback (one-way protocol)
- My position is stored per remote (ESPSomfy is separate from physical remote)
- Radio latency varies unpredictably
- Can't have multiple My positions
- My position saves both height AND slat angle
- Timing-based positioning is unreliable due to:
  - Variable radio latency
  - Soft start/stop motor characteristics
  - No feedback to confirm position reached

#### Critical Mistakes to Avoid
- DON'T add delays to all scripts (blocks execution)
- DON'T use cover.* services - use espsomfy_rts.send_command
- DON'T use "StepUp"/"StepDown" - use "Up"/"Down" with send_step_command
- DON'T forget to program My position FROM ESPSomfy (not just physical remote)
- DON'T make blueprint complex - keep it simple for state tracking only

### Issues Encountered & Solutions

1. **Blueprint import "unknown error"**: Repository was private (must be public)
2. **Lounge My button not working**: My position must be programmed per remote
3. **Commands appearing to fail randomly**: Delays in scripts were blocking execution
4. **Step commands failing**: Direction must be "Up"/"Down" not "StepUp"/"StepDown"

### Future Improvements

#### Stepper Motor Conversion (Planned)
Replace RTS motors with DIY stepper solution:
- ESP32 + TMC2209 driver (silent operation)
- NEMA 17 stepper motor
- Hall effect sensor or encoder for position feedback
- ESPHome integration for full position control
- Reuse IQ2 motor housing and gears

Benefits:
- Exact position control (go to 37% exactly)
- Position feedback (always knows position)
- Unlimited presets
- No radio latency issues
- Silent operation with TMC2209

### Current File Structure
```
vertical-blinds-blueprint/
├── vertical_blinds_simple.yaml  # State tracking blueprint
├── simple_control.md            # Control setup documentation
├── simple_config.yaml           # Configuration examples
├── README.md                    # User documentation
└── CLAUDE.md                    # This file (project context)
```

### ESPSomfy Configuration
- Blind type: Blind (not shade)
- Tilt type: Integrated
- Invert position: OFF
- Entities created: cover.dining_blinds, cover.lounge_blinds

### Wall Switch Setup
Wall switches call the same scripts as dashboard buttons - single source of truth.

## Summary
Project pivoted from complex timing-based positioning to simple 3-position control with state tracking. This works reliably within RTS limitations. Future upgrade path is stepper motor conversion for true position control.