# Service interfaces (Binder / AIDL)

Method signatures recovered statically from the DEX of the extracted APKs
(androguard). Descriptors are Dalvik‑style (`I`=int, `Z`=boolean, `[B`=byte[],
`V`=void, `L…;`=object). These are the real API boundary between the HIMS apps
and the braille hardware.

## Keyboard / system service

**`com.jawon.han.input.IHanBrailleInputService`**

```
injectKey(I I I)V
set6DotMode(Z)V                     is6DotMode()Z
setAvailableSystemKey([I)V          setKey2StopTTS([I)V
lockHomeKey()V                      unlockHomeKey()V
lockSystemKey()V                    releaseSystemKey()V
playMacro(Ljava/lang/String;)V      setMacroPeriod(I)V   getMacroPeriod()I   setMacroDelayInputMode(Z)V
powerOff()V                         powerOffAppResponse()V
setAtlasFrequency(I)I
setProductionMode(I)I
updateScanChipFirmware([B)I
printEmulatorLcd(Landroid/graphics/Bitmap;)V
appLoadingComplete()V               getLoadingCompleteStatus()I
checkUpgradeStatus()Z               setUpgradeStatus(Z)V
checkTTSStopFlag()Z                 eraseTTSStopFlag()V
getJpnInputMode()I / setJpnInputMode(I)V / getJpnDialogState()I / setJpnDialogState(I)V / setJpnComposingText(String)V
registerCallback(IHanBrailleInputServiceCallback)V   unregisterCallback(…)V
```

**`com.jawon.han.input.IHanBrailleInputServiceCallback`**
```
onResponse(I I Ljava/lang/String;)Z     // key / event delivery back to the app
```

Reading: the keyboard + system keys are mediated by a **scan‑chip MCU**
(`updateScanChipFirmware`), with `injectKey` for synthetic input,
`set6DotMode` for 6‑ vs 8‑dot, and `setAtlasFrequency`/`setProductionMode`
exposing the internal **"Atlas"** controller. `printEmulatorLcd(Bitmap)` suggests
a visual LCD emulator distinct from the braille cells.

## Display service (external DotPad accessory)

**`com.selvashc.dotpad.IDotPadLineInterface`**

```
onLineData(Ljava/lang/String;)V
onLineDataWithCursor(Ljava/lang/String; I I)V
onRawLineData([B)V                  // raw dot bytes — the low-level path
onMultiLine(Ljava/lang/String;)V
onMultiLineString(Ljava/lang/String;)V
onMultiLineRaw(Ljava/lang/String;)V
onMultiLineDataAndLinePos(Ljava/lang/String; I)V
onMultiLineLinePos(I)V
onDotPatternImage(Landroid/graphics/Bitmap;)V
onIcon(Landroid/graphics/Bitmap;)V
onAllDown()V                        // clear all cells
isConnectDevice()Z
```

Also present: `IDotPadMathGraphInterface`, `IDotPadMathFormulaInterface` for
tactile math/graphics.

## Braille character model

**`com.jawon.han.haninputmethod.data.IBrailleChar`**
```
getBrailleText()String   getReadBrlText()String   getLCDText(I)String   getTTSText(I)String
getPasswordText()String  getEnglishChar()Z / setEnglishChar(Z)V
getBrailleCharAt(I)IBrailleChar   indexOf(IBrailleChar)I   include(IBrailleChar)Z   size()I
getWordId()I / setWordId(I)V
```

## Open question — reachability

Whether these services are `exported` and callable by a **non‑system** app, or
locked behind a HIMS **signature/system permission**, was not determined (the
manifest check was not completed). Expectation: they are system‑level and most
likely signature/permission‑protected, which would block a plain third‑party
bridge app. This is the key thing to verify for the chroot‑bridge path — see
[next-steps.md](next-steps.md).
