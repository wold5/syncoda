+++
title = "Macros and templates for Zola"
description = "Macros and templates used on syncoda.nl"
weight = 1
draft = false
date=2024-02-23

[taxonomies]
tags = ["Zola"]

[extra]
toc = true
series = "Zola"
+++

These macros are used on Syncoda.nl.

- latest versions can be accessed via Github or -lab (soon), but are included here for learning purposes. For a '80s BASIC vibe, type them over manually ;)
- inline CSS is used for now, preferring portability, re-use and making these standalone. Generally, the used visual styles are functional, and can handle light/dark mode 
- example usage of macros in a template: `<p>{{- syncoda_macros::print_download_remarks(array=page.extra.remarks) -}}</p>`

## Navigation

### List section pages upto x
`macro list_section_pages(sectionobject, page_list_limit=-1)`

Get a section its pages, and output each within `<li>` tags.
Upon reaching `page_list_limit`, output a "More..." link to the section.

Used on the [software section index](/software/).

### Section/page index listing project pages
`macro show_page_index(sectionobject, depth=0, maxdepth=1)`

Recursive function for building a menu with subsections and their pages.
Currently limited to `depth=1`, and uses `<h*>` headers.
Todo: build a contiguous unordered list 

Used on the [software section index](/software/).
```jinja2
{% macro show_page_index(sectionobject, depth=0, maxdepth=1) -%}
{% if not sectionobject.extra.page_list_skipme %}
	{# Top level H3 headers #}
	{% if depth == 1 %}
		<h3>{{ self::list_section_title(sectionobject=sectionobject) }}</h3>
		{%- if sectionobject.description -%}
			<p>{{ sectionobject.description }}</p>
		{%- endif -%}
	{%- elif depth > 1 -%}
		<h4>{{ self::list_section_title(sectionobject=sectionobject) }}</h4>{# open li tag#}
	{% endif %}

	{#- Output available pages -#}
	{% if not sectionobject.extra.page_list_hide | default(value=false) and sectionobject.pages %}
		<ul>
		{{ self::list_section_pages(sectionobject=sectionobject) }}
		</ul>{# todo remove for nested li #}
	{% endif %}
		
	{% if depth <= maxdepth and sectionobject.subsections %}
		{% for s in sectionobject.subsections %}
			{# start recurse call on subsection {{s}}<br> #}
			{% set section_subsection_object = get_section(path=s, metadata_only=false) %}
			{{ self::show_page_index(sectionobject=section_subsection_object, depth = depth + 1) }}
		{% endfor %}
	{% else %}
		       {#</ul></li> todo, close level 2+ li tag, current <h4>#}
	{%- endif -%}
	{#</ul>#}
{%- endif -%}
{% endmacro list_subsection_pagelinks -%}
```


### Get a section's first page its path
Get a section its first ordered page path
- Assumes the original sorting suffices.
- Use `| trim_start_matches(pat="/")` to remove any leading slashes

```Jinja2
{% macro get_section_first_page_path(path) -%}
	{%- set section = get_section(path=path, metadata_only=false) -%}
	{%- for page in section.pages -%}
		{{ page.path }}
		{%- if loop.index0 == 0 -%}
			{%- break -%}
		{%- endif -%}
	{%- endfor -%}
{% endmacro get_section_first_page_path %}
```


### Providing a 'latest' version redirect
This uses a redirecting subsection like `/project/releases/latest/`, that points to the first ordered post of the parent section `/project/releases/`. One could also use a frontmatter `alias` to each newest release, but this is error prone.

Why a section? It can access its parent sections using `{{ section.ancestors }}`; its direct parent being `{{ section.ancestors | last }}`. This avoids hardcoding the section path.

For the redirection page, a custom page template was modelled after Zola's redirection page (`internal/alias.html`). Its `{{ url }}` will then need to point to the latest version. 

```Jinja2
{%- import "macros/syncoda_macros.html" as syncoda_macros -%}
{%- set_global parent_section = section.ancestors | last -%}
{%- set redirect_url = syncoda_macros::get_section_first_page_path(path=parent_section) -%}
<!doctype html>
<header>
<meta charset="utf-8">
<link rel="canonical" href="{{ redirect_url | safe }}">
<link rel="stylesheet" href="/abridge.css" /><!-- manually specified, no base URL -->
<meta http-equiv="refresh" content="0; url={{ redirect_url | safe }}">
<title>Redirect to latest</title>
</header>
<body>
<p><a href="{{ redirect_url | safe }}">Click here</a> to be redirected to the latest release.</p>
</body>
```

Enable the template in the frontmatter using:

    template = "section_redirect_to_parent_firstpage.html"


