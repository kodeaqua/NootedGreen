# Alder Lake-U bring-up (i3-1215U, device-id 0x46B3)

Status: untested on hardware. This is a first-boot checklist, not a guarantee.

## Hardware facts

- iGPU: UHD Graphics, PCI device-id `0x46B3`, 64 EU = 4 DSS x 16 EU.
- The TGL accelerator binary counts sub-slices, so this is 8 SS x 8 EU.
- NootedGreen patches only match a spoofed `0x9A49` (TGL GT2). Never leave the real `0x46B3` visible.

## OpenCore `DeviceProperties`

`PciRoot(0x0)/Pci(0x2,0x0)`:

| Key | Type | Value |
|-----|------|-------|
| `AAPL,ig-platform-id` | Data | `499A0000` |
| `device-id` | Data | `499A0000` |

Optional: `edid` (Data) to inject your panel EDID.

## Boot args

First boot, with logging and the topology pinned so it does not depend on fuse reads:

```
-v keepsyms=1 debug=0x100 IGLogLevel=8 -NGreenDebug -ngreentglwithgfx -disablegfxfirmware -allow3d ngreen-dmc=adlp ngreen-dss=4 liludump=250 msgbuf=725288 liludbuf=725288
```

Display only (no accelerator), if the full stack hangs early: replace `-ngreentglwithgfx` with `-ngreentglfb` and drop `-allow3d`.
Expect fragmented output in this mode (DBUF is not programmed without the HW kext).

Second boot, to validate fuse detection: remove `ngreen-dss=4` and compare the `topology fuses:` / `topology (fuse)` log lines with 4 DSS.

## Logs to collect

```
log show --last boot | grep -E "ngreen|HANGCHECK|topology"
```

Key lines: `topology fuses:`, `topology (...)`, `HANGCHECK V206`, `HANGCHECK V204: CSB`, `HANGCHECK RCS HEAD/TAIL/ACTHD`.
