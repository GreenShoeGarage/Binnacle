# BINNACLE v0.3.0: the bench grows teeth, and captures find the exit

Draft. Field Instrument FI-073.

The first cut of BINNACLE gave the Bus Pirate 5 a lit console in the browser: a
real terminal in a brass bezel and a follow-along logic analyzer that draws
whatever transaction you just ran. v0.2 made that analyzer real by reading the
firmware source instead of guessing at the protocol. v0.3 is about turning the
protocol bench from a convenience into something you would actually reach for, and
giving captures a way out of the browser.

The bench now enters a bus mode properly. Ask it to scan I2C and it sends the mode
command, walks the setup menus by accepting the defaults, runs the scan, and reads
the result back. The addresses that answer light up on a compact map of the whole
0x00 to 0x7F space, and when the Bus Pirate recognizes a part, the name is right
there on hover. It is a small thing, but seeing the bus as a grid of lit cells is
so much faster than reading a column of hex.

SPI got the same treatment. There is a one-tap JEDEC id read for flash chips, and a
transaction field where you type something like a read command and BINNACLE wraps
it, runs it, and lists the bytes that come back. I kept the raw device reply on
screen next to the parsed bytes, because scraping text is never perfect and I would
rather show you the truth than hide it behind a clean number.

The other half of v0.3 is export. A capture that only lives in one browser tab is a
capture you cannot really work with, so BINNACLE now writes both VCD and CSV. The
VCD is a proper change-only dump with a picosecond timescale, so it drops straight
into PulseView or GTKWave and lines up with the rest of your tools. The CSV is one
row per sample for when you just want to throw it at a script. Both stay available
even after you unplug, so you can capture, disconnect, and still get your data out.

As always this ships with a test harness. Open the page with a test flag and it
checks the terminal emulator, the analyzer state machine including data that
arrives in fragments, the I2C scan parser, the SPI byte extraction, and both export
formats down to the timing of individual edges in the VCD.

One honest note on what did not make it. I wanted a UART monitor in this release,
but the firmware monitor macro is disabled and the only path is the transparent
bridge, which takes over the port and needs careful handling to enter and exit
cleanly. That is going in v0.4 alongside telling the two serial ports apart
automatically, so you stop guessing which is which.

Make. Hack. Learn. Share. The scan grid is the part I keep opening just to look at.
EOF
