# Commands

The user can use either the RS-422 or the CAN-FD interface, depending on the variant, to transmit commands to the IMU in ASCII format. This command set is identical across the whole MK2 family. Below is an exhaustive and descriptive list of the available commands.

Remember that, for the RS422 variant, commands must be *COBS*-encoded and terminated with a `0x00` byte before transmission. Refer to [RS422 Frame Decoding & Encoding — COBS](data-frames.md#rs422-frame-decoding-encoding-cobs-consistent-overhead-byte-stuffing) if needed.

## Enter Configuration State
 
<table markdown="1" class="oscp-table">
<colgroup>
<col style="width:20%">
<col style="width:25%">
<col style="width:55%">
</colgroup>
<thead>
<tr><th>Command</th><th>IMU State</th><th>IMU Response</th></tr>
</thead>
<tbody>
<tr><td><code>CONFIG\r\n</code></td><td>Operating State</td><td class="text-left">IMU switches to Configuration State.</td></tr>
</tbody>
</table>

## Exit to Operating State
 
<table markdown="1" class="oscp-table">
<colgroup>
<col style="width:20%">
<col style="width:25%">
<col style="width:55%">
</colgroup>
<thead>
<tr><th>Command</th><th>IMU State</th><th>IMU Response</th></tr>
</thead>
<tbody>
<tr><td><code>EXIT\r\n</code></td><td>Configuration State</td><td class="text-left">IMU switches to Operating State.</td></tr>
</tbody>
</table>

## Soft-Reset
 
<table markdown="1" class="oscp-table">
<colgroup>
<col style="width:20%">
<col style="width:24%">
<col style="width:56%">
</colgroup>
<thead>
<tr><th>Command</th><th>IMU State</th><th>IMU Response</th></tr>
</thead>
<tbody>
<tr><td><code>RESET\r\n</code></td><td>All</td><td class="text-left">IMU resets (switches to Startup State, from software).</td></tr>
</tbody>
</table>

## Get a Startup Frame
 
<table markdown="1" class="oscp-table">
<colgroup>
<col style="width:20%">
<col style="width:25%">
<col style="width:55%">
</colgroup>
<thead>
<tr><th>Command</th><th>IMU State</th><th>IMU Response</th></tr>
</thead>
<tbody>
<tr><td><code>SUF\r\n</code></td><td>Configuration State</td><td class="text-left">IMU sends a SUF with its current configuration.</td></tr>
</tbody>
</table>

## Get an Operating Frame
 
<table markdown="1" class="oscp-table">
<colgroup>
<col style="width:32%">
<col style="width:25%">
<col style="width:43%">
</colgroup>
<thead>
<tr><th>Command</th><th>IMU State</th><th>IMU Response</th></tr>
</thead>
<tbody>
<tr><td><code>OF&lt;Frame Type*&gt;\r\n</code></td><td>Configuration State</td><td class="text-left">IMU performs and transmits an OF.</td></tr>
</tbody>
</table>

!!! info "Details"
    *Frame Type* parameter must be encoded with: **R**, **E**, **Q**, **M**, **G**, or **D** only. For example, to receive a Quaternions Operating Frame from the IMU, send *OFQ\r\n*.
 
## Change Operating Mode
 
<table markdown="1" class="oscp-table">
<colgroup>
<col style="width:32%">
<col style="width:25%">
<col style="width:43%">
</colgroup>
<thead>
<tr><th>Command</th><th>IMU State</th><th>IMU Response</th></tr>
</thead>
<tbody>
<tr><td><code>OM&lt;Operating Mode*&gt;\r\n</code></td><td>Configuration State</td><td class="text-left">IMU changes its OM.</td></tr>
</tbody>
</table>

!!! info "Details"
    *Operating Mode* parameter must be encoded with: **I**, **L**, or **M** only. For example, to configure the IMU to Medium Speed, send *OMM\r\n*.
 
## Change Operating Frame Types
 
<table markdown="1" class="oscp-table">
<colgroup>
<col style="width:32%">
<col style="width:25%">
<col style="width:43%">
</colgroup>
<thead>
<tr><th>Command</th><th>IMU State</th><th>IMU Response</th></tr>
</thead>
<tbody>
<tr><td><code>EOFT&lt;Frame Type*&gt;\r\n</code></td><td>Configuration State</td><td class="text-left">IMU enables selected OF type.</td></tr>
<tr><td><code>DOFT&lt;Frame Type*&gt;\r\n</code></td><td>Configuration State</td><td class="text-left">IMU disables selected OF type.</td></tr>
</tbody>
</table>

!!! info "Details"
    *Frame Type* parameter must be encoded using one of the following characters: **R**, **E**, **Q**, **M**, or **G**. For example, to enable the Euler Angles frame, send *EOFTE\r\n*. If the Rotation Matrix frame is no longer needed, send *DOFTM\r\n*.
 
## Change Dynamic Range of Gyroscopes
 
<table markdown="1" class="oscp-table">
<colgroup>
<col style="width:32%">
<col style="width:25%">
<col style="width:43%">
</colgroup>
<thead>
<tr><th>Command</th><th>IMU State</th><th>IMU Response</th></tr>
</thead>
<tbody>
<tr><td><code>DRG&lt;Range*&gt;\r\n</code></td><td>Configuration State</td><td class="text-left">Updates gyroscopes DR.</td></tr>
</tbody>
</table>

!!! info "Details"
    *Range* parameter must be encoded with: **0125**, **0250**, **0500**, **1000**, **2000** or **4000** only. For example, to configure gyroscopes with a DR of *250°/sec*, send *DRG0250\r\n*. This command targets the MEMS gyroscopes; on the MK2E2, the optical gyroscope's fixed range is unaffected.
 
## Change Dynamic Range of Accelerometers
 
<table markdown="1" class="oscp-table">
<colgroup>
<col style="width:32%">
<col style="width:25%">
<col style="width:43%">
</colgroup>
<thead>
<tr><th>Command</th><th>IMU State</th><th>IMU Response</th></tr>
</thead>
<tbody>
<tr><td><code>DRA&lt;Range*&gt;\r\n</code></td><td>Configuration State</td><td class="text-left">Updates accelerometers DR.</td></tr>
</tbody>
</table>

!!! info "Details"
    *Range* parameter must be encoded with: **02**, **04**, **08** or **16** only. For example, to configure accelerometers with a DR of *4g*, send *DRA04\r\n*.
 
## Change Dynamic Range of Inclinometers
 
<table markdown="1" class="oscp-table">
<colgroup>
<col style="width:32%">
<col style="width:25%">
<col style="width:43%">
</colgroup>
<thead>
<tr><th>Command</th><th>IMU State</th><th>IMU Response</th></tr>
</thead>
<tbody>
<tr><td><code>DRI&lt;Range*&gt;\r\n</code></td><td>Configuration State</td><td class="text-left">Updates inclinometer DR.</td></tr>
</tbody>
</table>

!!! info "Details"
    *Range* parameter must be encoded with: **0.5**, **1.0**, **2.0** or **3.0** only. For example, to configure inclinometer with a DR of *1g*, send *DRI1.0\r\n*.
 
## Change Misalignment Correction Status
 
<table markdown="1" class="oscp-table">
<colgroup>
<col style="width:20%">
<col style="width:25%">
<col style="width:55%">
</colgroup>
<thead>
<tr><th>Command</th><th>IMU State</th><th>IMU Response</th></tr>
</thead>
<tbody>
<tr><td><code>EMCORR\r\n</code></td><td>Configuration State</td><td class="text-left">IMU enables the misalignment correction.</td></tr>
<tr><td><code>DMCORR\r\n</code></td><td>Configuration State</td><td class="text-left">IMU disables the misalignment correction.</td></tr>
</tbody>
</table>

## Write User Calibration Register, AHRS & Filters Configurations
 
<table markdown="1" class="oscp-table">
<colgroup>
<col style="width:32%">
<col style="width:25%">
<col style="width:43%">
</colgroup>
<thead>
<tr><th>Command</th><th>IMU State</th><th>IMU Response</th></tr>
</thead>
<tbody>
<tr><td><code>WR&lt;Reg*&gt;&lt;Value*&gt;\r\n</code></td><td>Configuration State</td><td class="text-left">Updates specified register.</td></tr>
</tbody>
</table>

!!! info "Details"
    *Reg* parameter must be encoded using one of the following three-character codes: **GXB, GYB, GZB, AXB, AYB, AZB, IXB, IYB, MXB, MYB, MZB, MXX, MYX, MZX, MXY, MYY, MZY, MXZ, MYZ, MZZ, FCO, FHS, FRT, FGA, FAR, FMR, GFI, GLP, GHP, AFI, ALP,** or **AHP** — plus **GOB** on the **MK2E2** only, for the Optical Gyroscope Z Bias.
 
!!! info "Details"
    *Value* parameter must be encoded as exactly 8 hexadecimal characters following the type of the register such as 32-bit floating-point value (*float32*) for calibration registers and *FGA*, *FAR*, *FMR*, or unsigned 32-bit for the rest. For example, to configure the user gyroscope x-axis bias with an offset of *1.0*, send *WRGXB3F800000\r\n*. Another example, to configure the fusion heading source with the internal magnetometer which is *1*, send *WRFHS00000001\r\n*.
 
## Save Current Configuration to Flash Memory
 
<table markdown="1" class="oscp-table">
<colgroup>
<col style="width:20%">
<col style="width:25%">
<col style="width:55%">
</colgroup>
<thead>
<tr><th>Command</th><th>IMU State</th><th>IMU Response</th></tr>
</thead>
<tbody>
<tr><td><code>SAVE\r\n</code></td><td>Configuration State</td><td class="text-left">IMU saves its current configuration.</td></tr>
</tbody>
</table>

## Restore to Factory Settings
 
<table markdown="1" class="oscp-table">
<colgroup>
<col style="width:20%">
<col style="width:25%">
<col style="width:55%">
</colgroup>
<thead>
<tr><th>Command</th><th>IMU State</th><th>IMU Response</th></tr>
</thead>
<tbody>
<tr><td><code>REFS\r\n</code></td><td>Configuration State</td><td class="text-left">IMU is reset to factory settings.</td></tr>
</tbody>
</table>

## Response to a Recognized Command
 
When a recognized command is executed while the IMU is in Configuration State, the following commands indicate successful or failed execution by sending a human-readable ASCII message terminated by **\r\n**: *CONFIG*, *OM*, *EOFT*, *DEOFT*, *EMCORR*, *DMCORR*, *WR*, *DRG*, *DRA*, *DRI*, *SAVE*, and *REFS* (only if failed for the latter).
 
## Undefined Command
 
When a command is not recognized or incorrectly formatted in Configuration State, the IMU responds with a human-readable ASCII error message terminated by **\r\n**. A message not recognized in Operating State will simply be ignored without any response.