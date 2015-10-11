---
layout: post
title: Focusrite SaffireControl LE responsiveness fix for Mac OS X
---

I've recently bought used Firewire **Focusrite Saffire LE** audio interface to connect an electric guitar. Despite it's old age, it works perfectly on OS X, however **SaffireControl LE** software that is used for metering and to configure the internal mixer behaved really slugish. I soon discovered that the reason was the UI not being redrawn on updates – this happened only when app window got focus or tooltip appeared. Unfortuanetly, the device and software are no longer supported by the vendor and that means – no updates. Accidentally, I found the solution to unblock the UI.

**Workaround:** After you open **SaffireControl LE**, click **Save** and cancel the file chooser dialog. The interface will unblock and work properly.
