<div>

# iPad Restore on Linux — What Went Wrong and How We Fixed It

[Overview](#overview) [Early Mis‑Steps](#failures) [DFU
vs Recovery](#modes) [New Build Workflow](#solution)
[Takeaways](#lessons) [Resources](#resources)

</div>

<div role="main">

<div id="overview" class="section">

## Project Overview

We wanted to wipe and reinstall iPadOS on an older iPad using **only
Linux**. Sounds simple, but we ran into a string of *very* familiar
pain‑points: flaky cables, half‑broken packages, USB‑port weirdness, and
confusion about DFU versus Recovery mode. This page documents the
dead‑ends and, more importantly, the repeatable fix using the latest
open‑source stack.

</div>

<div id="failures" class="section">

## Where We Went Wrong

<div class="warn">

**Cable Chaos:** our first micro‑USB → Lightning cord turned out to be a
charge‑only knock‑off. It would power the iPad but *never* enumerated as
a USB device, so `lsusb` stayed silent and all restore tools failed.

</div>

-   **USB 3.0 Quirks:** DFU negotiates only 480 Mb s‑1. Plugging into a
    blue USB 3.x port caused intermittent disconnects on our kernel;
    swapping to a black USB 2.0 port (or a USB‑C hub that falls back to
    2.0) fixed link flakiness.
-   **Mis‑reading Modes:** We kept booting into *Recovery* (Apple logo)
    instead of full DFU (black screen) so `irecovery` threw `CPID` but
    firmware upload stalled.
-   **Old Packages:** Debian/Ubuntu repos still ship
    `libimobiledevice 1.2.x`. That build predates iPadOS 17 USB
    handshake and fails with
    `ERROR: Unable to connect to device in DFU mode`.
-   **Wrong Flags:** We tried `irecovery --dfu`, but the correct trigger
    flag is `-f`. Passing `--dfu` just printed `unrecognized option`.
-   **Kernel Lockdown:** AV Linux shipped with USB lockdown enabled.
    Udev rules never granted `0666` permissions to the device.

</div>

<div id="modes" class="section">

## DFU vs Recovery – Spot the Difference

**Recovery Mode** shows the iTunes/computer icon and can run a normal
restore *only if* your iPadOS is healthy enough to boot a minimal
ramdisk. **Device Firmware Update (DFU)** is a lower‑level bootstrap
that loads `iBoot` over USB even when main flash is corrupted.

    # Check which mode you're in
    lsusb -d 05ac:12a8   # Recovery (iBoot)
    lsusb -d 05ac:1227   # DFU (pre-iBoot)

If you see an Apple logo, you're *not* in DFU. DFU is a pitch‑black
screen (the back‑light may flick once).

</div>

<div id="solution" class="section">

## The Working Solution – Rebuilding the Stack

1.  **Use a certified data cable *and* a USB 2.0 port.** We ended up
    with an Anker PowerLine plugged into a classic black USB 2.0 port.
    Avoid blue USB 3.x ports unless you confirm stable DFU enumeration.

2.  **Install build deps.**  

        sudo apt install --no-install-recommends \
          build-essential libusbmuxd-dev libssl-dev libplist-dev \
          libusb-1.0-0-dev git pkg-config

3.  **Clone and build the bleeding‑edge libs.**

        git clone https://github.com/libimobiledevice/libimobiledevice.git
        git clone https://github.com/pmateti/libirecovery.git

        cd libirecovery && ./autogen.sh --enable-static && make -j4 && sudo make install
        cd ../libimobiledevice && ./autogen.sh --without-cython && make -j4 && sudo make install
        sudo ldconfig

4.  **Create udev rule.**

        echo 'SUBSYSTEM=="usb", ATTR{idVendor}=="05ac", MODE="0666"' | \
          sudo tee /etc/udev/rules.d/39-ios.rules
        sudo udevadm control --reload-rules && sudo udevadm trigger

5.  **Enter DFU & restore.**

        # Hold HOME + POWER for 4 s, release POWER, keep HOME for 10 s
        irecovery -q   # should print CPID / DFU

        idevicerestore -l            # list available IPSWs
        idevicerestore -e -w *.ipsw   # erase & reinstall

6.  **Post‑restore pairing.** Unplug/re‑plug, then:

        idevicepair pair
        ideviceinfo | head

    You should see info without errors.

After installing from source, pin the repo versions so they don’t get
overwritten by apt upgrades: `apt-mark hold libimobiledevice*`.

</div>

<div id="lessons" class="section">

## What We Learned

-   60 % of failures were hardware (cheap cables); always verify with
    `lsusb` first.
-   USB 2.0 ports are more reliable for DFU than many USB 3.x
    controllers.
-   Apple changes USB handshakes quietly—build the newest `lib*` stack.
-   Put iPad into DFU *before* running any restore command.
-   Write a quick udev rule instead of debugging `permission denied` at
    2 a.m.

</div>

<div id="resources" class="section">

## Helpful Resources

-   <a href="https://libimobiledevice.org/" target="_blank"
    rel="noopener">libimobiledevice Project</a>
-   <a href="https://github.com/libimobiledevice/idevicerestore"
    target="_blank" rel="noopener">idevicerestore</a>
-   <a href="https://theiphonewiki.com/wiki/DFU_Mode" target="_blank"
    rel="noopener">iPhone Wiki – DFU Mode</a>

</div>

</div>

Last updated: May 26 2025 • Page by Brian & ChatGPT
