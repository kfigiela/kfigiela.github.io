---
layout: post
title: Hacking Denon DRA-F109 remote connector
---

Recently I bought Denon DRA-F109 Stereo Receiver. In this post I describe how the receiver may be integrated with Raspberry Pi by using receiver's *Remote Connector*, enabling to use Denon remote to control both devices. Integration covers also alarm clock built into the DRA-F109, display dimmer, sleep timer and power control, as it was native Denon system device.

----

### Hardware setup

The device itself is only tuner with amplifier. It also has couple of external inputs (2 coaxial, 1 optical and 2 analog). The other parts F109 system are CD player (DCD-F109) and network player (DNP-F109) that are connected using coaxial inputs. The remote control bundled with DRA-F109 is an overkill for just amplifier, but you can connect all three devices if they are connected using *Remote Connector* cable which is 3.5 mm TRS connector. 

Since I have only the receiver and use [Raspberry Pi](http://raspberrypi.org) with [HiFiBerry Digi](http://hifiberry.com/hbdigi) as an audio player I wanted to use Denon remote to controll both. Till date I was using a generic remote with [TSOP4838](http://www.vishay.com/docs/82459/tsop48.pdf) IR receiver connected to Raspberry GPIO pin, then IR codes were translated into actions by [lirc](http://lirc.sourceforge.net). This was also working with Denon remote, but was in some way limited: buttons except input selection, volume control, etc. didn't do anything when receiver was in external input, but some of them actually did have function in tuner mode (e.g. rewind/forward buttons are used to tune radio). That was motivation to find a better solution.

![Denon DRA-F109](/img/2014-06-15-denon-remote-connector_receiver.jpg)

### Hacking remote connector

> Remember, you may break your Raspberry or Denon receiver. Try at own risk!

At present, nobody wants to reinvent the wheel. So expected that they must have used standard bus for data transmission in *Remote Connector*. The initial guesses were: IR receiver (TSOP4838-like output), RS232 and I<sup>2</sup>C. I expected that bus requires two way communication for Network Player, so serial connection seemed the most possible as UART is present in most µC. First of all, I needed to determine pinout – I took the multimeter and when buttons were pressed there was voltage drop on ring of TRS plug. I took 3.3V USB-to-Serial adapter, two resistors (1.8kΩ and 3.2kΩ in my case) to make [voltage divider](http://en.wikipedia.org/wiki/Voltage_divider) (Denon outputs signal in 5V logic) and started to experiment. Suprisingly, it worked! The data is transmitted at standard 115.2 kbps. The next step was to connect it to Raspberry Pi UART RX pin.

> **Wiring diagram**
> ![Wiring diagram](/img/2014-06-15-denon-remote-connector_circuit.png) 

For now, it is enough just to receive data from the receiver – it is also the easier part to implement as it requires only voltage divider and the protocol may be reverse engineered from data we receive. Unfortuanetly, tip of the connector which is supposed to be RX of the receiver is at 5V when floating. I don't have 5V USB-to-Serial adapter and definetly don't want to fry my Raspberry Pi, so for now, we will keep on receiving data from DRA-F109.

> **Note:** It is important to disable TTY and kgdb at serial port in `/boot/cmdline.txt` and in `/etc/inittab`. Otherwise your Raspberry Pi will crash (or at least will be halted by kgdb via serial).

![Denon RC-1163 Remote Control](/img/2014-06-15-denon-remote-connector_remote.jpg)

### The protocol

The next challenge was to decode the incoming data. Stream is organized in packets. The packet format is `0x00 0xff 0x55 [payload length] 0x00 0x00 [id] [destination] [payload]`. Sometimes, but not always there is one or two more bytes sent by amp, but they don't seem to be important – possibly it is some kind of padding.

#### Denon AVR serial protocol

The receiver implements two message formats. If the `id` is set to `0x80` and destination is `0x00` then payload follows the Denon AVR serial protocol present in Home Theater receivers in form of standard RS232 port and it has [official documentation](http://www.ip-symcon.de/forum/attachment.php?attachmentid=23493&d=1384502367). Commands are ASCII with trailing `\r`. DRA-F109 implements a small subset of the protocol as it is only stereo amp, but still any documentation is better than nothing. Till now I identified the following commands (in [regular expression](http://en.wikipedia.org/wiki/Regular_expression) notation), the list for sure lacks DAB+ messages as I don't have DAB+ broadcast at my place yet:

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
* `SITUNER`: Source set to Tuner.
* `SINETWORK`: Source set to Network Player.
* `SIAUX1`: Source set to Analog In 1.
* `SIAUX2`: Source set to Analog In 2.
* `SIDIGITAL_IN`: Source set to Digital In.
* `TMANFM`: Tuner band set to FM.
* `TMANMANUAL`: Tuner set to manual (mono) mode.
* `TMANAUTO`: Tuner set to automatic stereo mode.
* `TPAN(\d\d)`: Tuned to preset `$1`.
* `TPANOFF`: Tuned out of preset (manual tuning or tuner off).
* `TFAN(\d{6})`: Tuner tuned to FM `$1/100.0` MHz.
* `SSTPN(\d\d)(.{9})(\d{8})`: Preset `$1` is station named `$2` at `$3/100.0` MHz.
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

I don't have DNP-F109 player to see what is available in order to control the receiver. Probalby, the same packet format will work for controlling the receiver.

#### Binary protocol

For the other packets, payload length is usually 1. If three bytes are given below in this section consider it as `[id] [destination] [payload]`.

##### Source and function selection

When function selection button in pressed on the remote the receiver sends:

* Network Player
  * `0x61 0x00 0x00`: Online Music 
  * `0x62 0x00 0x00`: Internet Radio
  * `0x63 0x00 0x00`: Music Server
  * `0x64 0x00 0x00`: iPod/USB
* CD Player
  * `0x5f 0x00 0x00`: CD
  * `0x60 0x00 0x00`: iPod/USB
  
It also sends when source is changed (also by mechanical button on the receiver) – these are less interesting, as the same data is also sent using AVR protocol:

* `0x01 0x03 0x00` and `0x33 0x00 0x9b`: Network Player
* `0x01 0x04 0x00` and `0x33 0x14 0x00`: CD Player
* `0x01 0x05 0x00` and `0x33 0x15 0x00`: Analog In 1
* `0x01 0x06 0x00` and `0x33 0x16 0x00`: Analog In 2
* `0x01 0x07 0x00` and `0x33 0x17 0x00`: Digital In
* `0x33 0x08 0x00`: Tuner (both FM and DAB)

##### Network and CD control

Depending on selected source (CD or Network Player) the `destination` field is `0x25` for CD player or `0x26` for network player and the payload is always `0x00`. The button presses are mapped to ids as follows:

* `0x32`: Play/Pause
* `0x33`: Stop
* `0x34`: Play (sent by Amp by Alarm function)
* `0x44`: Next
* `0x45`: Previous
* `0x46`: Forward
* `0x47`: Rewind
* `0x48`: Up
* `0x49`: Down
* `0x4a`: Left
* `0x4b`: Right
* `0x4c`: Enter
* `0x4d`: Search
* `0x4e`: Mode
* `0x4f`: 1
* `0x50`: 2
* `0x51`: 3
* `0x52`: 4
* `0x53`: 5
* `0x54`: 6
* `0x55`: 7
* `0x56`: 8
* `0x57`: 9
* `0x58`: 0
* `0x59`: +10
* `0x5a`: Clear
* `0x5c`: Random
* `0x5d`: Repeat

For example, packet of `0x00 0xff 0x55 0x01 0x00 0x00 0x32 0x26 0x00` means *Play/pause* Network Player. 

##### Special functions

The dimmer function sends `0x43 0x00 [brightness]` where brightness may be 0, 1, 2 or 3 which means bright, dim, dark, display off respectively.

The receiver also sends the following commands:

* `0x84 0x00 0x00` when the System Settings are opened,
* `0x02 0x01 0x00` when the receiver goes to stand-by,
* `0x42 0x00 0x00` when the receiver is turned on,

#### What is missing?

The receiver does not send anything if *Add*, *Call*, *Search* and *Network Setup* buttons are pressed on the remote. These buttons are used exclusively by network player. TSOP receiver is still needed at Raspberry Pi to handle these buttons. Unfortuanetly, it seems to have a little worse reception than receiver one.

### The code

The ruby code is available under MIT license at [GitHub repository](https://github.com/kfigiela/denon-raspberry). The `demo.rb` is example of receiving data from Denon – it is not Rasbperry Pi specific. The repository includes my setup that is described in [second post](/2014/06/20/denon-meets-raspberrypi/)
