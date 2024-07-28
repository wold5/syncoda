+++
title = "AVATeR v0.16 release"
date = 2024-07-27
weight = 0
aliases = []
draft = false
template = "page_software_release.html"

[taxonomies]
tags = ["AVATeR"]

[extra]
toc = true
screenshots = ["avater-screenshot-0.16-1.png"]

+++

[AVATeR](/software/avater/) v0.16 fixes bugs for PB sorting and when enabling Local Mirrors, amongst others; improves darkmode support (Windows mainly); adds image caching and other internal changes.

<!-- more -->

## Details

Read on for additional details.

### PocketBook sorting bug fixed
![](avater-screenshot-faulty_sorting.png)

For PocketBooks, titles with an empty author field weren't sorted properly, collecting at the top. This regression was due to adding Kobo and Sony support earlier.

_If upgrading isn't possible: click in the table header to manually sort columns, first by date (or page) and next by title._


### Rare local mirror enabling/syncing bug
In certain situations, a newly enabled LM wouldn't sync (the reload popup doesn't appear). Here it occurred only on Windows. Another device scan (F5) would solve this, but it's conceivable that on startup LM syncing could also fail.

While LM handling was already simplified, incorrect device ordering caused the sync to target an incomplete LM source.


### Added image caching for drawings (PocketBook and Sony)

Displayed bitmap images are now cached. This avoids drawings (vector SVGs) getting converted to bitmaps over and over. At worst this could halt the program momentarily when scrolling.

![](avater-screenshot-settings_imagecache.png)

- Cache size is configurable under Settings > Advanced (Hover the mousecursor over the input field for more info). Note stored bitmaps can be large, being likely stored uncompressed and with higher (redundant) color depths by default, so be liberal when increasing the cache size.

### Improved darkmode support (Windows mainly)
Darkmode support was improved, mainly for Windows 11. For Linux, the situation is less straightforward and under investigation, but may benefit also (one can try themes like `Adwaita-Dark` and `kvantum`, though these don't provide 100% coverage[^0]).

![](avater-screenshot-0.16-1.png)

- GUI icons will be kept at 60% grey for now to simplify things. The main nag being the device connection state icons, where in dark mode the 'faded' unmounted items show up brighter.
- export support is another topic. Currently default HTML colors aren't touched. Allowing customized stylesheets is another option.

This was pursued after Qt 6.7 on Windows 11 defaults to the 'windows11' Qt style[^1]. This suggested checking out dark mode as well. En route, a Qt bug was discovered and reported for this style, where table text color wasn't adjusted properly. Hence, AVATeR builds for Windows using Qt 6.7.2 or earlier, will now force the generic Qt "fusion" style.

### Minor optimizations

- When switching annotation sources, annotation models were accidentally created twice. Data loading occurs in a separate stage, so this could go unnoticed. 

- Aforementioned annotation models are now also sorted during creation, while redundant filtering calls were prevented. With larger annotation counts, this might provide gains.

### Other changes
- the sorting mode and export sorting mode are stored with new variable names. This affects switching between v0.16 and any earlier versions.
- PocketBook Color 3 detection was added. AFAIK, they now use a generic Linux Foundation VID, oft used (as a standin?) for USB hubs. Given AVATeR scans only 'block' devices, we don't expect problems. And if so, let us know :-)

### Internal changes (development)
There were a ton of internal changes: the CMake config was cleaned up, compile flags were added, and 'circular' header references were reduced to a minimum[^2]

Along the way, there were the usual hickups: VCPKG and library configurations getting swamped, exploring an INI library replacement and other stuff.

## Next release

A few things are planned for v0.17:

- Some exporter changes. Adding sorting case sensitivity is one: for this the exporter will switch to using the annotation model sorting modes. This is a step-up towards moving the annotation table out of the 'mainwindow' code; with the future option of having multiple tabs. Lastly, Kobo chapter exports be included as well. 

- Backup tool internal improvements. Some are implemented already, like the file iterator - as may be mentioned, this opens up the option for adding KOReader support eventually. 

- For Windows, AVATeR preference and data folder may be moved into the user home directory. When restoring a Windows install, these are lost in their current locations; and likely also be ignored whenever a backup is made.

- Mac support will be evaluated late 2024 or early 2025. There's 'surplus cash' (though little from the rare AVATeR donations), so stay tuned[^4].


---

[^0]: One telltale is the spacing between top menu items, which initially looked weird.
[^1]: Force a style by appending i.e. `-style Fusion` when invoking AVATeR from the shell/CLI.
[^2]: Or whack-a-mole with #includes. Use forward declarations whenever possible in headers. As an aside, the C++ Modules alternative looks enticing, but support currently seems iffy. 
[^4]: still loathing the soldered memory on the 8GB base model: to be remedied by a 'mere' 200 currency tokens =/
