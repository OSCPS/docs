# Frame Reference

Each frame emitted by the unit is decoded into a struct carried by a variant of
`OscpFrame`.

| `OscpFrame` variant | Header value | Struct | Length constant | Length |
|---------------------|--------------|--------|-----------------|--------|
| `OscpFrame::Raw`     | `0x00` | `OscpRaw`     | `OSCP_RAW_LEN`     | 61 B |
| `OscpFrame::Euler`   | `0x01` | `OscpEuler`   | `OSCP_EULER_LEN`   | 25 B |
| `OscpFrame::Quat`    | `0x02` | `OscpQuat`    | `OSCP_QUAT_LEN`    | 29 B |
| `OscpFrame::RotMat`  | `0x03` | `OscpRotMat`  | `OSCP_ROT_MAT_LEN` | 49 B |
| `OscpFrame::Gnss`    | `0x04` | `OscpGnss`    | `OSCP_GNSS_LEN`    | 64 B |
| `OscpFrame::Debug1`  | `0x05` | `OscpDebug1`  | `OSCP_DEBUG_1_LEN` | 58 B |
| `OscpFrame::Debug2`  | `0x06` | `OscpDebug2`  | `OSCP_DEBUG_2_LEN` | 58 B |
| `OscpFrame::Startup` | `0x07` | `OscpStartup` | `OSCP_STARTUP_LEN` | 40 B |

