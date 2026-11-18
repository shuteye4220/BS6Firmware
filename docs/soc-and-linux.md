# SoC and Linux feasibility

## The chip

Reported by the device (`getprop ro.board.platform` etc.):

- **MediaTek MT6873 = Dimensity 800**
- Octa‑core: 4× Cortex‑A76 + 4× Cortex‑A55 @ ~2.0 GHz, 7 nm
- GPU: Mali‑G57
- 6 GB RAM, UFS storage
- Android 12 (firmware V2.0+); vendor security patch level ≈ Aug 2023

Note: HIMS marketing brochures list an "8‑core 4×A73 + 4×A53" part, but the
device itself reports MT6873 (A76+A55). Trust the device.

## Mainline / postmarketOS status

- **No ready‑made MT6873 (Dimensity 800) port exists** — there is no
  `linux-postmarketos-mediatek-mt6873` kernel to flash.
- `mt6873` *is* listed among the SoCs recognised by the postmarketOS MediaTek
  bring‑up tooling, so getting a *minimal* kernel to boot is a reachable first
  step, but everything after that is manual work.
- Comparable MediaTek phone SoCs that **do** have community kernel forks
  (mt6768, mt6785, mt6763, mt6878/Dimensity 7300) illustrate the realistic
  ceiling of an early port: e.g. the mt6763 port "just about boots in ~3 minutes
  with one core, logs on simplefb/UART, and USB gadget mode" — not a usable
  daily system.
- MediaTek phone‑SoC vendor drivers can only be used as **reference**; mainline
  drivers must be rewritten. (Chromebook/IoT MediaTek SoCs — MT8xxx/Genio — are
  well supported upstream via Collabora, but that doesn't transfer to MT6873.)

## Feasibility verdict

| Goal | Verdict |
|------|---------|
| Replace Android with native Linux | Large from‑scratch port; locked bootloader (unverified but likely); even booted, **won't drive the braille display** without custom drivers. High effort, real brick risk. |
| Faster than the current OS? | No. See below. |
| More secure than stale Android? | No — see below. |
| Linux *environment* (chroot) on top of Android | The practical path; near‑native with a real chroot, braille still an open question (see braille-architecture.md + next-steps.md). |

### Speed
Running Linux via **proot** (no root) is *slower* than native on syscall‑heavy
work (ptrace tax), near‑native on pure compute. A real **chroot** (needs root)
is essentially native speed. A full native port would be only marginally leaner
than a root chroot (mainly RAM headroom), not meaningfully "faster." The
Dimensity 800 comfortably out‑classes a Raspberry Pi 4 (the BTSpeak's SoC) on
cores, RAM and storage, so a chroot desktop/CLI would feel quicker than a Pi‑4
BTSpeak — *for the speech‑driven parts*. Braille output is the gating
problem (see braille-architecture.md).

### Security
Neither approach fixes the real exposure (stale kernel + frozen vendor
firmware). A chroot updates only *userland*; the Aug‑2023 kernel underneath is
unchanged. A native port would swap the kernel for an unmaintained fork while
the remotely‑exploitable blobs (Wi‑Fi/BT/modem/TEE firmware) stay exactly as
stale. Better lever: network‑isolate the device (VLAN/guest SSID) and treat it
as an appliance; keep high‑value accounts on a maintained device.
