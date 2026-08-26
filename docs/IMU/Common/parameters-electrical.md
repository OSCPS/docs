# Parameters & Electrical

## Configurable Parameters

The OSCP-MK2 IMU offers several real-time configurable parameters, which can also be saved in non-volatile memory to maintain them at each start-up. Refer to the [Commands](commands.md) page for how to configure them.

<table markdown="1" class="oscp-table">
<caption><strong>Table: Configurable Parameters</strong></caption>
<colgroup>
<col style="width:22%">
<col style="width:33%">
<col style="width:45%">
</colgroup>
<thead>
<tr><th>Parameter</th><th>Configuration</th><th>Comment</th></tr>
</thead>
<tbody>
<tr><td>Output Data Rate</td><td>0 Hz, 100 Hz, 500 Hz</td><td class="text-left">Operating mode.</td></tr>
<tr><td>Gyroscope Dynamic Range</td><td>±125, ±250, ±500, ±1000, ±2000, ±4000 °/sec</td><td class="text-left">Selectable range, applies to the MEMS gyroscopes. On the MK2E2, the optical gyroscope's range is fixed and not user-configurable.</td></tr>
<tr><td>Accelerometer Dynamic Range</td><td>± 2 g, ± 4 g, ± 8 g, ± 16 g</td><td class="text-left">Selectable range.</td></tr>
<tr><td>Inclinometer Dynamic Range</td><td>± 0.5 g, ± 1.0 g, ± 2.0 g, ± 3.0 g</td><td class="text-left">Selectable range.</td></tr>
<tr><td>Enabled Frame Type</td><td>RAW, EULER, QUATERNION, ROTATION MATRIX, GNSS</td><td class="text-left">Each type of frame can be enabled or disabled. Controller Overruns can occur.</td></tr>
</tbody>
</table>

---

## Electrical Interface

!!! warning "Precautions"
    - Do not power-up the device with reverse polarity.
    - Do not exceed the input power-up voltage beyond + 34 VDC.

### Connections

In order to operate correctly, the IMU must be connected as shown in the figure below, depending on selected protocol.

=== "RS422 Connection Diagram"

    ![RS422 Connection Diagram](assets/images/imu_connection_diagram_rs422.svg){ .svg .padded }


=== "CAN-FD Connection Diagram"

    ![CAN-FD Connection Diagram](assets/images/imu_connection_diagram_can_fd.svg){ .svg .padded }


!!! info "RST Pin"
    The **active-high** reset pin (number 5, wire color orange) can be **left floating** if unused.

The terminal provided to connect the IMU is open wired on the other side of the IMU connector. The table below shows wire identification. This pinout, connector, and wiring are identical across the whole MK2 family.

<table markdown="1" class="oscp-table">
<caption><strong>Table: Wire Color Identification of the supplied open wired terminal</strong></caption>
<colgroup>
<col style="width:10%">
<col style="width:15%">
<col style="width:32%">
<col style="width:43%">
</colgroup>
<thead>
<tr><th>Pin</th><th>Wire Color</th><th>Function Assignment</th><th>Comment</th></tr>
</thead>
<tbody>
<tr><td>1</td><td>Red</td><td>RS422: <strong>RX-</strong></td><td>Connected to user <strong>TX-</strong> (RS422)</td></tr>
<tr><td>2</td><td>Yellow</td><td>RS422: <strong>RX+</strong></td><td>Connected to user <strong>TX+</strong> (RS422)</td></tr>
<tr><td>3</td><td>Black</td><td><strong>Ground</strong></td><td>Power Ground</td></tr>
<tr><td>4</td><td>White</td><td><strong>Power</strong></td><td>12V Typical</td></tr>
<tr><td>5</td><td>Orange</td><td><strong>Active-High Reset</strong></td><td><strong>Can be left floating.</strong> 3.3V, 5V Tolerant</td></tr>
<tr><td>6</td><td>Blue</td><td>RS422: <strong>TX+</strong> / CAN-FD: <strong>P</strong></td><td>Connected to user <strong>RX+</strong> (RS422)</td></tr>
<tr><td>7</td><td>Green</td><td>RS422: <strong>TX-</strong> / CAN-FD: <strong>N</strong></td><td>Connected to user <strong>RX-</strong> (RS422)</td></tr>
<tr><td>8</td><td>Purple</td><td>RS422 / CAN-FD: <strong>GND</strong></td><td>Isolated Ground</td></tr>
<tr><td>Shell</td><td>Shield</td><td>-</td><td>-</td></tr>
</tbody>
</table>


Otherwise, the IMU connector is **54-00251**, its mating connector is **50-00983** and an example of mating cable assembly is **10-03817**. All are from *Tensility International Corp*.