# Integration Paths

The library provides one receive path per transport. Both return `OscpFrame`,
so code downstream of reception does not depend on the transport.

---

## Selecting a path

| Variant | Entry point | Behavior |
|---------|-------------|----------|
| RS422 (stream) | `OscpParser` | Input has no message boundaries. The parser synchronizes on the `0x00` delimiter, COBS-decodes, checks the CRC and returns one `OscpFrame` per valid frame. |
| CAN-FD (framed) | `oscp_frame_decode()` | Input is one complete message. Stateless; no parser, buffer or COBS decoding. |

Use the parser when the interface delivers bytes (UART/RS422). Use the decoder
when the interface delivers complete payloads (FD-CAN mailbox).

---

## Transport Examples

=== "Transport A - RS422 byte-stream parser"

    Bytes can be passed one at a time (e.g. drained from an ISR queue) or as a
    buffer. The parser returns a frame each time a valid frame is completed.

    ```text
    bytes ─► feed() / feed_buf() ─► sync on 0x00 ─► COBS decode ─► CRC check ─► OscpFrame
    ```

    The parser holds a fixed-size buffer inside `OscpParser` and does not
    allocate. Framing, COBS, CRC and overflow errors are counted in
    `OscpStats`; they are not returned to the caller.

    ```rust
    use oscp_imu::OscpParser;

    let mut parser = OscpParser::new();

    let mut buf = [0u8; 256];
    loop {
        let n = uart_read(&mut buf);
        for frame in parser.feed_buf(&buf[..n]) {
            /* one OscpFrame per valid frame */
        }
    }
    ```

    !!! warning "`feed_buf` returns a lazy iterator"
        Bytes are consumed only while the iterator is advanced. A statement
        such as `parser.feed_buf(&buf);` whose result is discarded feeds no
        bytes. Consume the iterator with `for`, `.for_each()` or `.count()`.

    See [Examples](examples.md).

=== "Transport B - CAN-FD message decoder"

    Each call decodes one payload. No state is kept between calls.

    ```rust
    use oscp_imu::oscp_frame_decode;

    fn on_can_message(data: &[u8]) {
        if let Ok(frame) = oscp_frame_decode(data) {
            /* frame passed length and CRC checks */
        }
    }
    ```

    !!! note "DLC padding"
        CAN-FD pads a payload to the next DLC size; a 61-byte RAW frame is sent
        in a 64-byte message. `data.len()` can therefore exceed the frame
        length. The decoder takes the frame length from the frame type,
        computes the CRC over that length and ignores the remaining bytes.

    See [Examples](examples.md).

---

## Common output type

Both paths return the following enum:

```rust
pub enum OscpFrame {
    Raw(OscpRaw),
    Euler(OscpEuler),
    Quat(OscpQuat),
    RotMat(OscpRotMat),
    Gnss(OscpGnss),
    Debug1(OscpDebug1),
    Debug2(OscpDebug2),
    Startup(OscpStartup),
    CmdResponse(OscpFrameCmd),
}
```

Use `match` to select the variant and bind its struct, e.g.
`OscpFrame::Euler(e) => e.roll`. Frame contents are listed in the
[Frame Reference](frames.md).

---

Next: [Frame reference →](frames.md)
