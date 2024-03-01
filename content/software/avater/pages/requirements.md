+++
title = "AVATeR System requirements"
description = "Project info"
date = 2022-03-02T23:44:21+01:00
updated = 2023-05-05T00:00:00+01:00
weight = 5

[taxonomies]
tags = ["AVATeR"]

[extra]
toc = true


+++

System requirements (for running AVATeR). \
_For compatible e-reader devices, see the [compatibility](/software/avater/pages/compatibility/) page._

<!-- more -->

## Supported processors

Compiled versions are currently provided for:
- amd64

ARM versions will be investigated.

## Supported Operating Systems
Having currently few dependencies, AVATeR will generally run on compatible distributions. 

### ![](/images/debian.png) Linux Debian (Ubuntu, Mint, etc)	

Debian releases are tagged using the Debian release naming. Older releases used 'standard' and 'compatible' releases: these equate Bullseye and Buster respectively. Note newer distro releases may run applications from older distributions, the reverse is less likely, especially due to the Qt versions not being available.

- ***Bookworm*** (i.e. Ubuntu 22+) or later
    - starting 0.13.1+
	- Qt6 only
    - Minimal requirements: `glibc 2.3?`, `qt 6.4`, `libzip 1.6`, `udev`
- ***Bullseye*** (i.e. Ubuntu 20+) or later
    - Qt5 only
    - Minimal requirements: `glibc 2.33`, `qt 5.15.1`, `libzip 1.6`, `udev`
- ***Buster*** (i.e. Ubuntu 18-19).
    - Qt5 only
    - Minimal requirements: `glibc 2.28`, `qt 5.11`, `libzip 1.3`, `udev`
    - Omits the ability to cancel running backups
 

### ![](/images/redhat.png) Linux Red Hat (Fedora, OpenSUSE, etc)
Fedora releases follow the Fedora Workstation releases, and should be useable on newer or compatible systems with equal glibc and qt versions.

- Fedora Workstation 37
    - starting 0.13.1+
    - Qt6 only
	- Minimal: `glibc 2.??` `libzip` `qt6-qtbase`
- Fedora Workstation 35
    - up to 0.13.1 and newer
    - Qt5 only
	- Minimal: `glibc 2.34` `libzip` `qt5-qtbase`

### ![](/images/windows.png) Windows

- Windows 10/11+
	- all versions
    - Note 0.13.x and older (using Qt 6.2) don't start on newer Windows releases. However, the Qt5 version might work.
- Windows 8/7/Vista
    - Qt5 versions (starting with 0.15) might work, else
	- 0.13.x and older
- Windows XP
    - Not supported (perhaps if compiled without the USB scanner+monitor).


## Windows and Linux differences

! Note: the device monitor has not yet been implemented for Linux.

The USB implementations differ, but this is mostly a technical aspect. The Linux version generally appears to run faster: Window's API function for device names can stall at times; Qt's row resizing was excruciatingly slow on Windows, but has been replaced by smart-row-resizing. That has it's own caveats: the intention is to allow either, at least in the Linux version.
