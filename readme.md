# Selta's Keypad

Custom firmware for a CH552-based 3-button USB macropad with rotary encoder.

## Features

- KVM switching via hotkey sequences
- Volume control (encoder twist)
- Mute toggle (encoder short-press)
- LED toggle (encoder long-press)
- RGB rainbow LED effect
- USB HID keyboard + consumer control

## Current Configuration

| Input | Function |
|-------|----------|
| BTN 1 | KVM Switch → Input 3 |
| BTN 2 | KVM Switch → Input 2 |
| BTN 3 | KVM Switch → Input 1 |
| Encoder Twist CW | Volume Up |
| Encoder Twist CCW | Volume Down |
| Encoder Short-Press | Mute Toggle |
| Encoder Long-Press (≥500ms) | Toggle LEDs On/Off |

> Note: Buttons are mapped "backwards" (1→3, 2→2, 3→1), and encoder rotation is "from the front" with vertical, USB port down orientation.
Also noteworthy, my KVM (Level1Techs) uses Ctrl + Ctrl sequence for changing inputs.

## LEDs

- **Normal mode:** Slow rainbow color cycle
- **Bootloader mode:** All red
- **Long-press encoder** to toggle LEDs on/off

LED settings can be adjusted in `src/led.h`:
- `NEO_BRIGHT_KEYS` - Brightness (0=dim, 1=medium, 2=bright)

Rainbow speed can be adjusted in `src/led.cpp`:
- `cycle_counter_s >= 4` - Higher = slower cycle

## Hardware

