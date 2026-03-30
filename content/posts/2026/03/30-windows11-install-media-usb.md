---
title: "Windows 11 setup from USB: a possible fix for 'media driver is missing' error"
date: 2026-03-30T12:21:15
description: This small note describes a possible fix for 'media driver is missing' error on Windows 11 setup
tags: ["windows"]
draft: false
---

## TL; DR

Use the correct partitioning scheme for your flash drive which is GPT.

## Problem

Recently I had to make a fresh installation of Windows 11 Home on my desktop PC.
As my main system is Linux, I didn't have an option to use the official
[Windows 11 media creation tool](https://www.microsoft.com/en-us/software-download/windows11).
It was a good opportunity to try [Ventoy](https://www.ventoy.net/en/index.html) instead.
However, after booting into Windows 11 installer I got a message:

> A media driver your computer needs is missing. This could be a DVD, USB or Hard disk driver.
> If you have a CD, DVD, or USB flash device with the driver on it, please insert it now.

The tricky part was that this message didn't gave any clue about which exactly driver is missing.
My first thought was - well, the machine is able to draw something on the screen; in addition, I can use mouse and keyboard.
So the basic stuff looked alright. Maybe the problem is about hard drive(s)? But the `Browse` button displays all of them.

## What did NOT worked

I spent a while looking for possible solutions online and chatting with LLMs.
For sake of transparency I will share all workarounds which I tried below; maybe some of them will help you.
Or an opposite - you will know which actions you should skip.

### Switch the boot mode

Ventoy provides something [WIMBOOT mode](https://www.ventoy.net/en/doc_wimboot.html) as alternative if any trouble
occurs with "usual" mode. For me the only difference was that in "usual" mode I got some old-fashioned sort of screen
with error message and WIMBOOT mode at least ended up on more graphical screen. Or vice versa. It was not a solution anyway.

### Download the drivers from motherboard and/or CPU manufacturer websites

As the dialog box asks you to download drivers, your very first idea is to download them, right?
There might be some disappointment waiting you at this point: drivers are EXE installers while you need INF files.
And you don't have Windows installed YET at this point.
No, those files cannot be extracted from EXE. Let me even share a Linux command to extract an EXE:

```bash
$ 7z x your_driver.exe
```

Try it. You will get a set of files with different names, but none of them are drivers.
Because that EXE format is more complicated that we would like it to be.

### Disable the Secure Boot ###

No effect.

### Try a different way to write the USB drive

__First: dd if=... of=...__

Although there are some hints online that it might work, for me it didn't.

__Second: WoeUSB__

This is a command-line tool which does some heavy compilation during setup.
It took several attempts to find a working combination of command-line arguments.
However, at the very end I got the same error as with Ventoy.

## What DID worked

After few iterations with ChatGPT I got this note:

> ⚠️ Important conditions (don’t skip)
> 
> 1\. Install Windows in UEFI mode
> 
> Make sure:
> 
> Boot mode = UEFI (not Legacy/CSM)
> 
> Disk uses GPT partitioning

Wait, what partitioning am I using?

... ... ...

Altight, it was MBR, meanwhile Ventoy absolutely can create a GPT disk.

After re-formatting my USB drive with GPT I got a working setup.

## Some thoughts and conclusions

- Please pay attention to details when reading instructions.
  I most likely skipped this GPT part several times.
- I wish the error message in Windows 11 installer had more details than
  "please insert media with driver"
- Ventoy is cool, actually. A good alternative to Rufus for Linux users.