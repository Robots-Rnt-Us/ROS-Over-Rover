# Project Memory — roverrobotics_ros2

Persistent facts for agents. Update this file when discovering stable conventions or fixing recurring confusion.

## Stack

- **ROS 2 Humble** on Ubuntu 22.04 (primary); branches exist for Foxy and Jazzy.
- **Build:** `ament_cmake` + `colcon`; vendored `librover` compiled into the driver binary (not a separate `.so`).
- **Language:** C++17 driver, Python 3 launch/teleop.

## Protocol mapping (`robot_type` in config YAML)

| `robot_type` | Protocol class | Typical comm |
|--------------|----------------|--------------|
| `pro` | ProProtocolObject | varies |
| `zero2` | Zero2ProtocolObject | varies |
| `mini_2wd` | Mini2WDProtocolObject | serial only |
| `mini`, `miti`, `max`, `mega` | DifferentialRobot | CAN (`can0`) or serial |

## Launch pattern (every hardware robot)

1. `roverrobotics_driver` node with `config/<robot>_config.yaml`
2. `accessories.launch.py` (sensors from `accessories.yaml`)
3. `joint_state_publisher` + `robot_state_publisher` with URDF

Teleop launches add `ps4_controller.launch.py` or `ps5_controller.launch.py`.

## Known quirks

- README mentions `indoor_miti` launch — **does not exist**; use `miti` or `miti_65`.
- Mini/Miti default to CAN; serial requires `comm_type: serial` and correct `/dev/tty*`.
- `publish_tf: false` by default; localization stacks may enable TF elsewhere.
- Recommended install path uses external [rover_install_scripts_ros2](https://github.com/RoverRobotics/rover_install_scripts_ros2) (udev, CAN, systemd).

## Key topics

- Subscribe: `/cmd_vel` (configurable via `speed_topic`)
- Publish: `/odometry/wheels` (configurable via `odom_topic`)
- Frames: `odom` → `base_link` (when TF enabled)

## Package version

All four packages at **1.0.2**, Apache-2.0.
