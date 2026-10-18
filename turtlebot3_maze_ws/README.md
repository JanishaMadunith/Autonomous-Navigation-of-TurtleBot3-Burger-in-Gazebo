# Autonomous Navigation in a 6×6 Maze Using ROS 2 and Gazebo

## 1. Project Overview

This project implements a 6×6 maze simulation for autonomous mobile robot navigation using ROS 2 Jazzy, Gazebo, and the TurtleBot3 Burger robot.

The maze environment is defined in an SDF world file. A previously generated occupancy map is used for localization and navigation with the ROS 2 Navigation Stack (Nav2).

## 2. Project Structure

maze-autonomous-navigation/
├── worlds/
│   └── maze_6x6_centered.world
├── maps/
│   ├── maze_6x6_map.yaml
│   └── maze_6x6_map.pgm
└── README.md


## 3. Requirements

Install or configure the following software in Ubuntu 24.04:

- ROS 2 Jazzy
- Gazebo Sim
- TurtleBot3 packages
- ROS-Gazebo bridge
- TurtleBot3 Cartographer package (if generating a map)
- TurtleBot3 Navigation2 package (Nav2)

## 4. Environment Setup

Open a terminal and source ROS 2:

```bash
source /opt/ros/jazzy/setup.bash
```

Set the robot model:

```bash
export TURTLEBOT3_MODEL=burger
```

Configure Gazebo to locate the TurtleBot3 models:

```bash
export GZ_SIM_RESOURCE_PATH="/opt/ros/jazzy/share/turtlebot3_gazebo/models:/opt/ros/jazzy/share"
```

Run these setup commands in each new terminal before launching the relevant ROS 2 components.

## 5. Running the Maze Simulation

Navigate to the directory containing the downloaded repository. Start Gazebo using the maze world:

```bash
gz sim -r worlds/maze_6x6_centered.world
```

If the world file is stored elsewhere, provide its absolute path.

## 6. Spawning the TurtleBot3 Burger

In a second terminal, source ROS 2 and configure the model and Gazebo resources:

```bash
source /opt/ros/jazzy/setup.bash
export TURTLEBOT3_MODEL=burger
export GZ_SIM_RESOURCE_PATH="/opt/ros/jazzy/share/turtlebot3_gazebo/models:/opt/ros/jazzy/share"
```

## Spawn the robot into the running Gazebo world:

```bash
ros2 run ros_gz_sim create \
  --world robot_maze \
  --file /opt/ros/jazzy/share/turtlebot3_gazebo/models/turtlebot3_burger/model.sdf \
  --name burger \
  --x 0.5 --y 0.5 --z 0.01
```

If the robot is already present in the simulation, do not spawn another copy.

## 7. Starting the ROS-Gazebo Bridge

In another terminal:

```bash
source /opt/ros/jazzy/setup.bash

ros2 run ros_gz_bridge parameter_bridge --ros-args \
  -p config_file:=/opt/ros/jazzy/share/turtlebot3_gazebo/params/turtlebot3_burger_bridge.yaml
```

This bridge connects the required simulation topics with ROS 2.

## 8. Running Robot State Publisher

Open another terminal:

```bash
source /opt/ros/jazzy/setup.bash
export TURTLEBOT3_MODEL=burger

ros2 launch turtlebot3_gazebo robot_state_publisher.launch.py
```

## 9. Running Autonomous Navigation

Once Gazebo is running, the robot is spawned, and the required ROS 2 topics are available, start Nav2 using the saved occupancy map.

```bash
source /opt/ros/jazzy/setup.bash
export TURTLEBOT3_MODEL=burger

ros2 launch turtlebot3_navigation2 navigation2.launch.py \
  use_sim_time:=True \
  map:="$(realpath maps/maze_6x6_map.yaml)"
```

Run this command from the repository root so that the relative map path resolves correctly.

In RViz:

1. Set the fixed frame to `map`.
2. Use **2D Pose Estimate** to initialize the robot's position and orientation.
3. Use **Nav2 Goal** to select a destination inside the maze.
4. Observe the planned route and robot movement.

The robot's localization, map alignment, and Nav2 lifecycle nodes must be functioning correctly before navigation goals can execute.

## 10. Generating a New Map (Optional)

If you need to generate a new map instead of using the supplied map, start the simulation and robot, then run Cartographer in a separate terminal:

```bash
source /opt/ros/jazzy/setup.bash
export TURTLEBOT3_MODEL=burger

ros2 launch turtlebot3_cartographer cartographer.launch.py \
  use_sim_time:=True
```

Move the robot through the maze to collect sensor data. Save the resulting map using the map-saving utility available in your ROS 2 installation, for example:

```bash
ros2 run nav2_map_server map_saver_cli \
  -f maps/maze_6x6_map
```

Do not run Cartographer and the saved-map Nav2 workflow simultaneously unless you have deliberately configured them to work together.

## 11. Troubleshooting

- **Gazebo cannot find models:** Check `GZ_SIM_RESOURCE_PATH`.
- **Robot does not appear:** Verify the Gazebo world name and robot model path.
- **Map does not load:** Check that the YAML file references the correct `.pgm` image.
- **Robot does not move to a goal:** Check localization, TF transforms, sensor topics, and the Nav2 lifecycle states.
- **Simulation time errors:** Avoid resetting the simulation clock while localization or mapping nodes are running.
- **Command not found:** Confirm that the required ROS 2 package is installed and source `/opt/ros/jazzy/setup.bash`.

## 12. Notes

This repository contains the maze world and saved occupancy map. The robot simulation, localization, and navigation rely on the installed ROS 2 Jazzy, Gazebo, TurtleBot3, and Nav2 packages. Custom launch files or ROS 2 packages are not included unless explicitly added to this repository.
