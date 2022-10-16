---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
# background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
# use UnoCSS
css: unocss
layout: center

# default: 980
# since the canvas gets smaller, the visual size will become larger
canvasWidth: 800
---

# MicroPython + esp32 + Bitcoin

## @KeithMukai

<div class="absolute bottom-30px">
Hardware sponsored by <img src="bitcoin_magazine_logo_white.png" class="w-50">
</div>
<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---

# What are we building?

### A basic platform for your own DIY Bitcoin project
<br/>

<img src="build_complete_01.jpg" class="w-75" style="float: right;">

* Cheap, easily-sourced microcontroller
* Hardware inputs, display, camera
* Coder-friendly MicroPython
* Bitcoin libraries onboard
* "Advanced" UI rendering library

---

# The workshop plan

<img src="kit_components.jpg" class="w-50" style="float: right;">

<br/>

* Phase I: Overview (this presentation)
* Phase II: You go off & assemble on your own time
* Phase III: Find me at the SeedSigner table to help debug, brainstorm project ideas, etc.

---

# Caveat

<div class="text-center" style="margin-top: 7em;">

## I AM NOT AN EXPERT!

</div>

---

# MicroPython:

<br/>

* Basically* same as Python3
* But lighter, meant for embedded devices
* Compiled/Optimized for each platform
* Live interactive REPL
* Trezor, Coldcard, Passport, Specter-DIY, Krux

---

# Why not...

<br/>

* C/C++?
* Rust?
* ...?

---

# MicroPython:

<br/>

Firmware interprets & executes code stored as:
<br/>

* *.py files (slowest)
* *.mpy bytecode
* "Frozen" bytecode
* Compiled C code w/python bindings (fastest)

---

## Bitcoin Core's `secp256k1` in MicroPython
<hr>


```python
import secp256k1, hashlib
from binascii import hexlify

secret = hashlib.sha256(b'reallysecuresecret').digest()

if not secp256k1.ec_seckey_verify(secret):
    raise ValueError("Secret key is invalid")

# computing corresponding pubkey
pubkey = secp256k1.ec_pubkey_create(secret)

# serialize the pubkey in compressed format
sec = secp256k1.ec_pubkey_serialize(pubkey, secp256k1.EC_COMPRESSED)

...

sig = secp256k1.ecdsa_sign(msg_hash, secret)
```

---
layout: two-cols
---

# `embit`

@StepanSnigirev's Bitcoin utility library

Used in:
* Specter-DIY
* Specter Desktop
* SeedSigner
* Krux

::right::

<br/>
<br/>
```python
from embit import bip32, bip39, script

seed = bip39.mnemonic_to_seed(mnemonic)
root = bip32.HDKey.from_seed(seed)
xprv = root.derive("m/84'/0'/0'")
xpub = xprv.to_public()

# generate first receive addr
pubkey = xpub.derive([0,0]).key
addr = script.p2wpkh(pubkey).address()
```

---
layout: two-cols
---

# LVGL
"Light and Versatile Graphics Library"

<br/>
Can't use Python Image Library (PIL) in MicroPython.
<br/>
<br/>

* Lightweight C library for nice UIs on embedded devices
* fonts, shapes, widgets, user input
* Has C-bindings for MicroPython (sorta)


::right::
<img src="lvgl_demo.png" class="absolute right-30px w-75">


---
layout: two-cols
---

# LVGL...
...is not fun

* Not pythonic at all
* Closer to CSS implemented as C structs
* Docs are somehow thorough yet not useful
* Must compile from their fork of MicroPython: `lv_micropython`

::right::

<br/>
<br/>
```python
import lvgl as lv
lv.init()
top_nav = lv.obj(scr)
top_nav.set_size(240, 48)
top_nav.align(lv.ALIGN.TOP_LEFT, 0, 0)

style = lv.style_t()
style.init()
style.set_pad_all(0)
style.set_border_width(0)
style.set_radius(0)
style.set_bg_color(lv.color_hex(0x000000))
top_nav.add_style(style, 0)
```
---

## Building our custom MicroPython firmware

<br/>
Lots of pieces:
<hr>

* ESP-IDF compiler
* @esixtyone's fork of LVGL `lv_micropython`, `lv_bindings`
* usermods:
  * `secp256k1-embedded` & `uhashlib`
  * `esp32-camera-driver`
  * QR decoder (Quirc? TBD)
* frozen modules:
  * `embit`
* custom SAOLA-1R board definition

---

# Managing all the pieces

* One main git repo to gather dependencies
* Compile within a Docker container

---

## Start here

<br/>

![](github_screenshot.png)

<div class="absolute bottom-30px">
  https://github.com/kdmukai/micropython-esp32
</div>

---

# The Hardware

## Inventory check
<img src="kit_components.jpg" class="w-100" style="float: right;">

<br/>

* ESP32-S2 Saola-1R
* Waveshare LCD hat
* 40-pin gpio adapter
* Alcohol wipe
* OV2640 camera module
* gpio expander board (not needed)
* Full- & half-size breadboard
* 15 male-to-male jumpers
* 16 male-to-female jumpers

