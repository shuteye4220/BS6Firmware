# Firmware package

## Where it lives

HIMS serves BrailleSense firmware from public S3 buckets. The file analysed here:

```
https://braillesense.s3.us-west-1.amazonaws.com/h620b/en12/H620B_apps_80249.bin
```

- Model code **H620B** = BrailleSense 6 mini; `en12` = English / Android 12 branch.
- `apps_80249` = the **application** partition, build 80249.
- Size: 1,877,851,837 bytes
- SHA‑256: `3c1a9687b4fb3f9a2a2c92566d52f8ca6490417c28adfc1523f6b7cbc8b90d83`

A separate **system** image is served from the other bucket, e.g. (32‑cell H632B
shown; the H620B equivalent follows the same pattern):

```
http://braillesense6.s3.ap-northeast-2.amazonaws.com/release/en/H632B_system_46966.bin
```

HIMS's own note warns that Windows may rename the `.bin` to `.zip` and that it
must not be extracted — consistent with the container format below.

## Container format

`H620B_apps_80249.bin` is a **ZIP** archive (351 entries). The central directory
(file names) is readable, but **every entry's data is encrypted** — extraction
requires a key HIMS does not publish. The contents were therefore **not** read
from this file, and no attempt was made to break that encryption.

Visible entry names (names only) indicate the package holds:

- `apps/` — HIMS application APKs (ScreenReader, Polaris2Service, BrailleDotPad,
  HanNotePad, etc.)
- `Tables/duxbury/*.btb`, `Tables/liblouis/*` — braille translation tables
- `Tables/google/*.apex`, `*.apk` — Google mainline/APEX modules
- `HanDictionary/` — dictionary/thesaurus data
- `Package.ini`, `Release_Note.*`, `3.version`, `en.lang`

## What is NOT in the `apps` package

No `boot.img`, no kernel, no **device tree (DTB)**, no `vendor` partition, no
bootloader. Those live in the **system/boot** images, not here. Consequently the
SoC‑wiring and kernel‑driver questions cannot be answered from this file — see
[next-steps.md](next-steps.md) for the GPL kernel‑source route, which is the
clean way to get the driver and device‑tree reference.

## How the app analysis was actually done

Because the package is encrypted, the braille apps were instead **extracted from
a running device** (they are world‑readable, decrypted, installed system APKs)
and read statically. See [braille-architecture.md](braille-architecture.md).
