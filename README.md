# Pyxel Mono

A custom monospaced pixel bitmap font for the [Pimoroni Pico Display Pack](https://shop.pimoroni.com/products/pico-display-pack) and the [PicoGraphics](https://github.com/pimoroni/pimoroni-pico/blob/main/micropython/modules/picographics/README.md) MicroPython library.

Pyxel Mono is designed as a drop-in display wrapper: wrap your `PicoGraphics` instance once and use `.text()`, `.character()`, and `.measure_text()` with the same API you already know, while all other display calls pass through untouched.

---

## Supported characters

The font covers the full printable ASCII range plus a small selection of extended characters:

- Uppercase `A–Z` and lowercase `a–z`
- Digits `0–9`
- ASCII punctuation: ``! " # $ % & ' ( ) * + , - . / : ; < = > ? @ [ \ ] ^ _ ` { | } ~``
- Extended: `£ ¥ – — • €`

---

## Installation

Copy `pyxel_mono.py` to the root of your Pico's filesystem using [Thonny](https://thonny.org/):

No other dependencies are required beyond the standard Pimoroni MicroPython build.

---

## Usage

### Basic setup

Import `PyxelMono` and wrap your `PicoGraphics` display. Use `display` exactly as you normally would — only `.text()`, `.character()`, and `.measure_text()` are handled by the font; everything else is forwarded to PicoGraphics transparently.

```python
from picographics import PicoGraphics, DISPLAY_PICO_DISPLAY
from pyxel_mono import PyxelMono

display = PyxelMono(PicoGraphics(display=DISPLAY_PICO_DISPLAY))
```

### Drawing text

Set a pen colour first, then call `.text()`:

```python
WHITE = display.create_pen(255, 255, 255)
display.set_pen(WHITE)
display.text("Hello World", 10, 10, scale=2)
display.update()
```

### Word wrap

Pass a pixel width as `wordwrap` and long strings will break automatically at word boundaries:

```python
display.text("The quick brown fox", 10, 10, wordwrap=100, scale=1, spacing=2)
```

### Letter spacing

Add extra pixels between characters with `spacing`. The default is `1`, which works well at larger scales. At `scale=1` (the smallest, default size), bumping spacing to `2` noticeably improves legibility:

```python
display.text("Hello World", 10, 10, scale=1, spacing=2)

display.text("Hello World", 10, 30, scale=2)
```

### Drawing a single character

Pass the decimal ASCII code to `.character()`:

```python
# Draw an ampersand (&) at position (20, 20), 2× scale
display.character(38, 20, 20, scale=2)
```

### Measuring text

Use `.measure_text()` to get the pixel width of a string before drawing — useful for aligning text:

```python
WIDTH, HEIGHT = display.get_bounds()
label = "Score: 100"

text_w = display.measure_text(label, scale=2)
x = (WIDTH - text_w) // 2  # horizontally centered

display.text(label, x, 10, scale=2)
```

### Full example

```python
from picographics import PicoGraphics, DISPLAY_PICO_DISPLAY
from pyxel_mono import PyxelMono

display = PyxelMono(PicoGraphics(display=DISPLAY_PICO_DISPLAY, rotate=0))

WIDTH, HEIGHT = display.get_bounds()

BLACK = display.create_pen(0, 0, 0)
WHITE = display.create_pen(255, 255, 255)
CYAN  = display.create_pen(0, 200, 200)

# Clear to black
display.set_pen(BLACK)
display.clear()

# Centered title
display.set_pen(WHITE)
title = "Hello World"
x = (WIDTH - display.measure_text(title, scale=2)) // 2
display.text(title, x, 10, scale=2)

# Subtitle in a different colour
display.set_pen(CYAN)
display.text("The quick brown fox jumps over the lazy dog.", 10, 46, wordwrap=WIDTH - 20, scale=1, spacing=2)

# Word-wrapped paragraph
display.set_pen(WHITE)
display.text("The quick brown fox jumps over the lazy dog.", 10, 82, wordwrap=WIDTH - 20, scale=1, spacing=2)

display.update()
```

---

## API reference

### `PyxelMono(display)`

Wraps a `PicoGraphics` instance. All methods not listed below are forwarded directly to the underlying display.

---

### `display.text(text, x, y, wordwrap=-1, scale=1, spacing=1)`

Draw a string using Pyxel Mono.

| Parameter  | Type  | Default | Description                                            |
| ---------- | ----- | ------- | ------------------------------------------------------ |
| `text`     | `str` | —       | String to render                                       |
| `x`        | `int` | —       | Left edge of the first character (pixels)              |
| `y`        | `int` | —       | Top edge of the first character (pixels)               |
| `wordwrap` | `int` | `-1`    | Wrap lines at this pixel width; `-1` disables wrapping |
| `scale`    | `int` | `1`     | Integer pixel scale factor                             |
| `spacing`  | `int` | `1`     | Extra pixels added between each character              |

Colour is determined by the pen currently set on the display via `display.set_pen()`.

---

### `display.character(char, x, y, scale=1)`

Draw a single character by its decimal ASCII code.

| Parameter | Type  | Default | Description                                      |
| --------- | ----- | ------- | ------------------------------------------------ |
| `char`    | `int` | —       | Decimal ASCII code (e.g. `65` = `A`, `38` = `&`) |
| `x`       | `int` | —       | Left edge of the glyph cell (pixels)             |
| `y`       | `int` | —       | Top edge of the glyph cell (pixels)              |
| `scale`   | `int` | `1`     | Integer pixel scale factor                       |

---

### `display.measure_text(text, scale=1, spacing=1, fixed_width=False)`

Return the pixel width of a string without drawing it.

| Parameter | Type  | Default | Description                                                                |
| --------- | ----- | ------- | -------------------------------------------------------------------------- |
| `text`    | `str` | —       | String to measure                                                          |
| `scale`   | `int` | `1`     | Integer pixel scale factor                                                 |
| `spacing` | `int` | `1`     | Extra pixels between characters — must match the value passed to `.text()` |

---

## Customising glyphs

Each character is defined in the `_RAW` dictionary near the top of `pyxel_mono.py` as a list of 12 strings, each exactly 5 characters wide. `#` is a filled pixel, `.` is empty.

To edit an existing glyph, find it by its character and change the pattern:

```python
ord('A'): [  # 'A'
    ".....",
    ".....",
    "..#..",
    ".#.#.",
    "#...#",
    "#...#",
    "#####",
    "#...#",
    "#...#",
    "#...#",
    ".....",
    ".....",
],
```

To add a new character, add a new entry to `_RAW` following the same 5×12 format:

```python
ord('¿'): [  # '¿'
    ".....",
    ".....",
    "..#..",
    ".....",
    ".....",
    "..#..",
    ".##..",
    "#....",
    "#...#",
    ".###.",
    ".....",
    ".....",
],
```

Glyphs are bit-packed at import time, so there is no runtime cost for larger character sets.
