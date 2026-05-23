---
name: add-robot-launch
description: Add or modify a Rover robot launch file, config YAML, and URDF wiring. Use when adding a new robot platform, duplicating launch patterns, or connecting driver config to description models.
---

# Add Robot Launch

## Checklist

```
- [ ] Config YAML in roverrobotics_driver/config/
- [ ] Launch file in roverrobotics_driver/launch/
- [ ] Optional teleop launch (*_teleop.launch.py)
- [ ] URDF in roverrobotics_description/urdf/ (if new model)
- [ ] Optional Gazebo launch in roverrobotics_gazebo/launch/
- [ ] CMakeLists.txt install rules (if new config/launch files)
```

## Step 1 — Config YAML

Copy nearest robot (e.g. `mini_config.yaml` → `newbot_config.yaml`).

Set:
- `robot_type` — must match driver protocol support
- `comm_type` / `device_port` — `can` + `can0` or `serial` + `/dev/tty*`
- Kinematics: `wheel_radius`, `wheel_base`, `robot_length`, `gear_ratio`

## Step 2 — Launch file

Copy `mini.launch.py` pattern:

1. Resolve URDF via `get_package_share_path('roverrobotics_description')`
2. Point driver to new config YAML
3. Include `accessories.launch.py`
4. Start joint + robot state publishers

## Step 3 — Teleop (optional)

Copy `mini_teleop.launch.py`; include robot launch + `ps4_controller.launch.py`.

## Step 4 — Gazebo (optional)

Copy `mini_gazebo.launch.py` in `roverrobotics_gazebo/launch/`.

## Step 5 — Verify

```bash
colcon build --packages-select roverrobotics_driver roverrobotics_description
source install/setup.bash
ros2 launch roverrobotics_driver newbot.launch.py
```

Use Gazebo launch if no hardware available.
