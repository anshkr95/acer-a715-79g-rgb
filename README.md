# Acer Aspire 7 (A715-79G) RGB Keyboard Controller for Linux

A lightweight, driverless utility to control the single-zone RGB keyboard backlight on the Acer Aspire 7 A715-79G running Linux.

## Background and Reverse Engineering

Although branded and distributed by Acer, the Aspire 7 A715-79G motherboard and chassis are manufactured by Clevo (NP50RN platform). 

Standard Linux drivers (such as `tuxedo-keyboard`, `clevo-wmi`, or `acer-wmi`) fail on this specific notebook for two main reasons:
1. The system DMI table reports `Manufacturer: Acer` instead of `CLEVO` or `TUXEDO`, triggering hardware check assertions in vendor kernel modules.
2. The OEM firmware strips standard Clevo ACPI/WMI GUID tables, preventing ACPI-based LED drivers from attaching to the hardware bus.

This utility operates directly at the hardware layer by issuing low-level commands to the motherboard Embedded Controller (EC) via the native Linux `ec_sys` kernel debug interface.

## Hardware Architecture and Register Map

The Clevo Embedded Controller monitors memory-mapped registers starting at offset `0xF8`:

| Register Address | Function | Operational Value |
| :--- | :--- | :--- |
| `0xF8` | Hardware Execution Strobe | `0xCA` (commits settings to physical hardware) |
| `0xF9` | Mode / Subcommand | `0x06` for Brightness, `0x03` for RGB Matrix |
| `0xFA` | Channel 1 | Blue Intensity (`0x00` to `0xFF` / 0 to 255) |
| `0xFB` | Channel 2 | Red Intensity (`0x00` to `0xFF` / 0 to 255) |
| `0xFC` | Channel 3 | Green Intensity (`0x00` to `0xFF` / 0 to 255) |
| `0xFD` | Command Terminator | `0x00` |

Notice: The hardware channel wiring on this specific chassis is ordered as Blue, Red, Green (BRG) rather than conventional RGB.

## Prerequisites

* Linux Kernel with `CONFIG_ACPI_EC_DEBUGFS` compiled as a module (`ec_sys`). This is included by default in standard distributions such as Arch Linux, Fedora, Debian, and Ubuntu.
* Root or `sudo` privileges to communicate with the debugfs EC node.

## Quick Installation

Clone this repository and run the automated installer:

```bash
git clone [https://github.com/](https://github.com/)<your-username>/acer-a715-79g-rgb.git
cd acer-a715-79g-rgb
chmod +x install.sh
sudo ./install.sh
