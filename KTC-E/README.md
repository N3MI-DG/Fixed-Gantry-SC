# Klipper-Toolchanger-Easy

To get fixed gantry working on klipper-toolchanger-easy, we need a custom pickup and dropoff path per `tool config` and custom pickup and dropoff gcode in the `[toolchanger]` section.

## Example Tool Sections
The pickup and dropoff paths for the tools need to mirrored. To do that we add a custom path to the `tool config`.

### T0

```
[tool T0]
tool_number: 0
extruder: extruder
fan: T0_part_fan
detection_pin: !nhk0:gpio10
params_park_x: 1.7
params_park_y: 0.8
params_park_z: 1
gcode_x_offset: 0
gcode_y_offset: 0
gcode_z_offset: 0.08
params_dropoff_path: [{'x':22, 'y':16}, {'x':22, 'y':0}, {'x':0, 'y':0}]
params_pickup_path: [{'x':0, 'y':0}, {'x':22, 'y':0}, {'x':22, 'y':16, 'verify':1}]
```

### T1

```
[tool T1]
tool_number: 1
extruder: extruder1
fan: T1_part_fan
detection_pin: !nhk1:gpio10
params_park_x: 302.7
params_park_y: 1.0
params_park_z: 0
gcode_x_offset: -0.290
gcode_y_offset: -1.240
gcode_z_offset: -0.60
params_dropoff_path: [{'x':-22, 'y':16}, {'x':-22, 'y':0}, {'x':0, 'y':0]
params_pickup_path: [{'x':0, 'y':0}, {'x':-22, 'y':0}, {'x':-22, 'y':16, 'verify':1}]
```

## Toolchanger Section
To add custom pickup and dropoff gcodes to the `[toolchanger]` section, add the following to the bottom of the `[toolchanger]` section. This will override the original pickup and dropoff gcodes which are intended for a Voron 2.4.

```
dropoff_gcode:
  {% set x = tool.params_park_x|float %}
  {% set y = tool.params_park_y|float %}
  {% set z = tool.params_park_z|float %}
  {% set fast = tool.params_fast_speed|float %}
  {% set path = tool.params_dropoff_path %}
  {% set max_z = printer.configfile.config["stepper_z"]["position_max"]|float %}
  {% set cur_z = printer.toolhead.position[2]|float %}
  RESPOND TYPE=echo MSG='Dropping off {tool.name}'
  G90
  ; Move 1 mm up to avoid crashing into things
  G0 Z{ [cur_z+1.0, max_z]|min } F{fast}
  #   ##############  Move up to the dock  ##############
  ROUNDED_G0 X={x + path[0]['x']|float} D=150 F={fast}
  ROUNDED_G0 Y={y + path[0]['y']|float} D=0 F={fast}
  STOP_CRASH_DETECTION
  #  ############## Run the path ##############
  {% for pos in path %}
    {% set speed = tool.params_path_speed|float * (pos.get('f', 1.0)|float) %}
    G0 {% if 'x' in pos %}X{x + pos['x']|float}{% endif %} {% if 'y' in pos %}Y{y + pos['y']|float}{% endif %} {% if 'z' in pos %}Z{z + pos['z']|float }{% endif %} F{speed}
  {% endfor %}
  ## Return to safe Y if no pickup_tool
  {% if pickup_tool is none %}
    G0 Y{tool.params_safe_y} F{fast}
  {% endif %}

pickup_gcode:
  {% set x = tool.params_park_x|float %}
  {% set y = tool.params_park_y|float %}
  {% set z = tool.params_park_z|float %}
  {% set fast = tool.params_fast_speed|float %}
  {% set path = tool.params_pickup_path %}
  RESPOND TYPE=echo MSG='Picking up {tool.name}'
  G90
  #   ##############  Fast to the last point  ##############
  ROUNDED_G0 Y={tool.params_close_y} F={fast} D=5
  # ROUNDED_G0 X={x} Z={z + path[0]['z']|float} F={fast} D=5
  ROUNDED_G0 X={x} F={fast} D=5
  ROUNDED_G0 Y={y + path[0]['y']|float} F={fast} D=0
  # Wait for temp before actually picking up the tool, while the nozzle is resting on it's pad.
  {% if tool.extruder %}
    M109 T{tool.tool_number} S{printer[tool.extruder].target|int}
  {% endif %}
  # Run the path
  {% for pos in path %}
    {% set speed = tool.params_path_speed|float * (pos.get('f', 1.0)|float) %}
    G0 {% if 'x' in pos %}X{x + pos['x']|float}{% endif %} {% if 'y' in pos %}Y{y + pos['y']|float}{% endif %} {% if 'z' in pos %}Z{z + pos['z']|float }{% endif %} F{speed}
    {% if 'verify' in pos %}
      VERIFY_TOOL_DETECTED T={tool.tool_number}
    {% endif %}
  {% endfor %}
  # Restore the position with smooth rounded move.
  ROUNDED_G0 Y={tool.params_safe_y} F={fast} D=20
  ROUNDED_G0 D=0
```
