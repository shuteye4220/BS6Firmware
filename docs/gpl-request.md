# GPL kernel-source request

The BrailleSense 6 mini ships a Linux kernel, which is licensed under **GPLv2**.
Under **GPLv2 section 3**, anyone who distributes the binary (HIMS / SELVAS) must
make the *complete corresponding source* available to recipients — and a
written offer to supply it must stand for **at least three years**. As the owner
of the device, you are a recipient entitled to ask.

## Who to contact

- **HIMS Inc.** (US distributor): `sales@hims-inc.com`, +1 512-837-2000 (Austin,
  TX). Ask them to route the request to engineering / software support.
- **SELVAS Healthcare, Inc.** (Korea) — the actual manufacturer; fall back to
  them if HIMS doesn't follow through.

Before sending, fill in your firmware version: on the device it's under the
About / version info (the build analysed here was the `apps` partition
**80249**, Android 12 / firmware V2.x).

---

## Email (copy from here)

**Subject:** GPLv2 source request — BrailleSense 6 mini (H620B)

Hello,

I own a HIMS BrailleSense 6 mini (model **H620B**), running firmware version
**<FILL IN YOUR VERSION>** (Android 12). The device includes the Linux kernel
and other software licensed under the GNU General Public License (GPL) and
Lesser GPL.

Under **section 3 of the GPL version 2**, I am requesting the **complete
corresponding source code** for all GPL- and LGPL-licensed components
distributed in this device's firmware. Specifically, I am asking for:

1. The full **Linux kernel source** as shipped on this device, including all
   HIMS/SELVAS and MediaTek patches applied to it;
2. The **kernel configuration** file (the `.config` / `defconfig`) used to build
   the kernel for this model;
3. Any **build scripts, Makefiles, and documented instructions** necessary to
   compile and install the kernel ("the scripts used to control compilation and
   installation of the executable", as GPLv2 section 3 requires);
4. The source for any **other GPL/LGPL components** in the firmware (for example
   the bootloader, if GPL-licensed, and GPL/LGPL userspace libraries).

I understand this covers only GPL/LGPL-licensed software, not proprietary
components. I am happy to receive the source by download link, or on physical
media if you prefer — in which case please let me know any reasonable cost for
the media and shipping, as permitted under GPLv2 section 3(b).

Please also let me know if a different address or department is the correct
place to direct GPL source requests.

Thank you,
<YOUR NAME>
<YOUR CONTACT / ORDER OR SERIAL IF THEY ASK>

---

## What to expect

- Large vendors (e.g. Samsung) post this openly; a smaller assistive-tech vendor
  may never have handled a GPL request, so expect to **follow up** and be
  patient and specific.
- What you should get: kernel source + config + build scripts. This is the
  reference for the **`brlkey` driver**, the **built-in display / scan-chip
  ("Atlas") driver**, and the **device tree** — see
  [next-steps.md](next-steps.md).
- What it will **not** include (not GPL): MediaTek HALs, the Wi-Fi/BT/modem/GPU
  firmware blobs, and HIMS's own braille applications.
- If a request is flatly refused or ignored over a long period, the Software
  Freedom Conservancy and the FSF handle GPL-compliance matters — a last resort,
  not a first step.
