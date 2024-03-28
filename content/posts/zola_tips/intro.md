+++
title = "Building a software section with a Zola SSG blog"
description = "Building a software development section with a Zola SSG based website: tips, tricks and macros"
weight = 0
draft = false
date = 2024-02-29
aliases = ["/software/zola_tips/intro/"]

[taxonomies]
tags = ["website", "Zola"]

[extra]
toc = true
series = "Zola"

+++


This series discusses the _internal_ Zola SSG setup used for this website, with an emphasis on the software pages. The (design) theme used isn't discussed ([Abridge](https://github.com/Jieiku/abridge/)). Parts of this series may prove useful, or jump start development of a website. The site source is available on [github](https://github.com/wold5/syncoda/).
<!-- more -->
The series is aimed at beginning to intermediate Zola users, that have some familiarity with Zola - experienced users may skip most (if not all) topics. A brief beginners tutorial is provided in the [Zola docs](https://www.getzola.org/documentation/getting-started/overview/).

_Note this pages re-integrates parts of an earlier article on integrating a manual._ 

### Motivation for building a (SSG) website
Do (re)consider your specific usecase. If Github, Gitlab, etc. suffices for your development efforts, a simple blogging setup will save time and effort. This site hosts several projects _with binaries_ and doubles as a portfolio, making it worthwhile enough.

As for SSGs: a static HTML site removes some headaches, but also introduces others. They do tend to be more fun to work with.

## Zola
Zola is one of many SSGs, in active development with a reasonably sized community. It is Rust-based and uses a Jinja2 derived templating dialect called Tera. Macros and examples may thus be somewhat compatible with other Jinja2 based SSGs like Hugo. 

**We use Zola v0.17-0.18 with Tera v1.x**.

## File layout considerations
For a typical SSG, the directory structure will correspond to some internal 'tree' of the website. Zola further distinguishes between sections and pages: sections can contain pages, and also other (sub)sections, and so on. The 'blog frontpage' itself is actually a section (as for sections, some changes may happen with Tera v2.0 eventually). \
Sections inherently have access to the metadata of their pages and subsections: this allows construction of navigation menus and page listings, but we are getting ahead of ourselves here. 

First an example, a partial view of the current (2024) website file layout. The website offers a separate blog and software section; the latter with subsections for each project, and further subsections containing the actual content pages, like:

```
├── posts/
│   ├── 2022/
│   ├── 2023/
│   │   ├── _index.md
│   │   └── 2023-01-04-some_blog_post.md
│   └── _index.md
├── software/
│   ├── _index.md             # section /software/ overview
│   ├── avater/
│   │   ├── _index.md         # section /avater/ index that forwards to pages/info/
│   │   ├── manuals/
│   │   ├── pages/
│   │   │   ├── _index.md     # section /pages/ overview
│   │   │   └── info.md       # page with project information 
│   │   └── releases/
│   │       ├── _index.md     # section /releases/ overview
│   │       └── 0.16/
│   │           └── index.md  # page, in 'co-located' form in the /0.16/ directory, with v0.16 release info
│   └── project #2
```

### Project homepages: forwards/redirects

A project's home URL would ideally be just `/project/`, without forwarding to a `/pages/info/` page. This is doable, if a section (`_index.md`, with underscore) is configured as a page. Theme support for this tends to be limited (no TOCs for one), requiring template modification. Personally, I prefer to keep the separation between pages and sections, and use this forwarding setup. Other reasons such as 'transparency' will be discussed later.

### Project releases sections
Structuring the software release pages inside a subsection like `/releases/0.15/` provides benefits:
- a directory (sub)structure that follows regular software projects, and can be distributed independently 
- allows easy internal referencing, using scripting or manual linking

Note `/0.15/` is actually a page ('co-located', i.e. it contains files for that page) as its `index.md` lacks the underscore prefix. We will get back on this in a moment.

#### Gotcha: version numbers and safe URLs (slugification)
Version numbers in paths like `0.15` are by default transformed into `0_15`, because of an URL sanitization feature called [slugification](https://www.getzola.org/documentation/content/page/#output-paths). For Zola, change the configuration setting to `"slugify.paths = "safe"` to allow additional characters in the URLs (such as a dot).

Note:
- any pre-existing external links will break. However, forwarding pages can be created for the old URLs automatically, by adding the old path (the one with the underscore) to the page frontmatter `aliases=[]` array
- currently the `"on"` setting doesn't slugify the asset paths, and whether an oversight or not, using `safe` is thus needed regardless

### Getting a new release on the frontpage: transparency
How to get a release post on the frontpage? There's no need to double post: sections have a toggle called `transparent`, which will forward their pages to a parent section for inclusion. This can happen all the way up to the frontpage (also a section). All 'transparent' sections will list these pages, including their own. As an example, the yearly posts from `/posts/2022/` and `/posts/2023/` can be amalgamated under `/posts/`, all the way up to the frontpage.

Some notes: 
- if a section contains pages we don't want to make transparent, these need to be relocated into a non-transparent section. The `/project/pages/` section is one such example. _There have been requests for creating a page exclusion flag. I however prefer the section approach, avoiding the need to chase flags (`grep` helps of course)._
- navigation links to the next or previous page/post links on a forwarded page, will take into account its siblings at its highest located section in the tree. For example, on the frontpage these will be other frontpage pages (one could possibly script around this). Disabling transparancy half-way can thus give odd results, as sorting is for example now done by `weight`.
- to list projects, one now needs to list _subsections_ for the `/software/` section. To this end, use a template like our [/software/] page, discussed on the macros page.


### Storing binaries and files in each release directory
_If your binary and/or source packages are stored externally, this topic doesn't apply._ For this website, release packages are currently bundled with each release page in a directory, called 'co-location', as discussed next. Alternatively, store them somewhere in `static`.

#### Co-locating files with the page
A page can be bundled with relevant files (images, binaries, etc.) into a directory, called '[co-locating assets](https://www.getzola.org/documentation/content/overview/#asset-colocation)'. It makes each page (and ultimately the project) an independent entity, easily moved and changed.

- Zola collects co-located file paths into the `page.assets` array. Currently (0.17+) this includes any subfolders, allowing subfolder structures like `.../0.15/windows/some_installer.exe`.

- Beware using URLs sanitization, as discussed previously

#### Storing files in `/static/` or locations other than the page
If files are not co-located with the page, store them in some `/static/` subdirectory, or an external location. The file locations does now need to be conveyed to the page somehow. These could be stored in the page content, frontmatter, or some (co-located) manifest file, or automatically generated using a template/macro script.

#### Gotcha: file syncs
One downside of (self-)hosting binaries, especially with the co-located approach, is that 'simple' file syncs now read (and transfer) all data, which can include large binaries. For remote connections, consider using `rsync --checksum` with a remote rsync instance. \
Syncing on size and/or date differences can help, but may detect spurious differences (presumably a side effect of building the website on various machines). And beware using a size-only match with files like update manifests, as these tend to have a fixed size.

#### Gotcha: file sizes and hosting webspace
Nowadays webspace is cheap and aplenty, but you may be tied to some `git` hosting service (that may also build the site using Zola), or want to keep costs low. *nix releases without dependencies canbe small, at 1-2MB. For Windows the file sizes can add up over time due to dependencies being included. Packaged Windows Qt GUI apps start around 20MB. As typically a `.zip` is bundled with an `.exe` and/or `.msi`, this figure doubles for each release.
