+++
title = "AVATeR Project info"
description = "Project info"
date = 2022-03-01T01:04:21+01:00
updated = 2023-06-10T00:00:00+01:00
weight = 0

[taxonomies]
tags = ["AVATeR", "C++", "Qt", "SQLite"]

[extra]
software = true
logo = "/images/logo-avater.png"
#image = "/files/avater/avater-screenshot-0.13-1.png"
toc = true


+++
![](/images/avater-35px.png)
AVATeR stands for Annotation Viewer and Tools for e-Readers.

It supports PocketBook, Kobo and Sony e-readers. Annotations can be interactively read, searched and exported to various formats, directly from a connected e-reader. It can locally store annotations for viewing after disconnecting your e-reader; and it provides several tools, for making backups and checking databases amongst others. 

<!-- more -->

![AVATeR screenshot](/files/avater/screenshots/avater-screenshot-0.16-1.png)

## Features 

We do:

- Interactive annotations reading, searching and exporting</strong> directly from the e-reader
- Store annotations locally</strong> for viewing after disconnecting
- provide basic <strong>tools</strong> for e-readers, including backup options

We don’t:
- manage device content or an e-book library
- modify e-book files

Some highlights:

- <strong>auto-detect e-reader (dis)connection</strong>
- cross-platform: Windows and Linux. (Apple planned)
- device settings/annotation files can be easily copied/synced.

See also [the screenshots page](/software/avater/pages/screenshots#screenshots).

## Supported e-reader devices

- PocketBook or Vivlio e-readers with 6.x or 5.x firmware.
- Kobo e-readers with 4.x firmware
- Sony e-readers

_For an overview of compatible e-reader devices, see the [compatibility](/software/avater/pages/compatibility/) page._

---

## Downloads

For the latest release use: [/software/avater/releases/latest/](/software/avater/releases/latest/). \
For [other versions see the release pages](/software/avater/releases/).

Supported are Windows, Linux Debian and Redhat, with Mac support being in the works. _See details with each release, or the [system requirements page](/software/avater/pages/requirements/)._


---

## Support
- [Online manual (English)](/software/avater/manuals/)

- [Mobilereads topic](https://www.mobileread.com/forums/showthread.php?t=345428)</a>
- e-mail using support [at] syncoda.nl

---

## Background

### Release roadmap

- Upcoming release
    - improve manual backup tool (internally)
    - ~add annotation type filter~
    - ~various bugfixes~
    - ~improve drawing and image processing and caching~
- Planned
    - ~improve PB translation not support~
    - add Apple Mac support
    - add KOReader support
    - add PocketBook tools for editing book titles, collections, etc.
    - edit annotations/notes
    - improve CLI support    
    - open source USB components


### Program structure

The AVATeR backend is split into two parts: the 'USBScanner' provides tools for checking USB devices, hotplug events and mountpoints. The ReaderManager manages all Reader objects. On changes it signals UI applications, such as the AVATeR UI, which contains the annotation viewer and some tools.

![](/files/avater/programdiagram.svg)

The USBScanner could be converted to plain C. It interacts directly with WIN32/udev APIs, though libusb could be adapted in the future. The other parts use C++ (in a somewhat C-style). There are still some dependencies on Qt, for SQL, preferences and debug logging.
The UI depends on C++/Qt, and is compatible with Qt 5.12.x+ (common on linux) and 6.x. 

The program sources are currently closed, with an intent to open them up. The current idea is to release the lower parts first, starting with the USBScanner. A liberal license is preferred.

### History
AVATeR started as the PocketBook Tools plugin for Calibre; the first standalone version had a tiny window with 6 tool buttons. The original annotation HTML export tool (a SQL query that produced HTML) was converted into an interactive viewer, and one thing led to another.

## Donate

You are encouraged to donate, if you are able to do so.

- [Donate using PayPal](https://www.paypal.com/donate/?hosted_button_id=4UATTKGGJ9V68)

---

For downloads [see the release pages](/software/avater/releases/).

