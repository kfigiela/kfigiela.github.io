---
layout: post
title: Baudline Spectrum Analyzer on macOS High Sierra & Mojave
---

[Baudline](http://www.baudline.com) unfortuanetly does not work on new macOS versions due to changes to XQuartz. Since source code is not avaliable there is no easy fix (which probably would be to just recompile app). Inspired by [StackOverflow post](https://stackoverflow.com/questions/42598306/baudline-app-crashes-after-opening-xquartz-osx-10-11-6) I found a non-invasive way to have it running again. The workaroud is to start Baudline with the following command: `DYLD_LIBRARY_PATH=/opt/X11/lib/flat_namespace /Applications/baudline.app/Contents/Resources/baudline`.

**Update [2019-01-02]:** You may make the fix permament by running `install_name_tool -change /usr/X11/lib/libXt.6.dylib /usr/X11/lib/flat_namespace/libXt.6.dylib /Applications/baudline.app/Contents/Resources/baudline`. Probably `install_name_tool` is bundled with Xcode, so you may need to install this. Then you may run baudline as usual.

**Update [2019-10-19]:** 32-bit apps (which baudline is) are no longer supported on macOS Catalina.
