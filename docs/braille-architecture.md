# Braille stack architecture

Derived from static analysis of these APKs (extracted from the device):
`BrailleSense_System_UI`, `BrailleSense_Settings`, `BrailleSense_Mobile_Screen_Reader`,
`BrailleSense_Service`, `BrailleSense_Accessibility`, `BrailleSense_Braille_Keyboard`,
`dotPad`, `lcp`.

The stack has three layers. Only the bottom one is a "driver."

## Layer 1 — Translation (in‑app native libraries)

All braille native libs are **32‑bit `armeabi-v7a` only**. They convert text to
braille dot patterns; none of them do device I/O (no `/dev`, no ioctl, no
serial/SPI references anywhere in them).

| Library | Role |
|---------|------|
| `libBes2BrlIf.so` | Text↔braille translation engine from **Extra Co.** (`jp.co.extra`). API: `convertFromBesToBrl` / `convertFromBrlToBes` / `convertFromBetToBes`, `getNumOfColumns/Lines/Pages`, `get/setPageLineImage`. |
| `libbraille.so`, `libbrltrn.so`, `libWHBrlLib.so` | Braille translation/formatting logic and tables. |
| `libGermanyTranslator.so`, `libJapanTranslator-lib.so` | Language‑specific translators. |
| (TTS libs: `libNuanceVocalizer`, `libpttsE-selvas_healthcare`, `libdtalkertts*`, `libvt_*`) | Speech synthesis — not braille. |

So: apps turn text into dot patterns in software, then hand them off.

## Layer 2 — IPC services (Binder / AIDL) — the real API boundary

The apps do not touch hardware directly; they call system services over Binder.
Full method lists in [service-interfaces.md](service-interfaces.md).

- **`com.jawon.han.input.IHanBrailleInputService`** (+ `…Callback.onResponse`) —
  the **keyboard / system** service. Notable surface: `injectKey`,
  `set6DotMode`/`is6DotMode`, `setAvailableSystemKey`, `playMacro`, `powerOff`,
  `setAtlasFrequency`, `setProductionMode`, `updateScanChipFirmware(byte[])`,
  `printEmulatorLcd(Bitmap)`, home/system‑key locks. Interpretation: the Perkins
  keyboard is scanned by a dedicated **"scan chip" MCU** with updatable
  firmware; **"Atlas"** appears to be an internal controller codename. Key
  events return via the `onResponse(int,int,String)` callback.
- **`com.selvashc.dotpad.IDotPadLineInterface`** — the **display‑write** API:
  `onLineData(String)`, `onLineDataWithCursor(String,int,int)`,
  `onRawLineData(byte[])`, `onMultiLine…`, `onDotPatternImage(Bitmap)`,
  `onIcon(Bitmap)`, `onAllDown()`, `isConnectDevice()`. **Caveat:** this is the
  **Dot Inc. "DotPad"** path — see below and [dotpad-accessory.md](dotpad-accessory.md).
- **`com.jawon.han.haninputmethod.data.IBrailleChar`** — braille‑cell data model
  (`getBrailleText`, `getLCDText`, `getTTSText`, `getReadBrlText`, …).
- **`com.selvashc.han.hanpolarisservice.HanPolarisService`** — a general
  lifecycle service (audio ducking, storage/volume events, `formatDisk`,
  `deleteFolder`). **Not** the braille I/O path.

## Layer 3 — Hardware

Confirmed kernel device nodes (referenced from the Java layer):

- `/dev/brlkey_lock`
- `/dev/brlkey_audiomode`

These are custom character devices from a HIMS **kernel driver** for the braille
keyboard (`brlkey`). Their presence is proof a kernel‑level driver exists.

## Built‑in display vs. external DotPad — important distinction

The `com.selvashc.dotpad` / `com.dotincorp` / `com.selvashc.dotPad` code is the
integration for the **external Dot Inc. DotPad** tactile display (an accessory),
connected over **USB serial** — not necessarily the BS6 mini's **built‑in
20‑cell display**. The built‑in display's exact write path is **not yet pinned
down**; the most likely route is the same low‑level system/scan‑chip ("Atlas")
controller that handles the keyboard, below the `IDotPadLineInterface` layer.
This is an open item — see [next-steps.md](next-steps.md).

## One‑screen summary

```
text
  │  Layer 1: libBes2BrlIf.so (+ friends)  — text → braille dots   [32-bit, in-app]
  ▼
dot patterns
  │  Layer 2: Binder/AIDL
  │    • IHanBrailleInputService      — keys, system, scan-chip/"Atlas", firmware
  │    • IDotPadLineInterface         — line/raw/graphic output  (external DotPad)
  ▼
  │  Layer 3: kernel driver(s)
  │    • /dev/brlkey_lock, /dev/brlkey_audiomode   (keyboard)
  │    • built-in 20-cell display controller        (path TBD)
  ▼
hardware (Perkins keys + 20 braille cells)
```
