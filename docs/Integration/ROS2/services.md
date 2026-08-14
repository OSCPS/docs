# OSCP IMU ROS 2 Services

This section documents the ROS 2 services provided by the OSCP IMU driver.

---

## Summary Table

| Service | Type | Purpose |
|---------|------|---------|
| `/oscp/get_config` | Service | Query current IMU configuration |
| `/oscp/stationary_calibrate` | Service | Calibrate gyro/accel biases (IMU stationary) |
| `/oscp/zero_orientation` | Service | Set current orientation as reference (0°) |
| `/oscp/apply_mag_config` | Service | Apply magnetometer calibration from YAML file |
| `/oscp/magnetometer_calibration` | Action | Calculate magnetometer soft iron and hard iron | 

--- 

## List Available Services

To see all services currently exposed by the IMU node:

```bash
ros2 service list
```

To inspect a specific service type:

```bash
ros2 service type /oscp/apply_mag_config
```

To view the service definition:

```bash
ros2 interface show oscp_imu_ros2/srv/ApplyMagConfig
```

Example output:
```
string config_path
---
bool success
string message
```

---

## Get IMU Configuration

Returns the current configuration of the connected IMU.

**Service:** `/oscp/get_config`  
**Type:** `oscp_imu_ros2/srv/GetIMUConfig`

### Call

```bash
ros2 service call /oscp/get_config oscp_imu_ros2/srv/GetIMUConfig "{}"
```

Pretty print version for proper formatting 

```bash
ros2 service call /oscp/get_config oscp_imu_ros2/srv/GetIMUConfig "{}" | sed -n "s/.*config_summary='//; s/')$//; s/\\\\n/\n/g; p"
```

---

## Stationary Calibration

Performs a stationary calibration of the IMU.

The IMU should remain **completely stationary** while the calibration is running. The calibration will measure the gyroscope and accelerometer biases.

**Service:** `/oscp/stationary_calibrate`  
**Type:** `oscp_imu_ros2/srv/StationaryCalibrate`

### Call

```bash
ros2 service call /oscp/stationary_calibrate oscp_imu_ros2/srv/StationaryCalibrate "{}"
```

---

## Zero Orientation

Sets the current IMU orientation as the zero/reference orientation.

The IMU should be positioned in the desired reference orientation before calling the service. All subsequent orientation measurements will be reported relative to this zero point.

**Service:** `/oscp/zero_orientation`  
**Type:** `oscp_imu_ros2/srv/ZeroOrientation`

### Call

```bash
ros2 service call /oscp/zero_orientation oscp_imu_ros2/srv/ZeroOrientation "{}"
```

---

## Apply Magnetometer Configuration

Applies a previously generated magnetometer calibration to the IMU.

The service reads the calibration values from a YAML file, writes the hard-iron and soft-iron calibration values to the IMU registers, and saves the configuration to IMU flash memory.

**Service:** `/oscp/apply_mag_config`  
**Type:** `oscp_imu_ros2/srv/ApplyMagConfig`

### Request

| Field | Type | Description |
|-------|------|-------------|
| `config_path` | string | Path to the magnetometer calibration YAML file |

### Response

| Field | Type | Description |
|-------|------|-------------|
| `success` | bool | `true` if configuration was successfully applied |
| `message` | string | Status or error message |

### Call

```bash
ros2 service call /oscp/apply_mag_config oscp_imu_ros2/srv/ApplyMagConfig \
  "{config_path: '/path/to/mag_calibration.yaml'}"
```

### YAML Configuration File Format

The magnetometer calibration YAML file must contain the following structure:

```yaml
magnetometer:
  hard_iron:
    x: -12.3456
    y: 18.9012
    z: -5.6789

  soft_iron:
    - [1.0, 0.0, 0.0]
    - [0.0, 1.0, 0.0]
    - [0.0, 0.0, 1.0]
```

**Field Definitions:**

- `hard_iron.x`, `hard_iron.y`, `hard_iron.z`: Three-axis hard-iron magnetic offset (in µT)
- `soft_iron`: 3×3 soft-iron correction matrix for handling local magnetic distortions

### Error Handling

If the specified file does not exist:
```
success: false
message: "Configuration file not found: /path/to/mag_calibration.yaml"
```

If the YAML is invalid or required fields are missing:
```
success: false
message: "Invalid YAML structure: missing 'soft_iron' field"
```

> **⚠️ Warning:** Applying a magnetometer configuration writes calibration values to the IMU and saves them to flash memory. Verify the calibration file contains correct values before applying it. Incorrect calibration can degrade magnetometer accuracy.

---

## (DEVELOPMENT) Magnetometer Calibration (Action Server)

The magnetometer calibration workflow is exposed as a ROS 2 **action server** (not a traditional service).

**Action:** `/oscp/magnetometer_calibration`  
**Type:** `oscp_imu_ros2/action/MagnetometerCalibration`

### Overview

The calibration action collects magnetometer samples over time and calculates hard-iron and soft-iron correction parameters using ellipsoid fitting. Once calibration completes, results can be applied to the IMU using the `ApplyMagConfig` service.

### Workflow

1. Client sends calibration goal
2. Server collects raw magnetometer samples (1000+ samples at ~100 Hz = ~10 seconds)
3. Server performs ellipsoid fitting to extract hard-iron offset and soft-iron matrix
4. Server returns calibration results
5. User can apply results via `ApplyMagConfig` service or manually via IMU registers

### Usage

Invoke calibration using the provided Python client:

```bash
ros2 run magcal_wrapper mag_calibration_client
```

Monitor real-time progress:
```bash
Progress: 0.0% - Collecting samples (0/1000)
Progress: 5.2% - Collecting samples (52/1000)
...
Progress: 100.0% - Calibration complete
```

### Calibration Procedure

1. **Start the IMU node:**
   ```bash
   ros2 launch oscp_imu_ros2 oscp_imu.launch.py
   ```

2. **Start the calibration server:**
   ```bash
   ros2 launch magcal_wrapper calibration.launch.py
   ```

3. **Launch the client and begin rotation:**
   ```bash
   ros2 run magcal_wrapper mag_calibration_client
   ```

4. **Rotate the IMU in all orientations:**
   - Roll full 360° (multiple times)
   - Pitch full 180°
   - Yaw full 360°
   - Include diagonal movements
   - Continue for 1–2 minutes

5. **Review results** when calibration completes

### Troubleshooting

**"Degenerate ellipsoid fit. Collect samples over more orientations."**
- Rotate the sensor in more diverse orientations
- Ensure you cover all planes (roll, pitch, yaw)
- Increase rotation time to 2–3 minutes

**"No samples collected"**
- Verify `oscp_imu_node` is running: `ros2 topic list | grep oscp/raw`
- Check raw data is flowing: `ros2 topic hz /oscp/raw`

**High fit error (>0.1)**
- Recalibrate away from metal objects, magnets, and electronic devices
- High fit error indicates local magnetic disturbances during collection

---