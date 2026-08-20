# OSCP ROS2 Driver

`oscp_ros2_driver` is a custom driver that integrates [OSCP IMU Products](https://www.oscp.com/technology) with ROS 2. It parses the OSCP data stream and publishes it on standard `sensor_msgs` topics for use by the wider ROS 2 ecosystem as well as custom messages, and includes serial communication handling along with sample launch files and parameter configurations for common setups.

:material-github: [https://github.com/OSCPS/oscp_ros2_driver](https://github.com/OSCPS/oscp_ros2_driver){:target="_blank"}

---

## ROS Architecture Overview 
This documentation covers the OSCP ROS2 driver stack, including:

**:material-github: [IMU Driver (oscp_imu_ros2)](https://github.com/OSCPS/oscp_ros2_driver/tree/main/oscp_imu_ros2){:target="_blank"}**
    <ul>
        <li>Low level serial driver</li> 
        <li>ROS IMU Topic publication</li>
        <li>Launch file based configurations </li>
    </ul>


**:material-github: [IMU Libraries (oscp_libraries)](https://github.com/OSCPS/oscp_ros2_driver/tree/main/oscp_libraries){:target="_blank"}**
    <ul>
        <li>COBS Decoding</li>
        <li>Frame Type Definitions</li>
    </ul>


**:material-github: [IMU Messages (oscp_msgs)](https://github.com/OSCPS/oscp_ros2_driver/tree/main/oscp_msgs){:target="_blank"}**
    <ul>
        <li>Custom OSCP packet message formats </li>
        <li>Standardized ROS topics</li>
        <li>Stamped with internal clock</li>
    </ul>

---

## Documentation map

<div class="grid cards" markdown>

-   [**__:material-download-multiple: Install__**](install.md)

    Install the ROS 2 package and its dependencies to get started quickly.

-   [**__:material-progress-wrench: Setup & Config__**](setup.md)

    Configure the driver, parameters, topics, frames, and communication settings.

-   [**__:material-code-block-parentheses: Launch & Usage__**](launch.md)

    Launch the ROS 2 nodes under different scenarios.

-   [**__:material-cog-sync: Services__**](services.md)

    Interact with the driver's ROS 2 services for calibration, orientation, and device control.

-   [**__:material-bug-check: Debugging__**](debug.md)

    Troubleshoot message output and general ROS 2 issues.


</div>

---