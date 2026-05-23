---
name: nav-slam-setup
description: Configure SLAM, Nav2, and robot_localization for Rover robots. Use when setting up mapping, navigation stacks, outdoor localization, or integrating accessories for nav.
---

# Nav & SLAM Setup

## SLAM (slam_toolbox)

```bash
ros2 launch roverrobotics_driver slam_launch.py
```

Requires lidar (configure in `accessories.yaml` → RPLidar S2).

## Indoor Nav2

```bash
ros2 launch roverrobotics_driver navigation_launch.py map_file_name:=/path/to/map.yaml
```

## Outdoor Nav2

```bash
ros2 launch roverrobotics_driver outdoor_navigation_launch.py map_file_name:=/path/to/map.yaml
```

## Localization

```bash
ros2 launch roverrobotics_driver robot_localizer.launch.py        # indoor EKF
ros2 launch roverrobotics_driver robot_localizer_outdoor.launch.py # outdoor GPS fusion
```

## Config locations

- SLAM: `roverrobotics_driver/config/slam_toolbox_*.yaml`
- Nav2: `roverrobotics_driver/config/nav2_*.yaml`
- EKF: `roverrobotics_driver/config/ekf_*.yaml`
- Maps: `roverrobotics_driver/maps/`

## Typical stack

```
Robot driver → /odometry/wheels
Accessories  → /scan (lidar), /imu (BNO055)
robot_localization → fused odom
slam_toolbox OR nav2 → mapping / planning
```

## Prerequisites

1. Robot launch running (driver + state publishers)
2. Lidar enabled in `accessories.yaml` for SLAM/Nav2
3. TF tree: `odom` → `base_link` → sensor frames