!!! info "Command responses"
    Command responses are reported as `OscpFrame::CmdResponse(OscpFrameCmd)`
    with one of `OscpFrameCmd::Success` (`0xA0`), `::Failed` (`0xA1`),
    `::Unknown` (`0xA2`) or `::NotImpl` (`0xA3`). These values are defined by
    the library, not by the IMU protocol. Only the RS422 parser produces this
    variant; see [Command classes](commands.md#command-classes).

---

## The header byte

The first byte of every operating frame is stored in `header_byte` as an
`OscpHeader`. Its fields are read with the following methods:

```rust
let t  = e.header_byte.frame_type();         /* bits [2:0] -> Result<OscpFrameType, u8> */
let om = e.header_byte.operating_mode();     /* bits [5:3] -> Result<OscpOperatingMode, u8> */
let mc = e.header_byte.misalignment_corr();  /* bits [7:6] -> Result<OscpMisalignmentCorr, u8> */
```

| Field | Bits | Values |
|-------|------|--------|
| Frame type | `[2:0]` | See table above (`OscpFrameType::Raw` … `::Startup`) |
| Operating mode | `[5:3]` | `OscpOperatingMode::Idle` / `::Low` / `::Medium` |
| Misalignment correction | `[7:6]` | `OscpMisalignmentCorr::Disabled` / `::Enabled` |

A method returns `Err(value)` if the bits do not map to a variant. The raw byte
is returned by `header_byte.to_byte()`.

---

## The status byte

Every frame contains a `status` byte. `0x00` indicates no fault. Other values
are a combination of the following masks:

| Mask | Value | Meaning |
|------|-------|---------|
| `OSCP_STATUS_OK`       | `0x00` | No fault |
| `OSCP_STATUS_OVERRUN`  | `0x01` | IMU real-time controller overrun |
| `OSCP_STATUS_MEMS_ERR` | `0x02` | MEMS sensor error |
| `OSCP_STATUS_INCL_ERR` | `0x04` | Inclinometer error |
| `OSCP_STATUS_MAG_ERR`  | `0x08` | Magnetometer error |
| `OSCP_STATUS_TEMP_ERR` | `0x10` | Temperature sensor error |
| `OSCP_STATUS_GNSS_ERR` | `0x20` | GNSS error _(G variants only)_ |
| `OSCP_STATUS_OG_ERR`   | `0x40` | Optical gyroscope error _(optical-based models only)_ |

```rust
if e.status & OSCP_STATUS_MEMS_ERR != 0 { /* handle MEMS fault */ }
```

---

## Raw frame - `OscpRaw`

Raw sensor output.

??? note "Fields"

    | Field | Type | Unit |
    |-------|------|------|
    | `header_byte` | `OscpHeader` | - |
    | `counter`     | `u8`  | - |
    | `timestamp_ms`| `u64` | ms |
    | `gyro_x/y/z`  | `f32` | dps |
    | `accel_x/y/z` | `f32` | g |
    | `incl_x/y`    | `f32` | mg |
    | `mag_x/y/z`   | `f32` | µT |
    | `temp`        | `f32` | °C |
    | `status`      | `u8`  | - |
    | `crc`         | `u16` | - |

---

## Euler frame - `OscpEuler`

AHRS attitude as Euler angles.

??? note "Fields"

    | Field | Type | Unit |
    |-------|------|------|
    | `header_byte` | `OscpHeader` | - |
    | `counter`     | `u8`  | - |
    | `timestamp_ms`| `u64` | ms |
    | `roll`        | `f32` | deg |
    | `pitch`       | `f32` | deg |
    | `yaw`         | `f32` | deg |
    | `status`      | `u8`  | - |
    | `crc`         | `u16` | - |

---

## Quaternion frame - `OscpQuat`

AHRS attitude as a unit quaternion `(w, x, y, z)`.

??? note "Fields"

    | Field | Type | Unit |
    |-------|------|------|
    | `header_byte` | `OscpHeader` | - |
    | `counter`     | `u8`  | - |
    | `timestamp_ms`| `u64` | ms |
    | `w`, `x`, `y`, `z` | `f32` | - (normalized) |
    | `status`      | `u8`  | - |
    | `crc`         | `u16` | - |

---

## Rotation matrix frame - `OscpRotMat`

AHRS attitude as a 3×3 direction-cosine matrix.

??? note "Fields"

    | Field | Type | Notes |
    |-------|------|-------|
    | `header_byte` | `OscpHeader` | - |
    | `counter`     | `u8`  | - |
    | `timestamp_ms`| `u64` | ms |
    | `rm`          | `[[f32; 3]; 3]` | Row-major direction-cosine matrix |
    | `status`      | `u8`  | - |
    | `crc`         | `u16` | - |

---

## GNSS frame - `OscpGnss`

Position, velocity and fix quality.

??? note "Fields"

    | Field | Type | Unit |
    |-------|------|------|
    | `header_byte` | `OscpHeader` | - |
    | `counter`     | `u8`  | - |
    | `timestamp_ms`| `u64` | ms |
    | `gnss_fix_type` | `u8` | - |
    | `num_satellites`| `u8` | count |
    | `longitude`   | `f32` | deg |
    | `latitude`    | `f32` | deg |
    | `height`      | `i32` | mm |
    | `horizontal_accuracy` | `u32` | mm |
    | `vertical_accuracy`   | `u32` | mm |
    | `velocity_north/east/down` | `i32` | mm/s |
    | `speed_accuracy` | `u32` | mm/s |
    | `heading_of_motion` | `f32` | deg |
    | `heading_accuracy`  | `f32` | deg |
    | `pdop`        | `f32` | - |
    | `gnss_flags`  | `GnssFlags` | See below |
    | `status`      | `u8`  | - |
    | `crc`         | `u16` | - |

    `GnssFlags` methods:

    | Method | Bits | Returns |
    |--------|------|---------|
    | `gnss_fix_ok()` | `[0]` | `bool`, fix valid |
    | `invalid_llh()` | `[1]` | `bool`, latitude/longitude/height invalid |
    | `last_correction_age()` | `[7:4]` | `u8` |

---

## Debug frames - `OscpDebug1` / `OscpDebug2`

Diagnostic frames. Debug 1 contains the per-axis sensor bias user registers and
the user filter settings. Debug 2 contains the user magnetometer calibration
matrix and the AHRS fusion parameters. Register values are raw `u32`; refer to
the datasheet for their interpretation. Both frames are sent in response to
`OFD` (`oscp_cmd_of(..., OscpFrameSel::Debug, ...)`) in CONFIG state.

??? note "Packed configuration bytes"

    | Field | Type | Methods |
    |-------|------|---------|
    | `gyro_filter_cfg` / `accel_filter_cfg` _(Debug 1)_ | `FilterCfg` | `filters()` `[1:0]`, `lpf()` `[4:2]`, `hpf()` `[7:5]` |
    | `fusion_cfg` _(Debug 2)_ | `FusionCfg` | `fusion_convention()` `[3:0]`, `fusion_heading_source()` `[7:4]` |

    Debug frames contain `counter` but not `timestamp_ms`.

---

## Startup frame - `OscpStartup`

Sent once at boot, and in response to `SUF` (`oscp_cmd_suf(...)`) in CONFIG
state. Contains the unit identification and saved configuration.

??? note "Fields"

    | Field | Type | Notes |
    |-------|------|-------|
    | `header_byte` | `OscpHeader` | - |
    | `mark_number` | `[u8; 10]` | Byte array; see note below |
    | `unit_number` | `u16` | Unit number |
    | `sw_major_ver` / `sw_minor_ver` / `sw_patch_ver` | `u8` | Firmware version |
    | `enabled_frames` | `u8` | Bitmask, see below |
    | `dyn_range_cfg` | `DynRangeCfg` | `gyro_dr()` `[3:0]`, `accel_dr()` `[7:4]` |
    | `gyro_filter_cfg` / `accel_filter_cfg` | `FilterCfg` | `filters()`, `lpf()`, `hpf()` |
    | `incl_ahrs_cfg` | `InclAhrsCfg` | `incl_dr()` `[3:0]`, `ahrs_convention()` `[5:4]`, `ahrs_heading_src()` `[7:6]` |
    | `ahrs_gain` / `ahrs_accel_rej` / `ahrs_mag_rej` | `f32` | AHRS configuration |
    | `ahrs_rec_trig_per` | `u32` | AHRS recovery trigger period (s) |
    | `status` | `u8` | - |
    | `crc` | `u16` | - |

`enabled_frames` masks:

| Mask | Value |
|------|-------|
| `OSCP_ENABLE_RAW`     | `0x01` |
| `OSCP_ENABLE_EULER`   | `0x02` |
| `OSCP_ENABLE_QUAT`    | `0x04` |
| `OSCP_ENABLE_ROT_MAT` | `0x08` |
| `OSCP_ENABLE_GNSS`    | `0x10` |

### Reading back dynamic ranges

`gyro_dr()`, `accel_dr()` and `incl_dr()` return the 4-bit code reported by the
unit. This code is not the value written by `DRG`, `DRA` or `DRI` and is not
sequential. Convert it with `from_dyn_range_code`:

```rust
use oscp_imu::{OscpGyroDr, OscpAccelDr, OscpInclDr};

let gyro  = OscpGyroDr::from_dyn_range_code(s.dyn_range_cfg.gyro_dr());   /* e.g. Ok(OscpGyroDr::Dps250) */
let accel = OscpAccelDr::from_dyn_range_code(s.dyn_range_cfg.accel_dr());
let incl  = OscpInclDr::from_dyn_range_code(s.incl_ahrs_cfg.incl_dr());
```

??? note "Readback code mapping"

    | Code | Gyroscope | Accelerometer | Inclinometer |
    |------|-----------|---------------|--------------|
    | `0b0000` | ±250 °/s  | ±2 g  | ±0.5 g |
    | `0b0001` | ±4000 °/s | ±16 g | ±3.0 g |
    | `0b0010` | ±125 °/s  | ±4 g  | ±1.0 g |
    | `0b0011` | -         | ±8 g  | ±2.0 g |
    | `0b0100` | ±500 °/s  | -     | - |
    | `0b1000` | ±1000 °/s | -     | - |
    | `0b1100` | ±2000 °/s | -     | - |

    Unlisted codes return `Err(code)`.

!!! warning "`mark_number` is a byte array"
    `mark_number` is 10 bytes with no terminator. UTF-8 validity is not
    guaranteed. Convert before printing and remove trailing padding:

    ```rust
    let mark = core::str::from_utf8(&s.mark_number)
        .unwrap_or("<invalid>")
        .trim_end_matches('\0');
    ```

---

!!! note "Struct layout"
    Frame structs are not packed. Each field is decoded with `from_le_bytes`
    into a naturally aligned struct, so field access has no alignment
    restrictions. All frame types implement `Copy`.

---

Next: [Command reference →](commands.md)
