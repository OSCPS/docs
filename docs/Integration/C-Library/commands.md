# Command Reference

Commands are **encoded**, not sent, by this library. Each `oscp_cmd_*` function
writes an ASCII command into a caller-owned buffer, framed for the transport selected at function call; Returned bytes are transmit separatly by the caller.

=== "Transport A - RS422 byte-stream parser"
    ```c
    uint8_t cmd[32];
    size_t  len = 0;

    if (oscp_cmd_config(cmd, sizeof(cmd), &len, OSCP_TRANSPORT_RS422) == OSCP_OK) {
        /* Transmit as-is */
        platform_dependant_rs422_write(cmd, len);
    }
    ```

=== "Transport B - CAN-FD message decoder"
    ```c
    uint8_t cmd[32];
    size_t  len = 0;

    if (oscp_cmd_config(cmd, sizeof(cmd), &len, OSCP_TRANSPORT_CANFD) == OSCP_OK) {
        /* Require 0x6FC CAN Command ID and DLC padding */
        platform_dependant_canfd_write(0x6FC, cmd, len);
    }
    ```

!!! tip "Buffer sizing"
    A **32-byte** buffer is sufficient for every command in the library and can be used for easier handling.


---

## How framing depends on transport

Every command function takes a trailing `oscp_transport_t` argument:

| Transport | Encoding |
|-----------|----------|
| `OSCP_TRANSPORT_RS422` | ASCII payload is **COBS-encoded** and terminated with a `0x00` delimiter. Transmit as-is. |
| `OSCP_TRANSPORT_CANFD` | Raw ASCII payload only. **Caller** set the OSCP CAN command identifier (`0x6FC`) and DLC. The controller only frames the message. No COBS, no delimiter. |

---

## Command classes

Commands differ in whether the unit replies and in which state they are valid. 

!!! tip "Catching Replies"
    Replies arrive back through the parser as
    `OSCP_FRAME_CMD_*` frame types, defined specificaly for the C Library. In the case of CAN-FD transport, they can be identified straight through the CAN ID `0x6FD`.

| Class | Reply? | Valid state |
|-------|--------|-------------|
| **Silent / ANY** | None | Any state |
| **Silent / CONFIG** | None | CONFIG only |
| **Ack / OPERATING** | Reply expected | OPERATING only |
| **Ack / CONFIG** | Reply expected | CONFIG only |

---

## Command catalogue

Every function shares the prototype shape
`oscp_err_t oscp_cmd_X(uint8_t *cmd, size_t cmd_max_len, size_t *cmd_len, [args,] oscp_transport_t transport)`.
The **wire** column shows the ASCII actually emitted (before transport framing such as COBS).

???+ note "Silent - valid in ANY state"

    | Function | Wire | Purpose |
    |----------|------|---------|
    | `oscp_cmd_reset` | `RESET` | Reset the unit |

???+ note "Silent - valid in CONFIG state"

    | Function | Wire | Purpose |
    |----------|------|---------|
    | `oscp_cmd_exit` | `EXIT` | Leave CONFIG state back to OPERATING state |
    | `oscp_cmd_refs` | `REFS` | Restore factory settings and reset the unit |

???+ note "Ack - valid in OPERATING state"

    | Function | Wire | Purpose |
    |----------|------|---------|
    | `oscp_cmd_config` | `CONFIG` | Enter CONFIG state |

???+ note "Ack - valid in CONFIG state"

    | Function | Args | Wire | Purpose |
    |----------|------|------|---------|
    | `oscp_cmd_suf`           | -                 | `SUF`        | Get a startup frame |
    | `oscp_cmd_of`            | `oscp_frame_sel_t`| `OF<x>`      | Get a specific output frame |
    | `oscp_cmd_om`            | `oscp_om_sel_t`   | `OM<x>`      | Set operating mode |
    | `oscp_cmd_enable_oft`    | `oscp_frame_sel_t`| `EOFT<x>`    | Enable an output frame type |
    | `oscp_cmd_disable_oft`   | `oscp_frame_sel_t`| `DOFT<x>`    | Disable an output frame type |
    | `oscp_cmd_drg`           | `oscp_gyro_dr_t`  | `DRG<nnnn>`  | Gyroscope dynamic range |
    | `oscp_cmd_dra`           | `oscp_accel_dr_t` | `DRA<nn>`    | Accelerometer dynamic range |
    | `oscp_cmd_dri`           | `oscp_incl_dr_t`  | `DRI<n.n>`   | Inclinometer dynamic range |
    | `oscp_cmd_enable_mcorr`  | -                 | `EMCORR`     | Enable misalignment correction |
    | `oscp_cmd_disable_mcorr` | -                 | `DMCORR`     | Disable misalignment correction |
    | `oscp_cmd_wr`            | `oscp_usr_reg_t`, `uint32_t` | `WR<REG><8·hex>` | Write a user register |
    | `oscp_cmd_save`          | -                 | `SAVE`       | Persist configuration |

---

## Argument enumerations

### Frame selector - `oscp_frame_sel_t`

Used by `oscp_cmd_of`, `oscp_cmd_enable_oft` and `oscp_cmd_disable_oft`.

