---
layout: post
title: Traktor Kontrol S5 MIDI mapping
---

One of the activities I do in my free time is DJing. Few months ago I bought Traktor Kontrol S5 that features two LCD displays. Unfortuanetly, customization possibilities are very limited. I [discovered](https://www.native-instruments.com/forum/threads/s5-s8-needs-spectrum-waveform-colors-on-display.267253/#post-1475226) then that displays & controller logic is implemented in QML and the source files are bundled with Traktor. This was really great, as it opened possibility for major customizations. Later on, some folks did also [even more](https://www.native-instruments.com/forum/threads/s8-s5-display-mods.288222/) hacking.

However, it is possible to modify Traktor S5/S8/D2 functionality even more. One of the missing functions of S5 is ability to change default mapping. It is possible to map some of the buttons and faders of Traktor S8 to MIDI, however it is not available for S5. Luckily, buttons are wired to Traktor functions also using QML and it is possible to remap some buttons to emulate S8 MIDI buttons.

---

First, I was looking for possible buttons for mix recorder function. I decided that `Shift + Snap` and `Shift + Quantize` are a good candidates. First, I had to modify QML files (see diffs below) and follow [S8/D2 MIDI mapping instructions](https://support.native-instruments.com/hc/en-us/articles/209571249-How-Can-I-Modify-my-TRAKTOR-KONTROL-S8-D2-Default-Mapping-Using-MIDI-Controls-). I also mapped the two buttons to the left of the display when in browser to navigate through playlist tree (jump to previous/next playlist). In this first case I use `TogglePropertyAdapter`, so that controller maintains state (on or off) and sends revelant MIDI when state changes. In the latter case, I use `TriggerPropertyAdapter`, so that press and release events are sent through MIDI. I was planning to use that mode for the mix recorder also, however I couldn't get the feedback LED working fine yet – there's a MIDI loop and it behaves non-deterministic.

{% gist kfigiela/d18eea71028bd3331135946ff9d8fd9d %}
