# External DotPad accessory (USB serial)

`dotPad.apk` integrates an **external Dot Inc. "DotPad"** tactile display — a
separate accessory, not the BS6 mini's built‑in 20‑cell line. This path is the
most tractable to drive from Linux, because it is a **standard USB‑serial**
device and Dot Inc. publishes its own SDK.

## Transport

`com.dotincorp.libs.usbserial.driver.*` is a fork of the open
**usb‑serial‑for‑android** library (mik3y). Standard CDC/FTDI/CP21xx‑style USB
serial:

```
UsbSerialPort / CommonUsbSerialPort:
  open(UsbDeviceConnection)      close()
  setParameters(I I I I)          // baud, dataBits, stopBits, parity
  read([B I)I / read([B I Z)I     write([B I)V
  setDTR(Z) / setRTS(Z) / getCD/CTS/DSR/RI
  getReadEndpoint() / getWriteEndpoint() / getPortNumber() / getSerial()
```

So from Linux this is reachable with ordinary `/dev/ttyUSB*` / `/dev/ttyACM*`
and termios — or BRLTTY — once the framing is known.

## Payload construction (where the wire protocol lives)

The byte‑level framing is built in these classes (method bodies not yet
decompiled — see [next-steps.md](next-steps.md)):

- **`com.selvashc.dotPad.device.DataSend`** — high‑level send API:
  `sendLineData(ctx,String)`, `sendLineDataWithCursor(ctx,String,int,int)`,
  `sendRawLineData(ctx, byte[])`, `sendMultiLineData`, `sendDotPatternImage`,
  `sendImageData`, `sendMathXMLData`, `allDownGraphic`.
- **`com.selvashc.dotPad.utils.DataSender`** — the implementation: paging,
  cursor create/delete, `getHexArray(String,int)` (hex framing),
  `sendLineAndWaitUntilComplete(...)`, bitmap→dots helpers
  (`textToBitmap…`, `skeletonize`, `dilateOnce` via OpenCV).
- **`com.selvashc.dotPad.utils.KeySender`** — input: `snedKey(int)` [sic],
  a `DOT_KEY` map, `sendKeyScroll(boolean)`, `get/setDotPadOption`.
- **`com.selvashc.dotPad.converter.Cell`** — character → dot‑coordinate mapping
  (`getBrailleDotCoordinate`, `getDots`).
- **`com.dotincorp.sdk_v3.model.DotPadKey`** — key enum.

## Why this matters

If the built‑in display proves too locked‑down to drive from Linux, an external
DotPad over USB is a fully open fallback for a Linux braille/tactile setup, and
reconstructing `getHexArray` + `sendRawLineData` yields a concrete, documentable
wire protocol.
