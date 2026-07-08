# BINNACLE

Field Instrument FI-073. A single-file browser console for the Bus Pirate 5.

BINNACLE gives the Bus Pirate 5 a graphical console in the browser over Web Serial.
It drives the command-line terminal on the first USB port and, on the second port,
runs a follow-along logic analyzer that captures and draws whatever bus transaction
you just ran. The name is the brass housing that holds a ship's compass and deck
instruments, which suits a console for a device literally called the Bus Pirate.

v1.0 is the MVP milestone: terminal, protocol bench, follow-along analyzer with
export, per-mode pin labels, and one-click port pairing, all in a single file.

## Requirements

- A Chromium browser (Chrome, Edge, or Opera) on desktop. Firefox and Safari do
  not implement the Web Serial API.
- The page must be served over HTTPS or from localhost. Web Serial is blocked on
  plain HTTP and inside sandboxed preview frames.
- A Bus Pirate 5 on current firmware.

## Running it

BINNACLE is one HTML file with no build step and no runtime dependencies. Serve
`binnacle.html` from any static host over HTTPS (for example Cloudflare Pages or
the mbparks.com static stack), or from a local server such as
`python3 -m http.server` opened at `http://localhost:8000/binnacle.html`.

## Connecting the two ports

The Bus Pirate 5 presents two USB serial ports: the VT100 command line and the
binmode port. They share a USB id, so the browser cannot tell them apart on its own.

Auto-pair (recommended) handles this for you. Click Auto-pair ports, choose the
first Bus Pirate port when the browser asks, and BINNACLE opens it, sends a probe,
and assigns it to the terminal or analyzer role based on how it answers. Click again
for the second port and it takes the remaining role. The order you pick them in does
not matter.

You can also connect each role by hand with the Connect terminal and Connect
analyzer buttons. When you connect the terminal by hand, BINNACLE still checks the
port and warns if it looks like the analyzer port instead.

## Terminal

The terminal pane is a VT100 subset rendered to a canvas, so the Bus Pirate color
output and the live statusbar draw correctly instead of leaking escape codes.
Click the screen to focus it, then type. Enter, Backspace, Tab, Escape, the arrow
keys, Ctrl combinations, and paste are all forwarded to the device.

On connect BINNACLE sends a carriage return and checks for a Bus Pirate prompt. If
it does not see one, it warns that you may have selected the analyzer port instead,
so you can swap without guessing. A pin panel below the screen shows the fixed
per-mode pin labels (for example SDA and SCL on IO0 and IO1 in I2C), and it follows
the active mode as you change it.

## Protocol bench

The bench enters a bus mode by sending `m <mode>` and accepting each submenu
default with Enter until the mode prompt returns, then runs a command and parses
the reply. v0.3 ships:

- Device info (`i`).
- Scan I2C bus: runs `scan`, parses the responder lines, and lights the answering
  7-bit addresses on a 0x00 to 0x7F map. Hover a lit cell to see the device name
  when the Bus Pirate recognizes it.
- SPI JEDEC id: reads three bytes with `[0x9f r:3]` and reports manufacturer, type,
  and capacity.
- SPI transaction: type a transaction such as `0x03 0x00 0x00 0x00 r:16` and
  BINNACLE wraps it in `[ ]`, runs it, and lists the returned bytes.
- UART bridge: enters UART mode and starts the transparent bridge. The bridge can
  only be exited by pressing the physical button on the Bus Pirate; there is no
  host side escape, so BINNACLE says so plainly.
- Raw command sender.

Replies are scraped from text, so non-default speeds or bit widths still need the
raw command line, and the raw device reply is shown alongside parsed results.

## Logic analyzer (FALA)

The analyzer speaks the Bus Pirate follow-along logic analyzer protocol (FALA) on
the binmode port. Set it up once:

1. On the terminal, type `binmode` and choose Follow along logic analyzer.
2. Connect the analyzer port in BINNACLE.
3. Run any bus transaction on the terminal.

After each transaction the Bus Pirate emits a `$FALADATA` metadata line on the
binmode port. With Auto-follow on (the default), BINNACLE requests the sample
buffer, decodes it, and draws eight lanes. Bit 0 of each sample is IO0 and bit 7
is IO7. Samples arrive newest first and are shown oldest to newest. Capture now
re-requests the most recent capture on demand.

### Export

Export VCD writes a change-only Value Change Dump with a 1 ps timescale, ready for
GTKWave or PulseView. Export CSV writes one row per sample with a time column and
one column per channel. Both act on the last capture and stay available after the
analyzer disconnects.

## Configuration

Constants at the top of the script:

- `VERSION` display and build string.
- `FEEDBACK_EMAIL` the address the Feedback button opens. Set this to your own.
- `BP5_USB` a soft USB vendor filter for the port chooser. Selection stays open.
- `COLS`, `ROWS`, `FONT_PX` terminal grid geometry.

## URL flags

- `?test` runs the in-page self-test harness instead of the app.
- `?debug` enables console logging.

## Testing

Open `binnacle.html?test=1`. The harness asserts the VT100 emulator, the value
formatters, the FALA state machine (metadata parsing, buffer request, chronological
reversal, and delivery split across multiple serial reads), the I2C scan parser,
SPI byte extraction, the VCD and CSV generators, the port role classifier, the auto-pair role decision, and the per-mode pin map. A browser-runnable test
harness is required before every release.

## Versioning and snapshots

The version appears in the footer and in a build marker comment immediately after
the DOCTYPE. Keep a per-release file snapshot and archive old builds rather than
deleting them.

## Known limitations (v1.0.0)

- Web Serial only: Chrome, Edge, or Opera on desktop, over HTTPS or localhost.
- Auto-pair opens each port and probes it to assign roles. If a port cannot be
  classified, connect that role by hand.
- The terminal is a VT100 subset; some rare escape sequences are ignored.
- The pin panel shows the fixed per-mode labels from the firmware. Live pin voltages
  stay on the device LCD and statusbar.
- Bench macros accept menu defaults; non-default speeds or bits need the raw line.
- SPI byte parsing extracts every 0xNN token, so the command echo can appear with
  read data. The raw reply is shown too.
- The UART bridge is transparent and can only be exited by pressing the physical
  button on the Bus Pirate.
- The analyzer uses FALA. Set the Bus Pirate binmode to Follow along logic analyzer
  first, then run a transaction on the terminal.
- VCD and CSV export the last capture; very long captures make large files.
- Standalone SUMP capture is out of scope for the 1.0 MVP; use PulseView for that.

## Beyond 1.0

v1.0 is the MVP and a deliberate pause. Candidate future work, none committed:
standalone SUMP capture, analyzer zoom and pan with protocol decode overlays, live
per-mode pin voltages, and saved bench macro presets.

## License

BINNACLE, a browser console for the Bus Pirate 5.
Copyright (C) 2026 M. B. Parks / Green Shoe Garage.

This program is free software: you can redistribute it and/or modify it under the
terms of the GNU General Public License as published by the Free Software
Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY
WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this
program. If not, see the file COPYING distributed with this program.
