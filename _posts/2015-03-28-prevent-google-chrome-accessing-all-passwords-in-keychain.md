---
layout: post
title: Prevent Google Chrome accessing all passwords in keychain (solved!)
---

Recently I found that iCloud keychain isn't covered by Mac OS X *Time Machine* backups or at least my quick research tells. As cloud services are concerned, *cloud* may stand for *Can't Locate Our User Data* and since I don't own any iDevice, so I don't benefit in any other way from iCloud, I decided to migrate all passwords to regular Mac OS X Keychain, which is stored in regular file. I followed [instructions](https://gist.github.com/rmondello/b933231b1fcc83a7db0b) by [Ricky Mondello](https://github.com/rmondello) to copy passwords from *iCloud Keychain* to regular one, then I disabled *iCloud Keychain* in *System Preferences*. 

Once I did it the problems started: every time I opened *Google Chrome* it asked for permissions **for each** password stored in KeyChain. Clicking *Always Allow* several hundred times is not an option and I don't necessarily want to send all my passwords to Google. There was no way to dismiss this dialogs at once (so annoying!), and when left that dialog in the background after few minutes the browser started to have connectivity issues. This bug was [reported](https://code.google.com/p/chromium/issues/detail?id=178358) for [years](https://productforums.google.com/forum/#!topic/chrome/BPLVDSeTmgI) and only workarounds I found are to remove passwords from KeyChain which is not an option.

On Quora I found discussion on it where [Stephen Ornelas](http://www.quora.com/Stephen-Ornelas ) [suggested](http://www.quora.com/Why-is-Google-Chrome-asking-for-my-keychain-password-so-frequently/answer/Stephen-Ornelas) to remove *Chrome Safe Storage* from KeyChain. Once I did it, I was logged out of Google Account in Chrome and the issue did no longer occur. This led me to final solution: log out Chrome from Google Account, and going further – it is enough to disable Chrome password sync.

## TL;DR: The solution

To stop Google Chrome accesing all passwords in keychain: disable Google Chrome [password sync](https://support.google.com/chrome/answer/165139) or [log out Chrome from Google Account](https://support.google.com/chrome/answer/2390059?hl=en).