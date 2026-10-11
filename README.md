# CYD Club MicroPython

**A MicroPython + LVGL firmware for the 2.8" "Cheap Yellow Display",
made for learning Python and for vibe coding.**

Flash it, plug the board into a computer, and tell an AI coding agent:

> *There's a MicroPython board connected over USB. Please make me a new app
> that does X.*

The board describes itself. Its own files include an `AGENTS.md` that tells
a coding agent what the board is, what the firmware can do and how to write
programs for it, and `help()` and `guide()` on the board document every
part of it. The agent doesn't need anything from you except the request.

The firmware was built for a students' computer club, where it is used to
write small games with graphics, touch and sound.

## Supported boards

| Board | Screen controller | |
|---|---|---|
| **ESP32-32E 240x320 2.8" Resistance Touch** (USB-C, battery connector, switchable speaker amplifier) | ILI9341 | tested |
| | ST7789 | tested |
| **ESP32-2432S028** (the classic CYD, ESP32-2432S028R) | ILI9341 | tested |
| | ST7789 | supported, not yet tested |

These boards all have a 240x320 SPI screen, XPT2046 resistive touch, a
microSD slot, a speaker output, an RGB LED, 4 MB of flash and no PSRAM.
Boards that look alike can differ inside: the LED and amplifier pins
differ between the two models, and the screen controller differs between
batches of the same model. The firmware reads the screen controller from
the screen itself on the boards tried so far, but the board model can't be
detected by software, so **the first time the board starts, it asks you**
(see below).

Other CYDs (capacitive touch, 3.5"/4.3" and larger screens, ESP32-S3
models) are **not** supported.

## What's in the download

| File | What it is |
|---|---|
| `cyd28-computerclub.bin` | The whole 4 MB flash: firmware plus an empty filesystem. Use this to install on a new board. Write it at **address 0x0**. |
| `cyd28-computerclub.fw` | The same firmware as an **upgrade file** for the SD card. It upgrades a board that already runs this firmware and keeps the board's files and settings. |
| `LICENSE` | The licence (MIT) and the licences of the parts it's built from. |

Each new release has the **same file names**, so the newest download always
has `cyd28-computerclub.bin` and `cyd28-computerclub.fw`. To tell versions
apart, look at the build date: the FIRMWARE screen (below) shows the
board's own and each `.fw` file's, and `club.firmware_info()` has it too.

## Installing: one click

> [!WARNING]
> **This erases everything on the board**: whatever firmware or Arduino
> sketch it has now, every program and file saved on it, and its touch
> calibration. The launchpad doesn't ask "are you sure?": a click or two
> after **Connect**, it is already erasing. If the board has work on it you
> want to keep, copy that to your computer first (with Thonny, or
> `mpremote cp`).

