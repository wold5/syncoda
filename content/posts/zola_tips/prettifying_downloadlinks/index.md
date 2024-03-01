+++
title = "Prettifying software package downloadlinks with Zola and Tera (Jinja2) templates"
draft = false
weight = 5
date=2024-02-29

[taxonomies]
tags = ["website", "Zola"]

[extra]
toc = true
series = "Zola"
+++

We explore improving the presentation of this website's software package downloadlinks, providing users an overview of the releases per OS/platform. One example being:

![](screenshot1.png)

We'll briefly list some approaches, with the [resulting script](#single-iteration-version) being intentionally kept simple. It utilises the filepath for OS/platform identification; and requires that sorting the filenames groups platforms together.

<!-- more -->

## Approaches, goals and constraints

1. Manually place downloadlinks (copy 'n paste)
2. Parse the filename and/or path of each package
3. Process some package manifest, detailing each package.

For this website, some automation was desired, without over-engineering a solution, suggesting approach #2. For commercial settings approach #3 provides more control and avoids parsing filepaths (fragile); one exception being the use of platform-specific subdirectories, for which approach #2 again suffices. 

## 1. Using manually placed links
Don't dismiss manually placing (or copy-pasting) links too fast: provided package filenames are kept consistent, only the version number needs updating. A variation utilises a shortcode/macro that inserts a `page.app_version` number. 

## 2. Parsing the filepath of page assets
To determine the OS/platform of the package, parsing the filepath offers an attractive shortcut. Some example package filenames are shown below (in order: Fedora, Debian with manually added description and Windows):
```
avater-0.11.0-1.x86_64.rpm
avater_0.11.0-1_amd64-buster.deb
avater_0.11.0-1_amd64-standard.deb
avater-0.11.0-win64.exe
avater-0.11.0-win64.zip
```

Notice that when sorted, the filenames group by platform - and while this cannot be garantueed, it can be helped... which will prove useful later. Using parent directories like `/windows/` simplifies the issue even more. 

### Filepath component considerations

When parsing one can utilize the path/directory, extension, filename stem or a mix of these. Also consider any legacy files. This involves some common knowledge, but let's walk through it.

#### Path or subdirectories
Pre-sorting packages into subdirectories like `/windows/` simplifies scripting considerably - but it still leaves the initial sorting task, to be done manually, by the build system or some external script.

_Note your SSG must also support iterating through subdirectories; Zola currently does so for co-located assets. If not, adding this should be reasonably doable._

#### File extension
Package file extensions tend to be unique to each OS/platform, with some overlap. Platform-agnostic examples are Flatpack and Snap; generic ones `.run` and `.sh` for Linux, and `.pkg` used by all BSDs. Consider also archives like `.tar.bz` or `.zip`. Re-compressed packages could be assigned `.exe.zip` and remain 'recognisable'. 

The exceptions suggest adding some OS/platform identifier to the filename, or using subdirectories. Once downloaded however, the file ought still to be identifiable suggesting the former be applied always.

#### Filename stem
For the CPack packaging system at least, platform identifiers in the filename (e.g. "windows", "fedora") are not present, except for Windows (`win64`), and need to be specifically set. The appendice section briefly discusses CPack.

Build systems like CPack allow full customization of the package filenames, though RPM and DEB tools gravitate towards a standard format geared at package repository _tools_. When ignored, compliance may still need to be implemented in the future. For archives this matters less, being created outside a repository orientated workflow.

##### Legacy files
Lastly, consider any legacy files, which may lack indentifiers: does one rename those? (and what for?) Or adapt the script? One option is to handle any legacy files transparantly, using a fallback mode that just lists the files.

### Basic scripting implementation 
The most basic KISS approach is to iterate `page.assets` repeatedly and list only specific regex matches. This provides full control over the layout and allows adding customized parts; while performance matters lless for an SSG build website. A template/macro setup will be most efficient, as shortcodes currently can't be nested/recursively called. We could however do somewhat better using a single iteration, discussed next.

### Single iteration version
Provided that (alphabetic) sorting provides adequate grouping of the packages, one can check each package for its platform, and upon encountering a new one, insert that associated platform header. This is sufficiently KISS, and sidesteps some scripting limitations (like being unable to append to _nested_ arrays). \
One catch is that higher version numbers will be sorted below older ones, so preferably build a separate page for each release (or iterate over `page.assets | reverse`).

#### Getting the OS/platform
First retrieve the platform using the filename, path, etc. Any number of variations exist: use the subdirectory paths (match on prefix), filestem indentifiers, or some mix. Keeping the platform function separately helps regardless.

```jinja2
{%- macro filename2platform(path) -%}
    {% set platform="" %}

    {# match extension #}
    {% if path is matching(".deb$") %}
        {% set platform = "debian" %}
    {% elif path is matching(".rpm$") %}
        {% set platform = "redhat" %}
    {% elif path is matching(".exe$|.zip$|.msi$") %}
        {% set platform = "windows" %}

    {# match filename #}
    {% elif path is matching(".tar.gz$") %}
        {% set platform = "source" %}
    {% elif path is matching("windows") %}
        {% set platform = "windows" %}
    {% elif path is matching("fedora") %}
        {% set platform = "redhat" %}
        {# nesting has issues #}
    {% else %}
        {% set platform = "other" %}
    {% endif -%}
{{platform}}
{%- endmacro %}
```

One can emulate a group capture to a degree, by abusing [`trim_start_match()`](https://keats.github.io/tera/docs/#trim-start-matches) and friends, or [`split`](https://keats.github.io/tera/docs/#split)(), like: `{% set platform = filename | split(pat="_") | nth(n=2) | lower %}` or for subdirectories `... | first | lower`.

When embedding such a loop, use `set_global` to preserve `platform` between iterations.

#### Building the downloadlinks
Finally, utilise the platform function for generating the menus:

```jinja2
{% macro assets2downloads(page, gethashes=true, include_matches="", exclude_matches=".png$|.jp[e]?g$|.gif$|.txt$|.pdf$|.bmp$|.mp4$") -%}
{% set lastPlatform = "" %}

{% for asset in page.assets | sort %}
    {# presents a lowercase filename #}
    {% set iconpath = "" %}

    {# include OR exclude files. Include takes priority #}
    {% if include_matches and asset is not matching(include_matches) %}
        {% continue %}
    {% elif exclude_matches and asset is matching(exclude_matches) %}
        {% continue %}
    {% endif %}

    {# get the filename lowercase. pat depends on platform #}
    {% set filename = asset | split(pat="/") | last | lower %}

    {# identify platform. When using subdirectories, splitting on the first part suffices instead #}
    {% set platform = self::filename2platform(path=filename) | trim %}

    {# output platform label #}
    {% if platform != lastPlatform %}
        {% set_global lastPlatform = platform %}

        {# overrides #}
        {%- if platform == "debian" -%}
<h3 id="downloads_debian"><img src="/images/debian.png"> Linux Debian (Ubuntu, Mint, etc)</h3>
        {%- elif platform == "redhat" -%}
<h3 id="downloads_redhat"><img src="/images/redhat.png"> Linux RedHat (Fedora, SUSE, etc)</h3>
        {%- elif platform == "macos" -%}
<h3 id="downloads_macos"><img src="/images/macos.png"> MacOS</h3>
        {%- elif platform == "windows" -%}
<h3 id="downloads_windows"><img src="/images/windows.png"> Windows</h3>
<p>For older Windows releases the Qt5 .zip version may work instead.</p>
        {# default #}
        {%- else -%}
<h3 id="downloads_{{platform}}"><img src="/images/{{platform}}.png">{{platform | capitalize}}</h3>
        {%- endif -%}
    {%- endif -%}

{{- self::filelink_decoration(asset=asset, filename=filename, gethashes=true) -}}

{%- endfor %}
{% endmacro assets2downloads -%}
```

Where `filelink_decoration` outputs the filename, link and hash.


#### Notes
- treat the 'Other' category as a warning - these could be stored up in a array, and processed last
- external repositories pose a challenge: 
    - for existing categories, place these under the headers
    - they could also at the start be [appended](https://keats.github.io/tera/docs/#concat) to a (sorted) copy of the `page.assets` array
    - or insert these last

## 3. Processing manifests
The last approach is to use some manifest file, that details each package, build externally. This circumvents all foibles involved with interpreting filenames. I haven't explored this, but will suggest some starting points: 

- get the build system to generate such a file, i.e. using CPack and [`CPACK_POST_BUILD_SCRIPTS`](https://c	make.org/cmake/help/latest/module/CPack.html#variable:CPACK_POST_BUILD_SCRIPTS). 
- use a OS/platforms packaging tools to query packages like DEB and RPM
- use a system like the [Open Build Service](https://openbuildservice.org/), which IIRC even generates downloadpages (for internal consumption)

Once some CSV or JSON manifest is available, import its data using [`load_data()`](https://www.getzola.org/documentation/templates/overview/#load-data), for processing or iteration. It could also import plaintext markdown, allowing for generating the list outside the SSG; or 'loading' a literal JSON string, if the page was generated externally.

## Appendices

### Fixing broken links due to URL slugification
Slugification replaces version dots by default. use the `safe` mode instead, as detailed on the [intro page](/posts/zola_tips/intro/).

### Changing CPack package filenames
`RPM` and `DEB` generators use their own filename variables. While changeable, mind any (future) repository (workflow) requirements as noted. Their defaults are a string like `DEB-DEFAULT` or `RPM-DEFAULT` that trigger an internal condition (i.e. if replaced it must be fully defined). 

For the other generators, like `archive` (ZIP), `NSIS` and MacOS (and perhaps others), modify the [`CPACK_PACKAGE_FILE_NAME`](https://cmake.org/cmake/help/v3.0/module/CPack.html#variable:CPACK_PACKAGE_FILE_NAME) string, before calling `include(CPack)`. You'll end up with some variation upon the following:
```
set(CPACK_PACKAGE_FILE_NAME "${AVATER_NAME_LOWER}_${CPACK_PACKAGE_VERSION}_${CPACK_SYSTEM_NAME}_${PROCESSOR}_${DESCRIPTION}")
message(STATUS "package filename: ${CPACK_PACKAGE_FILE_NAME}")

```
Note the extension is added automatically. The `description` variable here details the Linux distro: one can use a manually configured environment variable, set before compilation. Identifying a distro automatically turned out to be not that easy, with the better examples trawling through `/etc/` for distro-specific files. 

_Note: When selecting Cmake variables for a new filename, be mindfull of cross-compiling and similar setups; some CMake variables account for these situations._

### Operating system logos
Ending on a lighter note. Adding visual logos aid recognition, and liven up the page a bit. [Wikimedia commons](https://commons.wikimedia.org/) provides most icons in SVG format. Daredevils may find stylized logo variants on various sites.

And lastly children, pay attention, you're not allowed to have fun with logos (the graphical kind ;-):

![Redhat doesnt want parodies with its logo](screenshot-redhat-parody.png)


---

Original: 2023-06-10 (unreleased)

[^1]: Computing hash checksums can involve considerable I/O and processor usage