Getting the newest section post macro is discussed below.

## Release pages

### Compare latest version
Compare page paths, if unqueal a newer release is available.
Alternatively, compare using some `page.extra.version` variable.
Todo: CSS class reference

![](../zola_macro_latestversion.png)

```Jinja2
{% macro check_latest_version(section, currentpage) -%}
	{% set newest = self::get_section_first_page_path(path=section) %}
	{% if page.path != newest %}
		{%- set page = get_page(path=newest ~ "index.md" | trim_start_matches(pat="/") ) %}
		<p style="padding: 20px; border: solid 1px orange; background-color: yellow; color: black; text-align: center;">Newer release available: <a href="{{ page.permalink }}">{{ page.title }}</a></p>
	{% endif %}
{% endmacro get_section_first_page_path %}
```


### Release remarks

Users can't be expected to compare all release notes, so we provide color-coded warnings and remarks. 

![](../zola_macro_releaseremarks.png)

Output color-coded remarks, from an array of arrays (see example below).
Priority codes: 0=none/green; 1=low/yellow; 2=high/red

```
[extra]
remarks = [[1, "Windows EXE installer has been deprecated in favour of the MSI installer."],[0, "added Windows Qt5 release"]]
```
- Todo: use CSS classes
- Dictionary keys can't be referenced currently (v0.17), so use array offsets.
- The else condition could be simplified, if a default array argument (`default(array=[0,"No changes"])`) was supported (only literals currently). To fix this, preface the macro call by something like `{% set remarks = page.extra.remarks | default(value="") %}

```Jinja2
{% macro print_download_remarks(remarks, spacer="") -%}
	<p>
	{% if not remarks %}
		<span style="background-color: green">{{ spacer | safe }}</span> No requirements have changed<br>
	{% else %}
		{% for r in remarks %}
			<span style="margin-right: 5px; padding: 0px 10px; background-color:{% if r[0] == 0 %}green{% elif r[0] == 1 %}yellow{% else %}red{% endif %}">{{ spacer | safe }}</span> {{ r[1] }}<br>
		{% endfor %}
	{% endif %}
	</p>
{% endmacro print_download_remarks -%}
```

### Including .txt files like changelogs, requirements, etc.

For markdown content shortcodes like `load_data()` are available. 

For HTML templates, inclusion can be done as raw text, or using a preformatted code block - I prefer the latter. Using a `<pre><code>` block often suffices, otherwise check your current theme's method of styling code blocks.

### Decorating download links

The macro `assets2downloads` matches co-located filenames to a platform, generating a pretty menu. It currently depends on the filename sorting order to a) KISS, and b) work around some scripting limitations. The script is discussed separately here (or see the series menu).


## Beware...
### Shared variable scopes
For macros variables, stick to using its arguments (as recommended on the Tera page), as macros have access to the page/section scope. When porting shortcodes and/or building recursive functions, this can produce strange results. 

### The metadata_only flag
Functions like [`get_section`](https://www.getzola.org/documentation/templates/overview/#get-section) offer a `metadata_only` flag for efficiency - it will then not populate the `section.pages` and `section.subsections`.

### Passing either paths or objects
One may pass paths or objects. Object allow working with it without needing the various `get_section()` functions.


## Regular expressions
Provided are a few regular expressions for changing content, mostly URLs.

**Be careful when testing** with the `sed -n` and `p (print)` options. If not using version control, a pre/post directory comparison can be made using `diff -r`  or `diff -ru0`.

### Changing file/image URLs

It's easiest to store images with the post content (bundling or co-location). When moving such files to or from `/static/`, the content URLs need to be modified. The regex below suffices for this task, though it may not cover all situations.

```bash
sed -E 's+\!\[(.*)\]\((.\+)\)+![\1](/files/avater/manual/\2)+' somewhere/manual_en.md > manual_en.md 
# Test using sed -En 's+regexhere+p' manual_en.md
```
With Powershell avoid the default unicode conversion (always something... ;)), if desired.
```PowerShell
get-content somewhere/manual_en.md | 
 %{$_ -replace '!\[(.*)\]\((.+)\)' , '![$1](/files/avater/manual/$2)'} | 
 Out-File -Encoding ascii -FilePath manual_en.md
```

### Changing internal path URLs

File layout changes may also necessitate changes. A quick example using `find -exec`.

```bash
find . -iname "*.md" -exec sed -i -E 's+systemrequirements+requirements+' {} \;
# test using sed -n -E 's+systemrequirements+requirements+p' {} \;
```
Linux/GNU also offer the `sed -s` multi-file option:

```bash
sed -s -i 's+(@/software/avater/info.md#requirements)+(/software/avater/#requirements)+' *.md
```
