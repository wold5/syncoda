+++
title = "Building a software section with a Zola SSG blog"
description = "Building a software development section with a Zola SSG based website: tips, tricks and macros"
weight = 0
draft = false
date = 2024-02-29
aliases = ["/software/zola_tips/intro/"]

[taxonomies]
tags = ["website", "Zola", "Tera 1.x"]

[extra]
toc = true
series = "Zola"

+++


This series discusses the _internal_ Zola SSG setup used for this website, build over the past 2 years, with an emphasis on its software pages. The design (theme) isn't discussed. Parts of this series may prove useful, or jump start development of a website. The site source is available on [github](https://github.com/wold5/syncoda/).
<!-- more -->
The series is aimed at beginning to intermediate Zola users, that have some familiarity with Zola - experienced users may skip most (if not all) topics. A brief beginners tutorial is provided in the [Zola docs](https://www.getzola.org/documentation/getting-started/overview/).

_Note this pages re-integrates parts of an earlier article on integrating a manual._ 

### Motivation for building a (SSG) website
Do (re)consider your specific usecase. If Github, Gitlab, etc. suffices for your development efforts, a simple blogging setup will save time and effort. As this website hosts several projects _with_ binaries and doubles as a portfolio, it was worthwhile enough.

As for SSGs: a static HTML site removes some headaches, but also introduces others. They do tend to be more fun to work with. Some build their own SSG solution; sharing that task is easier.

## Zola
Zola is one of many SSGs, in active development with a reasonably sized community. It is Rust-based and uses a Jinja2 derived templating dialect called Tera. Macros and examples may thus be (somewhat) compatible with other Jinja2 based SSGs like Hugo. 

**The current setup discusses Zola v0.17-0.18 with Tera v1.x**.


## File layout considerations
For a typical SSG, the directory structure corresponds to some internal 'tree' of the website. Zola further distinguishes between sections and pages: sections can contain pages, and also other (sub)sections, and so on. The 'blog frontpage' itself is actually a section (as for sections, some changes may happen eventually with Tera v2.0). \
Sections inherently have access to the metadata of their pages and subsections: this allows construction of navigation menus and page listings, but we are getting ahead of ourselves here. 

### Sections or category/taxonomies/labels
One can organise posts using the aforementioned sections (or directories), creating a hierarchy. 

One alternative is the 'flat' model, where all posts reside under `/posts/`, and 'grouping' is achieved by assigning each post some taxonomy/category label, like `project #1 release`. Next, one retrieves all pages (and sections) for such a label (i.e. a release list), in that way providing some form of navigation or grouping. \
The downsides are minor: one can forget adding the label; and any changes require changing the metadata (called frontmatter) for multiple pages (but cue regex tools).

### Example using sections
As this website uses sections, these will be discussed: listed below is a partial file layout of the current (2024) website. The website contains separate blog and software section; the latter has subsections for each software project, with nested subsections containing the actual content pages, like:

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

A project's home URL would ideally be just `/project/`, but here such a _section_ forwards to some `/pages/info/` _page_. Configuring such a section to behave as a page is doable, avoiding this forwarding, but theme support for this tends to be limited (no TOCs for one), requiring template modification; and one loses the (conceptual) separation between pages and sections. Technical benefits exist such as 'transparency', which will be discussed later. Redirection does mean users get to see a redirect page briefly (this might be fixable in some future version, for internal website links). 

### Project releases sections
Structuring the software release pages inside a subsection like `/releases/0.15/` provides benefits:
- a directory (sub)structure that follows regular software projects, and can be distributed independently 
- allows easy internal referencing, using scripting or manual linking

Note `/0.15/index.md` is actually a page (being a directory, implies 'co-located' files, i.e. any files associated with that page) as it lacks the underscore prefix. We will get back on this in a moment.

#### Gotcha: version numbers and safe URLs (slugification)
Note: version numbers in paths like `0.15` are by default transformed into `0_15`, because of an URL sanitization feature called [slugification](https://www.getzola.org/documentation/content/page/#output-paths). For Zola, change the configuration setting to `"slugify.paths = "safe"` to allow additional characters in the URLs (such as a dot). We note this early on, to avoid later issues.

Note:
- changing the slugify settings will break any pre-existing external links. However, forwarding pages can be created for these 'old' URLs automatically: add the old path (the one with the underscore) to the page frontmatter `aliases=[]` array
- currently the `"on"` setting doesn't slugify the asset paths, and whether an oversight or not, using `safe` is thus needed regardless

### Getting a new release on the frontpage: transparency
How to get a release post on the frontpage? There's no need to double post: sections have a toggle called `transparent`: when enabled this will forward their pages to a parent section. This can happen all the way up to the frontpage (also a section). Each of the intermittent sections will list these pages (including any of their own). For example, the yearly posts from `/posts/2022/` and `/posts/2023/` can be amalgamated under `/posts/`, all the way up to the frontpage.

Some notes: 
- if a section contains pages we don't want to make transparent, these need to be relocated into a non-transparent section. The `/project/pages/` section is one such example. _There have been requests for creating a page exclusion flag. I however prefer the section approach, avoiding the need to chase flags (one can `grep` of course)._
- navigation links to the _next or previous page/post_ on a forwarded page, will take into account its siblings at its highest located section in the tree. For example, on the frontpage these will be other frontpage pages (one could possibly script around this).
- to list projects, one now needs to list _subsections_ for the `/software/` section. To this end, use a template like our [/software/](/software/) page, discussed on the macros page.


### Storing binaries and files in each release directory
_If your binary and/or source packages are stored externally, this topic doesn't apply._ For this website, release packages are currently bundled with each release page in a directory, called 'co-location', as discussed next. Alternatively, store them somewhere in `static`.

#### Co-locating files with the page
A page can be bundled with relevant files (images, binaries, etc.) into a directory, called '[co-locating assets](https://www.getzola.org/documentation/content/overview/#asset-colocation)'. It makes each page (and ultimately the project) an independent entity, easily moved and changed.

- Zola collects co-located file paths into the `page.assets` array. Currently (0.17+) this includes any subdirectories, allowing subdirectories structures like `.../0.15/windows/some_installer.exe`.

- Beware using URLs sanitization, as discussed previously

#### Storing files in `/static/` or locations other than the page
If files are not co-located with the page, store them in some `/static/` subdirectory, or an external location. These file locations do now need to be conveyed to the page somehow. These could be stored in the page content, frontmatter, or some (co-located) manifest file, or automatically generated using a template/macro script.

#### Gotcha: file syncs
One downside of (self-)hosting binaries, especially with the co-located approach, is that 'simple' file syncs now read (and transfer) all data, which can include large binaries.

- For remote connections, consider using `rsync --checksum` with a remote rsync instance.- Syncing on size and/or date differences can help, but may detect spurious differences (presumably a side effect of building the website on various machines). 
- Beware using a size-only match with fixed-size files like update manifests.

#### Gotcha: file sizes and hosting webspace
Nowadays webspace is cheap and aplenty, except for `git` hosting services (that may also build the site using Zola), or if want to keep costs low. 
For software, *nix releases without dependencies can be quite small, 1-2MB each. For Windows the file sizes do add up over time due to the included dependencies. Packaged Windows Qt GUI apps start around 20MB. As typically a `.zip` is bundled with an `.exe` and/or `.msi`, this figure doubles for each release.
