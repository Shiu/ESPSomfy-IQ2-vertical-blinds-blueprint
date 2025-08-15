# Project: ESPSomfy Vertical Blinds Controller

## Overview
This project provides a Home Assistant blueprint for controlling vertical blinds with Benthin IQ2 motors via ESPSomfy RTS. It works around ESPSomfy's timing limitations and the IQ2 motor's unique behavior.

## Problem Being Solved
Benthin IQ2 motors for vertical blinds have specific characteristics that don't work well with ESPSomfy's standard position calculations:
- **Soft start/stop**: Motor ramps up/down speed, making timing non-linear
- **180° slat rotation**: After reaching closed position, motor continues to rotate slats
- **Sequential operation**: Position movement first, then slat rotation (opposite of horizontal blinds)
- **Timing drift**: Multiple movements accumulate timing errors due to soft start

## Solution Approach
The blueprint bypasses ESPSomfy's position calculations entirely and uses direct motor control with timed movements:
1. Send RTS commands directly (Up/Down/My)
2. Use precise millisecond delays
3. Track position in Home Assistant (not ESPSomfy)
4. Self-contained scripts with embedded timings

## Key Design Decisions

### Why Not Modify ESPSomfy Code?
- ESPSomfy is designed for horizontal blinds with different tilt behavior
- IQ2 motor's soft start makes position calculations non-linear
- Easier to work around limitations than redesign core functionality
- Blueprint solution is non-invasive and easily shareable

### Why Event-Based Automation?
- Only one automation per blind (not 3 scripts)
- No scripts needed at all
- Flexible - can support any position value
- All configuration in one place per blind
- Each blind can have completely different timings

## Blueprint Architecture

### Components
1. **Position Tracker** (input_select): Tracks current position (0, 33/50, 100)
2. **Automation** (1 per blind): Listens for events and controls movement
3. **Event System**: Fire `vertical_blinds_move` events with blind_id and position
4. **Timings** (embedded): All movement times configured in the automation

### Movement Logic
- Automation listens for `vertical_blinds_move` events
- Filters by `blind_id` to identify target blind
- Checks current position from tracker
- Calculates which movement path to use
- Sends appropriate RTS command (Up/Down)
- Waits precise timing
- Sends stop command (My)
- Updates position tracker

### Position Reset Option
- Can optionally always go to 100% first
- Ensures known position before movement
- Prevents accumulating drift over time

## Configuration

### ESPSomfy Settings
```
Type: Blind
Tilt Type: None (or Integrated)
Up Time: ~30000ms (measure actual)
Down Time: ~30000ms (measure actual)
Invert Position: OFF (fixed bug in code)
```

### Home Assistant Setup
1. Create position tracker (input_select) with position options
2. Import blueprint
3. Create ONE automation per blind using the blueprint
4. Configure blind_id and timings in the automation
5. Add dashboard buttons that fire events

## Testing Procedures

### Timing Measurement
1. Start at 100% open
2. Time movement to each position
3. Average multiple measurements
4. Add buffer for soft stop

### Commands Used
- `espsomfy_rts.send_command` with "Up"/"Down"/"My"
- `send_step_command` for slat rotation (future enhancement)

## Known Issues & Workarounds

### Issue: Position Drift
**Workaround**: Enable "Always Reset" option to go to 100% before each movement

### Issue: Soft Start Timing
**Workaround**: Measure actual times, don't calculate. Each movement path needs individual timing.

### Issue: 0% Auto-Rotation
**Workaround**: Use 1% as "closed" position to avoid automatic slat rotation

## Future Enhancements
- [ ] Add slat rotation control via step commands
- [ ] Support for 4+ position presets
- [ ] Automatic timing calibration
- [ ] Position feedback from ESPSomfy (if available)

## Repository Structure

**IMPORTANT**: This project contains TWO separate git repositories:
1. `/home/shiu/projects/espsomfy/ESPSomfy-fork/` - ESPSomfy code (reference only, DO NOT commit here)
2. `/home/shiu/projects/espsomfy/vertical-blinds-blueprint/` - Blueprint repository (ONLY commit here)

The ESPSomfy folder is kept for code reference when needed. All commits should ONLY be made in the `vertical-blinds-blueprint` folder.

## File Structure
```
vertical-blinds-blueprint/    # Git repository - COMMIT HERE
├── vertical_blinds_controller.yaml  # The blueprint
├── README.md                        # User documentation
├── CLAUDE.md                        # This file
└── examples/
    └── configuration_example.yaml   # Example setup
```

## Commands Reference

### Direct Motor Control
```yaml
# Start movement
service: espsomfy_rts.send_command
data:
  entity_id: cover.dining_blinds
  command: "Down"  # or "Up"

# Stop movement
service: espsomfy_rts.send_command
data:
  entity_id: cover.dining_blinds
  command: "My"
```

### Step Commands (for slat rotation)
```yaml
service: espsomfy_rts.send_step_command
data:
  entity_id: cover.dining_blinds
  direction: "StepDown"  # or "StepUp"
  step_size: 50
```

## Development Notes
- Blueprint uses Home Assistant's `!input` tags for user configuration
- Scripts run in `parallel` mode to allow multiple blinds to move simultaneously
- Position tracking uses string comparison to handle "0" vs 0 type differences
- All timings in milliseconds for precision