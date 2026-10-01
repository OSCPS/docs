# Command Reference

The library encodes commands; it does not transmit them. Each `oscp_cmd_*`
function writes an ASCII command into a caller-owned buffer, framed for the
selected transport, and returns the number of bytes written. The caller
transmits those bytes.

=== "Transport A - RS422 byte-stream parser"
    ```rust
    let mut cmd = [0u8; 32];

    if let Ok(len) = oscp_cmd_config(&mut cmd, OscpTransport::Rs422) {
        /* Transmit as-is */
        platform_dependant_rs422_write(&cmd[..len]);
        /* Delay between consecutive commands */
        platform_dependant_delay_ms(10);
    }
    ```

=== "Transport B - CAN-FD message decoder"
    ```rust
    let mut cmd = [0u8; 32];

    if let Ok(len) = oscp_cmd_config(&mut cmd, OscpTransport::Canfd) {
        /* CAN command ID 0x6FC; caller applies DLC padding */
        platform_dependant_canfd_write(0x6FC, &cmd[..len]);
        /* Delay between consecutive commands */
        platform_dependant_delay_ms(10);
    }
    ```

!!! tip "Buffer sizing"
    A 32-byte buffer holds any encoded command. If the buffer is too small,
    the function returns `Err(OscpErr::Err)`.

---

## How framing depends on transport

The last argument of every command function is an `OscpTransport`:

| Transport | Encoding |
|-----------|----------|
| `OscpTransport::Rs422` | ASCII payload, COBS-encoded, terminated by `0x00`. Transmit unchanged. |
| `OscpTransport::Canfd` | ASCII payload only; no COBS, no delimiter. The caller sets the CAN command identifier (`0x6FC`) and the DLC. |

---

## Command classes

Commands differ in which state accepts them and in whether the unit replies.

!!! tip "Receiving replies"
    On RS422, replies are returned by the parser as
    `OscpFrame::CmdResponse(OscpFrameCmd)`. On CAN-FD, replies use CAN ID
    `0x6FD`. `oscp_frame_decode` does not decode replies; filter ID `0x6FD`
    before calling it.

| Class | Reply | Valid state |
|-------|-------|-------------|
| Silent / ANY | None | Any |
| Silent / CONFIG | None | CONFIG |
| Ack / OPERATING | `OscpFrameCmd` | OPERATING |
| Ack / CONFIG | `OscpFrameCmd` | CONFIG |
| Frame reply / CONFIG | Requested frame | CONFIG |

!!! warning "Frame reply commands"
    `SUF` and `OF<x>` are not acknowledged with `OscpFrameCmd::Success`. The
    reply is the requested frame: one `OscpFrame::Startup` for `SUF`, one frame
    of the selected type for `OF<x>`. `OscpFrameCmd::Failed` is sent only for an
    invalid selector, which `OscpFrameSel` cannot express. `OF` does not change
    the periodic output frames; use `oscp_cmd_enable_oft` / `oscp_cmd_disable_oft`.

---

## Command catalogue

All functions have the form
`fn oscp_cmd_X(cmd: &mut [u8], [args,] transport: OscpTransport) -> Result<usize, OscpErr>`.
The Wire column is the ASCII payload before transport framing.

???+ note "Silent - valid in ANY state"

    | Function | Wire | Purpose |
    |----------|------|---------|
    | `oscp_cmd_reset` | `RESET` | Reset the unit |

???+ note "Silent - valid in CONFIG state"

    | Function | Wire | Purpose |
    |----------|------|---------|
    | `oscp_cmd_exit` | `EXIT` | Return from CONFIG to OPERATING state |
    | `oscp_cmd_refs` | `REFS` | Restore factory settings and reset the unit. Replies `Failed` if the restore fails. |

???+ note "Ack - valid in OPERATING state"

    | Function | Wire | Purpose |
    |----------|------|---------|
    | `oscp_cmd_config` | `CONFIG` | Enter CONFIG state |

???+ note "Frame reply - valid in CONFIG state"

    | Function | Args | Wire | Purpose |
    |----------|------|------|---------|
    | `oscp_cmd_suf` | -              | `SUF`   | Request one Startup frame |
    | `oscp_cmd_of`  | `OscpFrameSel` | `OF<x>` | Request one frame of type x |

???+ note "Ack - valid in CONFIG state"

    | Function | Args | Wire | Purpose |
    |----------|------|------|---------|
    | `oscp_cmd_om`            | `OscpOmSel`       | `OM<x>`      | Set operating mode |
    | `oscp_cmd_enable_oft`    | `OscpFrameSel`    | `EOFT<x>`    | Enable an output frame type |
    | `oscp_cmd_disable_oft`   | `OscpFrameSel`    | `DOFT<x>`    | Disable an output frame type |
    | `oscp_cmd_drg`           | `OscpGyroDr`      | `DRG<nnnn>`  | Set gyroscope dynamic range |
    | `oscp_cmd_dra`           | `OscpAccelDr`     | `DRA<nn>`    | Set accelerometer dynamic range |
    | `oscp_cmd_dri`           | `OscpInclDr`      | `DRI<n.n>`   | Set inclinometer dynamic range |
    | `oscp_cmd_enable_mcorr`  | -                 | `EMCORR`     | Enable misalignment correction |
    | `oscp_cmd_disable_mcorr` | -                 | `DMCORR`     | Disable misalignment correction |
    | `oscp_cmd_wr`            | `OscpUsrReg`, `u32` | `WR<REG><8·hex>` | Write a user register |
    | `oscp_cmd_save`          | -                 | `SAVE`       | Save configuration to flash |

