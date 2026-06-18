#Setup and Configuration

The OSCP MK2M2 ROS 2 driver is configured via a YAML file passed at launch time. All parameters are loaded at startup and sent to the IMU over serial before streaming begins.

---
## Config Argument

```bash
ros2 launch oscp_imu_ros2 oscp_imu.launch.py config_file:=test.yaml
```

The `config_file` argument must point to a YAML file located in `oscp_imu_ros2/config/`.

---

## YAML Structure (ex. test.yaml)

```yaml
oscp_imu_node:
  ros__parameters:
    device: "/dev/ttyUSB0"
    baudrate: 921600
    frame_id: "imu_link"
    pub_accel_in_g: false

    operating_mode: "LOW"

    gyro_range: "DPS_250"
    accel_range: "G_4"
    incl_range: "G_1_0"

    gyro_filter_mode: "LP_AND_HP"
    gyro_lpf: "C0"
    gyro_hpf: "C1"

    accel_filter_mode: "LP_ONLY"
    accel_lpf: "C0"
    accel_hpf: "C0"

    ahrs_convention: "NWU"
    ahrs_heading: "INTERNAL_MAGNETOMETER"

    misalignment: true

    standard_ros_enabled: true
    oscp_raw_enabled: true
    oscp_euler_enabled: false
    oscp_quat_enabled: true
    oscp_rotation_matrix_enabled: false
```

---

## ROS Parameters

???+ note "Device"

    | Parameter | Type | Description |
    |---|---|---|
    | `device` | string | Serial port path |
    | `baudrate` | int | Must be `921600` for RS-422 variants except in custom builds |
    | `frame_id` | string | Frame ID used in all published messages |
    | `pub_accel_in_g` | bool | If `true`, publish acceleration in g. If `false`, publish in m/s² |

???+ note "Operating Mode"

    | Value | Output Data Rate |
    |---|---|
    | `IDLE` | 0 Hz |
    | `LOW` | 100 Hz |
    | `MEDIUM` | 500 Hz |

???+ note "Output Frame Enables"

    | Parameter | Type | Description |
    |---|---|---|
    | `standard_ros_enabled` | bool | Publish `sensor_msgs/Imu`, `MagneticField`, `Temperature` |
    | `oscp_raw_enabled` | bool | Publish raw OSCP frame |
    | `oscp_quat_enabled` | bool | Publish quaternion OSCP frame |
    | `oscp_euler_enabled` | bool | Publish Euler angles OSCP frame |
    | `oscp_rotation_matrix_enabled` | bool | Publish rotation matrix OSCP frame |

!!! Warning 
    Enabling too many output frames simultaneously at MEDIUM (500 Hz) may cause controller overruns on the IMU. Refer to the IMU datasheet for guidance.

---
## IMU Configurations

??? note "Gyroscope Dynamic Range"

    | Value | Range |
    |---|---|
    | `DPS_125` | ±125 °/sec |
    | `DPS_250` | ±250 °/sec |
    | `DPS_500` | ±500 °/sec |
    | `DPS_1000` | ±1000 °/sec |
    | `DPS_2000` | ±2000 °/sec |
    | `DPS_4000` | ±4000 °/sec |

??? note "Accelerometer Dynamic Range"

    | Value | Range |
    |---|---|
    | `G_2` | ±2 g |
    | `G_4` | ±4 g |
    | `G_8` | ±8 g |
    | `G_16` | ±16 g |

??? note "Inclinometer Dynamic Range"

    | Value | Range |
    |---|---|
    | `G_0_5` | ±0.5 g |
    | `G_1_0` | ±1.0 g |
    | `G_2_0` | ±2.0 g |
    | `G_3_0` | ±3.0 g |

??? note "Misalignment Correction"

    | Parameter | Type | Description |
    |---|---|---|
    | `misalignment` | bool | Enables gyroscope and accelerometer misalignment correction |

??? note "Gyroscope Filter"

    | Parameter | Options | Description |
    |---|---|---|
    | `gyro_filter_mode` | `DISABLED`, `LP_ONLY`, `HP_ONLY`, `LP_AND_HP` | Filter mode |
    | `gyro_lpf` | `C0` – `C7` | Low-pass cutoff selection |
    | `gyro_hpf` | `C0` – `C3` | High-pass cutoff selection |

??? note "Gyroscope Low-Pass Cutoff (gyro_lpf)"

    | Value | Cutoff (Low ODR) | Cutoff (Medium ODR) |
    |---|---|---|
    | `C0` | 33 Hz | 222 Hz |
    | `C1` | 33 Hz | 186 Hz |
    | `C2` | 33 Hz | 140 Hz |
    | `C3` | 33 Hz | 260 Hz |
    | `C4` | 34 Hz | 96 Hz |
    | `C5` | 31 Hz | 49 Hz |
    | `C6` | 19 Hz | 25 Hz |
    | `C7` | 11.6 Hz | 12.6 Hz |

??? note "Gyroscope High-Pass Cutoff (gyro_hpf)"

    | Value | Cutoff |
    |---|---|
    | `C0` | 16 mHz |
    | `C1` | 65 mHz |
    | `C2` | 260 mHz |
    | `C3` | 1.04 Hz |

??? note "Accelerometer Filter"

    | Parameter | Options | Description |
    |---|---|---|
    | `accel_filter_mode` | `DISABLED`, `LP_ONLY`, `HP_ONLY`, `LP_AND_HP` | Filter mode |
    | `accel_lpf` | `C0` – `C7` | Low-pass cutoff selection |
    | `accel_hpf` | `C0` – `C7` | High-pass cutoff selection |

??? note "Accelerometer Low-Pass and High-Pass Cutoff"

    | Value | Cutoff (Low ODR) | Cutoff (Medium ODR) |
    |---|---|---|
    | `C0` | 26 Hz | 208.25 Hz |
    | `C1` | 10.4 Hz | 83.3 Hz |
    | `C2` | 5.2 Hz | 41.65 Hz |
    | `C3` | 2.3 Hz | 18.5 Hz |
    | `C4` | 1.04 Hz | 8.33 Hz |
    | `C5` | 0.52 Hz | 4.17 Hz |
    | `C6` | 0.26 Hz | 2.08 Hz |
    | `C7` | 0.13 Hz | 1.04 Hz |

??? note "AHRS Configuration"

    | Parameter | Options | Description |
    |---|---|---|
    | `ahrs_convention` | `NWU`, `ENU`, `NED` | Earth axis convention for AHRS output frames |
    | `ahrs_heading` | `NONE`, `INTERNAL_MAGNETOMETER` | Heading source used by the AHRS filter |

---
