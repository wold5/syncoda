+++
title = "Building a software development section with a Zola blog"
description = "Hosting software package binaries with a Zola based website: macros, tips and tricks"
weight = 0
draft = false
date=2024-02-22

[taxonomies]
tags = ["website", "Zola"]

[extra]
toc = true
series = "Zola"

+++

This series discusses the internal Zola setup and macros used for syncoda.nl its software pages. Parts of it may prove useful to you, or jump start development of your website. 

Zola is a Rust-based SSG using a Jinja2 based templating dialect (Tera). Macros and examples may thus work for Jinja2 based SSGs like Hugo. 

<!-- more -->

Some familiarity with Zola is preferred, i.e., this supplements a beginners tutorial like that in the [Zola docs](https://www.getzola.org/documentation/getting-started/overview/). If you're experienced, consider skipping to the (future) macro section right away. The website source will be made available soon.

## Motivation for using a SSG website
Consider your specific usecase. If Github/lab suffices for your development efforts, sticking to a simple blogging setup will save both time and effort. Adding dedicated pages or building a full website, will be useful for large, closed source or legacy projects. \
For me, this site hosts several projects, and doubles as a portfolio, making it worthwhile enough. As for SSGs: providing a static HTML site removes a lot of potential headaches. And they can be fun to work with, if only to circumvent scripting limitations.

## File layout considerations

The current syncoda.nl layout contains a separate software section, with subsections for each project, and further subsections that contain the actual content pages, like:

```
├── posts/
│   ├── 2022/
│   ├── 2023/
│   │   └── _index.md
│   └── _index.md
├── software/
│   ├── _index.md      # the /software/ section index.html
│   ├── avater/
│   │   ├── _index.md  # section index forwarding to pages/info/
│   │   ├── manuals/
│   │   ├── pages/
│   │   └── releases/
│   └── project #2
```

Each project's home URL would ideally be just `/project/` without forwarding to the '/pages/info/' part. This is doable, when a section index (`_index.md`) is made to behave as a page, though this isn't supported by default by most themes (requiring DIY). I prefer to keep the separation between sections and pages.

### Project releases section
Creating release pages like `/releases/0.15/` inside a `project/releases/` (sub)section provides some benefits. It creates a directory (sub)structure that follows regular software projects, and can be distributed independently; and allows for easy internal referencing, especially programmatically using macro and template scripting. 

### Version numbers and safe URLs (slugification)
Version numbers in paths like `0.15` will by default be replaced by `0_15`, because of an URL sanitization feature called [slugification](https://www.getzola.org/documentation/content/page/#output-paths). For Zola, setting the config setting to `"slugify.paths = "safe"` limits the disallowed characters, allowing a dot.

- if your URLs used to contain dashes, forwards for the old URLs can be created by adding them to the page frontmatter `aliases=[]` array. 
- currently the `"on"` setting also doesn't slugify the asset paths, and whether an oversight or not, using `safe` is needed regardless.

### Getting a new release on the frontpage: transparancy
Sections have a `transparancy` toggle, that forwards their pages to the parent section, and so on. For blogs, yearly posts from `/posts/2022/` and `/posts/2023/` can thus be amalgated under `/posts/`, up to and on the frontpage. The same applies for the `/releases/` pages, which may appear on the frontpage.

There are two drawbacks: 
- if a parent section contains page we don't want to 'forward upwards', these need to be relocated to a non-transparant section. The `/project/pages/` section is one such example.
- any next/previous page navigation links _on_ a forwarded page, will take into account any siblings at its highest located section: for example, if this is the frontpage, these will be other frontpage pages (one may possibly script around this).

### Storing binaries and files in each release directory
_If your binary packages are stored externally, this topic is less relevant._

#### Files in `/static/` or locations other than the page
If not co-located with the page, the file locations need to be provided somehow. These may be stored in the page content, frontmatter, or some manifest file (perhaps even a co-located one) that can be processed.

#### Co-location with the page
Storing relevant files (images, binaries) together with the webpage in a directory, is called '[co-locating assets](https://www.getzola.org/documentation/content/overview/#asset-colocation)'. This makes each page and project directory an independent entity, more easily moved.

- Zola makes co-located files accessible via a `page.assets` array that can be iterated. 

    - This currently (0.17) includes any subfolders, allowing subfolder structures like `.../0.15/windows/`.

- Beware using URLs sanitization, as discussed previously

One downside of co-location is that 'simple' file syncs will read all data (and if remote also transfer), possibly including large binaries. For remote (SSH) connections, consider using `rsync --checksum` or some variant. Matching on sizes and dates will sometimes flag too many files (perhaps due to generating the site on various machines).

## Generic considerations

### Shortcodes vs macros and templates
Shortcodes are aimed at formatting small snippets of code, like image or youtube video. When starting, it can be attractive to use them for complex scripting tasks: they can be freely located within the content; support markdown; and don't require (overriding) templates (that also requires syncing updates eventually). For any complex scripting, really consider using macro functions and templates instead. 

Templates are used to build the actual pages (inserting content, etc.) and can contain scripting. A better convention is to call macro functions for scripting tasks. This better separates layout from content. Macros also provide more features in being able to access arbitrary sections and pages, and support both nesting and recursion. 


#### Converting shortcodes to macros
Beware macros can access their parent scope, which can be problematic when when recursing and variables overlap. Preferably use only their arguments (as recommended). 

### Template scripting limitations 

One last remark on the SSG scripting languages (usually Jinja2 variants): their commandset and features tend to be limited, but sufficient. Missing 'features' tend to imply there are different way(s) of achieving them. Given scripts run only locally during testing and building, optimization is also less of an issue. Mainly accept scripts won't necesserely be smart or pretty (as do mine, that was the whole point eh? ;). 

Listed are some specific limitations I encountered with Zola (v0.17, 2023) and its [Tera](https://tera.netlify.app/docs/) (Jinja-like) templating engine. _Not to diminish either, but to excuse any apparantly poor scripting on my, and perhaps your, behalve._

- regex matching: limited to Boolean matches, no capture groups _(rarely used)_
- can't append to **nested** arrays. There are [workarounds](https://zola.discourse.group/t/allow-load-data-to-take-a-literal/1165/3), but they're not pretty.
- Iterating over files requires using '[co-locating assets](https://www.getzola.org/documentation/content/overview/#asset-colocation)'. One can't (yet) get files from arbitrary directories, [but see this proposal](https://zola.discourse.group/t/new-global-functions-get-files-or-similar/307) _(BTW, I've come to prefer the co-locating approach)_
- nested or recursive macro functions are limited to templates; their in-content equivalent are 'shortcodes', but [shortcodes within shortcodes](https://github.com/getzola/zola/issues/515) aren't supported (yet). _(recommend just going with templates and macros)_

As noted, these and other limitations often pose nice challenges, with reasonable solutions being achievable. Do remember to watch the clock though.
