+++
title = "AVATeR Windows Qt5 release added to 0.15"
draft = false

[taxonomies]
tags = ["AVATeR"]

[extra]
toc = true
+++

A Windows Qt5-based release has been added to the [AVATeR 0.15 downloads](/software/avater/releases/0.15/#downloads_windows), for backward compatibility. 

<!-- more -->

While v0.15 (Qt 6.6) runs fine even on Win10 21H2, in the past v0.13 and older versions failed to start on newer Windows 11 versions due to some Qt 6.2.4 issue. To avoid similar situations, a Qt5 based version has been added; it might also run on older Windows 7/8 systems, as Qt6 requires Win10 at minimum.

Note
- Qt5 lacks some features of Qt6, better HDPI support being one
- included (C++) libraries could still prove incompatible with older Windows versions. It was tested with Win11 23H2 and Win10 21H2.
- Only a `.zip` archive version is shipped
- used are Qt 5.15.2, build for AMD64 on Win11 23H2 using MSVC 2022. 

<br>

---

<br>

## For developers: adding backward Windows Qt5 support
Note AVATeR's codebase supports Qt5 (for Linux releases), so this only required toggling some compiler flags (which QtCreator probably did earlier), as listed below. If needing to backport to Qt5 afresh, there will be more work to do. When sticking to 5.15.x, and not using specific Qt modules, it should be doable (the to-be-released AVATeR sources will showcase some required modifications, enclosed by preprocessor directives).

### For WIN32 API function calls and structs, set the UNICODE flag
This assumes you use wide or UTF16 (`w_char/std::wstring`) strings. \
For CMake with MSVC add: `string(APPEND CMAKE_CXX_FLAGS " /DUNICODE")`.

If not possible, any relevant WIN32 function calls and structs need to use their UTF-16 'Wide' `*_W` appendix variants (note by now a Windows beta UTF-8 mode also exists, that updates the ANSI functions to UTF-8).

### Remove 'written' logical operators like `not` and `and`
An odd one, apparently fixable with the `/permissive-` flag (preferred over `/Za`, according to sources), but using these operators was inconsistent anyway[^1]. When these don't compile, the associated `if` conditions appear out of place, creating an error avalanche, but don't be discouraged.


---

[^1]: Once after hunting a bug due to missing a negation (`!`), I temporarely switched to using `not` in some (older) code parts. Why, I now ask... but that's been reverted. `not not. Who's there? not not.`


