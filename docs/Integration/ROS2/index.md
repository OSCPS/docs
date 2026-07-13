# **OSCP ROS2 Driver**

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

-   [__:material-download-multiple: Install__](install.md)

    Install the ROS 2 package and its dependencies to get started quickly.

-   [__:material-progress-wrench: Setup & Config__](setup.md)

    Configure the driver, parameters, topics, frames, and communication settings.

-   [__:material-code-block-parentheses: Launch & Usage__](launch.md)

    Launch the ROS 2 nodes and learn how to subscribe and integrate the driver into your applications.

</div>

---