---
layout: post
title: Denon DRA-F109 control protocol
---

One of readers, Philipp Smutny, provided data dump from DNP-F109 network player. It was done by plugging Raspberry Pi in out connector of Network Player, where originally you would connect CD player. It looks like that protocol is quite similar to *binary protocol* described in [earlier post](/2014/06/15/denon-remote-connector) and the only difference is `0x01` instead `0x00` after `payload length` – possibly it marks this as *command*.

For instance, `00 ff 55 01 01 00 40 00 05 9b` means *Set volume to 05*, while `00 ff 55 01 01 00 41 00 01 98` means `Mute On` and `00 ff 55 01 01 00 41 00 00 97` is `Mute Off`. There're few more commands present in the dump provided by Philipp in [gist](https://gist.github.com/kfigiela/5d6c52131d78d40745f1) embeded below.

Unfortuanetly, retransmitting this data doesn't work with my setup. I get back data I send through DRA-F109 loopback circuit, but amp doesn't react. I suppose that this may be due to the fact that I'm using 3.3V (5V tolerant) serial USB adapter, however I can't verify this hypothesis now.

---

{% gist kfigiela/5d6c52131d78d40745f1 %}