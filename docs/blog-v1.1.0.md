# BINNACLE v1.1.0: the waveform learns to talk

Draft. Field Instrument FI-073.

BINNACLE hit 1.0 as a whole console for the Bus Pirate 5: a real terminal, a
protocol bench, and a logic analyzer that follows along with whatever you just ran.
I said I would pause there. I lied, but only a little, because the four things I
had deliberately left out turned out to be the four things that make it an
instrument instead of a viewer.

The big one is decode. Eight lanes of square waves is a picture of a bus, not a
reading of it. Now you pick a decoder and the waveform annotates itself: I2C marks
the START, the address with its read or write bit, every data byte, and each ACK or
NAK. SPI marks chip select and the bytes going both directions. UART marks each byte
and shows you the character if it is printable. Suddenly you are not counting clock
edges with your finger on the screen, you are reading what the chip said.

Underneath that, zoom and pan. Long captures used to be decimated down to the plot
width, which is fine until the thing you are hunting is a glitch narrower than a
pixel. Scroll to zoom around the cursor, drag to pan, hit Fit to get back. The time
axis labels the window you are actually looking at.

Then standalone capture. Follow-along is a lovely trick, but it only fires when you
run a bus transaction, which means you cannot watch a target that talks on its own
schedule. The Bus Pirate also speaks SUMP, the same protocol PulseView uses, so now
you can set a rate, a sample count, and a trigger pin, hit Arm, and just watch. That
one needed care: I went into the firmware for the exact divider math, because SUMP
clients assume a 100 MHz base clock and the sample count gets rounded to a multiple
of four. Better to know that and say so than to quietly hand you the wrong timebase.

The last two are small and I use them constantly. Bench presets: type a command,
name it, save it, and it is a button forever. And live pin voltages, which I had
assumed would mean scraping the statusbar, until I went looking and found the
firmware has had a perfectly good `v` command all along. The pin panel now shows
what each pin does *and* what it is sitting at.

Every decoder is tested against a synthetic waveform I generate in the harness: I
build a real I2C transaction bit by bit, feed it in, and assert that the bytes come
back out. Same for SPI and UART. That is the only way I would trust a decoder I
have not yet run against a scope.

Make. Hack. Learn. Share. The firmware keeps rewarding the read. Everything good in
this release came from opening the source instead of guessing.