| Enum | Wire char | Frame |
|------|-----------|-------|
| `OSCP_FRAME_SEL_RAW`        | `R` | Raw |
| `OSCP_FRAME_SEL_EULER`      | `E` | Euler |
| `OSCP_FRAME_SEL_QUATERNION` | `Q` | Quaternion |
| `OSCP_FRAME_SEL_ROT_MATRIX` | `M` | Rotation matrix |
| `OSCP_FRAME_SEL_GNSS`       | `G` | GNSS |
| `OSCP_FRAME_SEL_DEBUG`      | `D` | Debug - **`oscp_cmd_of` only** |

!!! warning "`OSCP_FRAME_SEL_DEBUG` is `oscp_cmd_of`-only"
    Passing `OSCP_FRAME_SEL_DEBUG` to `oscp_cmd_enable_oft` or
    `oscp_cmd_disable_oft` returns `OSCP_ERR_INVALID`. Debug frames are not meant to be sent periodically and are only receivable in CONFIG state, on-demand.

### Operating mode - `oscp_om_sel_t`

Used by `oscp_cmd_om`.

| Enum | Wire char | RAW ODR | GNSS ODR | Others ODR |
|------|-----------|------------------|------------------|------------------|
| `OSCP_OM_IDLE`   | `I` | 0 Hz | 0 Hz | 0 Hz |
| `OSCP_OM_LOW`    | `L` | 100 Hz | 1 Hz | 100 Hz |
| `OSCP_OM_MEDIUM` | `M` | 500 Hz | 1 Hz | 100 Hz |

### Gyroscope dynamic range - `oscp_gyro_dr_t`

Used by `oscp_cmd_drg`.

| Enum | Range |
|------|-------|
| `OSCP_GYRO_DR_125DPS`  | ±125 °/s |
| `OSCP_GYRO_DR_250DPS`  | ±250 °/s |
| `OSCP_GYRO_DR_500DPS`  | ±500 °/s |
| `OSCP_GYRO_DR_1000DPS` | ±1000 °/s |
| `OSCP_GYRO_DR_2000DPS` | ±2000 °/s |
| `OSCP_GYRO_DR_4000DPS` | ±4000 °/s |

### Accelerometer dynamic range - `oscp_accel_dr_t`

Used by `oscp_cmd_dra`.

| Enum | Range |
|------|-------|
| `OSCP_ACCEL_DR_2G`  | ±2 g |
| `OSCP_ACCEL_DR_4G`  | ±4 g |
| `OSCP_ACCEL_DR_8G`  | ±8 g |
| `OSCP_ACCEL_DR_16G` | ±16 g |

### Inclinometer dynamic range - `oscp_incl_dr_t`

Used by `oscp_cmd_dri`. Enum values are in tenths of g.

| Enum | Range |
|------|-------|
| `OSCP_INCL_DR_0G5` | ±0.5 g |
| `OSCP_INCL_DR_1G0` | ±1.0 g |
| `OSCP_INCL_DR_2G0` | ±2.0 g |
| `OSCP_INCL_DR_3G0` | ±3.0 g |

### User registers - `oscp_usr_reg_t`

Used by `oscp_cmd_wr`. The value is written as 8 uppercase hex digits.

??? note "Register mnemonics"

    | Group | Mnemonics |
    |-------|-----------|
    | Gyroscope bias | `GXB`, `GYB`, `GZB`, `GOB` _(`GOB` = optical gyro, optical-based models only)_ |
    | Accelerometer bias | `AXB`, `AYB`, `AZB` |
    | Inclinometer bias | `IXB`, `IYB` |
    | Magnetometer bias | `MXB`, `MYB`, `MZB` |
    | Magnetometer calibration matrix | `MXX`…`MZZ` (9 entries) |
    | AHRS fusion | `FCO` (convention), `FHS` (heading source), `FRT` (recovery period), `FGA` (gain), `FAR` (accel rejection), `FMR` (mag rejection) |
    | Gyroscope filters | `GFI` (enable bitmask), `GLP` (low-pass), `GHP` (high-pass) |
    | Accelerometer filters | `AFI` (enable bitmask), `ALP` (low-pass), `AHP` (high-pass) |

    Enum members are named `OSCP_USR_REG_<MNEMONIC>`, e.g. `OSCP_USR_REG_FGA`.

```c
/* Write AHRS fusion gain register */
oscp_cmd_wr(cmd, sizeof(cmd), &len, OSCP_USR_REG_FGA, 0x3F800000u, OSCP_TRANSPORT_RS422);
```

---

## Typical configuration sequence

1. `oscp_cmd_config` - enter CONFIG (from OPERATING), stopping periodic data output
2. `oscp_cmd_om` / `oscp_cmd_drg` / `oscp_cmd_dra` / `oscp_cmd_enable_oft` … - apply settings
3. `oscp_cmd_exit` - return to OPERATING state, enabling periodic data output

!!! success "Working Code"

    See the [command-sending example](examples.md#sending-commands) for working code.

---

Next: [Protocol & CRC →](protocol.md)
