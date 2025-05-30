+++
title = "Scripting hints for Zola/Tera"
description = ""
weight = 0
draft = false
date = 2024-03-09
aliases=["software/zola_tips/2024-02-26-usefull_regexes/"]

[taxonomies]
tags = ["Zola"]

[extra]
toc = true
series = "Zola"

+++

Discussed are some tips on scripting with Zola/Tera, based on building this website. These will be extended over time with new experiences. 

<!-- more -->

Note these notes and remarks apply to Tera 1.x.

## Introductory topics

### Shortcodes, macros and templates
Shortcodes are small functions suited for formatting snippets of text, like images or youtube links (they are likened to small templates). They can be freely called within the content, support markdown, and don't require templates. As they are evaluated when rendering the content, they lack some features that macros/templates offer. For complex scripting, consider templates with macro functions instead.

Templates build the actual pages and can include scripting. Using them separates the logic and layout from the content. At template build time, access is allowed to arbitrary page/section metadata and content. They also support shortcode-like 'macro' functions: these offer default arguments and support nesting (of macros) and recursion. Nesting allows building composite functions from smaller ones, but keep in mind macros remain basic 'return a string' functions.

#### Converting shortcodes to macros and/or templates
Shortcodes are arguably easier to start with, and can be converted into templates or macros eventually. Note the latter two use HTML instead of markdown. 

Beware macros can also access their parent scope (read: page/section variables), so preferably use only the macro argument variables (as recommended), and avoid overlapping variable names (i.e. `for section of section.subsection`).

### Storing a theme as submodule
Store a theme preferably as a `git` submodule within your project, so it can be updated independently. This also helps when contributing back any fixes via `git`.

