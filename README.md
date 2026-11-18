# BrailleSense 6 mini (H620B) — Linux / interoperability notes

Research notes on the **HIMS BrailleSense 6 mini** (model **H620B**, Android 12,
MediaTek MT6873 / Dimensity 800), gathered while investigating whether it could
run a Linux environment and whether its braille display and Perkins keyboard
could be driven from Linux (the goal: a BTSpeak‑like device with a braille
display).

These are **interoperability notes about hardware I own**. Everything here was
derived from:

- the **public** HIMS firmware update files (see [docs/firmware-package.md](docs/firmware-package.md)), and
- **app packages (APKs) extracted from my own device** — i.e. already‑decrypted,
  already‑installed software — read statically (no code was run, no DRM or
  encryption was circumvented).

No proprietary binaries, APKs, or firmware images are committed to this repo —
only descriptions of interfaces and architecture, at the level an open‑source
accessibility project (e.g. BRLTTY) would document a device.

## Contents

| Doc | What's in it |
|-----|--------------|
| [docs/firmware-package.md](docs/firmware-package.md) | Where the firmware lives, the `apps` package layout, encryption, what's *not* in it |
| [docs/soc-and-linux.md](docs/soc-and-linux.md) | MT6873 identification, mainline/postmarketOS status, feasibility of a native port |
| [docs/braille-architecture.md](docs/braille-architecture.md) | How text becomes dots and reaches the hardware: the three‑layer stack |
| [docs/service-interfaces.md](docs/service-interfaces.md) | The Binder/AIDL interfaces (keyboard + display) and their methods |
| [docs/dotpad-accessory.md](docs/dotpad-accessory.md) | The external Dot Inc. "DotPad" integration (USB‑serial) |
| [docs/next-steps.md](docs/next-steps.md) | Open questions and the most promising paths forward |

## TL;DR

- **Native Linux on the device** is a large from‑scratch port (no ready‑made
  MT6873 port exists) and, even if booted, would not drive the braille hardware
  without its own drivers. Not recommended as a first move.
- The braille hardware is reached through **Android Binder services**, not
  directly by the apps. That opens a **no‑reflash path**: a Linux environment in
  a chroot, bridged to the braille display/keyboard through those services —
  *if* the services are reachable by a non‑system app (open question; see
  [docs/next-steps.md](docs/next-steps.md)).
- The **Perkins keyboard** is backed by a **kernel driver** (`/dev/brlkey_*`) and
  a **scan‑chip MCU** (firmware‑updatable via a system service).
- There is an **external accessory** path (Dot Inc. DotPad over USB serial) that
  is fully tractable from Linux and uses an open USB‑serial library.
- The single most valuable thing to obtain is the device's **GPL kernel source**
  (see [docs/next-steps.md](docs/next-steps.md)).
