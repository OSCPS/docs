# API Reference

All public symbols live in `oscp_imu.h`. The header is C++-safe (`extern "C"`).
Functions are grouped by the integration path they belong to.

---

## Return codes - `oscp_err_t`

| Code | Value | Meaning |
|------|-------|---------|
| `OSCP_OK`          | `0`  | Success |
| `OSCP_ERR`         | `-1` | Generic failure (bad length, CRC, COBS, …) |
| `OSCP_ERR_NULL`    | `-2` | A required pointer argument was `NULL` |
| `OSCP_ERR_INVALID` | `-3` | An argument was outside its valid set |

The `void`-returning parser functions never fail loudly; they count problems in
[`oscp_stats_t`](#statistics) instead.

---

## Shared - every path

```c
uint16_t oscp_crc16(const uint8_t *data, size_t len);

oscp_err_t oscp_cobs_encode(const uint8_t *data, size_t data_len,
                            uint8_t *enc, size_t enc_max_len, size_t *enc_len);
oscp_err_t oscp_cobs_decode(const uint8_t *data, size_t data_len,
                            uint8_t *dec, size_t dec_max_len, size_t *dec_len);
```

| Function | Description |
|----------|-------------|
| `oscp_crc16` | Compute the CRC-16 over `len` bytes. See [Protocol & CRC](protocol.md#crc-16). |
| `oscp_cobs_encode` | COBS-encode `data` into `enc`; writes the encoded length to `*enc_len`. |
| `oscp_cobs_decode` | COBS-decode `data` into `dec`; writes the decoded length to `*dec_len`. |

---

## Transport Specific

=== "Transport A - RS422 byte-stream parser"

    ```c
    void oscp_parser_init(oscp_parser_t *parser, oscp_frame_callback_t cb, void *ctx);
    void oscp_parser_reset(oscp_parser_t *parser);
    void oscp_parser_feed(oscp_parser_t *parser, uint8_t byte);
    void oscp_parser_feed_buf(oscp_parser_t *parser, const uint8_t *buf, size_t buf_len);
    const oscp_stats_t *oscp_parser_stats(const oscp_parser_t *parser);
    void oscp_parser_stats_reset(oscp_parser_t *parser);
    ```

    | Function | Description |
    |----------|-------------|
    | `oscp_parser_init` | Initialize a caller-owned parser. `cb` fires once per valid frame; `ctx` is passed back to it unchanged. |
    | `oscp_parser_reset` | Drop any buffered bytes and clear sync. Stats are preserved. |
    | `oscp_parser_feed` | Feed a single byte - e.g. straight from a UART ISR. |
    | `oscp_parser_feed_buf` | Feed a buffer of bytes; equivalent to calling `oscp_parser_feed` per byte. |
    | `oscp_parser_stats` | Borrow a read-only pointer to the running statistics. |
    | `oscp_parser_stats_reset` | Zero the statistics counters. |

    ### The callback

    ```c
    typedef void (*oscp_frame_callback_t)(const oscp_frame_t *frame, void *ctx);
    ```

    Invoked synchronously from `oscp_parser_feed` / `oscp_parser_feed_buf` for each
    valid, CRC-checked frame. The `frame` pointer is only valid for the duration of
    the call - copy out anything needed.

    ### Statistics

    ```c
    typedef struct {
        uint32_t frames_ok;       /* frames successfully parsed */
        uint32_t framing_errors;  /* dropped: bad length / dispatch */
        uint32_t crc_errors;      /* dropped: CRC mismatch */
        uint32_t cobs_errors;     /* dropped: COBS decode failure */
        uint32_t overflows;       /* dropped: buffer overflow */
    } oscp_stats_t;
    ```

=== "Transport B - CAN-FD message decoder"

    ```c
    oscp_err_t oscp_frame_decode(const uint8_t *payload, size_t len, oscp_frame_t *frame);
    ```

    | Function | Description |
    |----------|-------------|
    | `oscp_frame_decode` | Decode one complete message into `frame`. Derives the frame type from `payload[0]`, tolerates DLC padding (`len` ≥ frame length), and verifies the CRC over the frame only. Returns `OSCP_ERR_NULL` on `NULL` args, `OSCP_ERR` on short length or CRC failure. |

---

## Commands

All command encoders share this shape:

```c
oscp_err_t oscp_cmd_X(uint8_t *cmd, size_t cmd_max_len, size_t *cmd_len,
                      /* [command-specific args] */
                      oscp_transport_t transport);
```

- `cmd` / `cmd_max_len` - caller-owned output buffer and its capacity (32 bytes is always enough).
- `cmd_len` - receives the number of bytes written.
- `transport` - `OSCP_TRANSPORT_RS422` (COBS + `0x00`) or `OSCP_TRANSPORT_CANFD` (raw ASCII).

| Function | Extra args |
|----------|-----------|
| `oscp_cmd_reset` | - |
| `oscp_cmd_exit` | - |
| `oscp_cmd_refs` | - |
| `oscp_cmd_config` | - |
| `oscp_cmd_suf` | - |
| `oscp_cmd_of` | `oscp_frame_sel_t frame` |
| `oscp_cmd_om` | `oscp_om_sel_t mode` |
| `oscp_cmd_enable_oft` | `oscp_frame_sel_t frame` |
| `oscp_cmd_disable_oft` | `oscp_frame_sel_t frame` |
| `oscp_cmd_drg` | `oscp_gyro_dr_t dr` |
| `oscp_cmd_dra` | `oscp_accel_dr_t dr` |
| `oscp_cmd_dri` | `oscp_incl_dr_t dr` |
| `oscp_cmd_enable_mcorr` | - |
| `oscp_cmd_disable_mcorr` | - |
| `oscp_cmd_wr` | `oscp_usr_reg_t reg, uint32_t value` |
| `oscp_cmd_save` | - |

Argument enumerations and per-command semantics are documented in the
[Command Reference](commands.md).

---

## Types at a glance

| Type | Role |
|------|------|
| `oscp_frame_t` | Tagged union of all decoded frames (`.type` + `.content`) |
| `oscp_frame_type_t` | Frame type enum (`OSCP_FRAME_RAW` … `OSCP_FRAME_STARTUP`, plus `OSCP_FRAME_CMD_*`) |
| `oscp_parser_t` | Caller-owned RS422 parser state |
| `oscp_stats_t` | Parser statistics counters |
| `oscp_frame_callback_t` | Per-frame callback signature |
| `oscp_transport_t` | `OSCP_TRANSPORT_RS422` / `OSCP_TRANSPORT_CANFD` |
| `oscp_err_t` | Return codes |
| `oscp_raw_t`, `oscp_euler_t`, `oscp_quat_t`, `oscp_rot_mat_t`, `oscp_gnss_t`, `oscp_debug_1_t`, `oscp_debug_2_t`, `oscp_startup_t` | Per-frame decoded structs |

### Accessor macros

| Macro | Returns |
|-------|---------|
| `oscp_raw(f)` / `oscp_euler(f)` / `oscp_quat(f)` / `oscp_rot_mat(f)` / `oscp_gnss(f)` / `oscp_debug_1(f)` / `oscp_debug_2(f)` / `oscp_startup(f)` | The matching `.content` member of `f` |
| `OSCP_HDR_FRAME_TYPE(f)` / `OSCP_HDR_OPERATING_MODE(f)` / `OSCP_HDR_MISALIGNMENT_CORR(f)` | Decoded fields of the header byte |

---

## Thread safety

!!! warning
    `oscp_parser_t` is **not** thread-safe. `oscp_parser_feed`,
    `oscp_parser_feed_buf` and the frame callback must run in the same execution
    context. The usual embedded pattern is a lock-free ring buffer written by
    the UART ISR and drained by a single task that owns the parser. The
    stateless `oscp_frame_decode`, `oscp_crc16` and command encoders keep no
    shared state and are safe to call concurrently on distinct buffers.

---

Next: [Examples →](examples.md)
