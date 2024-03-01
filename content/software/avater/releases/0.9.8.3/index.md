+++
title = "AVATeR v0.9.8.3 release"
date = 2022-06-20
weight = 0
aliases = ["/posts/2022/avater-release-0-9-8-3/"]
template = "page_software_release.html"

[taxonomies]
tags = ["AVATeR"]

[extra]
toc = true
remarks = [[0, "First public Debian release"]]

+++
[AVATeR](/software/avater/) v0.9.8.3 fixes various issues in 0.9.8 (and older), delaying new features.

First public Debian release also.

<!-- more -->

## Details

Read on for additional details.


### Changes
- **Fixed: Show creation date for annotations**
- **Fixed: Correct raw page numbers**
\
*Thanks go to Huwaetzel for reporting the previous issues.*

- **Linux fix: enforce UTF-8 for config files (0.9.8.3)**
Linux versions rely on Qt5, which requires UTF-8 to be set explicitly. 
*This may affect already stored file locations with special characters.*

- **Changed: write logfile to OS temporary folder**

- **Fixed: Debug messages UTF-8 display**
\
    - Mainly fixes debug log display of filenames with special characters (non-Qt dependent functions).
    - log OS-specific filepaths, instead of generic (Linux) ones (WIP).
    - Windows stdout logging still has issues with Asian characters (future option).

- **Windows: updated libraries**
\
zlib was updated to 1.2.12, mainly fixing a vulnerability; and Libzip to 1.8.0.

- **Minor fixes (UI, docs)**

See also the previous [0.9.8 release summary](/software/avater/releases/0.9.8/).



## Future changes
Add extra CSV export columns, colorized HTML export, and debug logging to stdout/terminal. 
Windows Qt will be updated to 6.4.2.
