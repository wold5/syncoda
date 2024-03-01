+++
title = "AVATeR Linux releases"
updated = 2022-06-11T00:00:00+01:00
draft = true

[taxonomies]
tags = ["AVATeR"]

[extra]
application_name = "avater"
application_logo = "/images/logo-avater.png"

toc = true

+++

[AVATeR](/software/avater/) installers are now available for GNU/Linux Debian and deriviatives (Ubuntu, etc.). These installers come in two flavours: a standard and a backwards compatible version.


<!-- more -->

# Standard version

Use 'standard' for any recent based distro. These releases will generally include all features, and receive the most testing.

Currently aimed at Debian Bullseye and derived distro's (Ubuntu 20-21+). It's dependencies are listed below, with `libqt5gui5-gles` being optional: 
```
Depends: libc6 (>= 2.34), libgcc-s1 (>= 3.0), libqt5core5a (>= 5.15.1), libqt5gui5 (>= 5.8.0) | libqt5gui5-gles (>= 5.8.0), libqt5network5 (>= 5.0.2), libqt5sql5 (>= 5.0.2), libqt5widgets5 (>= 5.14.1), libstdc++6 (>= 11), libudev1 (>= 183), libzip4 (>= 1.6.0)
Suggests: libqt5gui5-gles (>= 5.15.2)
```

# Compatible version

The 'compatible' release targets the previous 'stable' release of Debian. Older releases support older library versions - but doing so often introduces limitations, and possibly regressions and bugs.

## 'Buster compatible' release (0.9.9+)
Aimed at Debian Buster (2019+) and derived distros (Ubuntu 18+, etc.). 
Limitations/remarks include:

- disabled cancel button for the backup progress dialog (due to older Libzip 1.3 library[^0])
- disabled changelog menu item, as markdown support was missing (due to older Qt 5.11.x library[^0])

The dependencies are listed below. Note `libqt5gui5-gles` isn't officially available on Debian Buster, so it can do without - the reference was kept just in case.

```
Depends: libc6 (>= 2.14), libgcc1 (>= 1:3.0), libqt5core5a (>= 5.11.0~rc1), libqt5gui5 (>= 5.8.0), libqt5network5 (>= 5.0.2), libqt5sql5 (>= 5.0.2), libqt5widgets5 (>= 5.4.0), libstdc++6 (>= 6), libudev1 (>= 183), libzip4 (>= 1.3.0)
Suggests: libqt5gui5-gles (>= 5.15.2)
```

# Future

A Qt6 version will arrive one day; the compatible version may then remain at Qt5, or be phased out in favour of standard versions for either Qt5/6.

RPM installers may arrive eventually; if you'd like one, let us know. 
A Flatpack all-in-one distribution is also being considered.



# Linux vs Windows version

AVATeR has few platform dependent features, thanks to using cross-platform libraries such as Qt. The main difference involve USB-related code.

The Linux version does appear to run faster, most likely due to libraries (USB, Qt). Regarding Qt, the 'resize all rows' function was excruciatingly slow on Windows; and Window's API function for retrieving device names can stall at times.


---


[^0]: As AVATeR is currently distributed as a pre-compiled binary, these limitations are binding (i.e. we can't dynamically change features during runtime).
