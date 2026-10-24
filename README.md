# This fork

![bt-60-vuckale-layout.png](https://raw.githubusercontent.com/vuckale/bt-60-zmk/vuckale/bt-60-vuckale-layout.png)

# zmk-config
This is the ZMK-Config repo for the Polarity works BT60 V1 keyboard, with this you can customise your layout and keymap to suit your exact needs.
It builds against the current ZMK `main` (Zephyr 4.1, hardware model v2).

THIS IS NOT THE REPO FOR THE V2/BT65/BT75, those files can be accessed [here](https://github.com/PolarityWorks/zmk-config-ckp)

# Repository layout
```
.github/workflows/build.yml                  CI, uses ZMK's reusable build workflow
build.yaml                                   what CI builds (board bt60_custom)
config/west.yml                              pulls ZMK (main)
config/boards/vuckale/bt60_custom/
    bt60_custom.keymap                       layers and key bindings  <- edit this
    bt60_custom.dts                          hardware: matrix transform, pins, encoder, battery
    bt60_custom_defconfig, Kconfig.*,
    board.yml, board.cmake, bt60_custom.yaml board definition
```
The board is called `bt60_custom` (not `bt60`) because ZMK now ships its own in-tree `bt60` board and the two names would clash.

# Layers
| # | Name | How to reach it |
|---|------|-----------------|
| 0 | default_layer | always on |
| 1 | raise | hold Fn (the `&mo 1` key, right of Right Alt) |
| 2 | numpad_bt | toggle with Fn + the key between `=` and Backspace (`&tog 2`), same again to turn it off |

The numpad layer has the numpad on the right hand and Bluetooth / output control on the left hand:

| Keys | Action |
|------|--------|
| `7 8 9` / `U I O` / `J K L` / `M` | numpad 7 8 9 / 4 5 6 / 1 2 3 / 0 |
| `1 2 3 4 5` | `BT_SEL 0..4`: select / pair Bluetooth profile 1..5 |
| `Q W E R T` | `BT_DISC 0..4`: disconnect that profile (keeps the pairing) |
| `A` / `S` | `BT_PRV` / `BT_NXT`: previous / next profile |
| `D` / `F` | `OUT_USB` / `OUT_BLE`: choose USB or Bluetooth output |
| `Z` | `BT_CLR`: unpair the current profile |
| `C` | `BT_CLR_ALL`: wipe all profiles |

`BT_CLR` is also on the last key of the bottom row of the default layer.

# ZMK Studio (live remapping)
The firmware is built with ZMK Studio (`studio-rpc-usb-uart` snippet and `CONFIG_ZMK_STUDIO=y` in `build.yaml`). Open <https://zmk.studio/> in Chrome/Edge (or use the native app), connect the keyboard over USB and pick the serial port.
Changes are locked by default: hold Fn and press the last key of the bottom row (`&studio_unlock` on the raise layer), then Studio can edit the keymap.
The physical key positions shown in Studio come from `default_layout` in `bt60_custom.dts`. Two empty `reserved` layers in the keymap let you add layers 3 and 4 from Studio.
Once you change the keymap from Studio, edits to `bt60_custom.keymap` are ignored until you use "Restore Stock Settings" in Studio. Encoder (sensor) bindings can not be edited in Studio.

# Customisation
In order to change the keymap to meet your needs, you will need a GitHub account. Click the "fork" button in the top right to create your own version of the repo.
The keymap information is contained entirely within the file `config/boards/vuckale/bt60_custom/bt60_custom.keymap`. Edit it in GitHub's web editor (press `.` on the repo page) or any text editor, or use a visual editor such as [Keymap Editor](https://nickcoutsos.github.io/keymap-editor).
The list of ZMK keycodes and behaviors is in the [ZMK keymap docs](https://zmk.dev/docs/keymaps).
Commit the change, then click on "Actions" and wait for the build to complete (the orange dot will turn into either a red X or a green tick).

Every layer must have exactly 65 bindings (15 / 13 / 14 / 14 / 9 per row), one for each position in the matrix transform.

# Compilation and downloading
 If you have made any errors in the keymap the compilation will fail and there will be a red X in the actions dialog. Click on it and it will tell you where the problem is.
 If the build completes with no errors you will be able to find the firmware in the "Artifacts" section of the last action: download `firmware`, unzip it and you get `bt60.uf2`.
 Put the keyboard into bootloader (double tap the hardware reset button on the bottom of the keyboard, or use the `&bootloader` behavior if you have it on a key) and drag and drop the .uf2 file onto the usb drive that shows up.
 After the first flash of this version, remove the keyboard from your computer's Bluetooth list and pair it again.

 # More advanced
 If you want to exercise greater control over your BT60 and have a unique layout implemented you will need to change the matrix transform. The following diagram displays which keys connect to which row and column pins. Note that everything is zero indexed in the matrix transform
 	![alt text](rowcolmap.png)
  The matrix transform can be found in `config/boards/vuckale/bt60_custom/bt60_custom.dts`, more information on how to do the matrix transforms can be found in the official ZMK documentation [here](https://zmk.dev/docs/hardware-integration/new-shield)
  We're always willing to accept pull requests if you've developed your own layouts :)

# Updating ZMK
`config/west.yml` follows ZMK `main`, so every build uses the newest ZMK. If an upstream change ever breaks the build, replace `revision: main` with the commit SHA of a version that worked. What changed for out-of-tree boards in the Zephyr 4.1 update is described in the [ZMK blog post](https://zmk.dev/blog/2025/12/09/zephyr-4-1).

# Local builds

Steps taken from [ZMK Development Setup](https://zmk.dev/docs/development/local-toolchain/setup). Commands have been provided where the steps differ from the documentation.

1. Install the prerequisites, `west` and the toolchain as described in the [ZMK local toolchain setup](https://zmk.dev/docs/development/local-toolchain/setup)
2. Initialize West
    ```
    west init -l config
    ```
3. Fetch the modules, export Zephyr and install the Python dependencies
    ```
    west update
    west zephyr-export
    west packages pip --install
    ```
4. Build the firmware
    ```
    west build -s zmk/app -b bt60_custom -- -DZMK_CONFIG="$(pwd)/config"
    ```
5. Built firmware will be `build/zephyr/zmk.uf2`