In **Chrome or Edge on a desktop computer** (these browsers can talk to
USB serial ports; Firefox and Safari can't):

1. Plug the board in with a USB **data** cable (some cables only charge).
2. Open **[the club firmware in ESP Launchpad](https://espressif.github.io/esp-launchpad/minimal-launchpad/?flashConfigURL=https://raw.githubusercontent.com/chipguyhere/CYD28-ComputerClub/main/launchpad.toml)**
   (this erases the whole board: see the warning above).
3. Click **Connect**, pick the board's port (*USB-SERIAL CH340* or
   similar), and let it flash. It **erases the board** and writes the newest
   firmware, straight from this page: nothing to download first.
4. Press **RST** on the board (or unplug it and plug it back in).

If the launchpad won't connect or doesn't load, hold **BOOT**, press and
release **RST**, let go of BOOT and try again, or use the flasher below.

## Installing: the web flasher

No software to install either: use Espressif's web flasher, in the same
browsers.

1. Unzip the download.
2. Plug the board in with a USB **data** cable (some cables only charge).
3. Open **https://espressif.github.io/esptool-js/**
4. Under *Program*, set the baud rate to **921600**, click **Connect**, and pick
   the board's port (*USB-SERIAL CH340* or similar). If it won't connect,
   try 115200, or hold **BOOT**, press and release **RST**, then let go of
   BOOT and try again.
5. Set **Flash Address** to **`0x0`**, choose `cyd28-computerclub.bin`, and
   click **Program**. It takes about a minute.
6. Press **RST** on the board (or unplug it and plug it back in).

The image replaces **everything** on the board, so anything you had on it
is gone. The parts of the flash it doesn't use are blank (0xFF), so no
separate erase is needed.

With `esptool` on the command line, it's the same address:

```
esptool --port COM3 --baud 921600 write-flash 0x0 cyd28-computerclub.bin
```

(`/dev/ttyUSB0` or similar on Linux and macOS instead of `COM3`.) On most
computers the CH340 USB-serial chip works without a driver; older Windows
versions may need the CH340 driver from the chip maker, WCH.

## The first start: three questions

A freshly flashed board asks which board it is. None of the questions
depend on the touch screen being calibrated or on knowing which way up the
screen is:

1. **The screen** -- only if the screen didn't say which controller it has.
   The picture switches between the two screen-controller settings every 3
   seconds. Touch the screen anywhere (or press BOOT) while
   the words read normally, the box is red and the white frame goes all the
   way round.
2. **The board.** The LED is lit so that it shows a different colour on each
   model: **magenta on the ESP32-32E, blue on the ESP32-2432S028**. Tap
   the MAGENTA or the BLUE button, whichever matches.
3. **The touch screen.** *Calibrate now or later?* (Later: hold BOOT for 5
   seconds on the home screen, or run *Touch Calibration* from PROGRAMS.)

The answers are saved in a small file on the board, `/board.txt` (and the
board model in the board's settings too, so deleting the file is
harmless), and the board restarts into its home screen. If you copy a
whole flash image from one board to another with a different screen, it
asks again (once). To be asked again, hold BOOT for 10 seconds on the
home screen, tap **Board:** on the FIRMWARE screen, and then **Redetect**.
(Or put `ask` in that file, `open('/board.txt', 'w').write('ask')` at the
`>>>` prompt, and press RST.) Updating the firmware from the SD card keeps
the answers.

## Vibe coding with Claude Code (or another coding agent)

All you need is the coding agent on your computer and the board plugged
in. The agent talks to the board over USB with `mpremote`, MicroPython's
command-line tool. That is a Python package (`pip install mpremote`), and
the agent normally installs it itself when it finds it missing, so the one
thing your computer does need is **Python**. Then ask for what you want:

> There's a MicroPython board connected over USB. Please create me a new
> app for it: a reaction-time game where a circle appears at a random spot
> and I tap it as fast as I can, with a beep and a best-time score.

What happens next is the point of this firmware: the agent looks at the
board, finds `/AGENTS.md`, and learns from the board itself what it is
(screen size, touch, sound, memory limits, which pins are free), how
programs are laid out and started, and the `club` helper module's API. Then
it writes the program and copies it over. New programs appear on the
board's **PROGRAMS** screen.

- **With an SD card in the board** (recommended), each new app goes into
  a folder of its own at the top of the card, with its pictures and sounds
  next to it, and appears under *SD card* in PROGRAMS.
- **Without a card**, it is saved in the board's own storage, which holds
  only **384 KB**. Sounds, images and sprites fill that quickly.

Use a **FAT32**-formatted card. exFAT isn't supported, so format larger
cards as FAT32.

Two practical tips: close any other program that has the board's serial
port open (Thonny, a serial monitor, the web flasher's tab) before the agent
connects; on Linux your user needs to be in the `dialout` group.

## What's on the board

The home screen has a **PROGRAMS** button. It lists:

- **MP3 Player**: plays `.mp3` files from the SD card through the speaker.
- **Simple Shooter**: a small arcade game with sprites and mixed sound
  effects. It's a good test that the speaker works.
- **Bouncing Balls**: an LVGL animation demo.
- **Touch Calibration**: crosshairs to tap, then filter settings.
- **My programs** (the board's own files) and **SD card** programs.

These demos are ordinary `.py` files on the board, so you (or your agent)
can read and change them. `club.restore('mp3player.py')` puts back an
original.

**Networking and Bluetooth are compiled out.** The ESP32 on these boards
has no PSRAM, and WiFi alone would take a large part of its RAM, so that
memory is left for what students use: sound and LVGL graphics, i.e. games.

## Programming it yourself

Open a serial terminal at 115200 baud (Thonny, PuTTY, `mpremote`, or
Chrome's *Serial Terminal* page), press Enter, and at the `>>>` prompt:

```python
guide()          # the board's built-in manual, as linked pages (arrow keys, Enter, q)
help()           # the same help, page by page (use this one in Thonny)
help('club')     # everything the club module can do
```

The firmware's **`club`** module bundles what you need to drive the board
by hand or from a program:

- **Screen and touch:** `club.print()`, `club.input()`, `club.button()`,
  `club.popup()`; full LVGL 9.4 underneath; `club.set_rotation()` for
  landscape.
- **Sound:** `club.play('/sd/song.mp3')`, `pause()`, `volume()`, and
  `club.effect('/sd/boom.wav')` for sound effects mixed over the music.
- **Files and SD card:** `club.copyfile()`, `club.sd_info()`;
  `club.edit('game.py')` opens a full-screen editor over the serial port.
- **Programs:** `club.programs()`, `club.run_program()`, `club.restore()`.
- **Board:** `club.led(color=0xFF8000)`, `club.battery_raw()`,
  `club.calibrate()`.
- **Firmware:** `club.firmware_export('/sd/backup.fw')` saves the running
  firmware to the SD card as an upgrade file, to give to another board;
  `club.firmware_update('/sd/cyd28-computerclub.fw')` installs one. Both keep the
  board's own files and settings.

## Upgrading from the SD card (`.fw` files)

A board that already runs this firmware upgrades without a computer:

1. Copy `cyd28-computerclub.fw` to the **top folder** of the SD card (not
   into a folder; replace an older copy with the same name) and put the
   card in the board. If the board was already on, press **RST** so it
   sees the card.
2. On the home screen (or PROGRAMS), **hold BOOT for 10 seconds**. A countdown
   appears after one second; keep holding past *"Let go now for touch
   calibration"* until the **FIRMWARE** screen opens.
3. It lists the `.fw` files on the card. Tap one, then **Install**.
4. The board checks the file and restarts into its updater, which writes it
   (the screen shows progress) and starts the new firmware. It takes about
   half a minute. **Your files and settings are kept.**

From the `>>>` prompt the same is `club.firmware_update('/sd/cyd28-computerclub.fw')`.
A damaged file, or one for different hardware, is refused before anything
is written. If the power fails during an upgrade, the updater simply starts
again at the next power-on.

## Versions

MicroPython 1.27.0, LVGL 9.4.0, ESP-IDF 5.5.1. The build date is inside
the files (the file names stay the same from release to release): the
FIRMWARE screen and `club.firmware_info()` show it.

## Licence

MIT: see `LICENSE`, which also lists the licences of the parts the firmware
is built from (all permissive: MIT, Apache 2.0, and the public-domain
minimp3 MP3 decoder).