Based on a cheap CH552 macropad from Amazon:
- [Amazon link](https://www.amazon.com/dp/B0CD8DWT2S)

### What's Inside

- WCH CH552 microcontroller (unsure which variant)
- 3 mechanical buttons
- Rotary encoder with push button
- 3 addressable RGB LEDs (WS2812/NeoPixel)

### Pinout

| Function | Pin |
|----------|-----|
| Button 1 | P15 |
| Button 2 | P34 |
| Button 3 | P11 |
| Encoder Button | P33 |
| Encoder A | P32 |
| Encoder B | P14 |
| LEDs | P31 |

> Note: Different PCB revisions may have different pinouts. Check your board if things don't work.
The easiest way is to use a multi-meter, and check continuity between the key and the controller. You'll find continuity on the pin that the switch is connected to. Pin numbering usually is anti-clockwise, beginning with "1" where there is a notch or a small circle on the IC.

## Building & Flashing

### Requirements

- Arduino IDE
- CH55xduino board support

### Setup

1. Install Arduino IDE
2. Add CH55xduino board support:
   - Go to **Preferences → Additional Board Manager URLs**
   - Add: `https://raw.githubusercontent.com/DeqingSun/ch55xduino/ch55xduino/package_ch55xduino_mcs51_index.json`
3. Install the CH55xduino board from Board Manager

### Build Settings

In **Tools** menu:
- Board: **CH552**
- Bootloader: **P3.6 (D+) Pull up**
- Clock Source: **16MHz (internal) 3.5V or 5V**
- Upload Method: **USB**
- USB Settings: **USER CODE w/148B USB RAM**

### Entering Bootloader Mode

**First time / without custom firmware:**
- Short R12 or J4 (varies by PCB revision/brand) while connecting USB

> **WARNING:** Shorting the wrong component can damage your board or computer. Double-check which pads to short using a multimeter or reference photos for your specific PCB revision. I am not responsible for any damage caused by incorrect modifications.

**After flashing custom firmware:**
- Hold the encoder button while plugging in USB
- LEDs will turn red to indicate bootloader mode

### Flash

1. Get the macropad into bootloader mode
2. Click Upload (teal right arrow, or Sketch > Upload) in Arduino IDE

## Customization

### Button Mappings

Edit `configuration.cpp` to change button mappings.

#### Button Types

- `BUTTON_SEQUENCE` - Send keyboard key sequences
- `BUTTON_MOUSE` - Send mouse/media events
- `BUTTON_FUNCTION` - Call custom functions
- `BUTTON_NULL` - Disabled

#### Available Encoder Events

- `SCROLL_UP`, `SCROLL_DOWN`
- `LEFT_CLICK`, `RIGHT_CLICK`
- `UP`, `DOWN`, `LEFT`, `RIGH` (mouse movement)
- `VOLUME_UP`, `VOLUME_DOWN`, `MUTE_TOGGLE`

### Example Configurations

#### Single Key
```c
[BTN_1] = {
    .type = BUTTON_SEQUENCE,
    .function.sequence = {
        .sequence = {'a'},
        .length = 1,
        .delay = 0
    }
}
```

#### Two-Key Combo (Ctrl+C)
```c
[BTN_1] = {
    .type = BUTTON_SEQUENCE,
    .function.sequence = {
        .sequence = {KEY_LEFT_CTRL, 'c'},
        .length = 2,
        .delay = 0  // 0 = hold all keys together
    }
}
```

#### Three-Key Combo (Ctrl+Shift+Esc)
```c
[BTN_1] = {
    .type = BUTTON_SEQUENCE,
    .function.sequence = {
        .sequence = {KEY_LEFT_CTRL, KEY_LEFT_SHIFT, KEY_ESC},
        .length = 3,
        .delay = 0
    }
}
```

#### Sequential Keys (not held together)
```c
[BTN_1] = {
    .type = BUTTON_SEQUENCE,
    .function.sequence = {
        .sequence = {KEY_LEFT_CTRL, KEY_LEFT_CTRL, '1'},
        .length = 3,
        .delay = 50  // delay > 0 = press/release each key separately, in milliseconds
    }
}
```

#### Volume/Media Control
```c
[ENC_CW] = {
    .type = BUTTON_MOUSE,
    .function.mouse = {
        .mouse_event_sequence = {{.type = VOLUME_UP, .value = 1}},
        .length = 1,
        .delay = 0,
        .keypress = 0
    }
}
```

#### Mouse Scroll
```c
[ENC_CW] = {
    .type = BUTTON_MOUSE,
    .function.mouse = {
        .mouse_event_sequence = {{.type = SCROLL_UP, .value = 2}},
        .length = 1,
        .delay = 0,
        .keypress = 0
    }
}
```

#### Disabled Button
```c
[BTN_1] = {
    .type = BUTTON_NULL,
}
```

#### Re-enable Layer Switching
To re-enable the profile/layer menu on the encoder button:
```c
[BTN_ENC] = {
    .type = BUTTON_FUNCTION,
    .function.functionPointer = keyboard_press_enc,
}
```
Hold encoder and twist to switch profiles. See `configurations[]` array for profile definitions.

### Available Keys

Modifiers: `KEY_LEFT_CTRL`, `KEY_RIGHT_CTRL`, `KEY_LEFT_SHIFT`, `KEY_RIGHT_SHIFT`, `KEY_LEFT_ALT`, `KEY_RIGHT_ALT`, `KEY_LEFT_GUI`, `KEY_RIGHT_GUI`

Special: `KEY_ESC`, `KEY_TAB`, `KEY_RETURN`, `KEY_BACKSPACE`, `KEY_DELETE`, `KEY_INSERT`

Navigation: `KEY_UP_ARROW`, `KEY_DOWN_ARROW`, `KEY_LEFT_ARROW`, `KEY_RIGHT_ARROW`, `KEY_HOME`, `KEY_END`, `KEY_PAGE_UP`, `KEY_PAGE_DOWN`

Function: `KEY_F1` through `KEY_F24`

Printable characters: Use single quotes like `'a'`, `'1'`, `' '`

### Long-Press Threshold

Adjust in `src/buttons.cpp`:
- `LONG_PRESS_MS` - Default 500ms

### Multiple Profiles

The firmware supports multiple configuration profiles (see `configurations[]` array). The layer-switching menu is currently disabled but the code remains if you want to re-enable it.

## Credits

- Original CH55xduino by [Deqing Sun](https://github.com/DeqingSun/ch55xduino)
- KVM, volume control, and LED modifications by Selta

## Resources

- [CH55xduino on GitHub](https://github.com/DeqingSun/ch55xduino)
- [CH552 Datasheet](http://www.wch-ic.com/downloads/file/309.html)

## License

![license.png](https://i.creativecommons.org/l/by-sa/3.0/88x31.png)

This work is licensed under [Creative Commons Attribution-ShareAlike 3.0 Unported License](http://creativecommons.org/licenses/by-sa/3.0/).
