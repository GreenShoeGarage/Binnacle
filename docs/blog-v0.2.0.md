# BINNACLE: a lit deck console for the Bus Pirate 5

Draft. Field Instrument FI-073, v0.2.0.

The Bus Pirate 5 is a joy to talk to over a serial terminal, but a terminal is a
narrow window. You type a command, you read some text, you squint at a statusbar.
I wanted to sit the thing inside a proper console, something that feels like a
piece of bench equipment rather than a shell prompt. So I built BINNACLE.

A binnacle is the brass housing that holds a ship's compass and the instruments on
deck. The name fit twice over. The device is literally called a Bus Pirate, and the
look I was after is the one I keep coming back to: vintage instruments, engraved
plates, a lamp glowing under a hood. BINNACLE runs entirely in the browser, one
HTML file, no build step, night theme by default, and it drives the Bus Pirate over
the Web Serial API.

The part I am most pleased with is that the Bus Pirate 5 has two USB serial ports,
not one, and BINNACLE uses both. The first port is the command line, rendered in a
lit CRT bezel with a real VT100 emulator so the color output and the live statusbar
draw the way they do in a good terminal. The second port is where the follow-along
logic analyzer lives.

Here is the loop that makes it worth building. You run a transaction on the command
line, an I2C write, a SPI read, whatever you like. The Bus Pirate captures it and
sends a small notice on the second port. BINNACLE hears that notice, pulls the
sample buffer, and paints eight channels of the exact transaction you just ran.
Terminal and scope become one instrument. You do not arm a trigger or juggle a
second tool. You just work, and the waveform follows along.

Getting the analyzer right meant reading the firmware source rather than guessing.
The follow-along protocol is a short ASCII notice followed by a raw byte per
sample, newest sample first, one bit per IO pin. Once I had that from the source,
the decoder was small and, more to the point, correct. It ships with a
browser-runnable test harness that checks the terminal emulator, the formatters,
and the analyzer state machine, including the case where the serial data arrives
split across several reads, which is how it always arrives in real life.

v0.2.0 is an honest milestone, not a finished tool. The protocol bench is a thin
convenience over the command line for now. Long captures get decimated to fit the
screen, so zoom and pan are on the list, along with VCD and CSV export so captures
can move into PulseView. Port auto-identification is coming too, since right now you
pick the two ports by hand.

Make. Hack. Learn. Share. This one was a good hack, and reading someone else's
firmware to get the details right was the learn.
