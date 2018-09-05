---
layout: post
title: Baudline Spectrum Analyzer on macOS High Sierra
---

[Baudline](http://www.baudline.com) unfortuanetly does not work on new macOS versions due to changes to XQuartz. Since source code is not avaliable there is no easy fix (which probably would be to just recompile app). Inspired by [StackOverflow post](https://stackoverflow.com/questions/42598306/baudline-app-crashes-after-opening-xquartz-osx-10-11-6) I found a non-invasive way to have it running again. The workaroud is to start Baudline with the following command: `DYLD_LIBRARY_PATH=/opt/X11/lib/flat_namespace /Applications/baudline.app/Contents/Resources/baudline`.
