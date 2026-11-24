# Quansheng UV-K5 / UV-K6 / UV-5R
## Egzumer Custom Firmware v2.1.27 — Complete User Manual (UK Edition)

**A source-led operating guide to the keypad, display, menus, scanning, spectrum analyser, PMR446 reception and UK spectrum.**

> **Firmware scope.** This guide is written against the `egzumer/uv-k5-firmware-custom` repository and its documented v2.1.27 firmware family. It follows the repository's checked-in `Makefile` defaults. Different builds, forks, hardware revisions and EEPROM contents can change the menus or behaviour. Check the build options in your own binary before assuming that every optional item below is present.
>
> **Important terminology.** The feature entered with **F + 5** is called “Spectrum” in the source. It is a swept RSSI spectrum analyser with an automatic listening state and a manually tuned “STILL” state. It is not a conventional FFT display or a scrolling time-history waterfall. This manual uses “spectrum screen” and “waterfall” interchangeably where that is what users commonly call it.
>
> **Menu numbering.** Where this guide gives a menu number, it refers to the repository default build configuration audited for this manual: spectrum, DTMF calling, AM Fix, VOX and the audio bar enabled; NOAA, voice prompts, alarm, 1750 Hz transmit and crystal-calibration menu disabled. Optional build flags can insert or remove menu items and shift later numbers. The on-radio menu label is the reliable identifier.

---

## Contents

