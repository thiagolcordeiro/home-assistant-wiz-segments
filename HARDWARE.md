# WiZ segment integration: hardware validation

Target: WiZ RGBIC 5 m, product 605568.
Read-only UDP queries on 2026-09-10 returned:

- Module: ESP25_MHORGB_01
- Firmware: 1.38.0
- Initial power state: off

This matches the device/firmware used in the independent protocol investigation:
https://github.com/TechAntohere/WizScreenSyncController/

The official product sheet lists RGB, not five independently driven channels:
https://www.assets.signify.com/is/content/Signify/US.en_US.046677605568

Do not infer dedicated white LEDs from white presets or c/w fields in getPilot.
Per-region warm and cool controls have since been demonstrated by the visual
test below. This confirms functional controls but not dedicated white emitters.

The protocol investigation reports at most 12 color runs, physical block widths,
and no segment frame readback. A saved custom scene can override supplied data
while still returning success. Visual validation is required. Region state in a
future integration must distinguish desired state from observed device state.

## Reusable probe

Read-only, using Python 3 with no third-party dependencies:

```text
python tools/probe.py STRIP_IP
```

With the strip initially off and a person watching it, test three low-brightness
RGB regions for 20 seconds, then restore and verify the off state:

```text
python tools/probe.py STRIP_IP --test --slot 258 --width 3 --seconds 20
```

Width is in hardware blocks, not individual LEDs. Slot 258 worked on the tested strip but may be occupied on other devices. Do not automatically scan slots
or claim that an acknowledgment demonstrates independent segment control.

## Confirmed visual test

The user observed three contiguous regions in red, green and blue, with 18 LEDs
per region and the rest dark. Each wire step had width 3, establishing 6 LEDs
per hardware block. Custom slot 258 accepted the test visually, not just by ACK.
The user reports removing 42 LEDs from the end. With the original 150 LEDs,
the installed strip has 108 LEDs / 18 physical blocks.

After the observation, restoring the combined original color fields was rejected
with Invalid params. A standalone `setPilot` with `state: false` succeeded and
`getPilot` verified power off. The original saved color/scene was not restored.
The reusable probe now omits sceneId 0 from restoration and has a separate OFF
fallback. The integration's all-off command contains only `state: false`.

## Confirmed white-control test

Four width-3 regions (18 LEDs each) were sent with global brightness 30 and
per-step dimming 100, on slot 258:

| Physical LEDs | Fields sent | Observed output |
|---|---|---|
| 1–18 | R=255, other channels zero | Red |
| 19–36 | Step index 4=255, RGB and index 5 zero | Warm white |
| 37–54 | Step index 5=255, RGB and index 4 zero | Cool white |
| 55–72 | R=G=B=255, indices 4 and 5 zero | Similar cool white, subtly different |

The user first described red, warm white and cool white, then confirmed that the
last area was twice as long with a subtle change halfway. This distinguishes the
two final regions. The rest was not commanded. There was no automatic shutoff;
after feedback, a standalone OFF command succeeded and getPilot verified off.

Mapping for this hardware/firmware: wire index 4 is warm, index 5 is cool.
Home Assistant RGBWW ordering is R,G,B,cold,warm, so the encoder must swap whites.
White-only frames must not be treated as off merely because RGB is zero.
Optical emitter composition, Kelvin calibration, mixed-white behavior and
independent brightness curves remain unmeasured.

An experimental RGBWW Home Assistant integration is now in `custom_components/wiz_segments`.
It has not yet been installed or validated inside Home Assistant.
