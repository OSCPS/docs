# **OSCP ROS2 Driver**

## ROS Architecture Overview 
This documentation covers the OSCP ROS2 driver stack, including:

**[IMU Driver (oscp_imu_ros2)](https://github.com/OSCPS/oscp_ros2_driver/tree/main/oscp_imu_ros2){:target="_blank"}**
    <ul>
        <li>Low level serial driver</li> 
        <li>ROS IMU Topic publication</li>
        <li>Launch file based configurations </li>
    </ul>


**[IMU Libraries (oscp_libraries)](https://github.com/OSCPS/oscp_ros2_driver/tree/main/oscp_libraries){:target="_blank"}**
    <ul>
        <li>COBS Decoding</li>
        <li>Frame Type Definitions</li>
    </ul>


**[IMU Messages (oscp_msgs)](https://github.com/OSCPS/oscp_ros2_driver/tree/main/oscp_msgs){:target="_blank"}**
    <ul>
        <li>Custom OSCP packet message formats </li>
        <li>Standardized ROS topics</li>
        <li>Stamped with internal clock</li>
    </ul>

---