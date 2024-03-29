+++
title = "Integrating documentation markdown into a Zola (SSG) website"
draft = false
weight = 3
date=2024-03-23
aliases = ["posts/2023/2023-06-09-website_manual_integration/", "software/zola_tips/2024-02-24-website_manual_integration/"]

[taxonomies]
tags = ["Zola"]

[extra]
toc = true
series = "Zola"
+++

Brief notes on integrating the [AVATeR program user manual](/software/avater/manuals/manual_en/) into this Zola (SSG) based website. The intent is to let the SSG generate a webpage using the original markdown file, thus integrating it into the website, instead of hosting a separate 'website' (but see also the [documentation themes](#themes-for-documentation)).

<!-- more -->

## The manual
The manual in question is maintained separately, to allow inclusion with software packages and the website. It consists of a _single markdown file_, but may be extended to multiple files in the future.

_Note parts of the original article were moved into the introduction article._

## One to a few pages: importing markdown files

For one or just a few pages, an easy solution is to import the raw markdown (or HTML) content into an existing SSG page. This is currently used for the AVATeR program manual.

For Zola, use the [`load_data()`](https://www.getzola.org/documentation/templates/overview/#load-data) function with `format="plain"` to import markdown or HTML text. Note:

- using it within the markdown content, requires calling it indirectly from within a '[shortcode](https://www.getzola.org/documentation/content/shortcodes/)', or using a macro and template combination.
- local source file changes may require restarting Zola (a known issue)

## Multiple pages: adding 'frontmatter' metadata blocks

For multi-file manuals, it is more practical to transform the source files into a format that a SSG will accept. Zola and most SSGs require each page to start with a metadata ('[frontmatter](https://www.getzola.org/documentation/content/page/#front-matter)') block, as shown below:

```
+++
title = "Integrating documentation into a Zola (SSG) website"
draft = false
date = 2023-01-01
+++

Content here...
```
Note that for Zola _none of the frontmatter variables_ are mandatory, but the _block_ itself is. Templates and some section sorting modes (i.e. weight, date), may however complain if required variables are missing. Prepending such a block to each page can be achieved with scripting and/or a template-based documentation generator (i.e. Sphinx). 

This approach wasn't yet needed for syncoda.nl, so we won't go into it extensively. We do touch on the topic of navigation menus, which can be generated using Zola as discussed next. For a related example, look at the [Zola website themes section sources](https://github.com/getzola/themes/), specifically the Python script.


## Navigation and the directory layout

_If you've read the [intro](/posts/zola_tips/intro/), some parts will be recognisable, as they were derived from this page. Nevertheless, some additions are only discussed here._

For Zola, navigation menus _can be_ derived from the section structure, which in turn is derived from the directory layout. You may off course also hardcode (externally generated) menu's.

### File layout considerations
For smaller sites, posts, pages and documentation can co-exist with some planning - assuming we use sections to build the navigation structure.

```
├──layout1
│       ├── project1
│       │   ├── _index.md   # section index
│       │   ├── info.md
│       │   ├── page.md
│       │   └── manual.md
│       ├── project2
│       └── project3
```
For 'layout1', the section index `/project1/_index.md` could list any of its child pages (subsections, etc.) using a template script: it could iterate over a section's pages using its `section.pages` array. \
The template for its childpages in turn, could access their parent `/project1/` section, and use the same script to  render the same menu. This requires calling [`get_section`](https://www.getzola.org/documentation/templates/overview/#get-section) with the parent path as argument (hint: set `metadata_only=false`) to get the section object, which can then be processed in a similar way. Using some macro function simplifies re-use considerably.

### Circumventing the transparency mode
When a section has `transparency` enabled, it forwards its pages to its parent section: doing so will also forward the manual page(s). To prevent this, example `layout2` moves the manual into a subsection, which has `transparency` disabled: 

```
└── layout2
    └── software
        ├── project1
        │   ├── _index.md
        │   └── manual      
        │       ├── _index.md   # 'manual' its section index, with transparency=false
```
The `project1/manual/_index.md` section index could then list the available manuals (best). It may also forward to a manual page, or contain the manual itself (less). 

This does remove the manual page from a menu like that discussed for `layout1`:
- you may link to it manually from other pages, ignoring the new subsection
- create a `manuals.md` page in the `project1` root, that contains links or redirects to the `manual` subsection. Note the name can't overlap that of the `manual` section directory.
- change the menu script to include subsections, by looping over the `section.subsections` variable. 

You may as well consider grouping all pages into sections, and constructing the menu from sections instead. This allows news related pages to be transparently forwarded (i.e. to the frontpage), and not project related pages. This is the setup currently used for this website.

## Themes for documentation
An alternative is building the manual independently using some theme (collection of templates). Themes exist aimed at documentation, with current examples being:
- [book](https://github.com/getzola/book) (based on Gitbook, a basic book theme)
- [adidoks](https://github.com/aaranxu/adidoks) (a `doks` port for Zola)
- [doks (Hugo)](https://github.com/h-enk/doks). 

### Customizing themes
Any template files of a theme can be modified by overriding them completely, or partially (targetting 'blocks'): this is done by creating a modified copy in the website root directory called `/templates/`. The [Tera](https://keats.github.io/tera/docs/#inheritance) manuals provides some details, as do the sources of example website. 

Do consider updating your modified files regularly, as [discussed in the tips section](/posts/zola_tips/zola_scripting_tips/#updating-syncing-templates-and-config-files).

---

- Updated May 24th 2023: some text changes
- Updated June 10th 2023: major rewrite, use subsections instead
- Updated February 2024: split text over several articles
