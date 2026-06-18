# Launch & Usage

This section explains how to run the OSCP ROS2 driver stack, including individual nodes and launch files.

---

## Running the IMU Node (Standalone)

### Basic CLI execution (Ex. No IMU Config)

```bash 
ros2 run oscp_imu_ros2 oscp_imu_node \
  --ros-args \
  -p device:=/dev/ttyUSB0 \
  -p baudrate:=921600 \
  -p frame_id:=imu_link
```

### Launch the IMU node using yaml configurations:

```bash
ros2 launch oscp_imu_ros2 oscp_imu.launch.py config_file:=test.yaml
```

---

---
