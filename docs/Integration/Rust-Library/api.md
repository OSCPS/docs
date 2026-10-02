# API Reference

Public items are defined in the modules `oscp_imu::codec` (functions and
parser) and `oscp_imu::types` (types and constants). Both are re-exported at
the crate root, so items can be imported as `oscp_imu::<item>`.

---

## Errors

### Command and COBS errors - `OscpErr`

| Variant | Value | Meaning |
|---------|-------|---------|
| `OscpErr::Ok`         | `0`  | Defined for parity with the C Library; not returned |
| `OscpErr::Err`        | `-1` | General failure: output buffer too small, COBS decode failure |
| `OscpErr::ErrNull`    | `-2` | Defined for parity with the C Library; not returned |
| `OscpErr::ErrInvalid` | `-3` | Argument outside its valid set |

### Decoding errors - `OscpParseError`

| Variant | Meaning |
|---------|---------|
| `WrongLength { expected, actual }` | Payload shorter than the frame length (`oscp_frame_decode`), or not equal to it (`from_bytes`) |
| `UnknownFrameType(u8)` | Header byte does not map to a frame type |
| `CrcMismatch` | CRC-16 check failed |

The RS422 parser does not return errors. Dropped frames are counted in
[`OscpStats`](#statistics).

---

## Shared - every path

```rust
pub fn oscp_crc16(data: &[u8]) -> u16;
pub fn crc_check(frame: &[u8]) -> bool;

pub fn oscp_cobs_encode(data: &[u8], enc: &mut [u8]) -> Result<usize, OscpErr>;
pub fn oscp_cobs_decode(data: &[u8], dec: &mut [u8]) -> Result<usize, OscpErr>;

pub fn dispatch_frame(frame_type: u8, payload: &[u8]) -> Result<OscpFrame, OscpParseError>;
```

| Function | Description |
|----------|-------------|
| `oscp_crc16` | CRC-16 of `data`. |
| `crc_check` | Returns `true` if the last two bytes of `frame` (little-endian) equal the CRC-16 of the preceding bytes. |
| `oscp_cobs_encode` | COBS-encodes `data` into `enc` and returns the encoded length. Returns `Err(OscpErr::Err)` if `enc` is too small. |
| `oscp_cobs_decode` | COBS-decodes `data` into `dec` and returns the decoded length. Returns `Err(OscpErr::Err)` on failure. |
| `dispatch_frame` | Decodes a payload of exactly the frame length into an `OscpFrame`. Does not check the CRC. |

Each frame struct also provides `from_bytes(data: &[u8]) -> Result<Self, OscpParseError>`.
It requires a slice of exactly the frame length and does not check the CRC.

??? info "CRC-16"

    | Parameter | Value |
    |-----------|-------|
    | Polynomial (Koopman form) | `0xD175` |
    | Polynomial (normal form) | `0xA2EB` |
    | Initial value | `0xFFFF` |

    !!! warning "Polynomial form"
        `0xD175` is the Koopman representation. An MSB-first shift-register
        implementation requires the normal form `0xA2EB`
        (`(0xD175 << 1) | 1`); using `0xD175` in that loop gives incorrect
        checksums. The library uses a 256-entry lookup table derived from
        `0xA2EB`.

---

## Transport Specific

=== "Transport A - RS422 byte-stream parser"

    ```rust
    impl OscpParser {
        pub fn new() -> Self;
        pub fn reset(&mut self);
        pub fn feed(&mut self, byte: u8) -> Option<OscpFrame>;
        pub fn feed_buf<'a>(&'a mut self, buf: &'a [u8]) -> impl Iterator<Item = OscpFrame> + 'a;
        pub fn stats_reset(&mut self);
        pub stats: OscpStats,
    }
    ```

    | Item | Description |
    |------|-------------|
    | `OscpParser::new` | Creates a parser. Equivalent to `OscpParser::default()`. |
    | `reset` | Discards buffered bytes and clears synchronization. Statistics are kept. |
    | `feed` | Processes one byte. Returns `Some(frame)` when the byte completes a valid frame. |
    | `feed_buf` | Processes a buffer and yields each valid frame. Equivalent to calling `feed` for each byte. The iterator is lazy and must be consumed. |
    | `stats` | Public field containing the statistics counters. |
    | `stats_reset` | Sets all statistics counters to zero. |

    ### Returned frames

    Frames are returned by value. `OscpFrame` implements `Copy`; a returned
    frame does not borrow from the parser.

    ### Statistics

    ```rust
    pub struct OscpStats {
        pub frames_ok: u32,       /* frames successfully parsed */
        pub framing_errors: u32,  /* dropped: bad length / dispatch */
        pub crc_errors: u32,      /* dropped: CRC mismatch */
        pub cobs_errors: u32,     /* dropped: COBS decode failure */
        pub overflows: u32,       /* dropped: buffer overflow */
    }
    ```

    !!! note
        The fields `buf`, `buf_len` and `synced` of `OscpParser` are public.
        Do not modify them; call `reset()` instead.

=== "Transport B - CAN-FD message decoder"

    ```rust
    pub fn oscp_frame_decode(payload: &[u8]) -> Result<OscpFrame, OscpParseError>;
    ```

    | Function | Description |
    |----------|-------------|
    | `oscp_frame_decode` | Decodes one complete message. The frame type is read from `payload[0]`. `payload.len()` may exceed the frame length (DLC padding); the CRC is computed over the frame length only. Returns `WrongLength` for an empty or short payload and `CrcMismatch` on CRC failure. |

---

## Commands

All command encoders have this form:

```rust
pub fn oscp_cmd_X(cmd: &mut [u8],
                  /* [command-specific args] */
                  transport: OscpTransport) -> Result<usize, OscpErr>;
```

- `cmd`: caller-owned output buffer. 32 bytes is sufficient for every command.
- Return value: number of bytes written. Transmit `&cmd[..len]`.
- `transport`: `OscpTransport::Rs422` (COBS and `0x00` delimiter) or `OscpTransport::Canfd` (ASCII only).

| Function | Extra args |
|----------|-----------|
| `oscp_cmd_reset` | - |
| `oscp_cmd_exit` | - |
| `oscp_cmd_refs` | - |
| `oscp_cmd_config` | - |
| `oscp_cmd_suf` | - |
| `oscp_cmd_of` | `frame: OscpFrameSel` |
| `oscp_cmd_om` | `mode: OscpOmSel` |
| `oscp_cmd_enable_oft` | `frame: OscpFrameSel` |
| `oscp_cmd_disable_oft` | `frame: OscpFrameSel` |
| `oscp_cmd_drg` | `dr: OscpGyroDr` |
| `oscp_cmd_dra` | `dr: OscpAccelDr` |
| `oscp_cmd_dri` | `dr: OscpInclDr` |
| `oscp_cmd_enable_mcorr` | - |
| `oscp_cmd_disable_mcorr` | - |
| `oscp_cmd_wr` | `reg: OscpUsrReg, value: u32` |
| `oscp_cmd_save` | - |

Argument enumerations and command behavior are described in the
[Command Reference](commands.md).

---

## Types at a glance

| Type | Role |
|------|------|
| `OscpFrame` | Enum of decoded frames and command responses |
| `OscpFrameType` | Frame type from the header byte (`Raw` … `Startup`) |
| `OscpFrameCmd` | Command response (`Success` / `Failed` / `Unknown` / `NotImpl`) |
| `OscpParser` | RS422 parser state |
| `OscpStats` | Parser statistics counters |
| `OscpTransport` | `OscpTransport::Rs422` / `OscpTransport::Canfd` |
| `OscpErr` / `OscpParseError` | Error types |
| `OscpRaw`, `OscpEuler`, `OscpQuat`, `OscpRotMat`, `OscpGnss`, `OscpDebug1`, `OscpDebug2`, `OscpStartup` | Decoded frame structs |
| `OscpOperatingMode`, `OscpMisalignmentCorr` | Header byte fields |

### Accessor methods

| Method | Returns |
|--------|---------|
| `header_byte.frame_type()` / `.operating_mode()` / `.misalignment_corr()` | Header byte fields, as `Result<_, u8>` |
| `GnssFlags::gnss_fix_ok()` / `invalid_llh()` / `last_correction_age()` | GNSS flag bits |
| `FilterCfg::filters()` / `lpf()` / `hpf()` | Filter configuration bits |
| `FusionCfg::fusion_convention()` / `fusion_heading_source()` | AHRS fusion bits |
| `DynRangeCfg::gyro_dr()` / `accel_dr()`, `InclAhrsCfg::incl_dr()` / `ahrs_convention()` / `ahrs_heading_src()` | Startup configuration bits |
| `OscpGyroDr` / `OscpAccelDr` / `OscpInclDr` `::from_dyn_range_code(code)` | Range enum from readback code |

The packed-byte types also provide `from_byte(u8)` and `to_byte()`.

### Constants

| Constant | Value |
|----------|-------|
| `OSCP_FRAME_DELIM` | `0x00` |
| `OSCP_FRAME_MIN_LEN` | `25` |
| `OSCP_FRAME_MAX_LEN` | `72` |
| `OSCP_*_LEN` | Frame lengths; see [Frame Reference](frames.md) |
| `OSCP_STATUS_*` / `OSCP_ENABLE_*` | Status and enabled-frame masks; see [Frame Reference](frames.md) |

### `serde` feature

With `features = ["serde"]`, the following types implement
`serde::Serialize`: `OscpFrame`, the frame structs, `OscpFrameCmd`,
`OscpHeader` and the packed-byte types. The feature is disabled by default.

---

## Thread safety

!!! warning
    `OscpParser` methods take `&mut self`, so only one caller can use a parser
    at a time. `OscpParser` is `Send` and can be moved to another thread or
    shared through a `Mutex`. On embedded targets, the usual arrangement is an
    ISR writing to a lock-free ring buffer that one task drains into the
    parser. `oscp_frame_decode`, `oscp_crc16` and the command encoders have no
    shared state and can be called concurrently on separate buffers.

---

Next: [Examples →](examples.md)
