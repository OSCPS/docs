# Launch & Usage

This section explains how to run the OSCP ROS2 driver stack, including individual nodes and launch files.

---

## Running the IMU Node (Standalone)

### Basic CLI execution (Ex. No IMU Config)

```bash 
ros2 run oscp_imu_ros2 oscp_imu_node --ros-args -p device:=/dev/ttyUSB0 -p baudrate:=921600 -p frame_id:=imu_link
```

### Launch the IMU node using yaml configurations:

```bash
ros2 launch oscp_imu_ros2 oscp_imu.launch.py config_file:=test.yaml
```

---

### Launch IMU node using a launch files

```python
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument, IncludeLaunchDescription
from launch.launch_description_sources import PythonLaunchDescriptionSource
from launch.substitutions import LaunchConfiguration
from ament_index_python.packages import get_package_share_directory
import os


def generate_launch_description():
    imu_pkg = get_package_share_directory('oscp_imu_ros2')
    config_file = LaunchConfiguration('config_file')

    return LaunchDescription([
        DeclareLaunchArgument('config_file',default_value=os.path.join(imu_pkg,'config','oscp_imu.yaml'), description='Path to the IMU configuration YAML file'),

        IncludeLaunchDescription(PythonLaunchDescriptionSource(os.path.join(imu_pkg,'launch','oscp_imu.launch.py')),
            launch_arguments={
                'config_file': config_file
            }.items()
        ),
    ])
```

---

Next: [Services →](services.md)