1. [Read this first: safety, legality and limitations](#1-read-this-first-safety-legality-and-limitations)
2. [What this build contains](#2-what-this-build-contains)
3. [Keypad and physical controls](#3-keypad-and-physical-controls)
4. [Normal radio screen and operating modes](#4-normal-radio-screen-and-operating-modes)
5. [Entering frequencies and using memory channels](#5-entering-frequencies-and-using-memory-channels)
6. [The F+5 spectrum analyser, key by key](#6-the-f5-spectrum-analyser-key-by-key)
7. [The complete menu reference](#7-the-complete-menu-reference)
8. [Hidden transmit and calibration menu](#8-hidden-transmit-and-calibration-menu)
9. [Scanning, signal search and CTCSS/DCS detection](#9-scanning-signal-search-and-ctcssdcs-detection)
10. [PMR446 reception — normal screen and spectrum screen](#10-pmr446-reception--normal-screen-and-spectrum-screen)
11. [FM broadcast radio](#11-fm-broadcast-radio)
12. [DTMF functions](#12-dtmf-functions)
13. [UK band guide: what the radio can and cannot receive](#13-uk-band-guide-what-the-radio-can-and-cannot-receive)
14. [Recommended receive-only setup](#14-recommended-receive-only-setup)
15. [Practical troubleshooting](#15-practical-troubleshooting)
16. [Source map and authoritative references](#16-source-map-and-authoritative-references)

---

# 1. Read this first: safety, legality and limitations

## 1.1 A transceiver is not automatically legal to transmit everywhere it can tune

The UV-K5 is a transceiver: it can receive and transmit. The fact that the firmware allows a frequency to be entered, or that a hidden menu offers an “unlock” option, does **not** mean that transmission there is authorised.

In the UK, transmitting generally requires an Ofcom licence or a specific licence exemption. The exemption is equipment- and condition-specific. A frequency being used by a licence-exempt service does not make every radio capable of transmitting there licence-exempt.

### PMR446 is a particularly important example

UK PMR446 equipment operates in **446.0–446.2 MHz**, with a maximum of **500 mW ERP**. Ofcom describes it as handheld/mobile-only equipment; base stations are not permitted, airborne use is not permitted, and the service is on a non-interference/non-protected basis. Ofcom also stresses that a device is licence-exempt only when it is built and operated within the exemption conditions. The harmonised requirements include an integral antenna and no more than 500 mW ERP.

The UV-K5 has a detachable antenna and is a programmable multi-band transceiver. It is **not a compliant PMR446 handheld simply because you programme it to a PMR446 channel**. Do not transmit on PMR446 with the UV-K5. Use a compliant PMR446 radio for PMR446 transmission.

For a receive-only UV-K5 setup, use the hidden **F Lock → DISABLE ALL** option described in section 8. This is a useful firmware transmit inhibit, but it is not a substitute for understanding the law or for correctly operating the equipment.

## 1.2 Receiving private radio traffic is not the same as listening to broadcast radio

Ofcom says ordinary radio receivers are generally exempt from a receive licence, and describes general reception as including licensed broadcasting and hobby radio. It also warns that intentionally intercepting communications when you are not an intended recipient may be a criminal offence. Section 48 of the Wireless Telegraphy Act 2006 covers interception and disclosure of messages.

Practical rule: use the radio for broadcast services, amateur-radio hobby reception and communications for which you are an intended recipient. Do not deliberately eavesdrop on private business, personal, emergency-service or other non-public communications, and do not repeat or publish information heard from unauthorised interception. Ofcom recommends obtaining legal advice where the position is uncertain.

This is why the band guide later in this manual distinguishes **technically tunable** from **appropriate general reception**. A frequency appearing on the display is not permission to listen to the traffic carried on it.

## 1.3 The receiver has substantial hardware limitations

The firmware README is unusually candid about this, and it matters:

- The receiver front end has no proper track-tuned band-pass filtering across the whole tuning range.
- Strong nearby transmitters can overload the receiver and cause false signals, distorted audio or poor reception.
- AM is especially vulnerable to overload and distortion. The AM Fix can improve matters, but it cannot overcome the limits of the analogue hardware.
- The spectrum screen is a quick survey tool, not calibrated test equipment. RSSI and dBm readings are approximate and should not be treated as laboratory measurements.
- The firmware can tune some frequencies for which the radio has no suitable digital decoder. It does not turn the UV-K5 into a DAB, DMR, TETRA, LTE, DVB-T, ADS-B or satellite-data receiver.

A better antenna can improve reception on its intended band, but it can also make front-end overload worse in a strong-signal location. More antenna is not automatically better.

## 1.4 Hardware coverage in this build

The default build has `ENABLE_WIDE_RX=1`. The frequency tables therefore permit the following broad groups:

| Firmware band | Nominal tuning span | Notes |
|---|---:|---|
| 50 MHz band group | 18–108 MHz | Extended by `ENABLE_WIDE_RX`; not a dedicated HF receiver |
| 108 MHz group | 108–137 MHz | Useful for AM reception in the aviation band, subject to the legal warning above |
| 137 MHz group | 137–174 MHz | VHF services and amateur allocations |
| 174 MHz group | 174–350 MHz | Includes digital broadcast allocations the radio cannot decode |
| 350 MHz group | 350–400 MHz | Receiver only; performance varies greatly by frequency |
| 400 MHz group | 400–470 MHz | Includes amateur, SRD and PMR446 frequencies |
| 470 MHz group | 470–630 MHz and 840–1300 MHz | There is a hardware tuning gap from 630 to 840 MHz |

The BK4819 receive ranges used by the source are approximately **18–630 MHz** and **840–1300 MHz**. The **630–840 MHz gap is real**: the firmware rejects frequencies in it. The extended band table should not be mistaken for a guarantee of good sensitivity across every frequency.

---

# 2. What this build contains

The repository's default `Makefile` enables the following principal user features:

| Feature | Default | What it means to the operator |
|---|---:|---|
| UART programming | On | PC configuration/programming interface is compiled in |
| FM broadcast radio | On | Dedicated broadcast-FM receiver mode is available |
| NOAA weather channels | Off | The US NOAA channel shortcut is not present in this build |
| Voice prompts | Off | Menu voice announcements are not compiled in |
| VOX | On | Voice-operated transmit control and the VOX side-key action are available |
| DTMF calling | On | DTMF encoding/decoding and related menu items are available |
| Flashlight | On | Top LED flashlight action is available |
| Wide receive | On | Extended VFO tuning ranges are enabled |
| AM Fix | On | Dynamic AM front-end gain adjustment is available |
| More-sensitive squelch | On | The custom squelch thresholds are used |
| Faster channel scan | On | Faster scanning, with a more twitchy squelch response |
| RSSI bar | On | Signal indication uses the custom RSSI bar rather than only antenna symbols |
| Microphone audio bar | On | The transmit audio bar and its menu option are available |
| Copy memory channel to VFO | On | F+1 can copy a memory channel's settings into frequency mode |
| Spectrum analyser | On | F+5 opens the spectrum screen |
| Scan ranges | On | The range-scan additions to the spectrum/scan code are compiled in |
| AirCopy | Off | The AirCopy transfer screen is not present |
| Alarm / 1750 Hz TX | Off | Those optional transmit features are not compiled in |
| BYP/RAW demodulators | Off | Only FM, AM and USB are selectable |
| Frequency calibration menu | Off | FrCali is not compiled in |
| Temporary BLMin off | Off | That optional side-key action is absent |

There are other build options in the Makefile, including display layout, large frequency font, memory-name retention and low/mid TX power scaling. The exact menu is generated conditionally: a feature disabled at compile time will not appear as a menu item.

**Default modulation choices:** FM, AM and USB. LSB is not listed as a selectable mode. BYP and RAW only appear in a build with `ENABLE_BYP_RAW_DEMODULATORS=1`.

---

# 3. Keypad and physical controls

## 3.1 The main keypad

| Key | Normal role |
|---|---|
| `0`–`9` | Frequency entry in VFO mode; channel-number entry in memory mode; numeric menu selection |
| `MENU` | Open the menu; enter a menu item; confirm/save a setting |
| `UP` / `DOWN` | Step frequency or memory channel; move through menus; change settings |
| `EXIT` | Backspace during entry; leave a screen; cancel/back out of a menu; stop a scan |
| `* SCAN` | Short press enters DTMF input; long press starts/stops the channel/frequency scan |
| `F` | Function-key modifier; short press arms the F function; long press toggles keypad lock |
| `PTT` | Transmit in normal radio operation; special behaviour in the spectrum analyser (see section 6) |

The side buttons are called **Side1** and **Side2** in the source. Side1 is the upper side key used by the hidden-menu boot shortcut. The actions assigned to the side keys are configurable.

## 3.2 How F combinations work

Tap `F` once, then press the required number key. The F indicator appears in the status area while the modifier is armed. Do not hold F and the number together unless you deliberately want the long-press behaviour.

The default-build F combinations are:

| Combination | Action | Notes |
|---|---|---|
| **F + 0** | FM broadcast radio | Enters or leaves the dedicated FM receiver |
| **F + 1** | Band change in VFO mode | In memory mode, the enabled copy-to-VFO feature can copy the channel settings to frequency mode |
| **F + 2** | Switch A/B VFO | Changes the selected VFO |
| **F + 3** | Switch VFO / MR mode | Frequency mode ↔ memory-channel mode |
| **F + 4** | Frequency search | Starts the automatic frequency/CSS scanner screen |
| **F + 5** | Spectrum analyser | This is the spectrum screen described in section 6; with NOAA compiled instead, this key is used for NOAA channel handling |
| **F + 6** | Cycle TX power | Low → Mid → High → Low |
| **F + 7** | VOX toggle | VOX is compiled in by default; in builds without VOX this key toggles scan-list participation |
| **F + 8** | Frequency reverse | Swaps the RX/TX frequency roles, useful for checking repeater input/output relationships |
| **F + 9** | Recall the configured “1 Call” channel | Works only if the configured channel is valid |

A long press of `F` toggles the keypad lock. For most numeric keys, a long press invokes the corresponding F action without first tapping `F`. **Key `5` is the important exception:** its long-press action is `ScnRng` in VFO mode or scan-list membership in MR mode; it does not launch Spectrum. The table below shows this distinction.

### Main keypad: short press, F shortcut and long press

| Control | Short press | F + control | Long press / special hold |
|---|---|---|---|
| `MENU` | Open menu / accept or save | — | Run the action assigned to `M Long` |
| `EXIT` | Back out; backspace in numeric entry | — | Clear the active input or cancel the active range/input state where supported |
| `UP` / `DOWN` | Tune, change channel or menu value | Raise / lower squelch directly | Fast continuous movement while held |
| `1 BAND` | Enter digit 1 | Change VFO band group; in MR mode, copy the current channel to VFO when that feature is enabled | Same band/copy action as F+1 |
| `2 A/B` | Enter digit 2 | Switch active VFO A/B | Same as F+2 |
| `3 VFO/MR` | Enter digit 3 | Toggle VFO / MR mode | Same as F+3 |
| `4 FC` | Enter digit 4 | Open the frequency-search / CTCSS-DCS scanner | Same as F+4 |
| `5 NOAA` | Enter digit 5 | Open Spectrum in the default build; NOAA handling in a NOAA build | In VFO mode toggle `ScnRng`; in MR mode change the channel's scan-list membership |
| `6 H/M/L` | Enter digit 6 | Cycle TX power Low / Mid / High | Same as F+6 |
| `7 VOX` | Enter digit 7 | Toggle VOX in the default build | Same as F+7 |
| `8 R` | Enter digit 8 | Reverse RX/TX frequency roles | Same as F+8 |
| `9 CALL` | Enter digit 9 | Recall the configured 1-Call channel | Same as F+9 |
| `0 FM` | Enter digit 0 | Toggle FM broadcast radio | Same as F+0 |
| `* SCAN` | Open DTMF input in normal radio operation | Start CTCSS/DCS tone detection | Start / stop channel or VFO frequency scan; during MR scan, a further long press cycles scan-list selection |
| `F` | Arm the F modifier | — | Toggle keypad lock |
| `PTT` | Transmit in normal radio operation | — | `PTT` + digit sends live DTMF when configured; `PTT` + Side2 can send a 1750 Hz tone burst only in builds compiled with that feature |
| Side1 (upper) | Assigned `F1Shrt` action | — | Assigned `F1Long` action |
| Side2 (lower) | Assigned `F2Shrt` action | — | Assigned `F2Long` action |

The keypad legend `NOAA` on the `5` key does **not** mean NOAA weather reception is active in this default build. Here, `F+5` opens the spectrum analyser.

## 3.3 Side-key actions

The side keys have separate short-press and long-press assignments. The long-press action is triggered by holding the key; it is not the same action as a short tap. If the EEPROM has no valid saved assignments, the firmware fallback defaults are Side1 short = Monitor, Side1 long = None, Side2 short = Scan, Side2 long = None and MENU long = None. Existing saved settings can differ.

The function choices available in this build are:

- None
- Flashlight (compiled in)
- Power — cycle Low/Mid/High
- Monitor — open squelch while held/active
- Scan — start/restart scanning
- VOX (compiled in)
- FM Radio (compiled in by default)
- Keypad lock
- Switch VFO A/B
- Switch VFO/MR mode
- Switch demodulator (FM → AM → USB → FM)
- Spectrum (compiled in)

Alarm and 1750 Hz may also appear in builds where their corresponding options are enabled. `BLMin temporary off` only appears in builds that enable that custom option.

Configure the assignments in the menu items `F1Shrt`, `F1Long`, `F2Shrt`, `F2Long` and `M Long`.

## 3.4 Main-screen keypad shortcuts worth remembering

- **Short `MENU`:** open the menu.
- **Long `* SCAN`:** start/stop the channel or frequency scan.
- **Short `* SCAN`:** enter DTMF string input (in normal VFO/MR operation).
- **F + `* SCAN`:** start CTCSS/DCS detection on the current channel/frequency.
- **Long `5`:** in MR mode, shortcut for scan-list assignment; in VFO mode, toggles a bounded scan range between the selected VFO frequency and the other VFO's stored RX frequency (the default build has VOX and scan ranges enabled).
- **Long `MENU`:** run the action configured in `M Long`.
- **Long `EXIT`:** clear/cancel an active numeric input box.

---

# 4. Normal radio screen and operating modes

## 4.1 A/B VFO display

The main screen normally shows two VFO lines. The selected VFO is the one used for normal transmission; the receiver can temporarily use the other VFO when Dual Watch is active.

A line can display:

- Frequency, memory-channel number, channel name, or name plus frequency, depending on `ChDisp`.
- Modulation: `FM`, `AM` or `USB`.
- Tone/code indicators: `CT`, `DCS` or `DCR` when the corresponding FM receive/transmit code mode is configured.
- TX power: `L`, `M` or `H`.
- Offset direction: `+` or `-` when a repeater offset is configured.
- `R` when frequency reverse is active.
- Narrow-band indication when the channel is set to Narrow.
- Scan-list markers when that channel participates in Scan List 1 and/or Scan List 2.
- RSSI/signal level while receiving, and TX level while transmitting.

The status line can show TX/RX/power-save, scan, FM, Dual Watch/cross-band, VOX, keypad lock, F modifier, battery voltage or percentage, USB charging and battery level. Which symbols appear depends on compiled features and active modes.

## 4.2 VFO (frequency) mode and MR (memory) mode

- **VFO mode:** the display shows a tunable frequency. Use `UP`/`DOWN` to tune by the configured Step size, or type a frequency with the number keys.
- **MR mode:** the display shows saved memory channels. Use `UP`/`DOWN` to move between valid channels, or enter a three-digit channel number.

Use **F + 3** to switch between VFO and MR. Use **F + 2** to switch between VFO A and VFO B.

This firmware stores up to **200 memory channels** (channels 001–200 as shown to the operator). The additional VFO frequency positions are not the same thing as user-programmed memory channels.

## 4.3 Squelch and monitor

`Sql` controls how strong a signal must be before the speaker opens. Level 0 disables squelch (the receiver is effectively always open). Levels 1–9 progressively require a stronger signal. Because this build enables the more-sensitive squelch modification, the practical response may differ from stock firmware.

The **Monitor** side-key action opens the squelch so you can hear weak signals and background noise. It is useful for deciding whether the squelch threshold is simply too high.

For a weak signal, try a lower SQL setting before changing other receive settings. For a busy/noisy band, increase it gradually.

## 4.4 Dual Watch and cross-band receive

`RxMode` selects one of four modes:

| Display | Behaviour |
|---|---|
| MAIN ONLY | TX and RX use the main VFO only |
| DUAL RX RESPOND | Watch the two VFOs; when a signal is received, the radio can respond on the active/main side according to the firmware's Dual Watch logic |
| CROSS BAND | TX on the main VFO and RX on the secondary VFO |
| MAIN TX DUAL RX | Keep TX on the main VFO while receiving on both VFOs |

Dual Watch and cross-band functions are not needed for ordinary single-frequency listening. Leave `RxMode` at **MAIN ONLY** unless you understand why you need the other mode. Cross-band and Dual Watch should not be confused with a repeater or a lawful cross-band gateway.

## 4.5 Main-screen status indicators

The exact layout depends on the selected display mode and which features were compiled, but these are the principal indicators to recognise:

| Indicator | Meaning |
|---|---|
| `F` | The F modifier is armed and waiting for the next key |
| Padlock | Keypad lock is active; long-press `F` to toggle it |
| `L` / `M` / `H` | Low / Mid / High transmit-power selection |
| `FM` / `AM` / `USB` | Current VFO demodulation; AM and USB are receive modes in this build |
| `N` | Narrow channel bandwidth is selected; otherwise the channel is Wide |
| `CT` / `DCS` / `DCR` | CTCSS, DCS normal or DCS reverse squelch/code status |
| `+` / `−` | Repeater transmit-offset direction is configured |
| `R` | Frequency reverse is active |
| `VOX` | Voice-operated transmit is enabled |
| `SCR` | Voice scrambler is enabled for the current channel |
| `DW` / `XB` / `DWR` | Dual Watch / Cross Band / Dual RX Respond mode indicator |
| `I` / `II` | Current MR channel is assigned to Scan List 1 / Scan List 2 |
| `ScnRng` | A bounded VFO scan range is active |
| RSSI / S-meter / dBm | Approximate received signal indication; not calibrated test equipment |
| `MicBar` | Transmit audio-level bar, if enabled in the menu/build |
| Battery `%` or `V` | Estimated battery percentage or voltage, according to `BatTxt` |

---

# 5. Entering frequencies and using memory channels

## 5.1 Enter a frequency directly

1. Switch to **VFO mode** with F + 3 if you are in MR mode.
2. Make sure you are on the correct VFO (A or B).
3. Type the frequency with the number keys. The input is interpreted in MHz with 100 Hz internal increments; the current Step setting is used to round the final value.
4. Allow the input to complete. `EXIT` backspaces; a long `EXIT` clears the whole input.

Example: to enter approximately **446.00625 MHz**, type `446006` with the Step set to 12.5 kHz. The firmware's step rounding can land on the PMR446 channel centre at 446.00625 MHz. For other channel frequencies, see the table in section 10.

The VFO's displayed band group must permit the frequency. The firmware will reject frequencies outside its receive table and the BK4819 hardware gap.

## 5.2 Set the tuning step

Open `Step` in the menu. The firmware provides these choices:

**0.01, 0.05, 0.1, 0.25, 0.5, 1, 1.25, 2.5, 5, 6.25, 8.33, 9, 10, 12.5, 15, 20, 25, 30, 50, 100, 125, 200, 250 and 500 kHz.**

Use a step suited to the channel plan you are tuning. For PMR446 analogue channels, use **12.5 kHz**.

Note: the separate **spectrum analyser scan step** is configured with keys 1 and 7 while in the F+5 screen. It is not the same control as the VFO `Step` menu item.

## 5.3 Save a frequency as a memory channel

Use `ChSave` to select a memory-channel destination for the current VFO configuration. The channel number is entered as three digits (001–200). Confirm the destination when prompted. The saved channel can then be selected in MR mode.

The channel carries more than its frequency: it can retain its name, RX/TX tone settings, bandwidth, modulation, offset, power, scan-list participation and other channel attributes.

## 5.4 Rename or delete a memory channel

- `ChName`: choose a valid channel and edit its name. The editor uses `UP`/`DOWN` to change the character; `MENU` advances/accepts; the number keys can enter digits; `*` and `F` provide additional character operations in the name editor. The displayed name field is up to 10 characters.
- `ChDele`: choose the channel and confirm the deletion. Treat this as destructive.

The build option `ENABLE_KEEP_MEM_NAME=1` is on by default, so memory names are intended to survive channel re-saving where the firmware can preserve them.

## 5.5 Frequency reverse and repeater offsets

- `TxODir`: OFF, `+` or `-`.
- `TxOffs`: offset amount, entered numerically in MHz.
- `F + 8`: toggle frequency reverse, temporarily swapping the RX and TX frequency roles.

These are channel configuration tools, commonly used for amateur repeaters. They are not needed for PMR446 listening. Do not configure a transmit offset unless you have a lawful and intended transmit use for that channel.

---

# 6. The F+5 spectrum analyser, key by key

## 6.1 Open and understand the screen

From the normal main screen, tap `F`, then tap `5`. The firmware enters the spectrum analyser when `ENABLE_SPECTRUM=1` and `ENABLE_NOAA=0` (both conditions match the repository's default build).

The analyser sweeps a range around the current VFO frequency. It measures receiver RSSI at successive frequency points and draws a trace. It can automatically tune to a strong peak and open the audio path. It does not decode digital signals and it does not create a saved recording of signal history.

The principal display elements are:

- **Trace:** relative signal strength measured at each horizontal frequency position.
- **Peak arrow:** points to the strongest currently tracked peak.
- **Frequency labels:** show the scan start/end or centre/range step, depending on the scan-step mode.
- **`Nx`:** number of scan measurements (16, 32, 64 or 128).
- **Scan step:** the frequency interval between adjacent measurements.
- **dB scale:** approximate receiver level range. The current default starts at about −130 dBm to −50 dBm.
- **Horizontal trigger line:** the RSSI threshold that determines whether the analyser should switch from sweeping to listening.
- **Modulation and listen bandwidth:** shown at the top-right area.

The dBm scale is an estimate derived from RSSI and a band correction value. It is not a calibrated field-strength meter. Results can vary by frequency, antenna, radio sample and RF environment.

## 6.2 Default analyser settings

The source initialises the spectrum settings to:

| Setting | Default | Meaning |
|---|---:|---|
| Measurement count | 64 | 64 scan points across the displayed span |
| Scan step | 25.00 kHz | Spacing between measurements |
| Total swept span | 1.6 MHz | 64 × 25 kHz |
| Frequency movement step | 800 kHz | How far UP/DOWN moves the displayed scan range initially |
| RSSI trigger | 150 (raw scale) | Initial listen threshold, not a direct dBm value |
| Listen bandwidth | Wide | The filter used when audio listening is active |
| Modulation | FM | Starts using the current VFO's modulation; the initial static value is FM |
| Display scale | −130 to −50 dBm | Approximate vertical plot range |

When the analyser opens from a VFO frequency, it initialises the sweep so that the original VFO frequency is approximately at the middle of the default 1.6 MHz view. With a VFO set to 446.100 MHz, for example, the initial sweep is approximately 445.300–446.900 MHz.

## 6.3 Spectrum screen controls — complete key map

The following table describes the keys while the analyser is in its normal **SPECTRUM** state. These meanings are different from the same keys on the main radio screen.

| Key | Function in the spectrum screen | What it changes |
|---|---|---|
| `1` | Increase scan step | Moves to the next wider frequency interval |
| `7` | Decrease scan step | Moves to the next narrower frequency interval |
| `2` | Increase frequency movement step | Increases how far UP/DOWN shifts the scan range |
| `8` | Decrease frequency movement step | Decreases how far UP/DOWN shifts the scan range |
| `3` | Increase top dB limit | Raises the displayed top of the vertical scale by 1 dB per press, up to the firmware limit |
| `4` | Cycle measurement count | 64 → 128 → 16 → 32 → 64; unavailable while `ScnRng` is active |
| `9` | Decrease top dB limit | Lowers the displayed top of the vertical scale, while preserving a minimum display range |
| `UP` | Move the scan range upward | Shifts the swept frequencies higher by the movement step |
| `DOWN` | Move the scan range downward | Shifts the swept frequencies lower by the movement step |
| `*` | Raise RSSI trigger | Raises the signal level required to enter/continue listening |
| `F` | Lower RSSI trigger | Lowers the level required to enter/continue listening |
| `5` | Frequency input | Opens manual frequency entry for the analyser |
| `0` | Cycle modulation | FM → AM → USB → FM (BYP/RAW only exist in a special build) |
| `6` | Cycle listening bandwidth | Wide 25 kHz → Narrow 12.5 kHz → Narrower 6.25 kHz → Wide |
| Side1 | Blacklist the current peak | Marks the current peak bin to be ignored during the current scan session/range pass |
| Side2 | Toggle backlight | Turns the display backlight on/off for the analyser |
| `PTT` | Tune to the tracked peak and enter STILL state | This is **not a transmit command in the analyser**; it tunes to the detected peak and opens the detailed listening screen |
| `MENU` | No action | The spectrum handler deliberately does nothing for MENU in this state |
| `EXIT` | Leave the analyser | Restores the original frequency and saved radio registers, then returns to normal operation |

### Scan step choices

The spectrum analyser has its own list of intervals:

**0.01, 0.1, 0.5, 1, 2.5, 5, 6.25, 8.33, 10, 12.5, 15, 20, 25, 50 and 100 kHz.**

These are the distances between spectrum measurements, not necessarily the channel spacing of the service being viewed.

### Measurement count choices

The available counts are **16, 32, 64 and 128**. The `4` key cycles through them; from the default 64-point setting, the first press selects 128, then 16, 32 and back to 64. More points give a denser horizontal trace, but the width of the whole sweep is determined by both the point count and the scan step. The firmware calculates the total span as:

`measurement count × scan step`

For example:

- 64 points × 25 kHz = 1.6 MHz.
- 128 points × 12.5 kHz = 1.6 MHz.
- 128 points × 6.25 kHz = 800 kHz.

The 128 × 12.5 kHz combination is a useful starting point when viewing the complete 200 kHz PMR446 allocation in more detail.

### Frequency movement step

Keys 2 and 8 adjust the amount by which UP/DOWN shifts the sweep. It is a navigation increment, not the resolution between spectrum measurements. The source adjusts it in increments derived from the current scan step and limits it to a bounded range.

### dB scale

Keys 3 and 9 adjust the **top** of the display's dB range. They do not change the radio's actual RF gain or sensitivity. The lower end of the plotted range is updated from the measured noise floor. If a trace appears flattened at the top or bottom, adjust the display scale before assuming the signal itself has changed.

### RSSI trigger line

`*` raises the trigger; `F` lowers it. A lower trigger makes the analyser more willing to switch to listening, but it can also stop on noise or false peaks. A higher trigger reduces false stops but can miss weak transmissions.

The trigger is an internal RSSI threshold. It should not be treated as an exact dBm value.

## 6.4 What happens when a signal is found

During the sweep, the firmware measures RSSI at each point and tracks the strongest peak. When the peak crosses the trigger threshold, the analyser:

1. Stops sweeping temporarily.
2. Tunes the receiver to the peak frequency.
3. Enables the audio path and the selected listen bandwidth.
4. Listens for approximately one second before checking the signal again.
5. Continues listening while the signal remains above the trigger (or while monitor mode is active); otherwise it closes the audio path and resumes the sweep.

This is why you can hear a sufficiently strong analogue FM or AM signal while still on the spectrum screen. It is not a digital decoder; if the peak is a digital carrier, you may hear noise or nothing useful.

A weak signal below the trigger may appear in the trace but never open the speaker. Lower the trigger with `F`, or tune manually using the peak-hold procedure below.

## 6.5 Frequency input from the spectrum screen

1. Press `5` while in SPECTRUM.
2. Enter the frequency using the number keys.
3. Press `*` to insert the decimal point.
4. Use `EXIT` to backspace.
5. Press `MENU` to accept the entered frequency.

The analyser validates the frequency against its permitted input range. **Whether the entered value is the centre or the lower/start edge depends on the selected analyser scan step:**

- With scan steps **below 2.5 kHz** (0.01, 0.1, 0.5 or 1.0 kHz), the analyser is in *centre mode*. The entered value is the centre; the displayed sweep begins half a span below it.
- With scan steps **2.5 kHz and above**, the entered value is the *lower/start edge*. The displayed sweep begins at that frequency and extends upward. This includes the default 25 kHz analyser step and the 12.5 kHz PMR446 recipe in section 10.3.
- When `ScnRng` is active, the range endpoints define the sweep and the `5` frequency-input shortcut is disabled.

If no digits have been entered, `EXIT` returns to the previous analyser state. The key `*` is the decimal point in this input screen, not the trigger control.

## 6.6 PTT enters STILL mode — it does not transmit

Pressing PTT while in SPECTRUM calls `SetState(STILL)` and tunes to the tracked peak. The screen changes to a more detailed listening display with:

- The currently tuned peak frequency.
- A horizontal RSSI meter.
- Approximate S-meter and dBm values.
- The trigger marker.
- Four on-screen register adjustment boxes: `LNAs`, `LNA`, `PGA` and `IF`.

**The PTT key does not start transmission in the analyser's STILL state.** The source's PTT handler for STILL contains a TODO and the transmit code is commented out. Do not generalise this special behaviour to the normal main screen, where PTT is the transmit control.

## 6.7 STILL state controls — complete key map

| Key | Function in STILL | What it does |
|---|---|---|
| `3` / `9` | Adjust top dB limit | Same display-scale adjustment as SPECTRUM |
| `UP` / `DOWN` | Fine tune the receiver | If no register box is selected, tunes the displayed frequency |
| `UP` / `DOWN` with a register box selected | Adjust selected register | Increases/decreases the highlighted front-end/IF register value |
| `*` | Raise trigger | Raises listen threshold |
| `F` | Lower trigger | Lowers listen threshold |
| `5` | Frequency input | Opens manual frequency entry |
| `0` | Cycle modulation | FM → AM → USB → FM |
| `6` | Cycle listening bandwidth | 25 / 12.5 / 6.25 kHz |
| Side1 | Monitor toggle | Opens the receiver continuously, overriding the trigger threshold; press again to turn monitor off |
| Side2 | Backlight toggle | Turns the backlight on/off |
| `MENU` | Select register box | Cycles through LNAs → LNA → PGA → IF → LNAs |
| `EXIT` | Return to spectrum sweep | If a register box is selected, the first EXIT clears the selection; another EXIT returns to SPECTRUM and restarts the sweep |
| `PTT` | No action | The code currently contains no STILL-state transmit implementation |

### What the register boxes mean

These are low-level BK4819 receiver controls, not normal user settings:

- **LNAs:** adjusts the grouped low-noise amplifier stages.
- **LNA:** adjusts an individual LNA field.
- **PGA:** adjusts the programmable-gain amplifier field.
- **IF:** adjusts an intermediate-frequency register value.

Changing the LNA/PGA registers causes the firmware to lock the automatic gain-control setup for the session. These are experimental controls. They can make reception worse, cause distortion or make the displayed signal level misleading. For normal listening, leave them alone. Returning to the sweep and exiting the analyser restores the saved radio registers.

## 6.8 Blacklisting a peak

Side1 in SPECTRUM marks the currently tracked peak so it is ignored by subsequent measurements in that range. This is useful when a strong continuous signal dominates the display and you want to see weaker peaks nearby. It is temporary analyser behaviour, not a permanent frequency block list or a stored channel setting. Changing the scan range/step or restarting the analyser can clear the blacklist.

## 6.9 Useful spectrum recipes

**Survey the whole PMR446 allocation:**

1. Set the VFO to about 446.100 MHz.
2. Open F+5.
3. Set measurement count to 128 with `4` until `128x` appears.
4. Set scan step to 12.5 kHz with `1`/`7`.
5. The nominal sweep width is then 1.6 MHz, wide enough to include all 200 kHz of PMR446 with room either side.
6. Lower the trigger with `F` until real transmissions cause the receiver to pause, but not so low that it stops continuously on noise.

**Inspect a single narrow section:**

1. Use `5` to enter a frequency close to the section of interest.
2. Use `1`/`7` to select 6.25 or 12.5 kHz.
3. Use `4` to choose 64 or 128 measurement points.
4. Adjust the movement step with `2`/`8` and pan with UP/DOWN.

**Listen to a visible peak:**

1. Let the peak trigger the automatic listen state, or lower the trigger with `F`.
2. If it does not trigger, press PTT to enter STILL and tune to the strongest tracked peak.
3. Choose the correct demodulation with `0` and filter width with `6`.
4. Use Side1 to monitor continuously if you need to hear a signal below the trigger threshold.
5. Press EXIT to return to the sweep; EXIT again to leave the analyser.

---

# 7. The complete menu reference

## 7.1 Menu navigation

- Press `MENU` from the main screen to open the menu.
- Use `UP`/`DOWN` to move through menu items.
- Press `MENU` to enter the selected item.
- In a setting, use `UP`/`DOWN` to change the value. For many items you can also type the numeric value directly.
- Press `MENU` again to accept/save the setting.
- Press `EXIT` to back out without accepting the current edit.
- Type the menu's displayed index number to jump directly to that item. The numbers printed in the tables below match the default build described at the top of this manual. Optional menu items are compiled conditionally, so a custom build can shift the index; use the displayed menu label if a number does not match.

Most menu screens time out after approximately **20 seconds** of inactivity. Battery voltage, battery calibration and (if compiled) frequency calibration screens use a longer timeout of approximately **2 minutes**.

`*` has an extra function while editing **RxCTCS** or **RxDCS**: it starts/stops a CTCSS/DCS scan to identify the code used by an incoming FM signal.

### Numeric jump index — default build audited here

Use this index to locate an item quickly. Optional build flags can change the numbering; the menu label on the radio remains the reliable identifier. `FrCali` is omitted because it is not compiled in by default.

| 01–10 | 11–20 | 21–30 | 31–40 | 41–50 | 51–60 | 61–69 (hidden) |
|---|---|---|---|---|---|---|
| 01 `Step` | 11 `BusyCL` | 21 `SList2` | 31 `Mic` | 41 `Roger` | 51 `D Hold` | 61 `F Lock` |
| 02 `TxPwr` | 12 `Compnd` | 22 `ScnRev` | 32 `MicBar` | 42 `STE` | 52 `D Prel` | 62 `Tx 200` |
| 03 `RxDCS` | 13 `Demodu` | 23 `F1Shrt` | 33 `ChDisp` | 43 `RP STE` | 53 `D Decd` | 63 `Tx 350` |
| 04 `RxCTCS` | 14 `ScAdd1` | 24 `F1Long` | 34 `POnMsg` | 44 `1 Call` | 54 `D List` | 64 `Tx 500` |
| 05 `TxDCS` | 15 `ScAdd2` | 25 `F2Shrt` | 35 `BatTxt` | 45 `ANI ID` | 55 `D Live` | 65 `350 En` |
| 06 `TxCTCS` | 16 `ChSave` | 26 `F2Long` | 36 `BackLt` | 46 `UPCode` | 56 `AM Fix` | 66 `ScraEn` |
| 07 `TxODir` | 17 `ChDele` | 27 `M Long` | 37 `BLMin` | 47 `DWCode` | 57 `VOX` | 67 `BatCal` |
| 08 `TxOffs` | 18 `ChName` | 28 `KeyLck` | 38 `BLMax` | 48 `PTT ID` | 58 `BatVol` | 68 `BatTyp` |
| 09 `W/N` | 19 `SList` | 29 `TxTOut` | 39 `BltTRX` | 49 `D ST` | 59 `RxMode` | 69 `Reset` |
| 10 `Scramb` | 20 `SList1` | 30 `BatSav` | 40 `Beep` | 50 `D Resp` | 60 `Sql` | |

## 7.2 Receive and channel settings

| Menu | Options / range | What it means and how to use it |
|---|---|---|
| **60** · `Sql` | 0–9 | Squelch threshold. 0 opens the receiver continuously; higher values require a stronger signal. Start low and increase only enough to silence idle noise. |
| **02** · `TxPwr` | LOW / MID / HIGH | Selects the transmitter output-power level. |
| **01** · `Step` | 24 step values, 0.01–500 kHz | VFO frequency increment. Choose a step appropriate to the band/channel plan. |
| **13** · `Demodu` | FM / AM / USB | Selects the receive demodulator for the current VFO/channel. USB is upper sideband; LSB is not available in the default build. |
| **09** · `W/N` | Wide / Narrow | Channel receive/transmit filter width. Narrow is normally appropriate for 12.5 kHz land-mobile channels such as analogue PMR446 reception. |
| **04** · `RxCTCS` | OFF or CTCSS tone | Receive tone squelch. The speaker opens only when the received carrier has the configured sub-audible tone. For hearing any analogue PMR446 user on a channel, set OFF. |
| **03** · `RxDCS` | OFF, DCS normal or reverse | Receive digital-coded squelch. Set OFF unless you know the other radio's exact DCS code and polarity. |
| **06** · `TxCTCS` | OFF or CTCSS tone | Encodes a CTCSS tone on transmission. Not needed for receive-only use. |
| **05** · `TxDCS` | OFF, DCS normal or reverse | Encodes a DCS code on transmission. Not needed for receive-only use. |
| **07** · `TxODir` | OFF / + / − | Enables and sets the transmit offset direction. |
| **08** · `TxOffs` | Numeric MHz offset | Amount of transmit offset. Only configure for a legitimate, authorised repeater/channel use. |
| **11** · `BusyCL` | OFF / ON | Busy-channel lockout. When enabled, prevents transmission while the radio detects a busy channel; it does not guarantee that a frequency is clear. |
| **10** · `Scramb` | OFF; 2600–3500 Hz | Voice-scrambler setting. Both radios need compatible settings. This is not encryption or secure privacy. Global scrambler enable is in the hidden menu. |
| **12** · `Compnd` | OFF / TX / RX / TX/RX | Audio compander direction. Both ends generally need compatible settings. It changes audio processing, not the RF channel. |
| **56** · `AM Fix` | OFF / ON | Enables the firmware's dynamic AM front-end gain adjustment. Leave ON for AM listening unless it causes a specific problem. It cannot remove hardware overload limitations. |
| **19** · `SList` | LIST1 / LIST2 / ALL | Selects which scan-list set the channel scanner uses. ALL scans all valid memory channels. |
| **14** · `ScAdd1` | OFF / ON | Includes/excludes the current memory channel from Scan List 1. |
| **15** · `ScAdd2` | OFF / ON | Includes/excludes the current memory channel from Scan List 2. |
| **20** · `SList1` | Channel selector / priority display | Manages the Scan List 1 priority-channel entries. The screen indicates the priority slots when configured. |
| **21** · `SList2` | Channel selector / priority display | Manages the Scan List 2 priority-channel entries. |
| **22** · `ScnRev` | TIMEOUT / CARRIER / STOP | Scan-resume policy. TIMEOUT resumes after a fixed pause; CARRIER resumes when the carrier disappears; STOP leaves the scan parked on the found signal until you intervene. |
| **16** · `ChSave` | Channel 001–200 | Selects the memory-channel destination for the current VFO settings. Confirm the selected destination. |
| **17** · `ChDele` | Channel 001–200 | Selects a memory channel for deletion. Confirmation required. |
| **18** · `ChName` | Channel 001–200, then character editor | Edits a memory-channel name (up to 10 displayed characters). |
| **33** · `ChDisp` | FREQ / CHANNEL NUMBER / NAME / NAME + FREQ | Controls how a memory channel is rendered on the main screen. |
| **44** · `1 Call` | Channel 001–200 | Selects the memory channel recalled by F+9. |

### CTCSS and DCS in plain English

CTCSS is a continuous low-frequency tone sent alongside the voice transmission. DCS sends a repeating digital code. They are forms of selective squelch, not encryption. Someone with a receiver set to OFF can still hear an analogue transmission if it is otherwise receivable. A wrong receive code can make a perfectly good signal appear silent because the radio keeps the speaker muted.

The source contains **50 CTCSS tone choices** and **104 DCS code choices**. DCS is available in normal and inverse polarity, so the menu has OFF plus 104 `DxxxN` selections and 104 `DxxxI` selections. The code values shown by the radio are octal.

**CTCSS tones (Hz):**

`67.0, 69.3, 71.9, 74.4, 77.0, 79.7, 82.5, 85.4, 88.5, 91.5`

`94.8, 97.4, 100.0, 103.5, 107.2, 110.9, 114.8, 118.8, 123.0, 127.3`

`131.8, 136.5, 141.3, 146.2, 151.4, 156.7, 159.8, 162.2, 165.5, 167.9`

`171.3, 173.8, 177.3, 179.9, 183.5, 186.2, 189.9, 192.8, 196.6, 199.5`

`203.5, 206.5, 210.7, 218.1, 225.7, 229.1, 233.6, 241.8, 250.3, 254.1`

**DCS code values (octal, normal polarity `N`; inverse polarity `I` uses the same code with inverted polarity):**

`023 025 026 031 032 036 043 047 051 053 054 065 071 072 073 074 114 115 116 122 125 131 132 134 143 145`

`152 155 156 162 165 172 174 205 212 223 225 226 243 244 245 246 251 252 255 261 263 265 266 271 274 306`

`311 315 325 331 332 343 346 351 356 364 365 371 411 412 413 423 431 432 445 446 452 454 455 462 464 465`

`466 503 506 516 523 526 532 546 565 606 612 624 627 631 632 654 662 664 703 712 723 731 732 734 743 754`

## 7.3 Scanning and key behaviour settings

| Menu | Options / range | What it means |
|---|---|---|
| **29** · `TxTOut` | 30 seconds, 1–9 minutes, 15 minutes | Transmit timeout timer. Stops an overlong transmission after the selected time. It does not grant permission to transmit. |
| **30** · `BatSav` | OFF / 1:1 / 1:2 / 1:3 / 1:4 | Battery-save ratio. Higher save ratios reduce receiver duty cycle and can increase the chance of missing the beginning of a short signal. |
| **28** · `KeyLck` | OFF / AUTO | Automatic keypad lock. When enabled, the firmware locks the keypad after inactivity; `F` long-press toggles lock manually. |
| **59** · `RxMode` | MAIN ONLY / DUAL RX RESPOND / CROSS BAND / MAIN TX DUAL RX | Selects single-VFO, Dual Watch and cross-band behaviour (see section 4.4). |
| **40** · `Beep` | OFF / ON | Keypress/menu beep. |
| `Voice` | OFF / CHI / ENG | Voice prompt language. This build has `ENABLE_VOICE=0`, so the item is absent unless built with voice enabled. |
| **57** · `VOX` | OFF / levels 1–10 | Voice-operated transmit sensitivity. Enabled in the default build. Higher levels change the audio threshold; use only if you actually need VOX. |
| **23** · `F1Shrt` | Side-key function list | Assigns the short-press action for Side1. |
| **24** · `F1Long` | Side-key function list | Assigns the long-press action for Side1. |
| **25** · `F2Shrt` | Side-key function list | Assigns the short-press action for Side2. |
| **26** · `F2Long` | Side-key function list | Assigns the long-press action for Side2. |
| **27** · `M Long` | Side-key function list | Assigns the action for a long press of MENU. |

## 7.4 Audio, backlight and display settings

| Menu | Options / range | What it means |
|---|---|---|
| **31** · `Mic` | Five gain levels: approximately +1.5, +4, +8, +12, +15.5 dB | Microphone sensitivity/gain for transmission. It is not speaker volume. |
| **32** · `MicBar` | OFF / ON | Displays the experimental microphone/audio level bar while transmitting. Compiled in by default. |
| **36** · `BackLt` | OFF / 5 s / 10 s / 20 s / 1 min / 2 min / 4 min / ON | Backlight timeout. OFF means the backlight does not turn on automatically in the usual way; ON keeps it lit. |
| **37** · `BLMin` | 0–9 | Minimum backlight brightness. |
| **38** · `BLMax` | 1–10 | Maximum backlight brightness. The firmware keeps minimum below maximum. |
| **39** · `BltTRX` | OFF / TX / RX / TX/RX | Turns on the backlight when transmitting, receiving, or both. |
| **34** · `POnMsg` | FULL / MESSAGE / VOLTAGE / NONE | What is displayed at power-on. |
| **35** · `BatTxt` | NONE / VOLTAGE / PERCENT | Shows battery voltage or estimated percentage in the status line. |
| **58** · `BatVol` | Readout | Displays the measured battery voltage and estimated percentage; it is a readout, not a setting. |
| **68** · `BatTyp` | 1600 mAh / 2200 mAh | Selects the battery capacity profile used for the percentage estimate. It does not change the actual battery. |
| **67** · `BatCal` | 1600–2200 calibration parameter | Adjusts the battery-voltage scaling used by the firmware. Only alter after comparing with a reliable meter; do not guess. |

## 7.5 End-of-transmission and tone settings

| Menu | Options / range | What it means |
|---|---|---|
| **41** · `Roger` | OFF / ROGER / MDC | End-of-transmission indication. MDC is a signalling option, not a universal compatibility setting. Leave OFF unless the other system expects it. |
| **42** · `STE` | OFF / ON | Tail-tone elimination. Helps reduce the squelch tail heard when a CTCSS transmission ends. |
| **43** · `RP STE` | OFF / 1–10 × 100 ms | Repeater tail-elimination delay, from 100 ms increments up to about 1 second. Primarily relevant to repeater use. |

## 7.6 DTMF menu items

DTMF is Dual-Tone Multi-Frequency signalling. It is not digital voice, DMR or encryption. Since the default build enables DTMF calling, these entries are present:

| Menu | Meaning |
|---|---|
| **45** · `ANI ID` | Displays the radio's own DTMF identification string. In this firmware's front-panel menu handler, ANI ID is display-only; the menu key does not open an editor. |
| **46** · `UPCode` | Displays the DTMF string used for the configured “up”/start code. This is also display-only from the front-panel menu. |
| **47** · `DWCode` | Displays the DTMF string used for the configured “down”/end code. This is also display-only from the front-panel menu. |
| **48** · `PTT ID` | Selects when the PTT ID is sent: OFF, UP CODE, DOWN CODE, UP+DOWN CODE or APOLLO QUINDAR tone sequence. |
| **49** · `D ST` | DTMF sidetone ON/OFF. Controls whether locally generated DTMF is heard in the speaker. |
| **50** · `D Resp` | Decoder response: DO NOTHING / RING / REPLY / BOTH. |
| **51** · `D Hold` | How long the DTMF call/response state is held before auto-reset: 5–60 seconds. |
| **52** · `D Prel` | DTMF pre-load duration: 3–99 units of 10 ms (30–990 ms). |
| **53** · `D Decd` | Enables/disables DTMF decoding for the current channel. |
| **54** · `D List` | Selects one of up to 16 DTMF contacts. A valid selected contact can be used to initiate a DTMF call. |
| **55** · `D Live` | Global live DTMF decoder display ON/OFF. |

DTMF identifiers and call codes are signalling values, not passwords. The EEPROM initialisation code includes fallback values for an uninitialised ID/code set, and the remote-kill permission flag defaults to enabled when its stored value is invalid. Do not leave predictable defaults in a system where DTMF calling is actually used, and do not enable remote-control/kill features unless you understand the consequences and have authority to operate that system. The front-panel ANI ID/UPCode/DWCode entries are display-only in this build; use an appropriate programming method supported by the firmware if those values must be changed.

---

# 8. Hidden transmit and calibration menu

## 8.1 Enter the hidden menu

The radio has **two distinct startup layers**. Do not confuse the factory bootloader's flash mode with Egzumer's application-level startup shortcuts.

### A. Factory bootloader flash mode — PTT + Power On

On the standard original-style UV-K5 V1 hardware, the factory bootloader enters flash/download mode when you:

1. Turn the radio off.
2. Press and hold **PTT**.
3. While holding PTT, turn the radio on with the volume knob.
4. Confirm the expected bootloader indication: the display remains blank and the top white flashlight LED lights steadily.

This bootloader runs **before** Egzumer's application firmware. It is part of the radio's factory boot code, so the sequence is not implemented in `helper/boot.c`. A K1-style programming cable and compatible flashing software can then communicate with the bootloader. Hardware revisions differ; check the flasher's instructions for the exact board in your radio rather than assuming that every UV-K5-family revision uses the same key combination.

### B. Egzumer application startup shortcuts — PTT + side key

These are checked later by the firmware's own `helper/boot.c` handler, after control has passed from the factory bootloader:

| Power-on hold | Egzumer application action | Build condition |
|---|---|---|
| **PTT + Side1** (upper) | Opens the hidden menu / F Lock section | Built into this firmware |
| **PTT + Side2** (lower) | Enters AirCopy mode | Only when `ENABLE_AIRCOPY=1`; disabled in the default build audited here |

At normal operating time, `PTT + Side2` may have yet another function: a 1750 Hz repeater tone burst is only available when the relevant TX1750 build option is enabled. The default build audited here has that option disabled. These are three separate mechanisms: factory flash bootloader, firmware startup shortcuts and in-session key actions.

## 8.2 Hidden menu reference

| Menu | Options | What the source does |
|---|---|---|
| **61** · `F Lock` | DEFAULT+; FCC HAM; CE HAM; GB HAM; 137–174 / 400–430; 137–174 / 400–438; DISABLE ALL; UNLOCK ALL | Applies a firmware TX frequency filter. In the source, DEFAULT+ permits 137–174 and 400–470 MHz, with 174–350, 350–400 and 470–600 MHz conditionally opened by their separate toggles. DISABLE ALL blocks TX through the firmware frequency check. UNLOCK ALL is deliberately protected by repeated confirmation. |
| **62** · `Tx 200` | OFF / ON | Enables/disables the additional 174–350 MHz TX range in the default TX-lock logic (the source's band boundary begins at 174 MHz despite the menu label). |
| **63** · `Tx 350` | OFF / ON | Enables/disables the additional 350–400 MHz TX range. |
| **64** · `Tx 500` | OFF / ON | Enables/disables the additional 470–600 MHz TX range. |
| **65** · `350 En` | OFF / ON | Makes the 350–400 MHz band group available in VFO band selection. The TX-lock logic also requires this flag, together with `Tx 350`, before it permits TX in that segment. It is not a general sensitivity enhancer. |
| **66** · `ScraEn` | OFF / ON | Global enable for the scrambler settings selected per channel. |
| `FrCali` | −50 to +50 | Reference crystal calibration; not present in the default build. Only use with suitable frequency-measurement equipment and a known reference. |
| **67** · `BatCal` | 1600–2200 | Battery-voltage scaling calibration. Use a reliable meter; random adjustment makes the battery display less accurate. |
| **68** · `BatTyp` | 1600 mAh / 2200 mAh | Selects the battery profile. |
| **69** · `Reset` | VFO / ALL | VFO reset or full settings reset. ALL is destructive and should not be used casually. |

## 8.3 Important warning about the “GB HAM” lock

Do **not** treat `F Lock → GB HAM` as a legal compliance certificate. The source code for that profile permits **144–148 MHz** and **430–440 MHz**. The UK 2-metre amateur allocation is 144–146 MHz, so the firmware profile is wider than the UK 2-metre allocation. The lock is only a simple firmware filter; it is not a complete implementation of the UK band plan, licence conditions, equipment requirements or all Ofcom restrictions.

For a receiver-only radio, the safest configuration is **F Lock → DISABLE ALL**. This causes the firmware TX frequency check to reject all transmit frequencies. Do not select UNLOCK ALL.

---

# 9. Scanning, signal search and CTCSS/DCS detection

The firmware has more than one scan operation. They are not interchangeable.

## 9.1 Channel/frequency scan — long press `* SCAN`

This is the main scan function:

- In **MR mode**, it steps through valid memory channels, respecting the selected scan list.
- In **VFO mode**, it steps through frequencies using the VFO Step value.
- Use `SList` to choose List 1, List 2 or ALL.
- Use `ScAdd1`/`ScAdd2` on individual memory channels to include or exclude them from each list.
- Press `UP` or `DOWN` while scanning to jump to the next channel/frequency in that direction.
- Press `EXIT` to stop and return to the original frequency/channel.
- Press `MENU` while scanning to stop and retain the found channel/frequency where the scan state permits; this is the “remember the last found transmission” behaviour described by the project README.

The default build enables faster scanning. It uses a short dwell/pause, so very short transmissions can be missed. If you are trying to catch weak or brief signals, a slower scan method or a smaller custom scan list may be more effective. The build also enables `ENABLE_NO_CODE_SCAN_TIMEOUT`, so the CTCSS/DCS code-detection scan does not automatically stop at the usual 32-second timeout; use `EXIT` to end it.

## 9.2 Frequency-range scanning — long press `5` in VFO mode

With `ENABLE_SCAN_RANGES=1` (the default build), long-pressing `5` while in VFO mode toggles a scan range between the current VFO frequency and the other VFO's stored RX frequency. The firmware sorts the two endpoints so the lower frequency becomes the start and the higher frequency becomes the stop. The main display marks the range as `ScnRng`.

This range constrains the normal VFO frequency scan and can also constrain the F+5 spectrum analyser. When active, the analyser derives its measurement count from `(range stop − range start) / spectrum scan step`; it starts at the range's lower edge and automatically uses the range width. In this mode, spectrum UP/DOWN panning, the `4` measurement-count control and the `5` frequency-input shortcut are disabled because the range endpoints define the sweep. Long-pressing `5` again clears the custom range.

In MR mode, the same long-press shortcut toggles scan-list participation instead of setting a frequency range. If a memory channel is in only one list, the shortcut mirrors that membership to the other list; if its two list flags match, it toggles List 1. The explicit `ScAdd1` and `ScAdd2` menu items provide direct control.

## 9.3 Frequency search — F + 4

F+4 opens the dedicated scanner screen and performs a frequency search. The scanner attempts to find an active frequency and then identifies the associated CTCSS/DCS code where available.

The display shows:

- `FREQ:` — frequency being searched/found.
- `CTC:` — detected CTCSS tone, when one is found.
- `DCS:` — detected DCS code, when one is found.
- `SCAN` — search in progress.
- `SCAN CMP.` — scan completed/found.
- `SCAN FAIL.` — no valid result found.

Press `*` to restart the search. Press `MENU` to enter the save process when a valid result is available. The firmware can round a found frequency to a suitable 2.5 kHz or 6.25 kHz step before saving. Follow the prompts to choose a memory channel and confirm.

## 9.4 CTCSS/DCS scan — F + `* SCAN`

F+* starts a scan for the CTCSS or DCS code used by a signal on the current frequency. This is a code-detection operation, not a frequency sweep. It is useful when you already know the channel but not its selective-squelch code.

You can also open `RxCTCS` or `RxDCS` in the menu and press `*` while inside the submenu. The menu's code scanner searches in the background and, when it finds a code, places the result into the corresponding receive-code setting.

## 9.5 Scan-resume modes

`ScnRev` controls what happens when a scan stops on a signal:
- **TIMEOUT:** pause for a time, then resume.
- **CARRIER:** remain stopped while the carrier is present; resume after it disappears.
- **STOP:** remain stopped on the found signal until you intervene.

Choose STOP when you want to inspect a found signal manually. Choose CARRIER for a conventional “stay while someone talks” scan. TIMEOUT is useful when you want the scan to keep moving even if a signal is present.

---

# 10. PMR446 reception — normal screen and spectrum screen

## 10.1 The UK analogue PMR446 channel frequencies

The analogue PMR446 channel plan occupies 446.0–446.2 MHz with 12.5 kHz channel spacing. The channel centres are:

| PMR channel | Frequency (MHz) | PMR channel | Frequency (MHz) |
|---:|---:|---:|---:|
| 1 | 446.00625 | 9 | 446.10625 |
| 2 | 446.01875 | 10 | 446.11875 |
| 3 | 446.03125 | 11 | 446.13125 |
| 4 | 446.04375 | 12 | 446.14375 |
| 5 | 446.05625 | 13 | 446.15625 |
| 6 | 446.06875 | 14 | 446.16875 |
| 7 | 446.08125 | 15 | 446.18125 |
| 8 | 446.09375 | 16 | 446.19375 |

The UV-K5 firmware is an analogue FM receiver for this purpose. It will not decode digital dPMR446 voice. The wider PMR446 allocation also has digital channel plans; those are not the 16 analogue channel centres listed above.

## 10.2 Receive PMR446 on the normal main screen

To receive an analogue PMR446 channel, configure the VFO as follows:

1. Press **F + 3** until the radio is in VFO/frequency mode.
2. Press **F + 1** until the 400 MHz band group is selected.
3. Open `Step` and select **12.5 kHz**.
4. Open `Demodu` and select **FM**.
5. Open `W/N` and select **NARROW**.
6. Set `RxCTCS` to **OFF** and `RxDCS` to **OFF** if you want to hear any analogue transmission on the selected channel regardless of its tone/code.
7. Set `Sql` to 1 initially. Lower it if weak signals are being muted; raise it only if idle noise is a problem.
8. Enter the required frequency. For Channel 1, enter `446006` with the 12.5 kHz Step; the firmware rounds to 446.00625 MHz. Use the channel table above for the other channels.
9. Listen through the speaker. If a signal is too weak to open squelch, use the Monitor side-key action briefly.

**CTCSS/DCS note:** a tone code is a squelch aid, not encryption. With `RxCTCS`/`RxDCS` OFF, the receiver does not filter audio by tone or code.

## 10.3 Find PMR446 activity with F+5

A useful setup is:

1. On the normal main screen, tune the VFO to **446.100 MHz** (roughly the middle of the PMR446 block).
2. Press **F + 5**.
3. The default analyser sweep is about 1.6 MHz wide, approximately 445.300–446.900 MHz, so the full PMR446 block should appear in the initial view.
4. For a more detailed plot, set **128 points** using `4` and set the spectrum scan step to **12.5 kHz** using `1`/`7`. The nominal span remains 1.6 MHz.
5. Use `F` to lower the RSSI trigger until real transmissions make the radio enter listen mode. Use `*` to raise it if it keeps stopping on background noise.
6. A strong analogue signal should cause the analyser to tune to the peak and open the audio. If the signal is visible but does not trigger, lower the threshold.
7. Press PTT to enter STILL and tune to the current peak. This is a receive/listen action inside the analyser, not a TX command.
8. In STILL, use `0` for FM, `6` for 12.5 kHz listening bandwidth and Side1 to monitor continuously if needed.
9. Press EXIT to return to the analyser sweep, then EXIT again to return to the normal radio screen.

### Exact 16-bin PMR446 view (one measured point per analogue channel)

For a channel-by-channel view of the 200 kHz analogue block, use the analyser's **16-point** setting with a **12.5 kHz** scan interval. The firmware measures 16 points: 446.00625, 446.01875, …, 446.19375 MHz. In the plot these measurements are expanded horizontally, so each channel has its own bar group. The computed sweep width is 16 × 12.5 kHz = 200 kHz; the right-hand displayed edge is the upper boundary after Channel 16, not a seventeenth channel centre.

1. Tune the radio near PMR446 and open `F+5`.
2. Press `4` until `16x` is shown.
3. Press `1` or `7` until the spectrum interval reads `12.50k`.
4. Press `5` to open the analyser's frequency-input screen.
5. Enter `446*00625` (the `*` key is the decimal point) and press `MENU` to accept. At 12.5 kHz the analyser is in lower-edge mode, so this sets Bin 0 to Channel 1 (446.00625 MHz).
6. Set `0` to FM and `6` to the `12.5k` listen bandwidth. Adjust the trigger with `F` and `*`.

**Do not enter `446*10625` for this 16-bin/12.5 kHz setup.** At this scan step, key `5` accepts the lower edge, not the centre; entering Channel 9 would start the sweep at Channel 9 and omit Channels 1–8.

This is a live swept measurement, not 16 independent receivers. The radio measures channels sequentially, so a very short transmission can begin and end between visits to its frequency. If no peaks appear, the band may simply be quiet or signals may be too weak to cross the analyser's trigger threshold.

## 10.4 Why the UV-K5 must not be used as a PMR446 transmitter

> **PMR446 transmit warning:** Do not use the UV-K5 to transmit on PMR446. Programming 446 MHz, selecting Low power or enabling a firmware TX lock does not make it compliant; see section 1.1 for the equipment conditions.

---

# 11. FM broadcast radio

The dedicated FM broadcast receiver uses a separate BK1080 receiver chip. In the default build it is enabled.

## 11.1 Enter and leave FM radio

- Press **F + 0** to enter FM broadcast mode.
- Press **F + 0** again or `EXIT` to return to the two-way radio screen.

The FM screen shows the selected FM band, current frequency or memory channel, and scan/save/delete prompts.

## 11.2 FM radio controls

| Key | Function |
|---|---|
| Number keys | Enter a broadcast frequency in VFO mode, or a stored station number in memory mode |
| `UP` / `DOWN` | Tune to the next/previous frequency step or station |
| `F + 1` | Cycle the FM broadcast band selection |
| `F + 3` | Toggle FM VFO / station-memory mode |
| `F + *` or long-press `*` | Start automatic FM scan; the search populates the 20-entry FM station bank, so it can replace the saved station list |
| `MENU` | Save the found station in VFO mode; delete a selected memory station in MR mode, subject to confirmation |
| `F + 0` or long-press `0` | Leave FM broadcast mode (the same action that enters it from the main screen) |
| `EXIT` | Stop a scan, cancel a save/delete prompt, or leave FM radio when idle |

The FM receiver has its own station-memory bank (20 entries). It is separate from the 200 two-way radio memory channels.

---

# 12. DTMF functions

DTMF is a set of paired audio tones used for signalling. This firmware supports DTMF entry, transmission, ANI identification and optional decoding/calling features. It does not decode DMR, dPMR, P25 or other digital voice protocols.

## 12.1 Enter a DTMF string

In normal radio operation, short-press `*` to enter DTMF input mode. Enter the digits/DTMF symbols with the keypad. PTT transmits the entered string according to the channel's `PTT ID` and DTMF configuration. `EXIT` can backspace/cancel the input.

## 12.2 Configure DTMF for a known system

Only configure DTMF for a system whose operator has supplied the correct ID and signalling strings. **`ANI ID`, `UPCode` and `DWCode` are display-only in this firmware's front-panel menu; you cannot edit them on the radio.** If these fields need changing, use a PC programming application or another programming method that explicitly supports these settings for the Egzumer build installed on your radio. Back up the radio before writing configuration data; this manual does not assume that every version of CHIRP exposes these firmware-specific fields.

The remaining front-panel settings can be configured as follows:

1. Set `PTT ID` to the required send timing (OFF / UP / DOWN / UP+DOWN / APOLLO QUINDAR).
2. Set `D ST` according to whether you want to hear locally generated tones.
3. Enable `D Decd` for channels where DTMF decode is required.
4. Set `D Resp` only if the system expects an audible ring/reply response.
5. Set `D Hold` and `D Prel` to match the system timing.
6. Enable `D Live` only if you want the live decoded string displayed.

Do not assume DTMF is secure. It is signalling, not encryption.

---

# 13. UK band guide: what the radio can and cannot receive

This section is a practical orientation guide, not a replacement for Ofcom's full UK Frequency Allocation Table. The UKFAT is the authoritative reference for the allocations and services assigned across UK spectrum; its 2026 edition was updated on 3 July 2026.

The UV-K5 can tune only the hardware ranges described in section 1. It can demodulate FM, AM and USB in the default build. Many UK services use digital modulation, encrypted/private systems or data bursts that the radio cannot decode.

| Approximate frequency range | UK activity you may encounter | UV-K5 capability / caution |
|---|---|---|
| 18–30 MHz | HF services and various short-range/radio uses | Firmware can tune this with Wide RX, but the UV-K5 is not a proper HF receiver. Expect poor sensitivity/selectivity; no general HF performance guarantee. |
| 26.965–27.405 MHz and 27.60125–27.99125 MHz | UK Citizens' Band: 40 CEPT channels and the separate 40-channel UK band | Tunable with Wide RX. The firmware is not a substitute for compliant CB equipment; use the correct channel plan and mode for reception. |
| 30–88 MHz | Land-mobile, specialist and other services; amateur 4 m/6 m activity lies in parts of this wider region | Technically tunable. The UV-K5 does not have a dedicated VHF low-band or 6 m/4 m front end. |
| 50–52 MHz | UK amateur 6-metre activity: 50–51 MHz is primary and 51–52 MHz is secondary under the Ofcom amateur licence terms | FM/AM/USB receiver modes may be useful depending on the signal and band segment. |
| 70–70.5 MHz | UK amateur 4-metre activity | The RSGB plan identifies 70.260 MHz as the 4 m AM calling frequency and 70.294–70.500 MHz as a 12.5 kHz channelised segment. 70.450 MHz is also commonly used in UK amateur practice as an FM calling/net reference, but check current local band-plan guidance. |
| 87.5–108 MHz | Analogue FM broadcast radio | Best accessed with **F+0** and the separate FM broadcast receiver. This is a general broadcast service. |
| 108–118 MHz | Aeronautical navigation systems | Primarily navigation aids rather than analogue voice channels. |
| 118–137 MHz | Civil aeronautical communications (AM) | Set `Demodu` to AM. Use the VFO `Step` at **8.33 kHz** for the modern civil channel plan; this changes tuning increments, not the radio's fixed filter bandwidth. |
| 137–174 MHz | VHF services, including amateur 2 m (144–146 MHz), maritime VHF and satellite signals | The radio can receive analogue FM/AM where its demodulator and bandwidth are suitable. It cannot decode digital satellite telemetry or images. Historic NOAA APT carriers were 137.100 MHz (NOAA-19), 137.620 MHz (NOAA-15) and 137.9125 MHz (NOAA-18); all three spacecraft and the POES constellation were decommissioned in 2025, so these are **legacy reference frequencies**, not current NOAA APT targets. |
| 156.000–162.025 MHz | Maritime VHF channels, including Channel 16 at **156.800 MHz** | Marine VHF uses 25 kHz channel spacing and FM voice. The UV-K5 is not a type-approved marine safety radio and is not a substitute for ship-station equipment. |
| 174–240 MHz | DAB digital radio (Band III) and other allocations | The UV-K5 cannot decode DAB. A signal may appear as a broad feature in the spectrum display, but there will be no programme audio. |
| 240–350 MHz | Various aeronautical, military, satellite and other allocations | Technically within the firmware range; much activity is non-public, digital or specialised. Not a general-purpose decoder. |
| 350–400 MHz | Specialist and operational allocations | Tunable; reception quality varies with frequency and local RF conditions. |
| 400–430 MHz | UHF services, SRD and other shared/operational allocations | Some analogue signals may be demodulated; much activity is data or private communications. |
| 430–440 MHz | UK amateur 70 cm allocation; sub-band status/conditions vary (including a London-area restriction on amateur use of 431–432 MHz) | Includes FM simplex/calling and repeater output segments. The radio can receive analogue FM; digital-voice repeaters are not decoded. |
| 433.05–434.79 MHz (within 400–470 MHz) | Short-range devices and telemetry in the 433 MHz ISM/SRD area | Many devices send short digital/data bursts. The radio may show activity but cannot decode many payloads. |
| 440–446 MHz | Other UHF allocations and systems | Technically tunable; not all traffic is public/general reception. |
| 446.0–446.2 MHz | PMR446 analogue and digital handheld radio | The analogue 16-channel plan is listed in section 10. The default firmware receives analogue FM but cannot decode dPMR digital voice. |
| 446.2–470 MHz | Other UHF business and specialist allocations | The receiver can tune this range, but many systems are digital and cannot be decoded by the default firmware. |
| 470–694 MHz | UK digital terrestrial television (Freeview) and other allocations | The radio may tune part of this span, but it cannot decode DVB-T/T2 television. Strong TV multiplexes can appear on the spectrum screen as broad RF activity. |
| 630–840 MHz | Mobile/broadcast-related allocations across parts of the range | **Not receivable by the BK4819 tuning ranges used by this firmware**; the source rejects 630–840 MHz. |
| 840–870 MHz | Short-range devices, mobile and other services | Technically within the upper receiver range. Most signals are data or private traffic; not decoded as voice by default. |
| 870–960 MHz | Mobile, telemetry and other allocations | Some RF energy may be visible, but the radio cannot decode cellular systems or most digital/data protocols. |
| 960–1215 MHz | Aeronautical navigation, GNSS-related allocations and other services | Not ordinary analogue voice. The radio is not an ADS-B, GNSS or navigation-data decoder. |
| 1215–1300 MHz | Navigation, satellite and other allocations | The chip's upper tuning range extends here, but this is not a practical general voice-reception band for the UV-K5. |

## 13.1 Useful hobby-reception targets

### Where to start listening

- **FM broadcast:** F+0, tune local stations.
- **Amateur VHF/UHF:** the 2 m (144–146 MHz) and 70 cm (430–440 MHz) bands are useful starting points; FM and digital voice both exist in parts of these bands.
- **Amateur 6 m and 4 m:** reception depends on antenna, location, propagation and the signal mode.
- **Spectrum observation:** use F+5 to see whether RF energy is present before selecting an analogue demodulation mode.

Useful UK amateur reference points include **51.510 MHz** (6 m FM calling), **70.260 MHz** (4 m AM calling, identified by the RSGB plan), **70.450 MHz** (commonly used 4 m FM calling/net reference), **145.500 MHz** (2 m FM calling) and **433.500 MHz** (70 cm FM calling). The RSGB 70 MHz page lists 70.294–70.500 MHz as a 12.5 kHz channelised segment; treat 70.450 MHz as a commonly used operating reference rather than a separately reserved channel. Use the current RSGB plan for mode and segment detail.

### Common UK amateur repeater shifts (receive output → transmit input)

| Band / system | Repeater output to receive | Radio TX offset to input | Notes |
|---|---:|---:|---|
| 2 m | 145.6000–145.7875 MHz | **−0.600 MHz** | RV48–RV63 main output channels; low-power outputs also exist at 145.5750 and 145.5875 MHz. |
| 70 cm, 1.6 MHz system | 432.994–433.381 MHz | **+1.600 MHz** | RU240–RU270 outputs; nominal channel centres are 433.000–433.375 MHz at 12.5 kHz spacing; inputs are 434.594–434.981 MHz. |
| 70 cm, 7.6 MHz system | 430.825–430.975 MHz | **+7.600 MHz** | RU66–RU78 outputs; inputs are in 438.425–438.575 MHz. |

The sign shown is from the handheld's point of view when it receives the repeater output and transmits to its input. Local repeater details and coordinated channels can change; confirm a specific repeater's pair before programming it.

### Other useful reference frequencies

- **Civil airband:** 118–137 MHz AM; set the VFO step to 8.33 kHz for 8.33-channel tuning.
- **Maritime VHF:** Channel 16 is 156.800 MHz (25 kHz channel plan; FM). It is a distress/safety/calling channel, not a general-purpose test frequency.
- **ISS amateur cross-band repeater:** ARISS lists **437.800 MHz downlink** and **145.990 MHz uplink with 67 Hz tone** for the Columbus station. This is a receive reference, not a guarantee that the radio is active at any particular moment.
- **UK CB:** CEPT channels occupy 26.965–27.405 MHz; the separate UK 40-channel band occupies 27.60125–27.99125 MHz.
- **Historic NOAA APT:** 137.100, 137.620 and 137.9125 MHz are the retired NOAA-19, NOAA-15 and NOAA-18 APT frequencies; the NOAA POES constellation was decommissioned in August 2025.

## 13.2 What this radio does not decode

The default firmware does not demodulate or decode:

- DAB digital radio;
- DVB-T/T2 digital television;
- DMR or dPMR digital voice;
- TETRA public-safety/business systems;
- P25 or other digital voice standards;
- LTE/5G cellular signals;
- ADS-B aircraft position data;
- GNSS/GPS navigation data;
- most FSK/LoRa/telemetry/data packets;
- encrypted or trunked digital radio systems.

The waterfall may show energy from these systems, but a trace peak is not the same as a decodable voice transmission.

---

# 14. Recommended receive-only setup

This is a conservative baseline for learning the firmware and using the UV-K5 as a receiver.

## 14.1 Main settings

| Setting | Suggested value | Reason |
|---|---|---|
| RxMode | MAIN ONLY | Avoids unnecessary Dual Watch/cross-band behaviour |
| Sql | 1 initially | Sensitive starting point; adjust for local noise |
| Demodu | FM for ordinary analogue VHF/UHF voice; AM for AM services; USB for upper-sideband hobby signals | Match the signal's modulation |
| W/N | Narrow for 12.5 kHz land-mobile channels; Wide where the signal requires it | Match channel bandwidth |
| RxCTCS / RxDCS | OFF unless you know the code | Prevents the receiver muting signals with an unknown tone/code |
| BatTxt | Voltage or Percent | Gives useful battery information |
| BackLt | 10–20 seconds or ON while bench testing | Choose for visibility and battery life |
| Beep | Preference | Key feedback only |
| AM Fix | ON | Recommended for AM reception in this build |
| F Lock (hidden menu) | DISABLE ALL | Firmware transmit inhibit for receive-only operation |

## 14.2 A sensible PMR446 receive profile

- VFO mode, 400 MHz band group.
- Frequency: one of the 16 analogue PMR446 channel centres listed in section 10.1.
- Step: 12.5 kHz.
- Demodulation: FM.
- Bandwidth: Narrow.
- RxCTCS and RxDCS: OFF.
- Squelch: 1, adjusted to suit the noise floor.
- F Lock: DISABLE ALL.

Save frequently used channels in memory only for communications you are entitled to receive. Give each stored channel a meaningful name.

## 14.3 Scan-list strategy

Do not scan every saved channel if you only care about a few. Put the channels you actually use into Scan List 1, keep Scan List 2 for a second group, then set `SList` to the desired list. This reduces scan time and missed short transmissions.

## 14.4 First-time configuration checklist

A sensible setup sequence after flashing is:

1. **Choose the transmit policy first.** For a receiver-only radio, enter the hidden menu and set `F Lock` to **DISABLE ALL** (see section 8.2).
2. **Set ordinary receive behaviour.** Use `RxMode` = MAIN ONLY, `Sql` = 1 as a starting point, and turn `RxCTCS` and `RxDCS` OFF unless you specifically need selective squelch for a system you are authorised to monitor.
3. **Set channel defaults to suit the service.** Use FM and Narrow for 12.5 kHz analogue PMR446 channels. Select AM for AM services and USB only for upper-sideband signals. Turn `AM Fix` ON for AM listening in this build.
4. **Choose a useful battery display.** Set `BatTxt` to VOLTAGE or PERCENT and select `BatTyp` for the battery actually fitted. Adjust `BatCal` only after comparing the displayed voltage with a reliable multimeter at the battery terminals.
5. **Make the display usable.** Select a backlight timeout and sensible `BLMin`/`BLMax` values. `ChDisp` = NAME + FREQ is useful in MR mode if you name your memory channels.
6. **Set convenient side-key actions.** For example, Side1 short = Monitor, Side1 long = Switch Demodulator, Side2 short = Flashlight, Side2 long = Scan. This is only a suggested ergonomic layout; choose actions that you will recognise and not trigger accidentally.
7. **Check scan configuration.** Put only the memory channels you want into Scan List 1 or 2, then set `SList` to the list you intend to scan.

The `350 En` option controls whether the 350–400 MHz band group is available in VFO band selection (and is also part of that segment's TX-lock check). It is not needed for PMR446, and changing it does not improve receiver sensitivity.

## 14.5 Back up calibration data before programming

The source loads factory calibration values from the upper EEPROM region, including receive-calibration data around `0x1EC0`/`0x1EC8`, battery calibration around `0x1F40`, and further radio calibration values near `0x1F88`. Before using a PC programming tool that can write raw EEPROM data, make a full read/backup where the tool supports it and keep the original file safe. A settings reset is not the same as a raw EEPROM overwrite: the firmware's factory-reset routine deliberately leaves the upper calibration region intact, but an external programmer that writes the wrong image can still damage calibration. Restore only a backup made from that exact radio.

Never edit raw EEPROM offsets just because a guide lists them. Use a programming application and driver version that explicitly supports your radio and this firmware build.

## 14.6 Antenna and reception advice

- Use an antenna designed for the band you are listening to.
- Keep the antenna vertical for vertically polarised handheld/land-mobile signals unless you know the station uses another polarisation.
- Test in an open location away from computers, USB hubs, switching power supplies and other local noise sources.
- If the spectrum screen shows a forest of signals or AM audio becomes distorted in a strong RF area, try moving location or using an appropriate attenuator/filter rather than immediately increasing gain.
- Do not assume an external high-gain antenna will improve every band. The receiver's wide-open front end can overload.

---

# 15. Practical troubleshooting

| Symptom | Likely cause | What to try |
|---|---|---|
| Frequency entry is rejected | Frequency outside the firmware band table or in the 630–840 MHz gap | Check the tuning range in section 1.4; choose a frequency supported by the BK4819 ranges |
| No audio, but the frequency looks right | Wrong modulation, squelch too high, or RX CTCSS/DCS mismatch | Set FM/AM correctly, reduce SQL, set RxCTCS/RxDCS OFF for an initial test |
| PMR446 signal is visible but silent | Trigger is too high, bandwidth too wide/narrow, or digital PMR446 signal | Lower trigger with F; set FM and 12.5 kHz; remember dPMR is not decoded |
| Waterfall stops constantly | Trigger below local noise floor or a strong unwanted peak dominates | Raise trigger with `*`; blacklist the peak with Side1; try a quieter location |
| Waterfall never pauses | Trigger too high or the signal is weak | Lower trigger with `F`; check correct VFO frequency and scan range |
| Peak is visible but voice is distorted | Wrong demodulation, wrong listen bandwidth or front-end overload | Try FM/AM/USB as appropriate and cycle bandwidth with `6`; move away from strong RF sources |
| A channel is silent despite a strong signal | Incorrect RxCTCSS/RxDCS | Set both to OFF, then identify any required code only if appropriate |
| Scan misses short signals | Fast-scan build and short dwell time | Use fewer channels, scan a smaller list, or use manual tuning |
| Memory channel does not appear in scan | Channel is not valid or not included in the selected scan list | Check ScAdd1/ScAdd2 and `SList` |
| Battery percentage is implausible | Wrong battery type or calibration | Set BatTyp correctly; adjust BatCal only against a reliable voltage meter |
| FM broadcast mode will not open | FM receiver feature disabled in that build | Check `ENABLE_FMRADIO` in the Makefile used to build the firmware |
| F+5 does not open the spectrum screen | Spectrum not compiled, or NOAA feature takes the F+5 path | Check `ENABLE_SPECTRUM` and `ENABLE_NOAA` in the build options |
| Menu item is missing | Its feature is disabled at compile time | Compare your build's Makefile with section 2 |
| Radio receives badly on several bands | Wide-open receiver front-end overload, antenna mismatch or strong local RF | Test in another location, use a band-appropriate antenna or a suitable filter/attenuator |

## 15.1 Before changing hidden calibration values

Do not use `FrCali`, `BatCal`, the IF register or the LNA/PGA controls as general-purpose “make reception better” settings. These are calibration or low-level controls. An arbitrary value can make the radio less accurate or less sensitive. Record the original value before changing anything, and use proper test equipment where calibration is genuinely required.

## 15.2 Before flashing or changing firmware builds

The repository warns that firmware is used at the operator's own risk and that an incompatible build can render a radio unusable. Before flashing:

1. Confirm the exact radio model and hardware revision.
2. Back up the current configuration/EEPROM where your flashing tool supports it.
3. Confirm that the firmware build is intended for the radio's hardware.
4. Keep a known-good firmware file available for recovery.
5. Do not assume every UV-K5, UV-K6, UV-5R variant or clone has identical hardware.

The repository README links to the project's recommended UV Tools flasher. Follow the current flasher instructions and the specific hardware guidance for your radio.

---

# 16. Source map and authoritative references

## 16.1 Firmware source files used to build this manual

The user-facing behaviour described here was traced primarily through these repository files:

| Source file | What it establishes |
|---|---|
| `README.md` | Firmware lineage, features, build warning, wide-open receiver limitations and feature overview |
| `Makefile` | Default compile-time feature switches |
| `app/main.c` | Main screen keypad functions and F-key combinations |
| `app/generic.c` | F-key modifier, PTT and DTMF input/transmit handling |
| `ui/menu.c` | User-visible menu item names and option strings |
| `app/menu.c` | Menu limits, setting acceptance, menu key handling and setting application |
| `settings.c` / `settings.h` | EEPROM settings, defaults, calibration and conditional options |
| `app/spectrum.c` / `app/spectrum.h` | Spectrum analyser defaults, sweep calculation, listen behaviour, frequency input and every spectrum/STILL key handler |
| `frequencies.c` / `frequencies.h` | Frequency bands, tuning steps and RX/TX frequency checks |
| `radio.c` / `radio.h` | VFO configuration, modulation choices, RX/TX setup and channel data |
| `app/chFrScanner.c` | Channel/frequency scan and scan-resume behaviour |
| `app/scanner.c` / `ui/scanner.c` | Frequency search, CTCSS/DCS detection and scanner save process |
| `app/fm.c` / `ui/fmradio.c` | FM broadcast radio operations and controls |
| `helper/boot.c` | Egzumer application startup shortcuts: PTT + Side1 for F Lock, and PTT + Side2 for AirCopy when compiled |
| `ui/main.c` / `ui/status.c` | Main screen labels, status icons and channel display modes |

Repository: <https://github.com/egzumer/uv-k5-firmware-custom>

Project README: <https://github.com/egzumer/uv-k5-firmware-custom/blob/main/README.md>

Spectrum source: <https://github.com/egzumer/uv-k5-firmware-custom/blob/main/app/spectrum.c>

Menu definitions: <https://github.com/egzumer/uv-k5-firmware-custom/blob/main/ui/menu.c>

Main keypad handling: <https://github.com/egzumer/uv-k5-firmware-custom/blob/main/app/main.c>

Frequency checks: <https://github.com/egzumer/uv-k5-firmware-custom/blob/main/frequencies.c>

Default build options: <https://github.com/egzumer/uv-k5-firmware-custom/blob/main/Makefile>

The upstream project Wiki is also linked from the README: <https://github.com/egzumer/uv-k5-firmware-custom/wiki>

## 16.2 UK regulatory references

1. **Ofcom — Business Radio Guidance (published 5 December 2025), section 9 PMR446.** Covers 446.0–446.2 MHz, 500 mW ERP, mobile-only/no base stations, no airborne use, non-interference basis and the equipment conditions for licence exemption.  
   <https://www.ofcom.org.uk/siteassets/resources/documents/spectrum/business-radio-licences/business-radio-guidance-document.pdf>
2. **Ofcom — Rules on using radio equipment.** Explains general radio reception, scanners, unauthorised interception and disclosure.  
   <https://www.ofcom.org.uk/spectrum/radio-equipment/radio-spectrum-and-the-law>
3. **Wireless Telegraphy Act 2006, section 48.** Statutory text covering interception and disclosure of messages.  
   <https://www.legislation.gov.uk/ukpga/2006/36/section/48>
4. **Ofcom — UK Frequency Allocation Table (UKFAT), 2026 edition.** The full UK allocation reference; Ofcom lists the 2026 table as last updated 3 July 2026.  
   <https://www.ofcom.org.uk/spectrum/frequencies/uk-fat>
5. **Ofcom — Amateur radio licence terms and conditions.** Lists the UK amateur frequency bands and their primary/secondary status and conditions, including 50–52 MHz, 70.0–70.5 MHz, 144–146 MHz and 430–440 MHz.  
   <https://www.ofcom.org.uk/__data/assets/pdf_file/0027/62991/amateur-terms.pdf>
6. **Analogue PMR446 channel plan.** The frequency list in section 10 is the standard 16-channel 12.5 kHz analogue plan within 446.0–446.2 MHz. The Ofcom guidance is the authority for UK use conditions; consult your radio manufacturer's current channel table for tone/code defaults.
7. **Radio Society of Great Britain (RSGB) — UK Band Plans, January 2026.** Use the online master for current segment and channel information. This manual links directly to the 70 MHz, 144 MHz and 432 MHz band pages for the examples in section 13.1.
   <https://rsgb.org/main/operating/band-plans/>
   <https://rsgb.org/main/operating/band-plans/vhf-uhf/70mhz-band/>
   <https://rsgb.org/main/operating/band-plans/vhf-uhf/144mhz-band/>
   <https://rsgb.org/main/operating/band-plans/vhf-uhf/432mhz-band/>
8. **Ofcom — Citizens' Band (CB) radio spectrum use.** Official UK and CEPT channel tables and equipment requirements.
   <https://www.ofcom.org.uk/siteassets/resources/documents/spectrum/spectrum-enforcement/citizens-band.pdf>
9. **Ofcom — Information for maritime radio licensees.** Ofcom lists the maritime interface requirements, including IR 2026 for VHF radiotelephone equipment for general communications and associated Class D DSC equipment. This is an equipment requirement, not a general maritime channel-allocation table.
   <https://www.ofcom.org.uk/spectrum/radio-equipment/maritime-information>
   <https://www.ofcom.org.uk/siteassets/resources/documents/spectrum/interface-requirements/ir_2026.pdf>
10. **ARISS — Current Status of ISS Stations.** Current station configuration and the 145.990 MHz / 437.800 MHz cross-band voice repeater pair; status can change.
   <https://www.ariss.org/current-status-of-iss-stations.html>
11. **NOAA OSPO — POES spacecraft status archive.** Records the NOAA-15, NOAA-18 and NOAA-19 APT carrier frequencies and confirms the POES constellation's 2025 decommissioning.
   <https://ospo.noaa.gov/operations/poes/status.html>

---

## Final quick-reference card

| What you want to do | Keys / menu |
|---|---|
| Open menu | `MENU` |
| Switch VFO A/B | `F + 2` |
| Switch VFO / memory | `F + 3` |
| Change frequency band group | `F + 1` |
| Change modulation | `Demodu` menu or configured Switch Demodulator side key |
| Start channel/frequency scan | Long `* SCAN` |
| Frequency search | `F + 4` |
| CTCSS/DCS search on current signal | `F + * SCAN` |
| Open spectrum analyser | `F + 5` |
| Spectrum: finer/wider measurement interval | `1` / `7` |
| Spectrum: change scan-range movement step | `2` / `8` |
| Spectrum: change dB scale top | `3` / `9` |
| Spectrum: change point count | `4` |
| Spectrum: enter frequency | `5`, digits, `*` decimal, `MENU` accept; centre below 2.5 kHz scan step, lower edge at 2.5 kHz and above |
| Spectrum: change listen demodulation | `0` |
| Spectrum: change listen bandwidth | `6` |
| Spectrum: lower/raise listen trigger | `F` / `*` |
| Spectrum: ignore current peak | Side1 |
| Spectrum: tune peak / STILL | PTT |
| Spectrum: exit to radio | `EXIT` (from STILL, first EXIT clears register selection if active) |
| Enter hidden menu | Power on holding PTT + Side1 |
| Block all firmware TX | Hidden `F Lock` → `DISABLE ALL` |
| PMR446 analogue receive | VFO, FM, Narrow, 12.5 kHz Step, RxCTCS/RxDCS OFF, one of the 16 channel centres |

**Transmit legality and equipment compliance are covered in section 1.** The PMR446 receive profile above is not a transmit configuration.