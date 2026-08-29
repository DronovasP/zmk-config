# zmk-config

ZMK firmware for two keyboards, built on GitHub Actions (every push → *Actions* tab → `firmware` artifact zip).

| Keyboard | Shield | Controller | Link between halves | Host |
|---|---|---|---|---|
| **Cosmos 5x6** (handwired, 30 keys/half) | `cosmos_left` / `cosmos_right` | nice!nano v2 | wired, TRRS (full-duplex UART) | USB into the **left** half |
| Cradio / Ferris Sweep (34 keys) | `cradio_left` / `cradio_right` | nice!nano v2 | BLE | BLE/USB, `_lc`/`_rc` variants |

ZMK is pinned to the `v0.3` line in `config/west.yml` and `.github/workflows/build.yml` — keep those two in sync.

## Layout

```
boards/shields/cosmos/      the Cosmos shield (pins, matrix, wired split, Studio layout)
boards/shields/cradio/      Kconfig overrides for ZMK's built-in cradio shield
config/cosmos.keymap|conf   Cosmos keymap + options
config/cradio.keymap|conf   Cradio keymap + options (+ combos/leader/mouse dtsi)
config/west.yml             ZMK version + extra modules (urob's, used by the cradio keymap)
build.yaml                  GitHub Actions build matrix
scripts/gen_cosmos_layout.py  regenerates the Studio physical-layout picture
```

## Cosmos

### Wiring (pins as reported; row/column *order* still to be confirmed by testing)

Both halves use the same pins. Labels are the `D`-numbers printed on the nice!nano.

| Function | nice!nano pins |
|---|---|
| Rows 0→4 (top → bottom/thumb row) | `D3 D4 D5 D6 D7` (silkscreen `020 022 024 100 011`) |
| Cols 0→5 (outer/pinky → inner/index) | `D21 D20 D19 D18 D15 D14` (silkscreen `031 029 002 115 113 111`) |
| TRRS data | `D1` = TX, `D0` = RX — must be **crossed** between halves (TX↔RX) |
| TRRS power | VCC + GND (put them on opposite ends of the jack) |
| Diodes | `col2row` (cathode/bar towards the row) |

All of that lives in **one block** at the top of `boards/shields/cosmos/cosmos.dtsi`.
Wrong key printed → reorder pins there. Nothing printed → try `diode-direction = "row2col"`.
The bottom row is treated as 2 outer finger keys + 4 thumb keys per half.

### Flashing

1. Download the `firmware` artifact from the latest green Actions run and unzip.
2. Plug a half into the PC, double-tap its reset button → a `NICENANO` drive appears.
3. Copy `cosmos_left.uf2` to the left half, `cosmos_right.uf2` to the right half.
   Each half needs to be plugged in over USB to flash (the right one is only powered via TRRS in normal use).
4. Later you can enter the bootloader from the keymap: hold both layer thumbs (`Sys` layer) and press
   the top-left key (left half) or top-right key (right half).

`settings_reset.uf2` wipes stored settings/keymap on a half if a Studio edit leaves it in a weird state.

Never hot-plug the TRRS cable while a half is powered — the pins are not protected.

### Keymap

Plain QWERTY, no hold-taps on the base layer (gaming-safe).

- `Base` — QWERTY; thumbs: `LALT · Nav · Space · Bksp` | `Enter · Sym · Del · RAlt`
- `Nav` (hold left inner-ish thumb) — F1-F12, arrows/home/end/pgup/pgdn on the right hand, media, Caps
- `Sym/Num` (hold right thumb) — shifted symbols on the top row, numpad on the right hand
- `Sys` (Nav + Sym together) — `&bootloader`, `&sys_reset`, `&studio_unlock`

### ZMK Studio

The left half is built with `CONFIG_ZMK_STUDIO=y` and the `studio-rpc-usb-uart` snippet; locking is disabled
so no unlock key is needed. Plug the left half in, open <https://zmk.studio>, pick the USB device, remap.
Studio edits are stored on the keyboard and survive re-flashing the same keymap; use `settings_reset` to throw them away.

## Cradio

Unchanged from before. `_lc` artifacts are left-central builds, `_rc` right-central. Uses urob's
zmk-helpers/auto-layer/leader-key/tri-state/unicode/adaptive-key modules pulled in via `config/west.yml`.
