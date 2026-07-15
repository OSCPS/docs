# Installation

Build, compile, and flash ArduPilot firmware with [OSCP IMU Products](https://www.oscp.com/technology){:target="_blank"} on Linux or Windows.

---

### Development Environment

This integration is developed and tested on the following tools and platforms:

<div class="grid cards" markdown>

-   :material-linux:{ .lg .middle } <b>[Ubuntu 22.04](https://ubuntu.com/download/desktop){:target="_blank"}</b>
    -
    Recommended development and build environment host used for compiling and testing firmware.
-   :material-git:{ .lg .middle } <b>[Git](https://git-scm.com){:target="_blank"}</b>
    -
    Source code version control and repository management.
-   :material-chip:{ .lg .middle } <b>[ARM GNU Toolchain](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain){:target="_blank"}</b>
    -
    GCC-based toolchain for compiling ARM firmware.
-   :material-usb:{ .lg .middle } <b>[STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html){:target="_blank"}</b>
    -
    Firmware deployment tool for STM32-based flight controllers.
-   :material-quadcopter:{ .lg .middle } <b>[Mission Planner](https://ardupilot.org/planner/docs/mission-planner-installation.html){:target="_blank"}</b>
    -
    Ground control station for configuration and flight management.
-   :material-serial-port:{ .lg .middle } <b>[OSCP IMU Hardware](https://www.oscp.com/technology){:target="_blank"}</b>
    -
    External IMU sensor unit for integration testing.

</div>


### Prerequisites

=== "Linux"

    !!! note "Ubuntu Installation"
        This guide targets Ubuntu 22.04 LTS (Jammy Jellyfish). Download the Desktop image from the official Ubuntu releases page and install natively.
    
        [Download Ubuntu 22.04.5 LTS Desktop](https://releases.ubuntu.com/22.04/){:target="_blank"}

    Update your package lists and install Git:

    ```bash
    sudo apt-get update
    sudo apt-get install git
    sudo apt-get install gitk git-gui
    ```

=== "Windows"

    !!! note "WSL Installation"
        This guide targets Ubuntu 22.04 LTS (Jammy Jellyfish). Install WSL with an Ubuntu 22.04 LTS distribution via the command line.

      Install the WSL distribution with the following command:

    ```powershell
    wsl --install -d Ubuntu-22.04
    ```

    Restart your computer when prompted.

    After restarting, launch <b>Ubuntu</b> from the Start Menu and complete the initial setup by creating a Linux username and password.

    Verify that WSL is installed successfully:

    ```powershell
    wsl --status
    ```

    Once Ubuntu has opened, update the package lists and install Git:

    ```bash
    sudo apt-get update
    sudo apt-get install git
    sudo apt-get install gitk git-gui
    ```

Clone the OSCP ArduPilot repository with required submodules:

```bash
git clone --recurse-submodules https://github.com/OSCPS/oscp_ardupilot.git
cd oscp_ardupilot
```

Install the ArduPilot build dependencies:

```bash
Tools/environment_install/install-prereqs-ubuntu.sh -y
. ~/.profile
```

Restart your terminal or run:

```bash
source ~/.profile
```

---

## Hardware Configuration

To ensure proper integration, the External AHRS (<b>OSCP IMU</b>) backend must be explicitly enabled in your flight controller's hardware definition.

!!! important "Board Flash Size"
    These steps apply regardless of your board's flash size. Boards with <b>1 MB</b> of flash have some features disabled by default to save flash.

    The defines below ensure the OSCP backend remains compiled in, but you may need to disable other features to save flash for the External AHRS driver on <b>1 MB</b> boards.


1. Navigate to your target flight controller board's hardware definition directory:

```text
    libraries/AP_HAL_ChibiOS/hwdef/<BOARD_NAME>
```

!!! tip "Board Name"
      Use the directory path corresponding to your actual target flight controller board.

      ```bash
      libraries/AP_HAL_ChibiOS/hwdef/Pixhawk6C-bdshot
      ```

2. Open the board's `hwdef.dat` file.

3. Append the following configuration block to the end of the file:

```bash
    #------------------------------------------------
    # External AHRS — OSCP IMU
    #------------------------------------------------
    define AP_EXTERNAL_AHRS_ENABLED 1
    define AP_EXTERNAL_AHRS_BACKEND_DEFAULT_ENABLED 1
    define AP_EXTERNAL_AHRS_OSCP_ENABLED 1
    define AP_AHRS_ENABLED 1
    define HAL_EXTERNAL_AHRS_DEFAULT 15

    # EKF3 & External Navigation Support
    define HAL_NAVEKF3_AVAILABLE 1
    define EK3_FEATURE_EXTERNAL_NAV 1
```

---

## Build the OSCP Custom Firmware

Configure the build for your target flight controller:

```bash
./waf configure --board <BOARD_NAME>
```

!!! tip "Board Name"
      Use the directory path corresponding to your actual target flight controller board.

      ```bash
      ./waf configure --board Pixhawk6C-bdshot
      ```

Build the OSCP firmware (ArduCopter shown below):

```bash
./waf copter
```

!!! note "Build Cleanup"
    Standard builds are the default and recommended workflow. Use `./waf clean` only if you encounter build issues or switch between development branches. For a full reset, use `./waf distclean`.

The generated OSCP firmware will be located in:

```text
build/<BOARD_NAME>/bin/
```

!!! tip "Board Name"
      Use the directory path corresponding to your actual target flight controller board and username.

      ```bash
      \\wsl.localhost\Ubuntu-22.04\home\USERNAME\oscp_ardupilot\build\Pixhawk6C-bdshot\bin\arducopter.apj
      ```

---

## Flash the OSCP Custom Firmware

The following table outlines the appropriate flashing method based on the flight controller board's bootloader status.

| Board State | Bootloader | Firmware File | Flashing Tool |
|--------------|:----------:|---------------|---------------|
| Already runs<br>ArduPilot | ✅ | `arducopter.apj` | Mission Planner |
| New board | ❌ | `arducopter_with_bl.hex` | Mission Planner<br> & STM32CubeProgrammer |
| Corrupted<br>bootloader | ⚠️ | `arducopter_with_bl.hex` | Mission Planner<br> & STM32CubeProgrammer |

!!! note "Why `_with_bl.hex`?"
    This file contains both the ArduPilot bootloader and the firmware, allowing you to install both in a single flash operation.

---

<script>
function openTab(tabName) {
  // Find all tab labels generated by MkDocs
  const labels = document.querySelectorAll('.tabbed-labels label, .tabbed-set label');
  for (const label of labels) {
    if (label.textContent.trim() === tabName) {
      label.click();
      // Smoothly scroll the new tab area into view
      label.scrollIntoView({ behavior: 'smooth', block: 'start' });
      break;
    }
  }
}
</script>

=== "STM32CubeProgrammer"

    !!! note "When you need this"
        Use this path only if the board has <u>no ArduPilot bootloader</u> or if the existing bootloader is <u>corrupted</u> and the board is no longer recognized by Mission Planner. This uses the STM32 ROM-level DFU bootloader and not the ArduPilot bootloader.

    Enter STM32 DFU mode to flash the `_with_bl.hex` file. Refer to your flight controller hardware instructions for the specific DFU entry procedure.

    === "Linux"

        Install `dfu-util` if not already available:

        ```bash
        sudo apt-get install dfu-util
        ```

        Verify the board has entered DFU mode:

        ```bash
        dfu-util --list
        ```

        Check the kernel log for USB device detection.

    === "Windows"

        Open <b>Device Manager</b>

        Expand <b>Universal Serial Bus devices</b>

        Confirm the board appears as <b>STM32 BOOTLOADER</b>

        ![STMBootloader](../assets/images/stm32bootloader.png)

    !!! warning
        If the device does not appear in either environment, disconnect the board and repeat/diagnose the DFU entry procedure.

    <b>Flash using STM32CubeProgrammer:</b>

    1. Open STM32CubeProgrammer.
    2. Click <b>Connect</b>  to display the flight controller information.

        ![STM32CubeProgrammer Connect](../assets/images/stm32bootloader_connect.png)

    3. Select <b>Open file</b> to locate the `arducopter_with_bl.hex` file:

        ![STM32CubeProgrammer OpenFile](../assets/images/stm32bootloader_openfile.png)

        ```
        build/<BOARD_NAME>/bin/arducopter_with_bl.hex
        ```

        !!! tip "Board Name"
            Use the directory path corresponding to your actual target flight controller board.

            ```bash
            /home/username/ardupilot/build/pixhawk6c-bdshot/bin/arducopter_with_bl.hex
            ```

    4. Press <b>Download</b> to flash the image.

         ![STM32CubeProgrammer Download](../assets/images/stm32bootloader_download.png)
        
    5. Wait until the progress bar reaches <b>100%</b> as this indicates the firmware has been successfully installed.
    6. Disconnect the software and proceed to <a href="#" onclick="openTab('Mission Planner'); return false;">Mission Planner</a>.
    
        ![STM32CubeProgrammer Connect](../assets/images/stm32bootloader_disconnect.png)
        
        !!! warning "Action Required: Complete Firmware Installation"
            Flashing via STM32CubeProgrammer only installs the bootloader. To complete the installation, you must switch to the [Mission Planner](#){: onclick="openTab('Mission Planner'); return false;" } tab and upload the custom `.apj` firmware.


=== "Mission Planner"

    !!! note "When you need this"
        Use this path for routine firmware updates on a board that already has ArduPilot installed. This flashes over the existing ArduPilot bootloader, so no DFU mode is needed.

    === "Linux"

        !!! warning "Installing Mission Planner on Linux"
            Mission Planner is designed for Windows. While Mono provides a way to run it on Linux, you may experience minor bugs. For a more reliable experience, consider using a Windows.

        1. Install the latest version of <b>Mono</b>:
        ```bash
        sudo apt install mono-complete
        ```

        2. Download <b>Mission Planner</b> as a zip file from the [here](https://ardupilot.org/planner/docs/mission-planner-installation.html#mission-planner-on-linux){:target="_blank"}.

        3. Unzip the downloaded file to a directory.

        4. Change to that directory and launch Mission Planner:
        ```bash
        cd /path/to/missionplanner/
        mono MissionPlanner.exe
        ```

    === "Windows"

        !!! note "Installing Mission Planner on Windows"
            Download the latest installer from [here](https://ardupilot.org/planner/docs/mission-planner-installation.html){:target="_blank"}.

    <b>Flash using Mission Planner:</b>

    1. Open Mission Planner.
    2. Navigate to <b>Setup</b> to access firmware installation options.
    3. Select <b>Install Firmware</b> to open the firmware selection screen.
    4. Click <b>Load Custom Firmware</b> to browse for your custom `.apj` firmware file.

        ![Mission Planner Firmware](../assets/images/mission_planner_firmware.png)

    5. Browse to the generated firmware:

        ```
        build/<BOARD_NAME>/bin/arducopter.apj
        ```

        !!! tip "Board Name"
            Use the directory path corresponding to your actual target flight controller board.

            ```bash
            /home/username/ardupilot/build/pixhawk6c-bdshot/bin/arducopter.hex
            ```

    6. Select the file and allow Mission Planner to complete the upload.
    7. Reboot the flight controller once flashing is complete.

        !!! tip "Successful Reboot"
            Disconnect and reconnect the flight controller to ensure a successful reboot.

---

!!! success "Next Steps"
    Continue to [Initialization](launch.md) for instructions on connecting and configuring your OSCP IMU in Mission Planner.

---