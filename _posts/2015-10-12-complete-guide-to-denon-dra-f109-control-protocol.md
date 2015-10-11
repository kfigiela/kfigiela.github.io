---
layout: post
title: Complete guide to Denon DRA-F109 control protocol
---

# Hardware specs

Denon DRA-F109/DNP-F109/DCD-F109 remote connector is in fact TTL-level (5V) serial port. The connector is  3.5mm TRS jack (standard stereo plug). Tip is RX, ring is TX, sleeve is ground. Data is transmitted at 115.2 kbps baud rate. It may be easily connected to non-Denon hardware by using cheap 5V tolerant USB-to-Serial adapter. The receiver (DRA-F109) echoes back data recieved from other devices.


----


# Data protocol

> **Note:** `00` in this chapter means `0x00` (in hex)

We may say that byte stream is organized in packets. Each packet starts with serial port [break condition](http://stackoverflow.com/questions/14803434/receive-read-break-condition-on-linux-serial-port). On POSIX compilant systems it is represented as `00` (null) byte when reading from the device. Break should have 16.25 ms (important when transmitting to the receiver) Packet has the following structure: `BRK ff 55 length direction 00 <data> checksum`, where:

* `length` is length of data minus 2,
* `direction` is `0` when receiving from DRA or `1` when sending command to it,
* `checksum` is one byte sum of all bytes, however it is sometimes missing when the value is supposed to be `19`

The receiver implements two protocol: text-based and binary.

## Text-based protocol

This protocol is only used when receiving data from the DRA, it does not react to sending such commands (we may be doing that wrong). This is a subset of Denon AVR serial protocol present in Home Theater receivers in form of standard RS232 port and it has [official documentation](http://www.ip-symcon.de/forum/attachment.php?attachmentid=23493&d=1384502367).

The `<data>` of packet has the following structure: `80 00 <command> 0d`, and possible commands (in regular expression notation) are:

* `MV(\d\d)`: Volume set to `$1`.
* `MUON`: Mute on.
* `MUOFF`: Mute off.
* `SLP(\d\d\d)`: Sleep timer set to `$1` minutes.
* `SLPOFF`: Sleep timer disabled.
* `PSBAS(\d\d)`: Equalizer bass set to `$1`.
* `PSTRE(\d\d)`: Equalizer treble set to `$1`.
* `PSBAL(\d\d)`: Balance set to `$1`.
* `PSSDB (ON|OFF)`: SDB tone turned on or off.
* `PSSDI (ON|OFF)`: Source direct turned on or off.
* `SICD`: Source set to CD.
* `SITUNER`: Source set to Tuner. That one doesn't have checksum!
* `SINETWORK`: Source set to Network Player.
* `SIAUX1`: Source set to Analog In 1.
* `SIAUX2`: Source set to Analog In 2.
* `SIDIGITAL_IN`: Source set to Digital In.
* `TMANFM`: Tuner band set to FM.
* `TMDA`: Tuner band set to DAB+.
* `TMANMANUAL`: Tuner set to manual (mono) mode.
* `TMANAUTO`: Tuner set to automatic stereo mode.
* `TPAN(\d\d)`: Tuned to preset `$1`.
* `TPANOFF`: Tuned out of preset (manual tuning or tuner off).
* `TFAN(\d{6})`: Tuner tuned to FM `$1/100.0` MHz.
* `TFDA(\d\d?[A-Z]})`: Tuner tuned to DAB+ `$1` frequency block (e.g. `12D`). Unfortuanetly, there is no information on tuned station (except if preset was tuned, then we get `TPAN(\d\d)`).
* `SSTPN(\d\d)(.{9})(\d{8})`: Preset `$1` is station named `$2` at `$3/100.0` MHz. Frequency is always 999999.99 MHz for DAB+ stations.
* `PWSTANDBY`: Amp is going to stand by.
* `PWON`: Amp turned on.
* `TS(ONCE|EVERY) 2(\d\d)(\d\d)-2(\d\d)(\d\d) (..)(\d\d)`: Alarm of type `$1` set to turn on the device at `$2`:`$3` and turn it off at `$4`:`$5` at function `$6` with preset `$7`, where function is:
  * `NW`: Internet Radio,
  * `NU`: iPad/USB (Network),
  * `CD`: CD,
  * `CU`: iPad/USB (CD),
  * `TU`: Tuner,
  * `A1`: Analog In 1,
  * `A2`: Analog In 2,
  * `DI`: Digital.
* `TO(ON|OFF) (ON|OFF)`: Alarm once is `$1` and every is `$2`.

## Binary protocol

Bi-directional communication is possible with simple binary protocol. Some of it's features overlap with text-based protocol.

### Source and function selection

When function selection button in pressed on the remote:

* Network Player
  * `61 00 00`: Online Music
  * `62 00 00`: Internet Radio
  * `63 00 00`: Music Server
  * `64 00 00`: iPod/USB
* CD Player
  * `5f 00 00`: CD
  * `60 00 00`: iPod/USB

When receiver wants to turn on specific device:

* `01 03 00`: Network Player
* `01 04 00`: CD Player
* `01 05 00`: Analog In 1
* `01 06 00`: Analog In 2
* `01 07 00`: Digital In

When source is changed (also by mechanical button on the receiver)

* `33 00 9b`: Network Player
* `33 14 00`: CD Player
* `33 15 00`: Analog In 1
* `33 16 00`: Analog In 2
* `33 17 00`: Digital In
* `33 08 00`: Tuner (both FM and DAB)

### Control of input devices

The receiver forwards key-presses on the remote control if source other than tuner is selected: this data frames will have the following structure `button destination 00`.

Depending on selected source on the receiver the `destination` field is:

* `23` for Analog 1,
* `24` for Analog 2,
* `25` for CD player,
* `26` for network player,
* `27` for Digital.

Buttons have the following identifiers:

* `32`: Play/Pause
* `33`: Stop
* `34`: Play (sent by Amp by Alarm function)
* `44`: Next
* `45`: Previous
* `46`: Forward
* `47`: Rewind
* `48`: Up
* `49`: Down
* `4a`: Left
* `4b`: Right
* `4c`: Enter
* `4d`: Search
* `4e`: Mode
* `4f`: 1
* `50`: 2
* `51`: 3
* `52`: 4
* `53`: 5
* `54`: 6
* `55`: 7
* `56`: 8
* `57`: 9
* `58`: 0
* `59`: +10
* `5a`: Clear
* `5c`: Random
* `5d`: Repeat

> **Example:** Packet of `BRK ff 55 01 00 00 32 26 00 ad` (checksum omited) means *Play/pause* Network Player.

### Dimmer

The dimmer function sends `43 00 [brightness]` where brightness may be 0, 1, 2 or 3 which means *bright*, *dim*, *dark*, *display off* respectively.

### Other

The receiver also sends the following commands:

* `42 00 00` when the receiver is turned on,
* `02 01 00` when the receiver goes to stand-by,
* `84 00 00` when the System Settings are opened.

### What is missing?

The receiver does not send anything if *Add*, *Call*, *Search* and *Network Setup* buttons are pressed on the remote. These buttons are exclusive for Network Player. You'll need additional IR receiver to handle them i.e. with LIRC (see [config file](https://gist.github.com/kfigiela/614ca5e986ded3510e68)).


## Controlling the DRA-F109 receiver

It is possible to control the receiver using binary protocol (remember to set `direction` to 1), for most commands `length` is 1. Till now we've identified the following commands:

* Set source and power on if in standby
  * Network Player:  `24 00 00`
  * CD:       `23 00 00`
  * Analog 1:     `25 00 00`
  * Analog 2:     `26 00 00`
  * Digital:  `27 00 00`
  * Tuner (FM):    `20 00 00`
  * Tuner (DAB):    `22 00 00`
* Set source, send play command and power on if in standby:
  * Network Player:  `14 00 00`
  * CD:       `13 00 00`
  * Analog 1:     `15 00 00`
  * Analog 2:     `16 00 00`
  * Digital:  `17 00 00`
  * Tuner (FM):    `10 00 00`
  * Tuner (DAB):   `12 00 00`
* Power On the Receiver: `01 02 00` (compare with received `01 03 00`, etc.)
* Power Off the Receiver: `02 01 00` (possibly should work also with NP and CD, second byte could be device id as in `01 03 00`, etc.)
* Power Off the Receiver: `85 00 00` (possibly turns off all devices)
* Set volume:   `40 00 00-3c` (higher values also interpreted as max)
* Mute on/off:  `41 00 01/00`
* Sound settings:
  * SDB on/off: `42 00 00 00/01`
  * Bass increase/decrease: `42 00 01 00/01`
  * Treble increase/decrease: `42 00 02 00/01`
  * Balance to left/right: `42 00 03 00/01`
  * S. Direct: `42 00 04 00/01`
* Dimmer: `43 00 00-03` (Note: no feedback for this command)
* Tuner preset forward/backward: `68 30 00/01`
* Set sleep timer: `6b 00 time`, `time` in minutes (amp will display only last two digits, but should count up to 255 minutes), 0 to disable
* Set auto standby On/Off: `83 00 01/00`
* Set alarm: `type 00 00 hh mm 00 hh mm <src>`, where `type` is `88` for "once" or `89` for "every", and the `<src>` is
  * Tuner: `00 preset`
  * Analog 1: `01 00`
  * Analog 2: `02 00`
  * Digital: `03 00`
  * Network: `04 00`
  * Network (USB): `05 00`
  * CD: `06 00`
  * CD (USB): `07 00`
* Alarm "every" On/Off: `8a 00 00/01` (**Note:** not sure about that one, needs more investigation how turn once alarm)
* Emulate remote keypress: `48-5d 00 00` – key numbers as in recieved ones, can be used to select i.e. radio station, navigate menu, etc.

Note that for the most of commands, DRA will provide feedback (usually with text-based protocol).

> **Example:** Set volume to 10: `BRK FF 55 01 01 00 40 00 0a a0`.

> **Warning:** Sending too short packet will hang the receiver (need to restart by disconnecting power).


## Example implementation

Example implementation of the protocol in Ruby is available on GitHub:[kfigiela/denon-raspberry](https://github.com/kfigiela/denon-raspberry) where I keep code of my setup.

