+++
title = "AVATeR v0.11 release"
date = 2022-10-30
weight = 0
aliases = ["/posts/2022/avater-release-0-11-0/"]
template = "page_software_release.html"

[taxonomies]
tags = ["AVATeR"]

[extra]
toc = true
remarks = [[0, "Added Fedora RPM installer"],[0, "Added Windows .exe installer"]]

+++
[AVATeR](/software/avater/) v0.11 adds new .rpm and .exe installers, and fixes minor issues.

<!-- more -->

## Details
New are an .exe installer and .rpm package, generated using CMake/CPack. The Linux packaging isn't perfect yet (changelog information) but that can wait.

Further, some bugfixes and minor changes. Fixes involve reloading the annotations upon device re-connection, if the count remained the same. For Linux users, scanning devices with unmounted e-reading devices present no longer triggers a delay.
