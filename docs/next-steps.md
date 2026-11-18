# Next steps & open questions

Ordered by value‑for‑effort.

## 1. Request the GPL kernel source (highest value, fully legitimate)

The device ships a Linux kernel (GPLv2), so HIMS/SELVAS must provide the
corresponding source to device owners on request (GPLv2 §3). This is the clean
route to the **`brlkey` driver**, the **built‑in display / scan‑chip ("Atlas")
driver**, and the **device tree** — everything needed to understand or rewrite
the hardware drivers for a native port.

Email HIMS (`sales@hims-inc.com`; US office 512‑837‑2000 — ask them to route to
engineering/support), and note SELVAS Healthcare (Korea) as the manufacturer if
HIMS punts. Be specific:

> I own a BrailleSense 6 mini (model H620B), firmware version <yours>. Under
> GPLv2 section 3, I request the complete corresponding source code for the
> Linux kernel and any other GPL/LGPL components in the firmware, including the
> kernel configuration and build scripts. Happy to cover reasonable media costs.

Does **not** cover (not GPL): MediaTek HALs, Wi‑Fi/modem/GPU firmware blobs, or
HIMS's own braille app logic.

## 2. Verify service reachability (decides the no‑root path)

Open question from [service-interfaces.md](service-interfaces.md): are
`IHanBrailleInputService` and `IDotPadLineInterface` callable by a non‑system
app, or signature/permission‑locked? Check each APK's `AndroidManifest.xml` for
`android:exported`, `android:permission`, and `android:sharedUserId="…system"`,
and whether the declaring app is platform‑signed. If reachable → a small Android
app could bridge the braille hardware to a Linux chroot. If signature‑locked
(likely) → that bridge needs platform signing or root.

## 3. Reconstruct the DotPad serial framing

Decompile method bodies of `DataSender.getHexArray`, `DataSend.sendRawLineData`,
and `KeySender` (see [dotpad-accessory.md](dotpad-accessory.md)) to produce a
byte‑level wire protocol for the external DotPad. Standard USB serial + an open
transport library make this the most self‑contained Linux target.

## 4. Find the built‑in 20‑cell display write path

Trace how output reaches the internal cells (vs. the external DotPad): likely
the scan‑chip / "Atlas" controller via a system service or a kernel node not yet
identified. The GPL kernel source (step 1) would answer this directly.

## Paths NOT worth pursuing first

- **Full native Linux reflash** — no MT6873 port exists, likely locked
  bootloader, braille hardware wouldn't work without drivers, high brick risk.
  See [soc-and-linux.md](soc-and-linux.md).
- **Breaking the firmware package encryption** — unnecessary; the apps are
  obtainable decrypted from the device, and the kernel source should come from
  the GPL request.

## Recommended overall approach

1. Send the GPL kernel‑source request now (slow to come back, so start early).
2. Stand up a Linux environment in a **chroot** (not a reflash) for the
   BTSpeak‑like CLI/desktop — speech works immediately.
3. Resolve braille I/O via whichever proves feasible: the Binder‑bridge app
   (if services are reachable), a native driver (once kernel source arrives), or
   an external DotPad over USB.
