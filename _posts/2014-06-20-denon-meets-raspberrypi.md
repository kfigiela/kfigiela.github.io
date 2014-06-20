---
layout: post
title: Denon DRA-F109 meets Raspberry Pi
---

As I described in [previous post](/2014/06/15/denon-remote-connector/), it is possible to connect Denon DRA-F109 to Raspberry Pi not only with audio cable, but also with *remote connector*. In this post I describe, what integration scenario I use.

---

### The setup

I connected Raspberry Pi audio output to *Network player* receiver input as it is most reasonable input. On the Raspberry Pi I run [mpd](http://www.musicpd.org) as media player and [Shairplay](https://github.com/juhovh/shairplay) for AirPlay support. I do not use dmix or PulseAudio, as I try to avoid signal resampling. In `tty1` which is set to auto-login I run [ncmpcpp](http://ncmpcpp.rybczak.net) in [fbterm](https://code.google.com/p/fbterm/) with big font to be able to control it from sofa. Initially, it may look weird, but actually it is very well designed UI that may be controlled by remote control. In addition to (big) LCD screen that may be turned off I also connected 2x16 HD44780 display.

### Requirements

What I really wanted to do was:

* use standard controls as Previous, Next, Play/Pause,
* use Rewind and Forward for skipping albums,
* use cursors to navigate through ncmpcpp,
* toggle beween playlist and browser,
* add and songs tracks from playlists,
* be able to turn on and off the display,

All these, only if the receiver is set to *Network* source. Some buttons are used also by tuner, so using IR receiver was not so convenient.

### Additional features

*Remote connector* is more than just passing button press information. You get information, when the alarm clock is set, when the receiver is turned off and on, what radio stations are programmed. This allows to get some more features:

* stop music playback when receiver source is changed to any other input (also works when physical button on receiver is used),
* start music playback when *Network* source is selected,
* control backlight of HD44780 with receiver display dimmer,
* start playback when receiver is turned on by alarm clock,
* toggle between MPD and AirPlay modes,
* turn off HD44780 backlight and stop playback when receiver is put into standby mode (either by button, remote or sleep timer).

### The code

The ruby scripts that control MPD, Denon, Shairplay and LCDproc display are available under MIT license at [GitHub repository](https://github.com/kfigiela/denon-raspberry). They are build specifically for my setup, but should be good staring point for integrating with Denon receiver.

### Future work: one remote to rule them all

I own old Harman Kardon CD player. It should be doable to implement IR code translator with Raspberry Pi. As soon as I buy some IR leds I will experiment with that. 