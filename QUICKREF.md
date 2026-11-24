# UV-K5 / UV-K6 / UV-5R — Egzumer v2.1.27 Quick Reference (UK)

**Companion to the [complete manual](UV-K5_Egzumer_v2.1.27_Complete_UK_Manual.md).** This sheet is a compact operator aid, not a substitute for the full instructions or current UK regulatory information.

**Build scope:** Egzumer source defaults audited for this project: Spectrum, FM radio, DTMF calling, AM Fix, VOX and audio bar enabled; NOAA, AirCopy, voice prompts, alarm and 1750 Hz TX disabled. Optional builds and hardware revisions can differ.

## Main-screen keys

| Key | Action |
|---|---|
| `F + 0` | Enter/leave FM broadcast radio |
| `F + 1` | Change VFO band; MR mode can copy channel to VFO |
| `F + 2` | Switch VFO A/B |
| `F + 3` | Toggle VFO/MR |
| `F + 4` | Frequency / CTCSS-DCS search screen |
| `F + 5` | Open swept-RSSI Spectrum analyser |
| `F + 6` | Cycle TX power Low/Mid/High |
| `F + 7` | Toggle VOX |
| `F + 8` | Reverse RX/TX frequency roles |
| `F + 9` | Recall configured 1-Call channel |
| Long `F` | Keypad lock/unlock |
| Short `*` | DTMF input |
| `F + *` | CTCSS/DCS tone scan |
| Long `*` | Start/stop channel or VFO scan |
| Long `5` | VFO: toggle `ScnRng`; MR: change scan-list membership — **not** Spectrum |
| Side1 / Side2 | User-programmable short/long actions |

During a scan, PTT or MENU stops and keeps the found channel; EXIT stops and returns to the starting frequency.

## F+5 Spectrum analyser

| Key | Action |
|---|---|
| `1` / `7` | Increase/decrease spectrum measurement interval |
| `2` / `8` | Increase/decrease UP/DOWN movement distance |
| `3` / `9` | Raise/lower top of plotted dB scale |
| `4` | Cycle 64 → 128 → 16 → 32 → 64 measurement points |
| `5` | Enter frequency |
| `0` | Cycle FM → AM → USB |
| `6` | Cycle listen filter Wide 25k → 12.5k → 6.25k |
| `*` / `F` | Raise/lower RSSI listening trigger |
| UP / DOWN | Shift the sweep frequency range |
| Side1 | Ignore/blacklist current peak for the scan session |
| Side2 | Toggle analyser backlight |
| PTT | Tune to tracked peak and enter STILL (receive; does not TX) |
| MENU | No action in normal SPECTRUM state |
| EXIT | Leave Spectrum and restore previous radio state |

### Frequency-entry rule — important

The value entered with key `5` is **not always the centre**:

- Scan step **below 2.5 kHz**: centre mode; entered frequency is the centre.
- Scan step **2.5 kHz or above**: lower-edge mode; entered frequency is the sweep start.
- `ScnRng` active: VFO A/B endpoints define the scan; frequency entry and point-count controls are disabled.

### Exact 16-bin PMR446 analogue view

1. Open `F+5`.
2. Press `4` until **16x**.
3. Use `1`/`7` to select **12.50 kHz**.
4. Press `5`, enter **`446*00625`**, then MENU.
5. Set `0` to FM and `6` to 12.5 kHz listen bandwidth; adjust the trigger with `F`/`*`.

At 12.5 kHz the input is the **lower edge**, so this places the 16 measurement points on PMR446 Channels 1–16 (446.00625–446.19375 MHz). **Do not enter `446*10625`** for this setup: that starts at Channel 9 and loses Channels 1–8. This is a sequential swept measurement, not 16 simultaneous receivers.

## PMR446 — receive setup

For analogue PMR446 reception in the normal radio screen:

- VFO mode; select the 400 MHz band group.
- Set VFO Step to 12.5 kHz.
- Select FM and Narrow (12.5 kHz) bandwidth.
- Set RxCTCS and RxDCS to OFF to hear users regardless of tone/code.
- Start with low squelch; store channels in memory if useful.

