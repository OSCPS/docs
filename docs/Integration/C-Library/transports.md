# Integration Paths

Transport splits the library cleanly in two sections.
Both halves produce the same decoded `oscp_frame_t`, such that an application can stay transport-agnostic.

---

## Which one to use?

| Variant | What to call | Why |
|---------|---------------|-----|
| **RS422** (stream) | `oscp_parser_*` | Bytes arrive with **no** message boundaries. The parser syncs on the `0x00` delimiter, COBS-decodes, checks CRC, and invokes a given callback once per valid frame. |
| **CAN-FD** (framed) | `oscp_frame_decode()` | The controller already delivers **complete** messages. One stateless call. No parser, no ring buffer, no COBS. |

!!! tip "Rule of thumb"
    If the hardware hands a byte at a time (a UART/RS422 line), use the
    **parser**. If it hands a complete payload (an FD-CAN mailbox), use the
    **decoder**.

---

## Transport Examples

=== "Transport A - RS422 byte-stream parser"

    Use this when bytes arrive without message boundaries. Feed the parser bytes as
    they arrive. Could be a single byte from an ISR, or a buffer drained from a ring. Parser calls the callback once per valid frame.

    ```text
    bytes ─► parser_feed() ─► sync on 0x00 ─► COBS decode ─► CRC check ─► callback
    ```

    The parser owns a fixed-size buffer inside `oscp_parser_t`; there is no dynamic
    allocation. Framing, COBS, CRC and overflow errors are counted in
    `oscp_stats_t` rather than thrown.

    ```c
    oscp_parser_t parser;
    oscp_parser_init(&parser, on_frame, NULL);

    uint8_t buf[256];
    size_t  n;
    while ((n = uart_read(buf, sizeof(buf))) > 0) {
        oscp_parser_feed_buf(&parser, buf, n); /* on_frame() fires per frame */
    }
    ```

    See [Examples](examples.md) for more details.

=== "Transport B - CAN-FD message decoder"

    Use this when the transport already delivers whole messages. A single stateless
    call decodes one payload into a frame, no parser state to carry between calls.

    ```c
    void on_can_message(const uint8_t *data, size_t len) {
        oscp_frame_t frame;
        if (oscp_frame_decode(data, len, &frame) == OSCP_OK) {
            /* frame is valid and CRC-checked */
        }
    }
    ```

    !!! note "DLC padding is handled by the decoder"
        CAN-FD pads payloads up to the nearest DLC bucket, so a 61-byte RAW frame
        rides inside a 64-byte message. `len` may therefore **exceed** the frame's
        own length. The decoder derives the true length from the frame type and
        verifies the CRC over the frame only. Padding is ignored.

    See [Examples](examples.md) for more details.

---

## Both paths converge

Whichever transport used, decoding yields a tagged union:

```c
typedef struct {
    oscp_frame_type_t type;
    union {
        oscp_raw_t     raw;
        oscp_euler_t   euler;
        oscp_quat_t    quat;
        oscp_rot_mat_t rot_mat;
        oscp_gnss_t    gnss;
        oscp_debug_1_t debug1;
        oscp_debug_2_t debug2;
        oscp_startup_t startup;
    } content;
} oscp_frame_t;
```

!!! success "Ease of use"
    Branch on `frame.type` and read the matching member through the accessor
    macros (`oscp_euler(&frame)`, `oscp_raw(&frame)`, …). The full frame catalogue
    is in the [Frame Reference](frames.md).

---

Next: [Frame reference →](frames.md)
