---
layout: post
title: "Arch, Wayland and NVIDIA. The one kernel flag let me live."
date: 2025-11-03 12:00:00 -0000
categories: blogging
tags: [arch, wayland, hyprland, nvidia, troubleshooting]
---

## TL;DR

If your NVIDIA card keeps crashing on Arch + Wayland, add `nvidia.NVreg_EnableGpuFirmware=0` to your kernel parameters. Also: VRR on, 10-bit color off. Details below.

---

## The Situation

Running Linux on NVIDIA in 2025 is... an experience. 

My first Linux install was Gentoo—yes, really. I liked to suffer back then. The documentation was incredible until the day we lost it all. No backups. Fun times.

These days, things mostly work out of the box. Even NVIDIA! Sort of.

**The problem:** Your system will boot. You'll see graphics. You might even get 30 minutes of uptime before everything goes sideways.


## The system

CPU: AMD 7800X3D

RAM: Doesn't matter

Motherboard: Something that fits the above.

GPU: **NVIDIA** 4070 Super (Asus dual something)

OS: Arch, btw (I'm sure it's not just arch though)

+Wayland +Hyprland (but KDE does the same thing).

NVIDIA Driver: the propietary, or open, same results tbh. Just not nouveau.


## The Problem

Random crashes. Three flavors:

1. Cursor slowdown → freeze. SSH doesn't help. System hangs indefinitely.
2. Graphical glitch → instant reboot.
3. No warning. Just reboot.

Happens while browsing, watching videos, gaming. Completely unpredictable.

## What Didn't Work

I tried everything. None of this helped:

- **Driver updates** — Problem exists from 560 onwards
- **Cable swaps** — HDMI vs DisplayPort, makes no difference
- **Browser changes** — Firefox, Chromium, all crash equally
- **Compositor tweaks** — Transparency, blur, animations, VSync
- **Hardware settings** — CPU overclocking, RAM timings
- **Kernel versions** — LTS, mainline, zen, all crash

If you've tried all this too, keep reading.

## Diagnostic Steps

Before we fix it, let's check your current settings.

### Check Your Display Settings

Two things matter:
- **VRR (Variable Refresh Rate)** — Is it on?
- **10-bit color output** — Is it enabled?

On Hyprland? Install this:

<div class="terminal-prompt">
  <span class="prompt-user">n@arch</span>:<span class="prompt-path">~</span>$ <span class="prompt-command">pacman -S nwg-displays</span>
</div>

<div class="terminal-prompt">
  <span class="prompt-user">n@arch</span>:<span class="prompt-path">~</span>$ <span class="prompt-command">nwg-displays</span>
</div>

This shows all your display settings in a GUI.

**Note:** If adaptive sync isn't selectable in the GUI, edit your config manually:

<div class="terminal-prompt">
  <span class="prompt-user">n@arch</span>:<span class="prompt-path">~</span>$ <span class="prompt-command">nvim ~/.config/hyprland/monitors.conf</span>
</div>

Add this at the end:

```conf
misc {
    vrr = 1
}
```

### Check NVIDIA Driver Parameters

Run this to see your current driver configuration:

<div class="terminal-prompt">
  <span class="prompt-user">n@arch</span>:<span class="prompt-path">~</span>$ <span class="prompt-command">cat /proc/driver/nvidia/params | sort</span>
</div>

<div class="command-output">
<pre><code>CoherentGPUMemoryMode: ""
CreateImexChannel0: 0
DeviceFileGID: 0
DeviceFileMode: 438
DeviceFileUID: 0
DmaRemapPeerMmio: 1
DynamicPowerManagement: 3
DynamicPowerManagementVideoMemoryThreshold: 200
EnableDbgBreakpoint: 0
EnableGpuFirmware: 0
EnableGpuFirmwareLogs: 2
EnableMSI: 1
EnablePCIeGen3: 0
EnablePCIERelaxedOrderingMode: 0
EnableResizableBar: 0
EnableS0ixPowerManagement: 0
EnableStreamMemOPs: 0
EnableUserNUMAManagement: 1
ExcludedGpus: ""
GpuBlacklist: ""
GrdmaPciTopoCheckOverride: 0
IgnoreMMIOCheck: 0
ImexChannelCount: 2048
InitializeSystemMemoryAllocations: 1
KMallocHeapMaxSize: 0
MemoryPoolSize: 0
ModifyDeviceFiles: 1
NvLinkDisable: 0
OpenRmEnableUnsupportedGpus: 1
PreserveVideoMemoryAllocations: 1
RegisterPCIDriver: 1
RegistryDwords: ""
RegistryDwordsPerDevice: ""
ResmanDebugLevel: 4294967295
RmLogonRC: 1
RmMsg: ""
RmNvlinkBandwidthLinkCount: 0
RmProfilingAdminOnly: 1
S0ixPowerManagementVideoMemoryThreshold: 256
TemporaryFilePath: "/var/tmp"
UsePageAttributeTable: 4294967295
VMallocHeapMaxSize: 0</code></pre>
</div>


## The Fix

Three settings. All required.

### 1. VRR: ON

Yes, **ON**. I know it sounds backwards. Enable it anyway.

### 2. 10-bit Color: OFF

Disable it. I wish I could keep it on—some grays look better with 10-bit. But:
- Screenshots break in Hyprland with 10-bit enabled
- System still crashes with 10-bit on
- Turn it off

### 3. The Magic Kernel Flag

Add this to your kernel parameters:

```
nvidia.NVreg_EnableGpuFirmware=0
```

**This is the key.** This single flag fixed everything.

I won't explain what it does—just know that all three settings combined create stability. I've installed Arch twice on this hardware (once with KDE, once with Hyprland). Both times, this was the solution.

Don't know if it's an ASUS thing or an NVIDIA thing. Don't care. It works.

---

## How to Apply the Kernel Flag

**For Limine users:**

Edit `/etc/default/limine` and add the flag to the end of `KERNEL_CMDLINE`:

<div class="terminal-prompt">
  <span class="prompt-user">n@arch</span>:<span class="prompt-path">~</span>$ <span class="prompt-command">sudo nvim /etc/default/limine</span>
</div>

Then run ```limine-update```

**For GRUB users:**

Edit `/etc/default/grub` and add it to `GRUB_CMDLINE_LINUX_DEFAULT`, then regenerate:

<div class="terminal-prompt">
  <span class="prompt-user">n@arch</span>:<span class="prompt-path">~</span>$ <span class="prompt-command">sudo grub-mkconfig -o /boot/grub/grub.cfg</span>
</div>

**For systemd-boot users:**

Edit your boot entry in `/boot/loader/entries/` and add it to the options line.

---

## Does It Have to Be Exact?

In my experience: **yes**.

My system becomes unstable if I change VRR or 10-bit settings. But your hardware might differ. Test it yourself.

The kernel flag, though? That's non-negotiable.