# acpi-gpu-off

Power off legacy AMD discrete GPUs on certain hybrid-graphics laptops using ACPI firmware methods and `acpi_call`.

## Overview

Some older Intel + AMD hybrid laptops remain significantly hotter under Linux even when PCI runtime power management reports the discrete GPU as suspended.

On my Dell laptop, runtime PM alone left the system idling around **62°C**, while executing the firmware ACPI power-off methods reduced idle temperature to approximately **50°C**.

This repository provides a simple script plus either a systemd service or a runit service so the GPU can be powered off automatically at boot.

## Tested Hardware

* 2013 Dell Inspiron 14z 5423 laptop (Intel Ivy Bridge + AMD Radeon Enduro)
* Oracle Linux 9 UEK 6.12
* antiX Linux (runit)
* `acpi_call` kernel module

The ACPI methods used are firmware-specific and may not exist on other machines.

## Requirements

* `acpi_call` kernel module
* `systemd` or `runit`

## Installation

Copy the script:

```bash
sudo install -m755 gpu-off /usr/local/sbin/gpu-off
```

### systemd

Install the service:

```bash
sudo cp gpu-off.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable gpu-off.service
```

### runit (antiX)

Install the runit service directory:

```bash
sudo mkdir -p /etc/sv/gpu-off
sudo cp runit/gpu-off/run /etc/sv/gpu-off/run
sudo chmod +x /etc/sv/gpu-off/run
sudo ln -s /etc/sv/gpu-off /var/service/gpu-off
```

(Optional) Prevent the Radeon driver from binding to the discrete GPU:

```text
blacklist amdgpu
blacklist radeon
```

Place the above in `/etc/modprobe.d/blacklist-amdgpu.conf`.

Reboot.

## Verification

Confirm the service executed:

systemd:

```bash
journalctl -b -u gpu-off.service
```

runit:

```bash
sv status gpu-off
```

Check the ACPI return value:

```bash
cat /proc/acpi/call
```

Expected result:

```text
0x0
```

Verify temperatures with:

```bash
sensors
```

## Files

* `gpu-off` - ACPI power-off script
* `gpu-off.service` - systemd service
* `runit/gpu-off/run` - runit service for antiX

## Notes

This is not a generic Linux GPU power management solution.

It relies on vendor-specific ACPI methods exposed by the laptop firmware. Many systems use different method names, require additional arguments, or do not support ACPI power-off at all.

This will only work if your firmware exposes working ACPI methods (_OFF, SGOF, or ATPX).
If your machine does not expose these methods, this project will likely not work without modification.

## License

MIT License.
