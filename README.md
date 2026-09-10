# WiZ Segments — experimental Home Assistant integration

**English** | [Português brasileiro](README.pt-BR.md)

Version 0.3.0 · [MIT license](LICENSE). Independent community project; not an official WiZ integration.

Control a WiZ RGBIC strip directly by IP, with one RGBWW light entity per named
segment. No ESP or WLED firmware is involved. Implemented independently from
the [documented community protocol findings](https://github.com/TechAntohere/WizScreenSyncController/).
See [HARDWARE.md](HARDWARE.md) for actual hardware observations.

## Supported hardware and controls

- WiZ 605568, module `ESP25_MHORGB_01`, firmware `1.38.0` was identified locally.
- A user visually confirmed three separate red/green/blue regions, each 18 LEDs
  long, with a width setting of 3. Therefore one physical block is 6 LEDs.
- The reported cut removed 42 LEDs from the original 150: that test installation
  has **108 LEDs / 18 blocks**. Adjust this if the original count differs.
- RGB, warm-white control, cool-white control, brightness and on/off are available
  per entity. A second visual test confirmed separate warm and cool outputs with
  RGB zeroed, alongside an RGB-white reference. This demonstrates functional
  controls, not whether the strip has physically separate white emitters.
- The implementation enforces the reported limit of 12 wire regions. Dark gaps
  and an unused tail consume regions too. This limit has not been retested on
  the user's strip.
- The firmware does not return segment colors. Entities use assumed state.

## Requirements and HACS installation

The declared minimum is Home Assistant 2026.3 for bundled local brand images;
this is not a claim of verified runtime compatibility across HA versions.
The strip must already be connected to Wi-Fi and reachable from Home Assistant
on UDP port 38899. Reserve its address in your router's DHCP settings.
No internet port forwarding is needed.

If HACS is not installed, follow its [official setup guide](https://www.hacs.xyz/docs/use/)
for your installation type first. Then:

1. Open **HACS → ⋮ → Custom repositories**.
2. Add `https://github.com/thiagolcordeiro/home-assistant-wiz-segments`
   with type **Integration**.
3. Find **WiZ Segments**, open it and select **Download**.
4. Restart Home Assistant.
5. Open **Settings → Devices & services → Add integration → WiZ Segments**.

This is a custom repository installation, not approval in the default HACS catalog.
See [HACS instructions](https://www.hacs.xyz/docs/faq/custom_repositories/).

## Manual installation and initial setup

Download the repository source or a release. Copy `custom_components/wiz_segments`
to `/config/custom_components/wiz_segments`. The final manifest path must be
`/config/custom_components/wiz_segments/manifest.json`. Restart Home Assistant
and add the integration as described above.

Enter your strip's IP, installed physical block count and custom mode (default 258).
There are six LEDs per block on the tested hardware: 150 LEDs = 25 blocks;
a strip cut to 108 LEDs = 18 blocks. The integration cannot detect cut length.
Up to three initial entities cover the strip. With 18 blocks these are 1–6,
7–12 and 13–18, or 36 LEDs each. Setup does not change lighting.
English and Portuguese interface translations are included. Rename the device
and entities through the usual Home Assistant interface.

## Create or adjust segments

Open the integration's **Configure** menu. Choose **Edit segment**, **Add
segment**, or **Remove segment**, then **Save changes**. Changes are staged
until Save, and physical lighting changes on the next light command.

Ranges are inclusive, numbered from 1, in blocks of six physical LEDs:

| Blocks | Physical LEDs |
|---|---|
| 1–3 | 1–18 |
| 4–6 | 19–36 |
| 7–9 | 37–54 |
| 10–18 | 55–108 |

Shrink or remove an existing region before adding another in its space.
Overlapping regions and layouts requiring more than 12 wire steps are rejected.
At least one region must remain configured. IDs stay stable when a region is
renamed or resized, so existing automations continue to target the same entity.
Removing a region deletes its entity from the entity registry after Save.

Home Assistant entities now declare the RGBWW color mode. Each action
rebuilds the full frame while retaining other currently controlled regions.
Version 0.3.0 adds the host-generated animations documented below. Individual
LEDs inside a six-LED block and native transition commands are not supported. The standard card's controls depend
on the installed HA frontend; five independent sliders are not bundled here.

For precise channel control, use `light.turn_on` with `rgbww_color` in the order
**[red, green, blue, cold white, warm white]**, each from 0 to 255, plus an
independent `brightness` value. For example:

| Output | rgbww_color |
|---|---|
| Red | [255, 0, 0, 0, 0] |
| Warm white only | [0, 0, 0, 0, 255] |
| Cool white only | [0, 0, 0, 255, 0] |
| White made by RGB | [255, 255, 255, 0, 0] |

Example action in Developer tools or an automation (replace the entity ID):

```yaml
action: light.turn_on
target:
  entity_id: light.wiz_rgbic_segment_1
data:
  brightness: 128
  rgbww_color: [0, 0, 0, 0, 255]
```

```yaml
action: light.turn_off
target:
  entity_id: light.wiz_rgbic_segment_1
```

The wire uses warm then cool, so the integration deliberately swaps the last two
values. This mapping was visually validated on the user's firmware 1.38.0.
Simultaneous white-channel mixing and optical brightness curves have not been
measured; no calibrated Kelvin range is advertised. Saved 0.1.0 RGB preferences
are loaded with both white channels set to zero.

## Interaction with the WiZ app and restarts

- Custom mode 258 worked in the visual test. Keep it free of saved custom modes
  in WiZ: an occupied slot may acknowledge commands while ignoring their colors.
- The integration stores last-commanded colors and brightness, but never
  automatically turns the strip on at startup or after reconfiguration.
- After a restart, layout change, connection failure or external scene change,
  per-segment on/off is unknown while the strip is on. Turn on a segment to
  resume control; unknown other regions are then set dark. An isolated OFF is
  rejected while an external scene is active, to avoid blacking out unrelated
  regions whose colors cannot be read back.
- When the strip reports off, all segment entities report off. Subsequent
  activation does not unexpectedly replay previously lit other regions.
- External writes to the same custom slot cannot be reliably detected.
  Avoid simultaneously controlling this strip with WiZ scenes, WLED or another
  Home Assistant integration while using per-segment control.
- An acknowledgment confirms command acceptance, not optical output. Color and
  brightness remain assumed values. Firmware behavior may change after updates.

## Updates, rollback and removal

Back up Home Assistant before updating. Download the desired version in HACS
and restart. To roll back, use HACS' redownload option to select a previous
available release, or restore a backup. For manual installation, replace the
integration directory with the desired version and restart.

To uninstall, remove the integration entry in Devices & services, remove the
HACS download (or manually installed folder), then restart. Unloading does not
switch the strip off; turn it off first if that is your desired final state.

## Troubleshooting

| Symptom | Check |
|---|---|
| Integration not listed | Verify the manifest path, restart HA and reload the page. |
| Cannot connect | Verify power, IP, Wi-Fi isolation/VLAN routing and UDP 38899 from HA. |
| Unsupported module | Report model/module/firmware; do not bypass the module check. |
| ACK but unchanged colors | Check whether a saved WiZ scene occupies the custom slot. |
| Wrong region lengths | Count installed six-LED blocks, accounting for cuts. |
| Rejected layout | Check bounds, overlaps and the 12-region limit including gaps/tail. |
| Unknown state | Turn a segment on to resume control as described above. |
| No white controls in card | Send `rgbww_color` through an action with RGB zeroed. |

Find `wiz_segments` messages in **Settings → System → Logs**. Include HA and
integration versions, module/firmware, layout and expected/observed output in
issues. Remove IP/MAC addresses, emails and credentials from shared logs.

## Segment lengths and dynamic distribution (0.3.0)

In **Configure → Edit segment**, set **Number of blocks**. Each block contains
six LEDs on the supported strip. **Auto-arrange segments** packs the existing
segments from block 1 in their current order, preserving entity IDs and removing
gaps. New segments are appended. With auto-arrange enabled, **First block** is
ignored; disable it to place a segment at a specific position. The requested
lengths must fit the physical strip: this does not create additional physical LEDs.

Use **Distribute segment count** to divide the entire strip into 1–12 segments
(also limited by installed block count). For 18 blocks, six segments have three
blocks each. Change individual lengths afterward if desired. Distribution replaces
previous lengths and gaps, retains the first existing names/IDs in physical order,
and removes entities at the end when reducing the count. Changes remain staged
until **Save changes**. Adjust automations targeting removed entities.

## Animated effects (0.3.0)

Each light exposes an effect list in Home Assistant:

| Effect | Behavior |
|---|---|
| `off` | Solid configured RGBWW color; stops animation without switching off. |
| `Rainbow` | Moving rainbow across the available regions of the segment. |
| `Chase` | A bright region moves across a dim background in the selected RGBWW color. |
| `Breathe` | Smoothly varying brightness using the selected RGBWW color. |
| `Color loop` | The whole segment cycles through RGB hues. |

```yaml
action: light.turn_on
target:
  entity_id: light.wiz_rgbic_segment_1
data:
  brightness: 128
  effect: Rainbow
```

Choose `effect: "off"` for a solid color, or call `light.turn_off` to switch the
segment off. A color command without an explicit effect stops its animation;
brightness-only commands retain it. Rainbow and Color loop generate their own
RGB colors; Breathe and Chase use the configured RGBWW channels.

In **Configure → Strip connection and length**, set the cycle duration (1–60 s,
default 6) and update rate (1–5 per second, default 2). These settings apply to the
strip's effects and require Save. Frames are sent sequentially; slower network
responses reduce the actual rate, with no queued backlog. Per-frame animation
data is not saved to disk or published as individual Home Assistant state changes.

These are original host-generated effects inspired by common LED animations,
not the WLED effect engine. Static regions, dark gaps and the tail reserve their
wire steps first. Animated segments fairly share the remaining space in the
12-region command budget. Long segments therefore animate in larger groups;
with no spare regions, Rainbow becomes a color cycle and Chase cannot move within
that segment. Use fewer segments/gaps for greater spatial detail. There is no
individual control within a six-LED block and no WLED frame-rate guarantee.

The HA server must remain running. Effects stop after a detected external scene,
power-off, communication failure, reload or HA shutdown and do not resume by
themselves. Stopping HA can leave the last transmitted colors lit. External writes
to the same slot cannot reliably be detected. The 0.2.1 integration was confirmed
working by its user; these new animations still need visual validation on hardware.

## Validation and remaining work

Offline tests cover the original behavior and the new layouts/effects. The frame builder and real UDP transport tests cover
layout bounds, gaps, brightness, timeout/retry and malformed replies. Isolated
coordinator tests check serialized concurrent writes, failed-write rollback,
preserving other regions and no startup replay. These use minimal substitutes
for HA infrastructure and do not constitute full Home Assistant lifecycle tests.

The asynchronous client successfully queried the real strip. The earlier
three-color payload and the separate warm/cool/RGB-white payload were visually
verified by the user. The production full-frame
payload (including dark gaps and independent brightness) still requires visual
verification from within Home Assistant. The strip was verified off afterward.

All Python files are syntax-checked with Python 3.11. The integration uses modern
HA config entries and coordinator APIs, and the user confirmed version 0.2.1 working in Home Assistant. The new 0.3.0
features still require runtime and visual validation. Confirm the installed HA version before deploying in a
production setup. The user installed and validated version 0.2.1.

Run offline tests using Python 3.11+:

```text
python -m unittest discover -s tests -v
python tools/validate_release.py
```

The `tools/probe.py` utility defaults to read-only device identification. Its
optional visual test requires the strip to start off and restores the off state.

## Sources

- [Independent WiZ RGBIC protocol investigation](https://github.com/TechAntohere/WizScreenSyncController/)
- [Official 605568 product sheet](https://www.assets.signify.com/is/content/Signify/US.en_US.046677605568)
- [Home Assistant light entity API](https://developers.home-assistant.io/docs/core/entity/light/)
- [Home Assistant config flow API](https://developers.home-assistant.io/docs/core/integration/config_flow/)

The implementation and tests were generated with AI assistance. Source code from
the protocol reference application was not copied into this integration.

See [contributing](CONTRIBUTING.md) and [release publishing](docs/PUBLISHING.md).
