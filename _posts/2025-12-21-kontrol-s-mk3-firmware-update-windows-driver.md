---
layout: post
title: 'Native Instruments Kontrol S MK3 Firmware Update Windows Driver'
tags:
- Native Instruments
---

I just set up a new [Native Instruments Kontrol S61 MK3](https://www.native-instruments.com/en/products/komplete/keyboards/kontrol-s49-s61-s88/) keyboard.
After installing the Hardware Connection Service as outlined in the [set up instructions](https://www.native-instruments.com/en/specials/komplete/kontrol-setup/), the next thing you'll need to do is update the firmware on the device.
Unfortunately, I ran into significant problems here.
The firmware updater could not detect my keyboard, displaying the message:

> Please connect your KONTROL S MK3 and click RESCAN.

The problem is that Windows couldn't find a driver for the "KONTROL S61 MK3 DFU" device.
Native Instruments provide some [troubleshooting instructions](https://support.native-instruments.com/hc/en-us/articles/16567321051805-How-Can-I-Update-the-Driver-and-Firmware-of-My-Kontrol-S88-MK3-on-Windows) for this scenario, but these didn't help either.
In step 18 of that guide, when manually choosing the driver, the troubleshooting instructions tell you:

> Select Native Instruments and Komplete Kontrol S88 MK3 DFU

However, there was no option for "Komplete Kontrol S88 MK3 DFU", nor S61 (which is my keyboard).
Checking for Windows Updates revealed no updates.

Eventually, I had to manually search the [Microsoft Update catalog](https://catalog.update.microsoft.com/) to find and download the right update.
Windows really should have found this update itself, but it didn't for some reason.
I'm not sure if this is a Native Instruments problem or a Microsoft problem.
Regardless, in case anyone else runs into this, here's what to do:

1. [Download this Windows update](https://catalog.s.download.windowsupdate.com/d/msdownload/update/driver/drvs/2024/10/a18c8381-2a63-4b67-9594-5f9fcecaf1a4_161ae1246920f8b53cb34162604877b2c40b8b89.cab).
2. Open the downloaded file and copy all of the files inside it to a folder you can easily find.
3. Open Device Manager and choose Other devices.
4. Open the context menu for KONTROL S61 MK3 DFU using the applications/menu key or right click.
5. Choose Update driver.
6. Press Browse my computer for drivers.
7. Choose the folder into which you copied the files in step 2.
8. Press Next.

Hopefully, Windows should now install the correct driver.