---


# The Hardware

### esp32-S2 Saola-1R dev board
<img src="esp32-s2_saola1-pinout.jpg" class="h-75" style="margin: auto; float: right;">

<br/>

* $8
* single-core @ 240 MHz
* 4MB ROM, 8MB RAM
* tons of gpios

---

# esp32-S2's killer feature

<img src="digikey_saola1r.png" class="h-75" style="margin: auto;">

---

# esp32-S2's killer feature

<img src="digikey_esp32s2.png" class="h-75" style="margin: auto;">

---

# #LowTimePreference, right?

<img src="speed_comparison.png" class="h-75" style="margin: auto;">

---

# Raspberry Pi RP2040 / Pico
<img src="raspi_pico.jpg" class="h-75" style="float: right;">

<br/>

* $4
* Dual-core @ 133 MHz
* 264kB RAM; NOT expandable
* Raspi supply chain

---

# esp32-S3

<img src="ESP32-S3_DevKitC-1_pinlayout.jpg" class="h-75" style="float: right;">

<br/>

* dual-core @ 240 MHz
* 8MB ROM / 8 MB RAM
* IDF compiler = 😡
* `lv_micropython` = 😡
* Need help, time

---

# Old esp32?

<img src="old_esp32.jpg" class="h-75" style="float: right;">

<br/>

### (not "-S" or "-C" series)

<br/>

* older, slower
* considered not secure

---

# Easily change esp32 chips/boards later

<img src="micropython_esp32_boards.png" class="h-75">

---

# Eventually support other platforms

<img src="micropython_ports.png" class="w-150" style="margin: auto; margin-top: 5em;">

---

# Waveshare 1.3" LCD hat

<img src="waveshare_lcd_hat.jpg" class="h-50" style="float: right;">

<br/>

* Same display as SeedSigner(!)
* ST7789 240x240 color display
* 5-way joystick, 3 side keys

---

# 40-pin gpio adapter

<img src="gpio_adapter.png" class="h-50" style="float: right;">

<br/>

* Makes LCD hat breadboard-able
* Convenient* labeled pins

---

# Waveshare OV2640 camera board

<img src="ov2640.jpg" class="h-50" style="float: right;">

<br/>

* Ubiquitous workhorse
* Jumper wire connection is iffy

---

# Solderless breadboards

<br/>

<img src="solderless_breadboards_2.jpg" class="h-50" style="float: right; margin: 10px;">
<img src="solderless_breadboards_1.jpg" class="h-50" style="float: right; margin: 10px;">

---

# Writing the firmware to the board

<br/>

* Connect USB
* Enter bootloader mode: Hold "BOOT", click "RST", release "BOOT"
* Find the board: `ls /dev/tty.*`
* `erase_flash` and `write_flash`:
```bash
esptool.py -p /dev/tty.usbserial-1110 erase_flash
esptool.py -p /dev/tty.usbserial-1110 -b 460800 --before default_reset --chip esp32s2  \
  write_flash --flash_mode dio --flash_size detect --flash_freq 80m 0x1000 bootloader.bin \
  0x8000 partition-table.bin 0x10000 micropython.bin
```

* Power cycle to exit bootloader mode

---

# Copying files/resources over

<br/>

```bash
# List the files on the board
ampy -p /dev/tty.usbserial-1110 ls

# Transfer a file
ampy -p /dev/tty.usbserial-1110 put demos/fonts/opensans_regular_17.bin
```

---

# Running local scripts

<br/>

```bash
mpremote connect /dev/tty.usbserial-1110 run demos/secp256k1_test.py
```

---

# Demos!

<br/>

* `micropython-esp32/demos`

---

# Caveats / Limitations

<br/>

* Networking is knocked out for esp32-S2
* No QR decoder yet
* Flaky camera

---

# Further Possibilities

<br/>

* Add an NFC reader?
* SD card reader?
* Touchscreen?
* Other microcontrollers

---

# Graduate from solderless breadboards

<img src="build_complete_02.jpg" class="w-50" style="float: right;">
<img src="build_complete_03.jpg" class="w-50" style="float: right;">

<br/>

* Soldered protoboard
* Custom PCB

---

# Help with SeedSigner!

<img src="seedsigner_diagram.jpg" class="w-100" style="float: right;">

<br/>

* QR decoder (Quirc?)
* Fork LVGL's QR encoder
* Load from SD card
* eFuse signed firmware
* Custom pcb
* Custom C modules
  * Memory/framebuffer management
  * Camera + QR decoder + live display
  * `embit` enhancements
* Port current Python codebase

---

# What will you build?

<br/>

* Lightning?
* Decentralized IDs?
* PGP signer?

<br/>

### Send me updates!
* @KeithMukai
* Write an article about your project for Bitcoin Magazine

<br/>
<img src="bitcoin_magazine_logo_white.png" class="w-50 absolute bottom">