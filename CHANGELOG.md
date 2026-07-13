# Changelog

All notable changes to BINNACLE, Field Instrument FI-073.

## v1.1.0
- SPI reply parsing now reads the firmware's TX/RX labels, so read bytes are
  separated from the command echo exactly instead of being guessed from token order.
- Mode entry is prompt-driven: each submenu question is answered as soon as it is
  detected, rather than after a fixed delay. Failed entry is reported instead of
  silently running the command in the wrong mode.
- Auto-pair probing is adaptive: a port that identifies itself is assigned
  immediately, and only a silent or ambiguous port waits out the budget. The old
  fixed window could both stall on fast ports and misread slow ones.
- Analyzer zoom and pan: scroll to zoom at the cursor, drag to pan, Fit to reset.
- Waveform columns aggregate every sample they cover, so a single-sample glitch
  survives zoom-out instead of being decimated away.
- Standalone SUMP capture: arm and trigger without a terminal transaction, with
  selectable sample rate, sample count, and a basic trigger.
- Timing tunables collected in a TIMING block.

## v1.0.0
- One-click Auto-pair: opens each port, probes it, and assigns the terminal and
  analyzer roles automatically, in any order.
- MVP milestone.

## v0.4.0
- Port role verification with a clear warning when a port looks like the wrong role.
- Per-mode pin panel sourced from the firmware, following the active mode.
- UART transparent bridge launcher (exit by pressing the device button).

## v0.3.0
- Protocol bench matured: mode entry with menu defaults, I2C address-map scan,
  SPI JEDEC read, and a SPI transaction runner.
- VCD and CSV export of the last capture.

## v0.2.0
- FALA follow-along logic analyzer, decoded from the firmware protocol.
- Auto-follow captures with an on-demand Capture now.

## v0.1.0
- Dual-port Web Serial transport.
- VT100 subset terminal in a lit CRT bezel.
- Protocol bench and analyzer scaffolding, with a browser-runnable test harness.
