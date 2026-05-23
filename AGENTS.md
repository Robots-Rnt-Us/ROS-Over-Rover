# Agent Guide — roverrobotics_ros2

This repository is the official **Rover Robotics ROS 2 driver stack** (Humble-focused). AI agents should treat it as a four-package colcon workspace, not a monolithic app.

## Quick orientation

| Package | Path | Role |
|---------|------|------|
| `roverrobotics_driver` | `roverrobotics_driver/` | C++ hardware driver, configs, SLAM/Nav2 launches |
| `roverrobotics_description` | `roverrobotics_description/` | URDF, meshes, RViz display |
| `roverrobotics_gazebo` | `roverrobotics_gazebo/` | Gazebo worlds and sim launches |
| `roverrobotics_input_manager` | `roverrobotics_input_manager/` | PS4/PS5 joystick teleop |

**Robots:** Zero, Pro, Mini, Mini 2WD, Miti, Miti 65, Max, Mega (+ Flipper in sim/URDF).

## Before making changes

1. Read `.cursor/MEMORY.md` for stable project facts and known quirks.
2. Match existing patterns — copy the closest robot launch/config pair before inventing new structure.
3. Keep diffs minimal; do not refactor unrelated code.
4. Branch per ROS distro: `main` (dev), `humble`/`jazzy`/`foxy` (stable).

## Build & test

```bash
# From ros2_ws root (after cloning into src/)
source /opt/ros/humble/setup.bash
colcon build --packages-select roverrobotics_driver roverrobotics_description roverrobotics_gazebo roverrobotics_input_manager
source install/setup.bash
```

Hardware testing requires a physical robot; use Gazebo launches for sim validation.

## Common tasks → skills

| Task | Skill |
|------|-------|
| Build, source, package-select builds | `.cursor/skills/colcon-build/SKILL.md` |
| Add or modify robot launch + config | `.cursor/skills/add-robot-launch/SKILL.md` |
| Driver comm/odometry/cmd_vel issues | `.cursor/skills/debug-driver/SKILL.md` |
| SLAM, Nav2, localization setup | `.cursor/skills/nav-slam-setup/SKILL.md` |

## Architecture & Visual Documentation

For detailed node diagrams, coordinate frame trees, communications protocols, and input manager translation pipelines, see the following:
*   [docs/architecture.md](file:///Users/aman.gupta/Documents/GitHub/roverrobotics_ros2/docs/architecture.md) — Node architecture, package layout, and global TF Trees.
*   [docs/hardware_protocols.md](file:///Users/aman.gupta/Documents/GitHub/roverrobotics_ros2/docs/hardware_protocols.md) — Serial packet maps, checksum formulas, and VESC CAN layouts.
*   [docs/input_manager_teleop.md](file:///Users/aman.gupta/Documents/GitHub/roverrobotics_ros2/docs/input_manager_teleop.md) — Gamepad joystick configurations, deadzones, and dynamic throttle scaling.

```
/cmd_vel ──► roverrobotics_driver ──► librover (CAN/serial/VESC)
                    │
                    ├──► /odometry/wheels
                    └──► TF (optional)

robot_state_publisher ◄── URDF (roverrobotics_description)
accessories.launch.py ◄── RPLidar, BNO055, RealSense (config-driven)
```


## File ownership

- **Driver logic:** `roverrobotics_driver/src/`, `library/librover/`
- **Per-robot params:** `roverrobotics_driver/config/*_config.yaml`
- **Launches:** `roverrobotics_driver/launch/`, `roverrobotics_gazebo/launch/`
- **Models:** `roverrobotics_description/urdf/`, `meshes/`

## Cursor rules (auto-applied)

- `.cursor/rules/project-overview.mdc` — always on
- `.cursor/rules/cpp-driver.mdc` — C++ / librover
- `.cursor/rules/python-launch.mdc` — launch files & teleop
- `.cursor/rules/ros-config.mdc` — YAML configs

## Commit conventions

Use imperative, scoped messages: `fix(driver): …`, `feat(launch): …`, `docs: …`. Do not commit unless asked.
