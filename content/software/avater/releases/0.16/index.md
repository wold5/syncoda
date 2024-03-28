+++
title = "AVATeR v0.16 release"
date = 2024-02-29
weight = 0
aliases = []
draft = true

[taxonomies]
tags = ["AVATeR"]

[extra]
toc = true
screenshots = ["avater-screenshot-0.15-1.png"]

+++
[AVATeR](/software/avater/) v0.16 

<!-- more -->





## Details

Read on for additional details.


### Added image caching for drawings (PocketBook and Sony)
By chance, a books.db arrived with several complex drawings - leave it to kids to break stuff :). v0.14 intermittingly slows to a crawl when viewing these, and remains laggy due to the lack of caching[^2]. Caching images fixes this, but requires a sizeable cache of at least 30MB.

This brought other issues to attention: the often huge size of these drawings, and the current, slightly, inefficient two-step conversion.

Note support for PocketBook drawings was silently (but intentionally) enabled in v0.14, after implementing this option for Sony readers. 



## Next release

A few things are planned for v0.16.

First are some additional internal improvents of the backup tool. Some are implemented already, like the file iterator - this also opens up the option for adding KOReader support eventually. 

- Some minor bugs and glitches need addressing. 
- lastly, Kobo chapter titles need to be included in the annotation exports. 
- And Mac support will be extensively investigated this time.

<!--
Lastly, the Debian 10 Buster release may be scrapped. The intent is to move to C++ 20 for technical improvements (compile time mostly, using modules) and its unclear if it is properly supported.
-->

\
[^1]: This can be configured when making such an annotation, using the hamburger menu.
\
[^2]: One workaround is to do a negative search on the text used for these drawings. The catches are this text isn't shown (yet), and it is locale dependent, i.e. `Pencil` (English), `Potlood` (Dutch), etc. A negative search can then be done (sort-of) using `^!pencil`.
