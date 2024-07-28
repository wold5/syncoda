+++
title = "AVATeR e-Reader compatibility"
description = "List of compatible e-reading devices and programms"
date = 2022-03-02T23:44:21+01:00
updated = 2023-05-05T00:00:00+01:00
weight = 5

[taxonomies]
tags = ["AVATeR"]

[extra]
toc = true


+++

Platforms and devices tested with AVATeR are listed below; their limitations are also noted. \
_For system requirements (for running AVATeR), see the [requirements](/software/avater/pages/requirements/)._

<!-- more -->
\
\

![](/images/logo-pocketbook-medium.png)

# PocketBook
AVATeR was originally developed for use with PocketBook readers, and is thus very compatible. Generally, 6.x and most 5.x firmwares should work with AVATeR.

## Tested device/firmware
- 6.8.3090 (Lux 5)
- 5.8.x (HD Touch 2)

\
\

![](/images/logo-rakutenkobo-medium.png)

# Kobo
Kobo support was added in v0.14, and is considered experimental, especially for kepub. Firmwares in the 4.x range, upto at least 4.33 are compatible. Later versions are expected to remain compatible.

## Tested device/firmware
- v0.14:
    - Firmware 4.33.19747 (13-07-2022); v171; Libra 2
    - Firmware 4.6.9995 (11-10-2017); v141; Kobo Glo

Any older e-readers/firmware may currently not be supported (drop us a note if you want to help). 

## No page numbers
For Kobo, page numbers are substituted with the book progress percentage. Page numbers are not (readily) available in the database; retrieving them is possible but complex. Alternatively, use the chapter information (the viewer has an extra column). 

## Kepub support experimental (v0.14+)
Current Kepub support should work, but may break.

\
\

![](/images/logo-sonyreader-medium.png)
# Sony
Sony support was added since v0.14, and has some limitations. As Sony has ceased development, support will be stable and improve eventually.

Sony support comes with a fair number of gotchas. Preferably use a memory card to store your books (and thus annotations), and enable the local mirror for the card device.

## Tested device/firmware
- PRS-T2N (firmware 1.0.00.13240)

## Annotations limited to 200 characters
Annotations in the DB are limited to 200 characters: a `(...)` is shown whereever this limit is assumed. Retrieving the full text requires access to the book files, which is still far off. Typed notes are not limited to 200 characters. 

## ![](/images/windows.png) Windows: rescan after confirming connection on the device screen
On Windows, when connecting a Sony reader while AVATeR is running, performing a manual re-scan is required (top-left button mainwindow) after you confirmed the connection on the device's screen. For Linux this is fixable with the device monitor, for Windows less so, unless using timers.

## Manually select main/card database
Databases on either the main or card storage needs to be manually selected using the (new) source button. The intent is to remember this setting - for now you can however enable the Local Mirror for one storage device (main or card), and avoid manual switching (read on).

## Local Mirror limited to one storage
the Local (Database) Mirror feature supports just one storage device at a time: either main or card memory. The LM sync assumes non-overlapping database paths (PocketBook), which the Sony doesn't have. A simple but hacky workaround was explored (using a card subdir), which could work, but was dropped.

## Sony SD card always detected
Sony readers always report their SD-card hardware device, and is thus detected by AVATeR, regardless if a card is inserted (and/or mounted). If no card is inserted, some tools (manual backup) may warn about the card storage not being mounted - you may ignore this.

## Drawn notes in highlight column
Drawn notes are shown under the Highlight column (not Notes), and omit the highlighed text. This a todo.

## Improving XML conversion (todo)
Sony rolled their own (XML) note container format, requiring conversion - future version will improve on the current crude(!) regex using XSTL (on the to-do list).

\
\

![](/images/logo-vivlio-medium.png)
# Vivlio
Vivlio readers use re-branded PocketBook firmware (and possibly hardware), and will generally work with AVATeR.

## Tested devices/firmware
- todo
