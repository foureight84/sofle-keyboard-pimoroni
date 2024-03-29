# [Pimoroni Trackball](https://shop.pimoroni.com/products/trackball-breakout) integration with [Sofle Keyboard v1.1 and v2.1](https://josef-adamcik.cz/electronics/let-me-introduce-you-sofle-keyboard-split-keyboard-based-on-lily58.html)

![SofleKeyboard with Pimoroni Trackball](https://raw.githubusercontent.com/foureight84/sofle-keyboard-pimoroni/master/images/full_view.jpg)

## Preface
This builds upon existing work by [sevanteri](https://github.com/sevanteri) and [drashna](https://github.com/drashna) from QMK to add trackball support to the Sofle Keyboard designed by [Josef Adamčík](https://josef-adamcik.cz/)

The keyboard uses a modified transport library created by [drashna's work on the Dactyl Manuform](https://github.com/drashna/qmk_firmware/tree/master/keyboards/handwired/dactyl_manuform/5x6_right_trackball) to send pointing movement from off-hand to the main-hand. I've also modified the Pimoroni library created by [seventari](https://github.com/sevanteri/qmk_firmware/tree/master/users/sevanteri) to use this custom transport. I couldn't have been able to get this working properly without Drashna's help (THANK YOU!).

You can find this modified firmware here: [https://github.com/foureight84/qmk_firmware/tree/sofle_foureight84](https://github.com/foureight84/qmk_firmware/tree/sofle_foureight84)
All relevant files are contained in the `keyboards/sofle/keymaps/foureight84/` directory. **(This is now outdated but usable. However if you decide to use this, then DO NOT merge upstream QMK master. I have already made changes and commited the keymap to [QMK master branch](https://github.com/qmk/qmk_firmware), use this instead if you want the latest firmware).** 

This modification takes away one encoder. The trade-off is gaining more features and not have to move your hands away from the keyboard often. ~~I haven't had a chance to look at the Sofle v2 yet to see where the new pin-outs have been rerouted to / or still easily accessible. However, you should be able to wire the Pimoroni directly to the MCU I2C and power pins.~~ Looks like the I2C pins are still available on the V2 revision as per this [reddit post](https://www.reddit.com/r/ErgoMechKeyboards/comments/o3zwgn/sofle_v21_rgb_pcb_with_trackball/) by [tenstaana](https://www.reddit.com/user/tenstaana/)

## Keyboard Build Guide
This guide only focuses on adding the Pimoroni trackball and assumes that you have already built your Sofle Keyboard. If you haven't gotten there yet, here is the [official build guide documentation](https://josefadamcik.github.io/SofleKeyboard/)

## QMK

### Cloning Firmware
https://github.com/qmk/qmk_firmware.git

'foureight84' keymap is now part of QMK's master branch. See [guide](https://github.com/qmk/qmk_firmware/blob/master/docs/newbs_getting_started.md) for setting up your QMK environment

```
qmk setup
```

### Flashing
Due to the way transport works with split keyboards, `EE_HANDS` is used to set handedness. When flashing the firmware, make sure to use the correct syntax based on your MCU.

#### Elite-C
Left hand:

```
qmk flash -kb sofle/rev1 -km foureight84 -bl dfu-split-left
```
Right hand:

```
qmk flash -kb sofle/rev1 -km foureight84 -bl dfu-split-right
```

#### Pro-Micro

Left hand:

```
qmk flash -kb sofle/rev1 -km foureight84 -bl avrdude-split-left
```
Right hand:

```
qmk flash -kb sofle/rev1 -km foureight84 -bl avrdude-split-right
```

### Firmware Build Customizations

#### Scrolling direction
The trackball motion will scroll when the `_LOWER` layer is active. The default scroll movement matches the trackball's direction. If you want to reverse the horizonal or verical scroll directions, add the following constants to ```keyboards/sofle/keymaps/foureight84/config.h```:

```
#define TRACKBALL_REVERSE_VSCROLL true //Vertical scroll
#define TRACKBALL_REVERSE_HSCROLL true //Horizontal scroll
```

#### Trackball LED Timeout
By default, no trackball activity will put its LED to sleep. Timemout default can be overwritten in `config.h` using the `TRACKBALL_LED_TIMEOUT` constant. Timeout is expressed in milliseconds. A value of `0` sets LED to always on. Alternatively, the method `trackball_set_rgbw` can be used in place of `trackball_set_timed_rgbw` to ignore LED timeout.

```
#define TRACKBALL_LED_TIMEOUT 5000 // time in milliseconds
```

### Keymap
Keymap has been changed slightly from Sofle's default. A new `_MOUSE` layer has been added to provide mouse button functionality (left and right clicks). Pressing the trackball on any layer will activate left click (trackball click is disabled in scroll mode). The `_MOUSE` layer can be activated while `_LOWER` layer is active and the `Left Control` key is pressed. Once `_MOUSE` layer is active, you only need to hold down the `Left Control` key to keep the layer active. `h` and `j` become left and right mouse buttons. To perform a click drag, for example, hold the `h` while manipulating the trackball.

Activating the `_RAISE` layer will change the encoder's functionality from the default volume up/down/mute to previous track/next tract/play and pause.

#### QWERTY
```
  ,-----------------------------------------.                    ,-----------------------------------------.
  | ESC  |   1  |   2  |   3  |   4  |   5  |                    |   6  |   7  |   8  |   9  |   0  | Bspc |
  |------+------+------+------+------+------|                    |------+------+------+------+------+------|
  | Tab  |   Q  |   W  |   E  |   R  |   T  |                    |   Y  |   U  |   I  |   O  |   P  |  \   |
  |------+------+------+------+------+------|                    |------+------+------+------+------+------|
  |LCtrl |   A  |   S  |   D  |   F  |   G  |-------.    ,-------|   H  |   J  |   K  |   L  |   ;  |  '   |
  |------+------+------+------+------+------|  MUTE |    |       |------+------+------+------+------+------|
  |LShift|   Z  |   X  |   C  |   V  |   B  |-------|    |-------|   N  |   M  |   ,  |   .  |   /  |RShift|
  `-----------------------------------------/       /     \      \-----------------------------------------'
             |   [  | LGUI | LAlt |LOWER | /Space  /       \Enter \  |RAISE | RCTR | RAlt |   ]  |
             |      |      |      |      |/       /         \      \ |      |      |      |      |
             `----------------------------------'            '------''---------------------------'
```
#### LOWER
```
  ,-----------------------------------------.                    ,-----------------------------------------.
  |  F1  |  F2  |  F3  |  F4  |  F5  |  F6  |                    |  F7  |  F8  |  F9  |  F10  | F11  | F12 |
  |------+------+------+------+------+------|                    |------+------+------+------+------+------|
  |  `   |   1  |   2  |   3  |   4  |   5  |                    |   6  |   7  |   8  |   9  |   0  |      |
  |------+------+------+------+------+------|                    |------+------+------+------+------+------|
  | MOUSE|   !  |   @  |   #  |   $  |   %  |-------.    ,-------|   ^  |   &  |   *  |   (  |   )  |   |  |
  |------+------+------+------+------+------|  MUTE |    |       |------+------+------+------+------+------|
  | Shift|  =   |  -   |  +   |   {  |   }  |-------|    |-------|   [  |   ]  |   ;  |   :  |   \  | Shift|
  `-----------------------------------------/       /     \      \-----------------------------------------'
             | LGUI | LAlt | LCTR |LOWER | /Enter  /       \Space \  |RAISE | RCTR | RAlt | RGUI |
             |      |      |      |      |/       /         \      \ |      |      |      |      |
             `----------------------------------'            '------''---------------------------'
```
#### MOUSE
```
  ,-----------------------------------------.                    ,-----------------------------------------.
  |      |      |      |      |      |      |                    |      |      |      |      |      |      |
  |------+------+------+------+------+------|                    |------+------+------+------+------+------|
  |      |      |      |      |      |      |                    |      |      |      |      |      |      |
  |------+------+------+------+------+------|                    |------+------+------+------+------+------|
  |      |      |      |      |      |      |-------.    ,-------|MS_BT1|MS_BT2|      |      |      |      |
  |------+------+------+------+------+------|       |    |       |------+------+------+------+------+------|
  |      |      |      |      |      |      |-------|    |-------|      |      |      |      |      |      |
  `-----------------------------------------/       /     \      \-----------------------------------------'
             | LGUI | LAlt | LCTR |LOWER | /Enter  /       \Space \  |RAISE | RCTR | RAlt | RGUI |
             |      |      |      |      |/       /         \      \ |      |      |      |      |
             `----------------------------------'            '------''---------------------------'
```

#### RAISE
```
  ,-----------------------------------------.                    ,-----------------------------------------.
  |      |      |      |      |      |      |                    |      |      |      |      |      |      |
  |------+------+------+------+------+------|                    |------+------+------+------+------+------|
  | Esc  | Ins  | Pscr | Menu |      |      |                    |      | PWrd |  Up  | NWrd | DLine| Bspc |
  |------+------+------+------+------+------|                    |------+------+------+------+------+------|
  | Tab  | LAt  | LCtl |LShift|      | Caps |-------.    ,-------|      | Left | Down | Rigth|  Del | Bspc |
  |------+------+------+------+------+------|  MPLY |    |       |------+------+------+------+------+------|
  |Shift | Undo |  Cut | Copy | Paste|      |-------|    |-------|      | LStr |      | LEnd |      | Shift|
  `-----------------------------------------/       /     \      \-----------------------------------------'
             | LGUI | LAlt | LCTR |LOWER | /Enter  /       \Space \  |RAISE | RCTR | RAlt | RGUI |
             |      |      |      |      |/       /         \      \ |      |      |      |      |
              `----------------------------------'           '------''---------------------------'
```
#### ADJUST
```
  ,-----------------------------------------.                    ,-----------------------------------------.
  |      |      |      |      |      |      |                    |      |      |      |      |      |      |
  |------+------+------+------+------+------|                    |------+------+------+------+------+------|
  | RESET|      |QWERTY|COLEMAK|     |      |                    |      |      |      |      |      |      |
  |------+------+------+------+------+------|                    |------+------+------+------+------+------|
  |      |      |MACWIN|      |      |      |-------.    ,-------|      | VOLDO| MUTE | VOLUP|      |      |
  |------+------+------+------+------+------|  MUTE |    |       |------+------+------+------+------+------|
  |      |      |      |      |      |      |-------|    |-------|      | PREV | PLAY | NEXT |      |      |
  `-----------------------------------------/       /     \      \-----------------------------------------'
             | LGUI | LAlt | LCTR |LOWER | /Enter  /       \Space \  |RAISE | RCTR | RAlt | RGUI |
             |      |      |      |      |/       /         \      \ |      |      |      |      |
              `----------------------------------'           '------''---------------------------'
```

## Pimoroni Trackball Add-on

On the hardware side, the Pimoroni trackball utilizes the open I2C pins above the rotary encoder and the VCC and GND for the LED.

You will need the following:
- **(1)** Pimoroni Trackball (of course)
- **(1)** 3d printed Pimoroni mount (https://www.thingiverse.com/thing:4756591)
- **(4)** M2 6+3mm Male-Female Standoffs
- **(4)** M2 4mm Screws
- **(4)** Jumper wires 
- **(4)** Dupont connectors
- **(5)** Round Needle Pin Headers (Male and Female) 2.54mm pitch
- **(5)** 90 degree pin header 2.54mm pitch

I've also designed a 7 degree base and PCB skirt for the PCB that can be 3d printed as well.
- Base: https://www.thingiverse.com/thing:4755855
- Skirt: https://www.thingiverse.com/thing:4754454
- MCU and OLED Housing: https://www.thingiverse.com/thing:4784085

### Steps:
The (5) Round needle pin female headers will need to be split into a set of two and a set of three pins. You will need to solder the two to the I2C pins above the encoder and the 3 pins for the LED. The I2C pins are labeled 2 SCL and 2 SDA, and the 3 pins labeled GND, LED, and VCC (see second PCB photo).

![Round Female Needle Pins locations](https://raw.githubusercontent.com/foureight84/sofle-keyboard-pimoroni/master/images/PCB.JPG)

![Round Female Needle Pins locations - closeup](https://raw.githubusercontent.com/foureight84/sofle-keyboard-pimoroni/master/images/PCB_I2C.JPG)

![Soldered Pins](https://raw.githubusercontent.com/foureight84/sofle-keyboard-pimoroni/master/images/wiring.jpg)

The (5) 90 Degree pin headers will be soldered onto the Pimoroni trackball. For the (4) jumper wires, you will need to crimp the Dupont connectors on one end and solder the male round needle pin headers on the other end.

Attach the (4) M2 6+3mm Male-Female Standoffs to the 3d printed the Pimoroni trackball mount.

![Pimoroni Mount](https://raw.githubusercontent.com/foureight84/sofle-keyboard-pimoroni/master/images/Pimoroni_mount.jpg)

When conecting the Pimoroni to the keyboard, you will not need to connect the INT pin on the trackball. Here's a table showing corresponding trackball to keyboard pins:

|  TrackBall Pins |   Keyboard Pins  |
|-----------------|------------------|
|       3-5V      |        VCC       |
|       SDA       | SDA (Next to R1) |
|       SCL       | SCL (Next to R2) |
|       GND       |        GND       |


![Pimoroni Mount assembled](https://raw.githubusercontent.com/foureight84/sofle-keyboard-pimoroni/master/images/assembled.jpg)
![Pimoroni Mount assembled](https://raw.githubusercontent.com/foureight84/sofle-keyboard-pimoroni/master/images/right_keyboard_side_view.jpg)
