+++
title = "Integrating documentation markdown into a Zola (SSG) website"
draft = false
weight = 3
aliases = ["posts/2023/2023-06-09-website_manual_integration/"]
date=2024-02-24

[taxonomies]
tags = ["website", "Zola"]

[extra]
toc = true
series = "Zola"
+++

Brief notes on integrating the [AVATeR program user manual](/software/avater/manuals/manual_en/) into this Zola SSG based website. The intent is to let the SSG generate webpages as part of the website, thus integrating them, instead of having them as a separate part.

<!-- more -->

The manual in question is maintained separately, to allow inclusion with the software package and the website. It currently consists of a _single markdown file_, but may be extended to multiple files in the future.

_Note this article was originally more extensive, including an introduction to transparancy and sections - most of this was moved to this series of articles._

## One to a few pages: importing markdown files

For one or just a few pages, one easy solution is importing the raw content into an existing SSG page. This is currently used for the AVATeR program manual.

For Zola, use the [`load_data()`](https://www.getzola.org/documentation/templates/overview/#load-data) function with `format="plain"` to import markdown text. Note:

- using it within the markdown content, requires calling it indirectly from within a '[shortcode](https://www.getzola.org/documentation/content/shortcodes/)', or using a macro and template combination.
- local source file changes may require restarting Zola (a known issue)

## Multiple pages: adding 'frontmatter' metadata blocks

For larger manuals, it will be more practical to transform the source files into a format the SSG will accept. Zola and most SSGs require each page to start with a metadata ('[frontmatter](https://www.getzola.org/documentation/content/page/#front-matter)') block like below:

```
+++
title = "Integrating documentation into a Zola (SSG) website"
draft = false
date = 2023-01-01
+++

Content here...
```
Note that for Zola _none of the_ frontmatter variables are mandatory, but the block itself is. Templates and some section sorting modes (i.e. weight), may however complain if expected variables are missing. Prepending such a block to each page can be achieved with scripting and/or a template-based documentation generator (i.e. Sphinx). 

This approach wasn't yet needed for syncoda.nl; we do touch on the topic of navigation menus, which can be generated using Zola as discussed next. 


## Navigation, templates and the directory layout

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
For 'layout1', the section index `/project1/_index.md` could list any of its child pages (subsections, etc.) using a template. The template can iterate over a section's pages using its `section.pages` array.

The template for its childpages in turn, can access their parent `/project1/` section, and provide a similar menu. Its template would then call [`get_section`](https://www.getzola.org/documentation/templates/overview/#get-section) with the parent path as argument (hint: set `metadata_only=false`), and similarly iterate over the pages. 

### Transparancy
When using transparancy (forwarding pages to parent sections), the manual page will also be forwarded. If this is not desired, layout 2 moves the manual into its own subsection, which can have `transparancy` disabled: 

```
└── layout2
    └── software
        ├── project1
        │   ├── _index.md
        │   └── manual      
        │       ├── _index.md   # 'manual' its section index
```
The `project1/manual/_index.md` section index could then list the available manuals (best), forward to a manual page, or show the manual itself.

## Themes for documentation
While these won't necesserely integrate with the main blog or website, themes (collections of templates) exist that allow building independent documentation websites, current examples being:
- [book](https://github.com/getzola/book)
- [adidoks](https://github.com/aaranxu/adidoks)
- [doks (Hugo)](https://github.com/h-enk/doks). 

Any template files thereof can be modified by overriding them competely, or partially (blocks), by creating a modified copy in the site root directory `/templates/`. The [Tera](https://keats.github.io/tera/docs/#inheritance) manuals provides some details, as do the sources of example website.

- Updated May 24th 2023: some text changes
- Updated June 10th 2023: major rewrite, use subsections instead
- Updated February 2024: split text over several articles
