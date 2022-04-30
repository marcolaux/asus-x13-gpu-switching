# ASUS X13 dGPU suspend draft

This is a draft on what works for the ASUS X13 to turn off the dGPU completely (even though hybrid graphics with an auto suspending dGPU works, this is an experiement).

I tested this with a 2021 G14 and it seems to work reliably.

## Status X13

While I thought that this might be a "proper" way to enable / disable the dGPU it never fully suspends on the X13 when the dGPU is active.
Dynamic power management seems to work kind of but it never goes below 11W.

When we have a look at the method of the main branch here (https://github.com/hyphone/asus-x13-gpu-switching), where we just remove the dPGU from the PCI bus it fully shuts down and either dGPU enabled or fully disabled does not make a difference in power consumption. On the X14 we can go down to 4.5W integrated and hybrid (main branch).

Here we achieve 4.5W on integrated but only 11W on hybrid (https://github.com/hyphone/asus-x13-gpu-switching/tree/dgpu_disable).

In comparison with supergfxctl we get like 5W on integrated and 6.5W on hybrid.

So by just removing and adding the dGPU from / to the PCI bus we seem to get the best results (https://github.com/hyphone/asus-x13-gpu-switching).

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