---

## Argument enumerations

### Frame selector - `OscpFrameSel`

Used by `oscp_cmd_of`, `oscp_cmd_enable_oft` and `oscp_cmd_disable_oft`.

| Enum | Wire char | Frame |
|------|-----------|-------|
| `OscpFrameSel::Raw`        | `R` | Raw |
| `OscpFrameSel::Euler`      | `E` | Euler |
| `OscpFrameSel::Quaternion` | `Q` | Quaternion |
| `OscpFrameSel::RotMatrix`  | `M` | Rotation matrix |
| `OscpFrameSel::Gnss`       | `G` | GNSS |
| `OscpFrameSel::Debug`      | `D` | Debug 1 and Debug 2; `oscp_cmd_of` only |

!!! warning "`OscpFrameSel::Debug` with enable/disable"
    `oscp_cmd_enable_oft` and `oscp_cmd_disable_oft` return
    `Err(OscpErr::ErrInvalid)` for `OscpFrameSel::Debug`. Debug frames are not
    periodic; they are requested with `oscp_cmd_of` in CONFIG state.

### Operating mode - `OscpOmSel`

Used by `oscp_cmd_om`.

| Enum | Wire char | RAW ODR | GNSS ODR | Other frames ODR |
|------|-----------|---------|----------|------------------|
| `OscpOmSel::Idle`   | `I` | 0 Hz | 0 Hz | 0 Hz |
| `OscpOmSel::Low`    | `L` | 100 Hz | 1 Hz | 100 Hz |
| `OscpOmSel::Medium` | `M` | 500 Hz | 1 Hz | 100 Hz |

### Gyroscope dynamic range - `OscpGyroDr`

Used by `oscp_cmd_drg`.

| Enum | Range |
|------|-------|
| `OscpGyroDr::Dps125`  | ±125 °/s |
| `OscpGyroDr::Dps250`  | ±250 °/s |
| `OscpGyroDr::Dps500`  | ±500 °/s |
| `OscpGyroDr::Dps1000` | ±1000 °/s |
| `OscpGyroDr::Dps2000` | ±2000 °/s |
| `OscpGyroDr::Dps4000` | ±4000 °/s |

### Accelerometer dynamic range - `OscpAccelDr`

Used by `oscp_cmd_dra`.

| Enum | Range |
|------|-------|
| `OscpAccelDr::G2`  | ±2 g |
| `OscpAccelDr::G4`  | ±4 g |
| `OscpAccelDr::G8`  | ±8 g |
| `OscpAccelDr::G16` | ±16 g |

### Inclinometer dynamic range - `OscpInclDr`

Used by `oscp_cmd_dri`. Discriminants are in tenths of g.

| Enum | Range |
|------|-------|
| `OscpInclDr::G0_5` | ±0.5 g |
| `OscpInclDr::G1_0` | ±1.0 g |
| `OscpInclDr::G2_0` | ±2.0 g |
| `OscpInclDr::G3_0` | ±3.0 g |

!!! note "Readback"
    The Startup frame reports dynamic ranges with a different encoding. Use
    `from_dyn_range_code`; see
    [Reading back dynamic ranges](frames.md#reading-back-dynamic-ranges).

### User registers - `OscpUsrReg`

Used by `oscp_cmd_wr`. The value is encoded as 8 uppercase hexadecimal digits.

??? note "Register mnemonics"

    | Group | Mnemonics |
    |-------|-----------|
    | Gyroscope bias | `GXB`, `GYB`, `GZB`, `GOB` _(`GOB`: optical gyroscope, optical-based models only)_ |
    | Accelerometer bias | `AXB`, `AYB`, `AZB` |
    | Inclinometer bias | `IXB`, `IYB` |
    | Magnetometer bias | `MXB`, `MYB`, `MZB` |
    | Magnetometer calibration matrix | `MXX`…`MZZ` (9 entries) |
    | AHRS fusion | `FCO` (convention), `FHS` (heading source), `FRT` (recovery period), `FGA` (gain), `FAR` (accel rejection), `FMR` (mag rejection) |
    | Gyroscope filters | `GFI` (enable bitmask), `GLP` (low-pass), `GHP` (high-pass) |
    | Accelerometer filters | `AFI` (enable bitmask), `ALP` (low-pass), `AHP` (high-pass) |

    Variant names are the mnemonic in title case, e.g. `OscpUsrReg::Fga`.

```rust
/* Write AHRS fusion gain register */
oscp_cmd_wr(&mut cmd, OscpUsrReg::Fga, 0x3F80_0000, OscpTransport::Rs422)?;
```

---

## Typical configuration sequence

1. `oscp_cmd_config`: enter CONFIG from OPERATING. Periodic output stops.
2. `oscp_cmd_om`, `oscp_cmd_drg`, `oscp_cmd_dra`, `oscp_cmd_enable_oft`, …: apply settings.
3. `oscp_cmd_exit`: return to OPERATING. Periodic output resumes.

See [Sending commands](examples.md#sending-commands) for the corresponding code.

---

Next: [API reference →](api.md)