| Ch | MHz | Ch | MHz |
|---:|---:|---:|---:|
| 1 | 446.00625 | 9 | 446.10625 |
| 2 | 446.01875 | 10 | 446.11875 |
| 3 | 446.03125 | 11 | 446.13125 |
| 4 | 446.04375 | 12 | 446.14375 |
| 5 | 446.05625 | 13 | 446.15625 |
| 6 | 446.06875 | 14 | 446.16875 |
| 7 | 446.08125 | 15 | 446.18125 |
| 8 | 446.09375 | 16 | 446.19375 |

**Transmit restriction:** The UV-K5 is not compliant PMR446 transmitting equipment. Programming 446 MHz does not make it type-compliant. Use an approved PMR446 handheld for transmission; see section 1.1 of the full manual.

## UK frequency starting points

| Service / activity | Frequency / range | Receive mode / note |
|---|---:|---|
| UK CB CEPT channels | 26.965–27.405 MHz | Mode and channel plan matter; use compliant CB radio for TX |
| UK CB extension | 27.60125–27.99125 MHz | Separate UK 40-channel band |
| 6 m amateur FM calling | 51.510 MHz | FM |
| 4 m amateur calling | 70.260 MHz | AM/FM calling; 70.450 MHz is commonly used FM calling |
| Civil airband | 118–137 MHz | AM; 8.33 kHz channel spacing |
| 2 m amateur FM calling | 145.500 MHz | FM |
| 2 m repeater outputs | 145.600–145.7875 MHz | Typical handheld TX offset −0.600 MHz |
| Maritime VHF Ch 16 | 156.800 MHz | FM; distress/safety/calling channel, not for radio checks |
| 70 cm amateur FM calling | 433.500 MHz | FM |
| 70 cm repeater outputs | 432.994–433.381 MHz | Typical handheld TX offset +1.600 MHz |
| 70 cm repeater outputs | 430.825–430.975 MHz | Typical handheld TX offset +7.600 MHz |
| ISS cross-band repeater | 437.800 MHz downlink | 145.990 MHz uplink, 67 Hz tone; check current ARISS status |
| Historic NOAA APT | 137.100 / 137.620 / 137.9125 MHz | NOAA-19/15/18 legacy frequencies; satellites retired in 2025 |

Use the current [RSGB band plans](https://rsgb.org/main/operating/band-plans/), [Ofcom UK Frequency Allocation Table](https://www.ofcom.org.uk/spectrum/frequencies/uk-fat) and applicable licence terms for current allocations and operating conditions. The UV-K5 does not decode DAB, DVB-T/T2, DMR/dPMR, TETRA, P25, LTE/5G, ADS-B or GNSS.

## Startup modes

| Hold while switching on | Mode | Notes |
|---|---|---|
| PTT + Power | Factory bootloader flash mode on applicable original hardware | Typically blank display and steady top LED; hardware revisions differ |
| PTT + Side1 | Egzumer hidden F Lock menu | Checked by firmware `helper/boot.c` |
| PTT + Side2 | AirCopy | Only when `ENABLE_AIRCOPY=1`; disabled in audited default build |

## DTMF and hidden-menu reminders

- `ANI ID`, `UPCode` and `DWCode` display values but are read-only in the front-panel menu on this build. Use a compatible PC programming method where supported.
- The hidden F Lock menu is a firmware filter, not a legal permission certificate. For example, GB HAM allows 144–148 MHz in source code, wider than the UK 2 m allocation of 144–146 MHz.
- Save/backup configuration before changing calibration or using Reset.

## Source references

- [Egzumer firmware repository](https://github.com/egzumer/uv-k5-firmware-custom)
- [Full manual](UV-K5_Egzumer_v2.1.27_Complete_UK_Manual.md)
- [Ofcom UK Frequency Allocation Table](https://www.ofcom.org.uk/spectrum/frequencies/uk-fat)
- [Ofcom amateur licence terms](https://www.ofcom.org.uk/__data/assets/pdf_file/0027/62991/amateur-terms.pdf)
- [RSGB band plans](https://rsgb.org/main/operating/band-plans/)
- [ARISS station status](https://www.ariss.org/current-status-of-iss-stations.html)
