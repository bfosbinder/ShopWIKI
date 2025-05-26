# iPad Restore on Linux — What Went Wrong and How We Fixed It

## Quick links
- [Overview](#overview)
- [Where we went wrong](#where-we-went-wrong)
- [DFU vs Recovery](#dfu-vs-recovery)
- [The fix — repeatable workflow](#the-fix—repeatable-workflow)
- [Takeaways](#takeaways)
- [Resources](#resources)

---

## Overview
We set out to wipe and reinstall **iPadOS** on an older iPad using **only
Linux**. It *should* have taken ten minutes. Instead, we hit every common
road‑block: bad cables, USB‑port quirks, half‑broken restore tools and confusion
over DFU vs Recovery mode.  
This page records the false starts and, more importantly, the reliable
procedure that finally worked.

---

## Where We Went Wrong

!!! danger "Cable chaos"
    Our first micro‑USB → Lightning lead was a *charge‑only knock‑off*.
    The iPad powered on, but never enumerated as USB;  
    `lsusb` stayed silent and every restore tool failed.

* **USB 3.x quirks** – DFU negotiates at 480 Mb s‑¹ (USB 2.0 speeds). Plugging
  into a blue USB 3.x port caused intermittent disconnects on 6.1+ kernels.
* **Mismatched restore tools** – `idevicerestore` from Debian 12 was too old
  for iPadOS 17. Needed a 2.0.2‑plus build from source.
* **DFU vs Recovery confusion** – We kept cycling the device into Recovery
  (connect‑to‑iTunes screen) when we actually needed DFU (blank screen).

---

## DFU vs Recovery

| Mode      | Screen looks like | When to use                       | How to enter (10th‑gen) |
|-----------|------------------|-----------------------------------|-------------------------|
| **DFU**   | **blank/black**  | Low‑level firmware & OS restore   | Hold *Volume‑down + Power* 4 s → release Power, keep Volume‑down 10 s |
| Recovery  | Cable‑to‑laptop  | Update / restore with iTunes/Finder | Hold *Volume‑down + Power* until logo appears |

---

## The fix — repeatable workflow

1. **Use a known‑good cable.**  
   Apple‑certified (MFi) or Anker/Nimaso data‑capable Lightning.
2. **Plug into a USB 2.0 port** (black) or a powered hub that forces 2.0 mode.
3. **Install fresh libimobiledevice stack:**
   ```bash
   sudo apt remove libimobiledevice-dev idevicerestore
   git clone https://github.com/libimobiledevice/libimobiledevice.git
   cd libimobiledevice && ./autogen.sh --enable-debug && make -j && sudo make install
   cd .. && git clone https://github.com/libimobiledevice/idevicerestore.git
   cd idevicerestore && ./autogen.sh --enable-debug && make -j && sudo make install
