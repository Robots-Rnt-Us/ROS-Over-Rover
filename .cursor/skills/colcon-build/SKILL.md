---
name: colcon-build
description: Build and source the Rover Robotics ROS2 workspace with colcon. Use when building packages, verifying compile errors, or setting up the workspace after code changes.
---

# Colcon Build — roverrobotics_ros2

## Prerequisites

- Ubuntu 22.04 + ROS 2 Humble sourced
- Repo cloned into `ros2_ws/src/roverrobotics_ros2`

## Full workspace build

```bash
cd ~/ros2_ws
source /opt/ros/humble/setup.bash
colcon build
source install/setup.bash
```

## Single-package builds (faster iteration)

```bash
colcon build --packages-select roverrobotics_driver
colcon build --packages-select roverrobotics_description roverrobotics_gazebo roverrobotics_input_manager
```

## After driver C++ changes

Rebuild driver only, then re-source:

```bash
colcon build --packages-select roverrobotics_driver --cmake-args -DCMAKE_BUILD_TYPE=RelWithDebInfo
source install/setup.bash
```

## Verify install

```bash
ros2 pkg list | grep roverrobotics
ros2 run roverrobotics_driver roverrobotics_driver --help
```

## Common failures

| Error | Fix |
|-------|-----|
| Package not found | Re-source `install/setup.bash` |
| Missing Eigen3/rclcpp | Install ros-humble-desktop and `rosdep install --from-paths src --ignore-src -r -y` |
| librover link errors | Ensure new `.cpp` files are in `CMakeLists.txt` |