### Updating/syncing templates and config files
Theme content can change over time. Such changes also apply to overridden parts, such as customized templates, macros, SASS and `config.toml` file(s). For syncing changes, 'visual diff and merge' tools are helpful. One example is the older but free and cross-platform [KDiff3](https://kdiff3.sourceforge.net/). Its GUI appears complicated, but only a few buttons really matter - various other tools exist as well. 

For beginners, a brief description of a KDiff3 session follows:

1. use 'open' to select two source files/directories (note the order, i.e. source/destination)
2. open any files with indicated differences (if working cross-platform, bogus line ending differences may occur)
3. start the merge feature. This opens two panes highlighting any file differences. Step through each difference, choosing either source A or B
4. when finished, choose "Save" ([for fixing line-ending differences](https://stackoverflow.com/questions/27382944/kdiff3-there-is-a-line-end-style-conflict), select either DOS or Unix under the 'conflicts' selector)

### Template scripting limitations 

Most SSG scripting languages offer a limited but _quite sufficient_ command set, sometimes requiring workarounds. Accept scripts won't necessarily be smart or efficient (as scripts run locally, optimization is less important anyway).

Listed are some limitations encountered with Zola (v0.17) and [Tera v1.x](https://keats.github.io/tera/docs/) (Jinja-like) templating engine. _Not to diminish either, but to excuse any apparently poor scripting on my, and perhaps your, behalf._

- regex matching: limited to Boolean matches, no capture groups _(rarely used)_ _(capture groups can be emulated somewhat)_
- no appending to **nested** arrays. _There are [workarounds](https://zola.discourse.group/t/allow-load-data-to-take-a-literal/1165/3), but they're not pretty. Support may arrive eventually._
- macros can't operate on dictionary literals (its fields). _but can use array offsets_
- macros only return strings, not arrays or objects
- macro default array arguments don't seem to work
- Iterating over files requires using '[co-locating assets](https://www.getzola.org/documentation/content/overview/#asset-colocation)'. One can't (yet) get files from arbitrary directories, [but see this proposal](https://zola.discourse.group/t/new-global-functions-get-files-or-similar/307) _(BTW, I've come to prefer the co-locating approach)_
- nesting or recursing functions are limited to macros; their in-content equivalent are 'shortcodes', but [shortcodes within shortcodes](https://github.com/getzola/zola/issues/515) aren't supported (yet). _(recommend going with templates and macros anyway)_

Such limitations also provide challenges; one may even propose and submit improvements for inclusion with Zola. Do remember to watch the clock though.

---

## Tips and hints

### Macros: prefer returning only content

Preferably let macros return the minimal amount of content possible: text (with links), without layout and styling elements, like `<p>`, `<br>` and `<pre>`: such elements can better reside in the templates. Other benefits include:

- each macro call can have customized styling (via template CSS ids/classes)
- better separates layout from content
- this allows assigning the output to a variable
	- which allows conditional rendering, based on an empty result (may require using `trim`)
- better indentation control: uses the macro call position

For example, the 'newer version check' on the software pages here currently works like:

```Jinja2
### Downloads
	{% set newer_version_path = syncoda_macros::check_latest_version(page_path=page.path) %}
	{% if newer_version_path %}
	<p class="warning_newer_release">Newer release available: {{ newer_version_path | safe }}</p>
	{% endif %}
```

Granted, sometimes this goal is harder to achieve, like when processing multiple items. Macro's could then be called piecemeal on each array object, moving the loop into the template. (Macro's can't return array objects. Though... a macro could return a 'string-ified' dict, importable as a literal using `load_data()`, but this isn't an easy route...)

### Macro CSS styling
As noted, macro content can also be styled via parent elements in the template, using Id, Class or (modern) CSS selectors. This avoids the need for ids and classes in the macros themselves, making them easier to re-use.

Including `style=""` CSS overrides at times can be useful and, frankly, easier; but do consider topics like theme compatibility (light/dark mode) and universality/generality, such as 'warning' colours being typically red, yellow and green ([future] Martians may prefer other combinations).

### Using path or object argument variables
Passing pages and sections as macro arguments, can be done using their path (string) or the object itself. Passing an _object_ allows working with it directly, removing the need to call `get_section()` or similar functions. Especially with translations, one doesn't need to (re-)construct the subsection path. _However_, when recursing into deeper subsections certain arrays like `section.pages` don't get populated (for reasons of efficiency), requiring using `get_section()` anyway, as discussed later.

_Note: Clarify an expected type in the argument variables names, using `path` or `*_object` affix._

### Avoid double checking arguments in the macro
Avoid macro code that double checks some provided argument against a value in the `page.extra` array or object. Better to move that check into the parent template/macro, and let each macro be all about its task.

### Including plaintext files

For HTML templates and macros, `load_data()` can import plain text (this may contain markdown content). For dealing with plaintext there are some options:
- convert the `load_data()` output to HTML by replacing the plaintext linebreaks with `<br>` tags. To do so, append the filters [`linebreaksbr`](https://keats.github.io/tera/docs/#linebreaksbr) _and_ `safe`
- embed the plaintext in a `<pre>` HTML block which respects the plaintext linebreaks. \
We use(d) `<pre><code>` in places, as it creates a pretty outlined block. While using CSS/SASS is best, this allows using the theme's style (do check your current theme's method for styling code blocks).

## Cleaning up macro and template raw source output
While a `minify_html` config option exists (removing all 'non-functional' whitespace in the outputted source), it can still be worthwhile to check out the generated HTML sources, even if once. Reasons include fixing forgotten `| safe` calls (escaping special characters) and improving whitespace control.

### Prevent HTML escaping
For safety reasons, standard variable output in templates and macros (in HTML format) have their special characters escaped, with `https://` becoming `"http:&#x2F;&#x2F;`. While not required, this can break scripts, complicate source reading, and marginally increase file sizes. Adding a `| safe` filter avoids this.

- For macros, using `safe` once in the macro is sufficient - it applies to the point where a variable is output using the `{{ }}` delimiters.
- When re-processing (assigned) output that had `safe` applied, character escapes are applied again.

### Whitespace control
Tera treats every line as 'output': even an `{% if true %}` line outputs a blank line. Webbrowsers ignore such whitespace, but they do contribute to the filesize (easily 5-20%), make the HTML source look sloppy, and for macros can add characters to a 'returned' string.

To fix this Tera offers [whitespace control](https://keats.github.io/tera/docs/#whitespace-control), that removes such whitespace between statements. It is activated by adding a dash to opening _and/or_ closing delimiters like `{%-`, `{{-` and `{#-`, removing any whitespace characters prior (or after) that delimiter, at the point of insertion. \
For example, with `{%-` the new line will be appended right next to the previous line (removing the source linebreak). Perfecting this requires some fine-tuning, and in rare instances will not completely work as one intended (`for` loops for example).

### Whitespace in macro 'return' values
Macros can be (ab)used to return some (string) value. If no whitespace control is applied within the macro, the 'return' string may include whitespaces. One indicator is when template whitespace control has no effect around the assigned variable. Other issues include 'if' checks on the assigned result being always true, or linebreaks being included. Whitespace can be removed in the macro, or by appending a `| trim` filter to the macro call that assigns the variable (note this negates any previous `safe` filters).

### Web standards compliance checks
It can't hurt pulling the HTML and CSS through some webstandards validator, like the [W3.org's Nu HTML checker](https://validator.w3.org/nu/). Do expect a fair share of "Trailing slash on void elements" warnings, as XHTML apparently has fallen out of grace ;)

## Beware...
Work on the website usually comes last, so you'll be too tired at some point ;)

### get_section metadata flag
Functions like [`get_section`](https://www.getzola.org/documentation/templates/overview/#get-section) offer a `metadata_only` flag for efficiency - if set true, it will not populate the `section.pages` and `section.subsections`.

### Shared variable scope
Shortcodes and macros share their parent scope: overlapping variable names can have odd results, especially when recursing.

### Renaming recursive functions
If ever renaming or duplicating a recursive function for experimentation, also rename the call inside the function... 

### Deeper levelled section.pages being empty
It seems that for reasons of efficiency, the `section.pages` array doesn't get populated for deeper positioned subsections. This requires a `get_section()` call to get the 'complete' section object.

When translations exists, constructing a section path can get more complicated, and is still a WIP for our page+section lister (upcoming changes in Zola 0.18.1+ also impact this).

---

## Regular expressions
Provided are a few regular expressions for changing content URLs, outside of Zola using your a *nix shell or Windows Powershell.

**Be careful when testing** with the `sed -n` and `p (print)` options. If not using version control, a pre/post directory comparison can be made using `diff -r`  or `diff -ru0`.

### Changing file/image URLs

It's easiest to store images with the post content (bundling or co-location). When moving such files to or from `/static/`, the content URLs need to be modified. The regex below suffices for this task, though it may not cover all situations.

```bash
sed -E 's+\!\[(.*)\]\((.\+)\)+![\1](/files/avater/manual/\2)+' somewhere/manual_en.md > manual_en.md 
# Test using sed -En 's+regexhere+p' manual_en.md
```
With Powershell avoid the default Unicode conversion (always something... ;)), if desired.
```PowerShell
get-content somewhere/manual_en.md | 
 %{$_ -replace '!\[(.*)\]\((.+)\)' , '![$1](/files/avater/manual/$2)'} | 
 Out-File -Encoding ascii -FilePath manual_en.md
```

### Changing internal path URLs

File directory/layout changes may also necessitate changing links. A quick example using `find -exec`.

```bash
find . -iname "*.md" -exec sed -i -E 's+systemrequirements+requirements+' {} \;
# test using sed -n -E 's+systemrequirements+requirements+p' {} \;
```
Linux/GNU also offer the `sed -s` multi-file option:

```bash
sed -s -i 's+(@/software/avater/info.md#requirements)+(/software/avater/#requirements)+' *.md
```

