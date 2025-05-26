# iPad Restore on Linux — What Went Wrong and How We Fixed It

[Overview](#overview) · [Where We Went Wrong](#where-we-went-wrong) · [DFU vs Recovery Mode](#dfu-vs-recovery-mode) · [New Build Workflow](#new-build-workflow) · [Key Takeaways](#key-takeaways) · [Resources](#resources)

## Overview

We wanted to wipe and reinstall iPadOS on an older iPad using **only Linux**.  
Sounds simple, but we ran into a string of *very* familiar pain-points: flaky cables, half-broken packages, USB-port weirdness, and confusion about DFU versus Recovery mode. This document captures those dead-ends and, more importantly, the repeatable fix using the latest open-source stack.

## Where We Went Wrong

> **Cable Chaos:** Our first micro-USB → Lightning cord was a charge-only knock-off. It would power the iPad but *never* enumerate as a USB device, so `lsusb` stayed silent and all restore tools failed.

- **USB 3.0 Quirks:** DFU negotiates only 480 Mb/s. Plugging into a blue USB 3.x port caused intermittent disconnects on our kernel.

## DFU vs Recovery Mode

DFU (Device Firmware Update) is the only mode that lets you completely reflash the firmware, whereas Recovery Mode simply reinstalls the OS without touching the bootloader. If you’re seeing the little cable-to-computer icon on the iPad screen, that’s Recovery Mode—not DFU—so tools like `libimobiledevice` won’t work until you force it into DFU (hold Power+Home for exactly 10 s, then release Power only).

## New Build Workflow

1. Switched our MkDocs config to use the **root** docs directory instead of `docs/`.  
2. Installed the **awesome-pages** plugin locally:
   
   ```bash
   pip install mkdocs-material mkdocs-awesome-pages-plugin
   ```
