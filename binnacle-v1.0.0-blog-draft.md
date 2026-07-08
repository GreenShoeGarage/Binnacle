# BINNACLE v1.0.0: the console is whole

Draft. Field Instrument FI-073.

A few weeks ago BINNACLE was a sketch: give the Bus Pirate 5 a lit console in the
browser instead of a bare terminal. Today it reaches 1.0, which for me means the
minimum viable version is complete and it is time to stop and use it for a while
before adding more. Here is what a whole console turned out to mean.

The heart of it is still the two-port trick. The Bus Pirate has a command line on
one USB port and a logic analyzer on the other, and BINNACLE uses both at once. You
type on the terminal, and the follow-along analyzer draws the exact transaction you
just ran. That loop is the reason the tool exists, and it has worked since v0.2 when
I stopped guessing at the analyzer protocol and read it out of the firmware.

Since then the bench grew teeth: an I2C scan that lights the responding addresses on
a map, a SPI JEDEC read and a transaction runner, and captures that export to VCD
and CSV so they drop straight into PulseView. A pin panel under the screen shows
what each of the eight IO pins does in the current mode, pulled from the firmware so
it is right rather than remembered. And a UART bridge, with the honest note that you
exit it by pressing the button on the device, because that is the only way out.

The last piece, and the one that makes 1.0 feel finished, is auto-pairing. Until now
you had to know which of the two identical-looking ports was the terminal and which
was the analyzer. Now you click Auto-pair, pick a port, and BINNACLE opens it, sends
a small probe, and figures out which role it should play by how it answers. Pick the
second port and it takes the other role. The order does not matter. It is the kind of
small thing that removes a paper cut you stopped noticing you had.

All of it still lives in one HTML file with no build step, runs from any HTTPS host
or localhost, defaults to a night theme, and carries its own browser-runnable test
harness. That harness now checks the terminal emulator, the analyzer state machine
including fragmented serial reads, the bench parsers, both export formats, the port
classifier, and the auto-pair decision in both connection orders. Green across the
board.

So 1.0 is a stopping point on purpose. There is plenty I could still add, standalone
logic captures, waveform zoom with protocol decoding, live pin voltages, saved bench
presets, but a tool you actually use teaches you what it really needs, and I would
rather learn that on the bench than guess at it now.

Make. Hack. Learn. Share. This one was a good long hack, and reading the firmware to
get every detail honest was the through-line. The binnacle is lit. Time to sail.
