# acpi-gpu-off

Power off legacy AMD discrete GPUs on certain hybrid-graphics laptops using ACPI firmware methods and `acpi_call`.

## Overview

Some older Intel + AMD hybrid laptops remain significantly hotter under Linux even when PCI runtime power management reports the discrete GPU as suspended.

On my Dell laptop, runtime PM alone left the system idling around **62°C**, while executing firmware ACPI power-off methods reduced idle temperature to approximately **50°C**.

This repository provides a simple script to power off the GPU via ACPI firmware methods.
A systemd service is included as a convenience for automatic execution at boot.

## Tested Hardware

* 2013 Dell Inspiron 14z 5423 laptop (Intel Ivy Bridge + AMD Radeon Enduro)
* Oracle Linux 9 UEK 6.12
* antiX Linux (manual systemd use or compatibility testing)
* `acpi_call` kernel module

The ACPI methods used are firmware-specific and may not exist on other systems.

## Requirements

- `acpi_call` kernel module
- Root privileges to write to `/proc/acpi/call`

## Optional

- `systemd` (for automatic execution at boot)

The script works independently of any init system. It can be executed manually or integrated into any service manager (systemd, OpenRC, etc.).

## Installation

Copy the script:

```bash
sudo install -m755 gpu-off /usr/local/sbin/gpu-off
```

### systemd setup

Install the service:

```bash
sudo cp gpu-off.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable gpu-off.service
sudo systemctl start gpu-off.service
```

## Optional: prevent driver binding

To avoid the Radeon driver attaching to the discrete GPU:

```text
blacklist amdgpu
blacklist radeon
```

Place in:

```bash
/etc/modprobe.d/blacklist-amdgpu.conf
```

Then reboot.

## Verification

Check service execution:

```bash
journalctl -b -u gpu-off.service
```

Check ACPI call result:

```bash
cat /proc/acpi/call
```

Expected:

```text
0x0
```

Verify GPU state:

```bash
lspci -nnk | grep -EA3 'VGA|Display|3D'
```

Check temperatures:

```bash
sensors
```

## Files

* `gpu-off` — ACPI power-off script
* `gpu-off.service` — systemd service

## Notes

This is not a generic Linux GPU power management solution.

It relies on vendor-specific ACPI firmware methods exposed by the laptop BIOS/UEFI. Method names vary across devices, and many systems will not support ACPI GPU power-off at all.

This will only work if the firmware exposes valid ACPI methods such as `_OFF`, `SGOF`, or `ATPX`.

---

## License

MIT License
