# Launch & Usage

Establish a serial connection between target flight controller and your [OSCP IMU Products](https://www.oscp.com/technology){:target="_blank"}, then verify it in [Mission Planner](https://ardupilot.org/planner/docs/mission-planner-installation.html){:target="_blank"} on either Linux or Windows.

### Find the Serial Port

=== "Linux"

    !!! warning "Important"
        Ensure your adapter (InertialGate, MicroGate, or custom) is properly connected to both the flight controller and the OSCP IMU.

    1. Connect the flight controller via USB.
    2. Open a terminal.
    3. List available serial devices:
    
        ```bash
        ls /dev/ttyUSB*
        ls /dev/ttyACM*
        ```
    
    4. Look for a device such as:
    
        ```
        /dev/ttyUSBX
        ```
    5. Note the device path such as `/dev/ttyUSB0` or `/dev/ttyACM0`.

        !!! important
            This serial device is required in Mission Planner.

=== "Windows"

    !!! warning "Important"
        Ensure your adapter (InertialGate, MicroGate, or custom) is properly connected to both the flight controller and the OSCP IMU.

    1. Connect the flight controller via USB.
    2. Open <b>Device Manager</b>.
    3. Expand <b>Ports (COM & LPT)</b>.
    
        ![Device Manager COMX](../assets/images/com_device.png)
    
    4. Look for a device such as:
    
        ```
        USB Serial Device (COMx)
        ```
    5. Note the COM number such as `COM3` or `COM7`.
    
        !!! important
            This COM number is required in Mission Planner.

---

### Connect in Mission Planner

1. <b>Launch</b>  Mission Planner.
2. <b>Select</b>  the correct communication port from the dropdown menu (e.g., `/dev/ttyUSB0` on Linux or `COM10` on Windows).
3. <b>Configure</b>  the baud rate to match the flight controller's interface requirements.
4. <b>Click Connect </b> to initialize the serial communication link.

    ![ArduPilot](../assets/images/ardupilot.png)

5. Wait for the flight controller to initialize and connect.

---
!!! success "Next Steps"
    If the connection is successful, Mission Planner should communicate with the flight controller without issue. To enable OSCP IMU functionality, continue to [Setup & Config](setup.md) for parameter configuration and final setup.
---