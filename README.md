# BINNACLE

Field Instrument FI-073. A single-file browser console for the Bus Pirate 5.

BINNACLE gives the Bus Pirate 5 a graphical console in the browser over Web Serial.
It drives the command-line terminal on the first USB port and, on the second port,
runs a follow-along logic analyzer that captures and draws whatever bus transaction
you just ran. The name is the brass housing that holds a ship's compass and deck
instruments, which suits a console for a device literally called the Bus Pirate.

v1.1 adds waveform zoom and pan with I2C, SPI, and UART decode overlays, standalone
SUMP capture, saved bench presets, and live pin voltages, on top of the v1.0 MVP.

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
- Read voltages: runs the firmware `v` command and shows each pin's voltage on the
  pin panel, read on demand rather than live.
- Raw command sender, with saved presets. Type a command, give it a name, and Save;
  it becomes a one-click button that persists across sessions.

Replies are scraped from text, so non-default speeds or bit widths still need the
raw command line, and the raw device reply is shown alongside parsed results.

## Logic analyzer

The analyzer reads the binmode port and offers two capture sources.

### FALA (follow along)

Set the Bus Pirate binmode to Follow along logic analyzer, connect the analyzer
port, and run any bus transaction on the terminal. The device emits a `$FALADATA`
metadata line, BINNACLE pulls the sample buffer, decodes it, and draws eight lanes.
Bit 0 of each sample is IO0 and bit 7 is IO7. With Auto-follow on (the default),
each transaction paints itself. Capture now re-requests the most recent capture.

### SUMP (standalone)

Set the Bus Pirate binmode to the SUMP logic analyzer, choose SUMP as the capture
source, then pick a sample rate, a sample count, and an optional trigger pin and
level. Arm starts the capture; the device waits for the trigger (or free-runs if
the trigger is None) and dumps when it is done. This is how you watch a target that
talks on its own, without running a terminal transaction.

The device rounds the sample count to a multiple of 4 and the rate to the nearest
100MHz/(n+1) divider, so the readout may differ slightly from what you asked for.
Only basic triggers are available: one pin, one level. The firmware does not
implement advanced triggers.

### Zoom, pan, and decode

Scroll to zoom around the cursor, drag to pan, or use the plus, minus, and Fit
buttons. The time axis labels the visible window. Each pixel column aggregates every
sample it covers rather than picking one, so a single-sample glitch still appears as
a riser when you are zoomed out. Zoom in to resolve its exact width.

Choose a decoder to annotate the waveform with what the bus actually said. I2C marks
START, the address with its read/write bit, each data byte, and ACK or NAK. SPI marks
chip select and each byte (MOSI and MISO). UART marks each byte with its ASCII
character where printable. Decoders read the standard per-mode pins (I2C: SDA on IO0,
SCL on IO1; SPI: IO4 through IO7; UART: RX on IO5). The UART decoder assumes 8N1 and
needs at least two samples per bit.

### Export

Export VCD writes a change-only Value Change Dump with a 1 ps timescale, ready for
GTKWave or PulseView. Export CSV writes one row per sample. Both act on the whole
last capture, not the zoomed view, and stay available after the analyzer disconnects.

## Configuration

Constants at the top of the script:

- `VERSION` display and build string.
- `FEEDBACK_EMAIL` the address the Feedback button opens. Set this to your own.
- `BP5_USB` a soft USB vendor filter for the port chooser. Selection stays open.
- `COLS`, `ROWS`, `FONT_PX` terminal grid geometry.
- `TIMING` probe and menu-walk budgets. Mode entry and port probing are prompt-driven
  and resolve as soon as the device answers, so these act as safety nets rather than
  fixed waits.

## URL flags

- `?test` runs the in-page self-test harness instead of the app.
- `?debug` enables console logging.

## Testing

Open `binnacle.html?test=1`. The harness asserts the VT100 emulator, the value
formatters, the FALA state machine (metadata parsing, buffer request, chronological
reversal, and delivery split across multiple serial reads), the I2C scan parser,
SPI byte extraction, the VCD and CSV generators, the port role classifier, the auto-pair role decision, the per-mode pin map, the SPI TX/RX reply parser, the
menu-prompt detection used for mode entry, the SUMP rate and count framing, and the
glitch-preserving column aggregation. A browser-runnable test harness is required
before every release.

## Versioning and snapshots

The version appears in the footer and in a build marker comment immediately after
the DOCTYPE. Keep a per-release file snapshot and archive old builds rather than
deleting them.

## Known limitations (v1.1.0)

- Web Serial only: Chrome, Edge, or Opera on desktop, over HTTPS or localhost.
- Auto-pair opens each port and probes it to assign roles. If a port cannot be
  classified, connect that role by hand.
- The terminal is a VT100 subset; some rare escape sequences are ignored.
- Mode entry is prompt-driven: each submenu question is answered with its default as
  soon as it appears. Non-default speeds or bit widths need the raw command line.
- SPI read bytes come from the firmware's TX/RX labels, so the command echo is not
  mixed into the read data. If a reply carries no labels, BINNACLE lists every 0xNN
  token and says so.
- The UART bridge is transparent and can only be exited by pressing the physical
  button on the Bus Pirate.
- SUMP rounds the sample count to a multiple of 4 and the rate to the nearest
  100MHz/(n+1) divider. Basic triggers only: one pin, one level.
- Decoders use the standard per-mode pins; custom pin assignments are not selectable
  yet. The UART decoder assumes 8N1 and needs at least two samples per bit.
- Zoomed-out columns aggregate every sample they cover, so a single-sample glitch
  still draws as a riser. Zoom in to resolve its exact width.
- Voltages are read on demand from the `v` command, not streamed live.
- VCD and CSV export the whole last capture, not the zoomed view.

## Beyond 1.1

Nothing committed. Candidates: custom decoder pin assignment, decoder annotations in
the CSV and VCD exports, exporting only the zoomed view, streaming live voltages, and
saved mode setups alongside command presets.

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
