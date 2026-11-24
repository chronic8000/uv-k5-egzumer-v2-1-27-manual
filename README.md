# Quansheng UV-K5 / UV-K6 / UV-5R — Egzumer v2.1.27 Manual

A comprehensive, source-checked operating manual for the Egzumer custom firmware family for the Quansheng UV-K5, UV-K6 and UV-5R handheld radios, with a UK-oriented focus.

## Main manual

**[Read the complete manual](UV-K5_Egzumer_v2.1.27_Complete_UK_Manual.md)**  
**[Open the printable quick reference](QUICKREF.md)**

The guide covers:

- Front-panel keys, side keys, short presses, long presses and F-key combinations
- Main display indicators, VFO/MR operation, frequency entry and memory channels
- The F+5 swept-RSSI spectrum analyser, including every control in sweep and STILL states
- The complete standard and hidden menu references
- Frequency and channel scanning, CTCSS/DCS detection, FM broadcast radio and DTMF
- Analogue PMR446 reception using both the normal radio screen and the spectrum analyser
- UK frequency allocations, receiver limitations and practical receive-only setup
- Troubleshooting and source references

## Firmware scope

The manual is written for the **Egzumer v2.1.27 firmware family**, using the default build configuration checked against the upstream source. Optional compile-time features, later forks, hardware revisions and EEPROM settings can change what appears on a particular radio. The manual identifies the build assumptions and advises checking the actual menu labels on the device.

Upstream firmware source: [egzumer/uv-k5-firmware-custom](https://github.com/egzumer/uv-k5-firmware-custom)

## Important notes

- This is an independent documentation project. It is not the firmware itself and does not distribute firmware binaries.
- It is not an official Quansheng or Egzumer publication and is not affiliated with either.
- The UV-K5 should not be treated as compliant PMR446 transmitting equipment merely because it can be programmed to PMR446 frequencies. The PMR446 sections explain reception and the relevant UK restrictions.
- Always check current Ofcom guidance and the terms applicable to your own licence and equipment before transmitting.

## Corrections and contributions

Please open an issue with the radio model, exact firmware build/version, relevant compile-time options (where known), and the menu label or key sequence involved. Clear reproduction steps help distinguish a firmware difference from a documentation error.

## Licence

No separate licence is granted for this manual in this repository. The upstream firmware remains subject to the licence and notices in its own repository.
