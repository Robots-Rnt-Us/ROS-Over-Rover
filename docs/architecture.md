# Workspace & Node Architecture — roverrobotics_ros2

This document provides a technical overview of the workspace structure, node data flow, and coordinate frames (TF) used in the Rover Robotics ROS 2 Driver Stack.

---

## 1. Workspace Packages Overview

The workspace consists of 4 focused ROS 2 packages rather than a monolithic application:

```mermaid
graph TD
    subgraph Packages [Rover Robotics ROS 2 Workspace]
        driver[roverrobotics_driver<br/>C++ hardware driver, SLAM/Nav2 configs]
        description[roverrobotics_description<br/>URDF Models, meshes, RViz setups]
        gazebo[roverrobotics_gazebo<br/>Gazebo simulation launch/worlds]
        input[roverrobotics_input_manager<br/>PS4/PS5 joystick controller map]
    end

    input -->|/cmd_vel| driver
    gazebo -->|/clock /scan /odom| driver
    description -->|/robot_description| driver
    description -->|/robot_description| gazebo
```

| Package Name | Path | Role |
| :--- | :--- | :--- |
| **`roverrobotics_driver`** | [`roverrobotics_driver/`](file:///Users/aman.gupta/Documents/GitHub/roverrobotics_ros2/roverrobotics_driver) | C++ nodes interfacing with physical hardware via CAN/Serial, loading per-robot parameters, EKF localization, and launching SLAM/Nav2. |
| **`roverrobotics_description`** | [`roverrobotics_description/`](file:///Users/aman.gupta/Documents/GitHub/roverrobotics_ros2/roverrobotics_description) | Houses 2WD/4WD/Flipper robot descriptions (URDF/Xacro), payload mounting links, sensor offset definitions, and 3D visual/collision meshes (`.dae`). |
| **`roverrobotics_gazebo`** | [`roverrobotics_gazebo/`](file:///Users/aman.gupta/Documents/GitHub/roverrobotics_ros2/roverrobotics_gazebo) | Integration folders with Gazebo simulator world definitions, plugins for differential and skid-steer drive simulation, and simulated sensor spawners. |
| **`roverrobotics_input_manager`** | [`roverrobotics_input_manager/`](file:///Users/aman.gupta/Documents/GitHub/roverrobotics_ros2/roverrobotics_input_manager) | Maps joystick hardware axes and buttons (DS4/DS5) into standardized velocity commands (`geometry_msgs/msg/Twist`). |

---

## 2. Driver Node Data Flow

The `roverrobotics_driver` node acts as a bridge between the ROS 2 computational graph and the physical robot controller (VESC/serial board/CAN channel).

```mermaid
flowchart LR
    subgraph Inputs [ROS 2 Graph Input]
        teleop_cmd["/cmd_vel<br/>(geometry_msgs/Twist)"]
        nav_cmd["/cmd_vel_nav<br/>(geometry_msgs/Twist)"]
    end

    subgraph Node [roverrobotics_driver Node]
        subgraph Lib [librover C++ Library]
            protocol_sel{Robot Protocol Selection}
            pro_proto[ProProtocolObject]
            zero2_proto[Zero2ProtocolObject]
            mini_2wd_proto[Mini2WDProtocolObject]
            diff_robot[DifferentialRobot <br/> CAN/VESC]
        end
    end

    subgraph Output [Hardware Interfaces]
        serial_dev["Serial Port<br/>(/dev/rover-*)"]
        can_bus["CAN Channel<br/>(can0)"]
    end

    subgraph Publisher [ROS 2 Graph Output]
        odom["/odometry/wheels<br/>(nav_msgs/Odometry)"]
        tf_pub["/tf<br/>(TransformStamped)"]
    end

    teleop_cmd & nav_cmd --> protocol_sel
    protocol_sel -->|pro| pro_proto
    protocol_sel -->|zero2| zero2_proto
    protocol_sel -->|mini_2wd| mini_2wd_proto
    protocol_sel -->|mini/miti/max/mega| diff_robot

    pro_proto & zero2_proto & mini_2wd_proto -->|RS232/USB| serial_dev
    diff_robot -->|SocketCAN| can_bus

    serial_dev & can_bus -->|Status Bytes| Lib
    Lib -->|Encoder ticks / RPM| odom
    Lib -->|odom -> base_link| tf_pub
```

---

## 3. Coordinate Frame (TF) Tree

A standard configuration provides coordinate frames starting from the navigation map down to individual sensor links.

```mermaid
graph TD
    map[map]
    odom[odom]
    base_link[base_link]
    chassis_link[chassis_link]
    payload_link[payload_link]
    
    subgraph Wheels [Drivetrain Links]
        fl[front_left_wheel_link]
        fr[front_right_wheel_link]
        rl[rear_left_wheel_link]
        rr[rear_right_wheel_link]
    end
    
    subgraph Sensors [Attached Sensors]
        lidar[lidar_link]
        imu[imu_link]
        camera[camera_link]
    end

    map -->|slam_toolbox| odom
    odom -->|robot_localization EKF| base_link
    base_link -->|fixed| chassis_link
    base_link -->|fixed| payload_link
    
    chassis_link --> fl
    chassis_link --> fr
    chassis_link --> rl
    chassis_link --> rr
    
    payload_link -->|accessories.yaml| lidar
    payload_link -->|accessories.yaml| imu
    payload_link -->|accessories.yaml| camera
```

*   **`map` $\rightarrow$ `odom`**: Handled by localization/mapping nodes such as `slam_toolbox`.
*   **`odom` $\rightarrow$ `base_link`**: Broadcast by `robot_state_publisher` using wheel odometry fused with IMU (via `robot_localization` EKF nodes).
*   **`base_link` $\rightarrow$ `chassis_link` / `payload_link`**: Static transforms set by the robot’s URDF xacro configurations inside the `roverrobotics_description` package.
*   **`payload_link` $\rightarrow$ Sensors**: Custom configurations set up using `accessories.launch.py` and parameters from `accessories.yaml`.
