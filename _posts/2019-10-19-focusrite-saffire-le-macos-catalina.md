---
layout: post
title: Focusrite Saffire LE on macOS Catalina
---

macOS 10.15 (Catalina) dropped support for legacy 32-bit apps including _SaffireControl LE_. Luckily, Firewire audio interfaces are still supported, so only control app is missing.

A few months ago I have reverse-engineer control app and since then I have implemented rough CLI-based implementation that is able to read and update mixer settings, as well as monitor inputs and outputs. While it's far from being convenient it allows for basic stuff. The implementation is available on [GitHub repo](https://github.com/kfigiela/saffire-mixer).

I plan to develop user interface with more or less the same features as original app and will be posting updates on this page.

<video width="626" autoplay loop>
    <source src="attachments/saffire-le-monitor.mp4" type="video/mp4">
</video>

**Update #1 (2019-12-09)** I released [application](https://kfigiela.github.io/2019/12/09/saffire-le-mixer-app-for-catalina/) with GUI.
