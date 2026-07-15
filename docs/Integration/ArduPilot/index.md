# OSCP ArduPilot Driver

`oscp_ardupilot` is a custom driver that integrates [OSCP IMU Products](https://www.oscp.com/technology){:target="_blank"} with ArduPilot. It processes and forwards OSCP IMU data to the ArduPilot EKF for sensor fusion, and includes serial communication handling along with sample `.hwdef` configurations for enabling support on compatible flight controllers.

:material-github: [https://github.com/OSCPS/oscp_ardupilot](https://github.com/OSCPS/oscp_ardupilot){:target="_blank"}

!!! note "Supported Flight Controllers"
    For a complete list of compatible flight controllers, refer to the [official ArduPilot autopilot documentation](https://ardupilot.org/copter/docs/common-autopilots.html){:target="_blank"}.

---

## External AHRS

The ArduPilot fork includes the following four added files:

!!!note "File Location"
    All files are located in `libraries/AP_ExternalAHRS/`.

| File | Description |
|------|-------------|
| `AP_ExternalAHRS_OSCP.h` | Declares the OSCP ExternalAHRS backend. |
| `AP_ExternalAHRS_OSCP.cpp` | Implements the OSCP ExternalAHRS backend. |
| `OSCP_common.h` | Header for the Craig McQueen COBS library used by OSCP. |
| `OSCP_common.c` | Implementation of the Craig McQueen COBS library used by OSCP. |

---

## Build & Install

We've kept integration as simple as possible. Just clone our repository and follow the [installation guide](install.md) to get your flight controller ready to fly with OSCP.

---

## Supported OSCP Hardware

| Model | Description |
|-------|-------------|
| [**MK2M2**](https://www.oscp.com/products/mk2m2){:target="_blank"} | OSCP Tactical-Grade MEMS IMU |
| [**MK2E2**](https://www.oscp.com/products/mk2e2){:target="_blank"} | OSCP Tactical-Grade MEMS + Optical Gyroscope IMU |
| [**MK2Z**](https://www.oscp.com/products/mk2z){:target="_blank"} | OSCP Navigation-Grade MEMS + Optical Gyroscope IMU |

---

## Documentation map

<div class="grid cards" markdown>

-   [__:material-download-multiple: Install__](install.md)

    Install ArduPilot drivers and dependencies to get started.

-   [__:material-progress-wrench: Setup & Config__](setup.md)

    Full ArduPilot parameter list for OSCP IMU setup.

-   [__:material-code-block-parentheses: Launch & Usage__](launch.md)

    Use Mission Planner to connect and configure flight modes.

</div>

---
