---
layout: page
permalink: "nvidia"
title: "Power management with Nvidia proprietary drivers (on Turing cards)"
date: 2022-07-25
showSummary: true
categories: ['config']
tags: ['nvidia', 'power', 'linux']
---
## Short story
I recently got a laptop with NVIDIA Turing GPU; after my trusty Lenovo Legion Y520's body no longer wanted to be a single piece.
Although it has served me quite well over the past 5 years, the build quality (and my abuse :p) has started to show up. Battery capacity depleted, pieces started breaking off, speaker cones blown off, screws missing etc.

nouveau drivers did excellent power management of my Turing card. Power management as in, kept in lowest possible power state during idle. Which is around 9watts for my (whole) system.

## But!
But when I installed Nvidia proprietary drivers for a little bit of casual gaming, my turing card became capitalist and claimed whole battery. ~ around 18w minimum on idle!

Thinking that the nvidia-open kernel modules might have better compatibility like the nouveau drivers for power management, I gave it a try too. But that was a bad choice :D

25w on most of the time. I've even seen 64w getting pulled from the battery doing nothing. Finally came back to the prop drivers and like every other normal person I started going through all the Nvidia links on arch wiki and stumbled upon this: https://wiki.archlinux.org/title/PRIME. There I found this section on PCI-Express Runtime D3 (RTD3) Power Management, which linked an NVIDIA documentation for dynamic power management. https://us.download.nvidia.com/XFree86/Linux-x86_64/465.27/README/dynamicpowermanagement.html.


## Solution

I just followed the wiki,

- Add the necessary udev rules (This would enable runtime power management for the VGA Controller / 3D Controller PCI function)
- Set nvidia kernel module param to NVreg_DynamicPowerManagement=0x02
- Enabled nvidia persistence service.

And viola 🎉

Got my 8-9w idle usage back and everything working as usual.

```shell
abbyck@prime ~ batstat
  native-path:          BAT0
  vendor:               Razer
  model:                Blade
  serial:               CNB1RC30-03280AU00240-A00
  power supply:         yes
  updated:              Mon 25 Jul 2022 12:54:06 AM IST (16 seconds ago)
  has history:          yes
  has statistics:       yes
  battery
    present:             yes
    rechargeable:        yes
    state:               discharging
    warning-level:       none
    energy:              49.819 Wh
    energy-empty:        0 Wh
    energy-full:         64.5568 Wh
    energy-full-design:  65.0034 Wh
    energy-rate:         8.1312 W
    voltage:             16.207 V
    charge-cycles:       N/A
    time to empty:       6.1 hours
    percentage:          77%
    capacity:            95.783%
    icon-name:          'battery-full-symbolic'
  History (rate):
    1658690646	8.131	discharging
    1658690616	7.839	discharging
    1658690586	8.901	discharging
    1658690556	8.455	discharging
```
(`alias batstat
batstat='upower -i /org/freedesktop/UPower/devices/battery_BAT0'`)

While you are at it, enable hardware video acceleration for your CPU. Now 12w power draw for 4K YouTube video playback on Firefox (you might have to turn some flags on).

Also, YMMV!

- നന്ദി.
