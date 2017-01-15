---
layout: page
title: Projects
---

## NodeLab – Visual-textual programming platform

[NodeLab](http://nodelab.io) is cloud-based programming platform empowered by Luna language, that is unique due to it's dual textual-visual representation.

---

## Raspberry Pi and Denon DRA-F109 integration

I have Denon DRA-F109 Stereo Receiver and Raspberry Pi as media player. I wanted to use one remote control for both. It was possible to use IR receiver and LIRC, but it has many drawbacks, so I reverse engineered *Remote Connector* which appeared to be 5V serial port and the data protocol. Now, I can use Raspberry Pi as it was native Denon system product. [For more, see the blog post](/2014/06/15/denon-remote-connector/).


---

## <img src="/img/dropsport_logo.png" alt="Dropsport. Do sports together. Inspire others." style="height:2em;">

<img src="/img/dropsport_web.png" alt="Web interface screenshot" style="width:40%; float: right; padding: 0 0 1ex 1em;">

[Dropsport](http://dropsport.com) is an on-line platform to find people to participate in fitness activities. After launch in January 2013 the project got some attention in Polish media and was [featured](https://medium.com/@marta/conversion-after-20-sec-of-our-startup-featured-in-the-biggest-polish-tv-news-b445a0ccb825) in two main TV news programmes.

---

## AWS-DNS – virtual DNS for AWS EC2

[AWS-DNS](https://github.com/kfigiela/aws-dns) is a DNS server that maps AWS EC2 instances into virtual `.aws` TLD. Just after you configure Mac OS X resolver you may point any app i.e. ssh or web browser to `my-instance-name.aws` or `i-1231231.eu-west-1.aws` and it will transparently resolve to public instance IP address. For more information check the repo on [GitHub](https://github.com/kfigiela/aws-dns).
