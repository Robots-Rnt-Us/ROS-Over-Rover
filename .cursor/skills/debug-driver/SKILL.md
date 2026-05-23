---
name: debug-driver
description: Diagnose Rover Robotics driver communication, odometry, and cmd_vel issues. Use when the robot does not move, odometry is wrong, CAN/serial fails, or driver node crashes on startup.
---

# Debug Rover Driver

## Startup checks

```bash
ros2 node list | grep roverrobotics
ros2 topic echo /odometry/wheels --once
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.1}, angular: {z: 0.0}}" -r 10
```

## Config inspection

Read `roverrobotics_driver/config/<robot>_config.yaml`:

| Symptom | Check |
|---------|-------|
| No movement | `comm_type`, `device_port`, `robot_type` |
| Wrong odometry | `wheel_radius`, `wheel_base`, `gear_ratio` |
| No TF | `publish_tf`, frame IDs |
| CAN errors | `ip link show can0`, udev/install script |
| Serial errors | `ls /dev/tty*`, permissions, `comm_type: serial` |

## CAN (Mini/Miti/Max/Mega)

```bash
ip link show can0
# Expected: UP, bitrate configured by install script
```

## Logs

Driver runs with `output='screen'` in launches. Look for protocol init errors and comm timeouts.

## Code paths

- Main node: `roverrobotics_driver/src/roverrobotics_ros2_driver.cpp`
- Protocols: `library/librover/src/protocol_*.cpp`, `differential_robot.cpp`
- Comm: `comm_can.cpp`, `comm_serial.cpp`, `vesc.cpp`

## Sim fallback

If hardware unavailable:

```bash
ros2 launch roverrobotics_gazebo mini_gazebo.launch.py
```
