# Installation

This section covers installing and building the OSCP ROS2 Driver stack, including dependencies, workspace setup, and hardware permissions required for IMU communication.

---

## Prerequisites

This driver is developed and tested on:

- ROS 2 Humble or Jazzy (Ubuntu 22.04-24.04)
- Python 3.10+
- C++17 toolchain

Clone inside your ROS 2 workspace:

```bash
cd ~/ros2_ws/src

git clone https://github.com/OSCPS/oscp_ros2_driver.git
```
Install dependencies

```bash
cd ..

rosdep update

rosdep install --from-paths src -y --ignore-src --rosdistro $ROS_DISTRO
```
Build and source workspace to verify installation

```bash
colcon build --packages-select oscp_msgs oscp_libraries oscp_imu_ros2 --symlink-install

source install/setup.bash
```

---
