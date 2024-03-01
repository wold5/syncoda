+++
title = "AVATeR updater woes (fixed)"
draft = false

[taxonomies]
tags = ["AVATeR"]

[extra]
toc = true
+++

The previous AVATeR v0.13.1 update didn't show up using the update feature. The updater manifest wasn't pushed to the server last July... this turned up early after v0.13.2, luckily.

<!-- more -->

# More updater woes: URL redirections and 404s
Checked your 404 URLs recently? For syncoda, this turned up an outdated favicon location, the usual wordpress intrusion attempts, and then this: `https://www.syncoda.nl/software/avater/https:/www.syncoda.nl/files/avater/version-windows-amd64.txt`. Well... 

An ancient AVATeR release using a wrong URL seems unlikely[^1]; more probable is that some URL redirect rule backfired, that relocated the older update manifest location (in `/avater/files/`). As the `Redirect 301` lacks the `[L]` stop affix of `RewriteRule`, it seems to have mixed badly with later redirection additions. Ouch.

# Bonus: how the update check works
The (opt-in) program update check is simple. A text file from `https://www.syncoda.nl/files/avater/version-windows-amd64.txt` (for Linux `version-linux-amd64.txt`) is loaded. If an update exists, a pop-up offers to visit the website. This follows the setup by [NoteCasePro](https://www.notecasepro.com/).

The manifest file contains only the version number. A manifest version number was considered, but one can use a new file location as well. On the security side, some internal checks are deployed against 'bad' manifests; v0.14 will see some [extra Qt HTTPS checks](https://www.volkerkrause.eu/2022/11/19/qt-qnetworkaccessmanager-best-practices.html); and our hoster finally enabled DNSSEC.

[^1]: Trolls aside, a more exotic source could be www.virustotal.com executing and digging through binaries (it goes some way towards this, evident from the scan details). As to why it would _now_ turn up with a 0.9.x URL is another question.
