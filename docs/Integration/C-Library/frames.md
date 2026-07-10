# Frame Reference

Every frame the unit emits decodes into a typed struct inside the
`oscp_frame_t` union. Branch on `frame.type`, then read the matching member via
its accessor macro.

| `frame.type` | Value | Struct | Accessor | Decoded size |
|--------------|-------|--------|----------|--------------|
| `OSCP_FRAME_RAW`        | `0x00` | `oscp_raw_t`     | `oscp_raw(&f)`     | 61 B |
| `OSCP_FRAME_EULER`      | `0x01` | `oscp_euler_t`   | `oscp_euler(&f)`   | 25 B |
| `OSCP_FRAME_QUATERNION` | `0x02` | `oscp_quat_t`    | `oscp_quat(&f)`    | 29 B |
| `OSCP_FRAME_ROT_MATRIX` | `0x03` | `oscp_rot_mat_t` | `oscp_rot_mat(&f)` | 49 B |
| `OSCP_FRAME_GNSS`       | `0x04` | `oscp_gnss_t`    | `oscp_gnss(&f)`    | 64 B |
| `OSCP_FRAME_DEBUG_1`    | `0x05` | `oscp_debug_1_t` | `oscp_debug_1(&f)` | 58 B |
| `OSCP_FRAME_DEBUG_2`    | `0x06` | `oscp_debug_2_t` | `oscp_debug_2(&f)` | 58 B |
| `OSCP_FRAME_STARTUP`    | `0x07` | `oscp_startup_t` | `oscp_startup(&f)` | 40 B |

!!! info
    Command responses from the IMU surface as additional `frame.type` values
    with no payload: `OSCP_FRAME_CMD_SUCCESS` (`0xA0`), `OSCP_FRAME_CMD_FAILED`
    (`0xA1`), `OSCP_FRAME_CMD_UNKNOWN` (`0xA2`) and `OSCP_FRAME_CMD_NOT_IMPL`
    (`0xA3`). These are purely added by the C Library and are not native frame types from the IMU.

---

## The header byte

Every operating frame begins with a packed header byte. Decode it with the
supplied macros rather than by hand:

```c
oscp_frame_type_t       t = OSCP_HDR_FRAME_TYPE(f);        /* bits [2:0] */
oscp_operating_mode_t  om = OSCP_HDR_OPERATING_MODE(f);    /* bits [5:3] */
oscp_misalignment_corr_t mc = OSCP_HDR_MISALIGNMENT_CORR(f); /* bits [7:6] */
```

| Field | Bits | Values |
|-------|------|--------|
| Frame type | `[2:0]` | see table above |
| Operating mode | `[5:3]` | `OSCP_OP_MODE_IDLE` / `_LOW` / `_MEDIUM` |
| Misalignment correction | `[7:6]` | `OSCP_MISALIGNMENT_CORR_DISABLED` / `_ENABLED` |

---

## The status byte

Every frame carries a `status` byte. `0x00` means all clear; otherwise mask
against the flags below.

| Mask | Value | Meaning |
|------|-------|---------|
| `OSCP_STATUS_OK`       | `0x00` | No bits set - healthy |
| `OSCP_STATUS_OVERRUN`  | `0x01` | IMU real-time controller overrun |
| `OSCP_STATUS_MEMS_ERR` | `0x02` | MEMS sensor error |
| `OSCP_STATUS_INCL_ERR` | `0x04` | Inclinometer error |
| `OSCP_STATUS_MAG_ERR`  | `0x08` | Magnetometer error |
| `OSCP_STATUS_TEMP_ERR` | `0x10` | Temperature sensor error |
| `OSCP_STATUS_GNSS_ERR` | `0x20` | GNSS error _(G variants only)_ |
| `OSCP_STATUS_OG_ERR`   | `0x40` | Optical gyroscope error _(optical-based models only)_ |

```c
if (e->status & OSCP_STATUS_MEMS_ERR) { /* handle MEMS fault */ }
```

---

## Raw frame - `oscp_raw_t`

Full raw sensor output.

??? note "Fields"

    | Field | Type | Unit |
    |-------|------|------|
    | `header_byte` | `uint8_t`  | - |
    | `counter`     | `uint8_t`  | - |
    | `timestamp_ms`| `uint64_t` | ms |
    | `gyro_x/y/z`  | `float`    | dps |
    | `accel_x/y/z` | `float`    | g |
    | `incl_x/y`    | `float`    | mg |
    | `mag_x/y/z`   | `float`    | µT |
    | `temp`        | `float`    | °C |
    | `status`      | `uint8_t`  | - |
    | `crc`         | `uint16_t` | - |

---

## Euler frame - `oscp_euler_t`

AHRS attitude as Euler angles.

??? note "Fields"

    | Field | Type | Unit |
    |-------|------|------|
    | `header_byte` | `uint8_t`  | - |
    | `counter`     | `uint8_t`  | - |
    | `timestamp_ms`| `uint64_t` | ms |
    | `roll`        | `float`    | deg |
    | `pitch`       | `float`    | deg |
    | `yaw`         | `float`    | deg |
    | `status`      | `uint8_t`  | - |
    | `crc`         | `uint16_t` | - |

---

## Quaternion frame - `oscp_quat_t`

AHRS attitude as a unit quaternion `(w, x, y, z)`.

