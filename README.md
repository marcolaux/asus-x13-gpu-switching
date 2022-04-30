# ASUS X13 dGPU suspend draft

This is a draft on what works for the ASUS X13 to turn off the dGPU completely (even though hybrid graphics with an auto suspending dGPU works, this is an experiement).

I tested this with a 2021 G14 and it seems to work reliably.

## Thoughts

`supergfxctl` does not turn off the dGPU even though it is not in the PCI device tree after setting the mode to `integrated`

This `dgpu_disable` branch introduces a switching method based on the `/sys/devices/platform/asus-nb-wmi/dgpu_disable` sysfile.

This stays on reboots (restoring the last state of dGPU on / off).

It has to get called twice with a PCI rescan inbetween - why that is I'm not sure. But if only called once the dGPU does not autosuspend after enablement.

## How to use

- remove `supergfxctl / gpu-manager / optimus-manager` or whatever you might use for switching GPU modes
- check for remaining files in `/etc/modprobe.d/` with `nvidia` references, back them up and **delete them**
- check for remaining files in `/etc/modules-load.d/` with `nvidia` references, back them up and **delete them**
- `git clone https://github.com/hyphone/asus-x13-gpu-switching.git`
- `cd asus-x13-gpu-switching`
- `git checkout dgpu_disable`
- `sudo ./install` to install the scripts and services

### How to switch modes
- `systemctl start asusgpuswitch` to switch between hybrid and integrated (**warning**: this will forcly log you out)
- `cat /sys/devices/platform/asus-nb-wmi/dgpu_disable` tells you the current mode you are in (0 = hybrid + autosuspend, 1 = integrated + off)