---
layout: post
title: Hacking Denon remote connector – progress update
---

I updated [original post](/2014/06/15/denon-remote-connector/) on hacking Denon DRA-F109 remote connector. Since I now have DAB+ broadcast in my place I was able to add DAB+ specific commands to the spec. I also found out that the extra byte that is (not always) present is simple checksum. Sadly, in some cases it is not being sent, so there is no use of it.