??? note "Fields"

    | Field | Type | Unit |
    |-------|------|------|
    | `header_byte` | `uint8_t`  | - |
    | `counter`     | `uint8_t`  | - |
    | `timestamp_ms`| `uint64_t` | ms |
    | `w`, `x`, `y`, `z` | `float` | - (normalized) |
    | `status`      | `uint8_t`  | - |
    | `crc`         | `uint16_t` | - |

---

## Rotation matrix frame - `oscp_rot_mat_t`

AHRS attitude as a 3×3 direction-cosine matrix.

??? note "Fields"

    | Field | Type | Notes |
    |-------|------|-------|
    | `header_byte` | `uint8_t`  | - |
    | `counter`     | `uint8_t`  | - |
    | `timestamp_ms`| `uint64_t` | ms |
    | `rm[3][3]`    | `float`    | row-major direction-cosine matrix |
    | `status`      | `uint8_t`  | - |
    | `crc`         | `uint16_t` | - |

---

## GNSS frame - `oscp_gnss_t`

Position, velocity and fix quality.

??? note "Fields"

    | Field | Type | Unit |
    |-------|------|------|
    | `header_byte` | `uint8_t`  | - |
    | `counter`     | `uint8_t`  | - |
    | `timestamp_ms`| `uint64_t` | ms |
    | `gnss_fix_type` | `uint8_t` | - |
    | `num_satellites`| `uint8_t` | count |
    | `longitude`   | `float`   | deg |
    | `latitude`    | `float`   | deg |
    | `height`      | `int32_t` | mm |
    | `horizontal_accuracy` | `uint32_t` | mm |
    | `vertical_accuracy`   | `uint32_t` | mm |
    | `velocity_north/east/down` | `int32_t` | mm/s |
    | `speed_accuracy` | `uint32_t` | mm/s |
    | `heading_of_motion` | `float` | deg |
    | `heading_accuracy`  | `float` | deg |
    | `pdop`        | `float`   | - |
    | `gnssFixOk` : 1 | `bitfield` | fix valid |
    | `invalidLLH` : 1 | `bitfield` | lat/lon/height invalid |
    | `reserved` : 2 | `bitfield` | - |
    | `lastCorrectionAge` : 4 | `bitfield` | - |
    | `status`      | `uint8_t`  | - |
    | `crc`         | `uint16_t` | - |

---

## Debug frames - `oscp_debug_1_t` / `oscp_debug_2_t`

User / diagnostic frames. Debug 1 carries per-axis sensor bias user registers and
user filter settings; Debug 2 carries the user magnetometer calibration matrix and AHRS
fusion parameters. Both are opaque `uint32_t` register dumps - see the datasheet
for interpretation. Request them with the `OFD` operating frame command
(`oscp_cmd_of(..., OSCP_FRAME_SEL_DEBUG, ...)`) while in `CONFIG` state.

---

## Startup frame - `oscp_startup_t`

Emitted once at boot or when requested with the `SUF` command (`oscp_cmd_suf(...)`) while in `CONFIG` state.. Identifies the unit and reports its saved configuration.

??? note "Fields"

    | Field | Type | Notes |
    |-------|------|-------|
    | `header_byte` | `uint8_t`  | - |
    | `mark_number[10]` | `char` | **not null-terminated** - see warning below |
    | `unit_number` | `uint16_t` | unit number |
    | `sw_major_ver` / `sw_minor_ver` / `sw_patch_ver` | `uint8_t` | firmware version |
    | `enabled_frames` | `uint8_t` | bitmask, see below |
    | `gyro_dr` / `accel_dr` / `incl_dr` | `bitfield` | dynamic ranges |
    | `gyro_*` / `accel_*` filters | `bitfield` | filter config |
    | `ahrs_convention` / `ahrs_heading_src` | `bitfield` | AHRS config |
    | `ahrs_gain` / `ahrs_accel_rej` / `ahrs_mag_rej` | `float` | AHRS config |
    | `ahrs_rec_trig_per` | `uint32_t` | AHRS config |
    | `status` | `uint8_t` | - |
    | `crc` | `uint16_t` | - |

The `enabled_frames` bitmask uses these flags:

| Mask | Value |
|------|-------|
| `OSCP_ENABLE_RAW`     | `0x01` |
| `OSCP_ENABLE_EULER`   | `0x02` |
| `OSCP_ENABLE_QUAT`    | `0x04` |
| `OSCP_ENABLE_ROT_MAT` | `0x08` |
| `OSCP_ENABLE_GNSS`    | `0x10` |

!!! warning "`mark_number` is not null-terminated"
    `mark_number` is a fixed 10-byte field with no terminator. Do not pass it
    straight to `printf("%s")`. Copy it into an 11-byte buffer and terminate it
    yourself:

    ```c
    char mark[11];
    memcpy(mark, s->mark_number, sizeof(s->mark_number));
    mark[10] = '\0';
    ```

---

!!! danger "Packed structs - do not take a bare `float *`"
    Frame structs mirror the wire format and are **packed**. Reading a field
    through the struct (`e->roll`) is safe, but taking a `float *` into a
    member and dereferencing it can fault on strict-alignment cores (an
    unaligned `VLDR` on Cortex-M7). Copy into aligned locals before doing float
    math:

    ```c
    const float gx = r->gyro_x, gy = r->gyro_y, gz = r->gyro_z;
    ```

---

Next: [Command reference →](commands.md)
