# BINNACLE v0.4.0: knowing which port is which, and what each pin does

Draft. Field Instrument FI-073.

The Bus Pirate 5 has two USB serial ports, and until now BINNACLE made you guess
which was which. One is the command line, the other is the binmode port for the
logic analyzer, and they look identical to the browser because they share a USB
id. If you connected them backwards, you got a blank screen and no explanation.

v0.4 fixes the guessing. When you connect the terminal, BINNACLE quietly sends a
carriage return and listens for a Bus Pirate prompt. If it hears one, all is well.
If it does not, it tells you plainly that this is probably the analyzer port and to
swap. The analyzer side does the mirror image: if it starts seeing terminal chatter
instead of capture data, it says so. You still pick the two ports by hand for now,
but you can no longer pick them wrong and be left in the dark. True one click
pairing is the v0.5 job.

The feature I enjoyed building most is the pin panel. Every bus mode on the Bus
Pirate assigns specific roles to specific IO pins. In I2C, IO0 is SDA and IO1 is
SCL. In SPI, the clock, data, and chip select land on IO4 through IO7. I pulled
these straight from the firmware so they are correct rather than remembered, and
the panel under the terminal now shows all eight pins with their current role,
following along as you change modes. It is the kind of small reference you did not
know you wanted until it is sitting right next to the screen.

There is also a UART bridge button. It enters UART mode and starts the transparent
bridge so you can talk to a target device directly through the terminal. One honest
detail I want to call out: the bridge can only be exited by pressing the physical
button on the Bus Pirate. There is no escape sequence from the host side, that is
just how the firmware works, so BINNACLE tells you that up front rather than letting
you wonder why it will not quit.

The test harness grew again. It now checks the port classifier against real VT100
output, prompts, and capture notices, and it checks the pin map for every mode. All
of it runs in the browser with a single flag, and it all runs green.

Make. Hack. Learn. Share. Reading the firmware to get the pin roles exactly right
was the learn this time, and the panel is the share.
