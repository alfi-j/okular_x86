# Okular – Universal Document Viewer

Okular can view and annotate documents of various formats, including PDF, Postscript, Comic Book, and various image formats.
It supports native PDF annotations.

### Downloads

For download and installation instructions, see https://okular.kde.org/download.php

### User manual

https://docs.kde.org/?application=okular&branch=stable5

### Bugs

https://bugs.kde.org/buglist.cgi?product=okular

Please report bugs on Bugzilla (https://bugs.kde.org/enter_bug.cgi?product=okular), and not on our GitLab instance (https://invent.kde.org).

### Mailing list

https://mail.kde.org/mailman/listinfo/okular-devel

### Source code

https://invent.kde.org/graphics/okular.git

The Okular repository contains the source code for:
 * the `okular` desktop application (the “shell”),
 * the `okularpart` KParts plugin,
 * the `okularkirigami` mobile application,
 * several `okularGenerator_xyz` plugins, which provide backends for different document types.

### Apidox

https://api.kde.org/okular/html/index.html

## Contributing

Okular uses the merge request workflow.
Merge requests are required to run pre-commit CI jobs; please don’t push to the master branch directly.
See https://community.kde.org/Infrastructure/GitLab for an introduction.

### Build instructions

Okular can be built like many other applications developed by KDE.
See https://community.kde.org/Get_Involved/development for an introduction.

If your build environment is set up correctly, you can also build Okular using CMake:

```bash
git clone https://invent.kde.org/graphics/okular.git
cd okular
mkdir build
cd build
cmake -DCMAKE_INSTALL_PREFIX=/path/to/your/install/dir ..
make
make install
```

Okular also builds tests in the build tree. To run them, you have to run `make install` first.

If you install Okular in a different path than your system install directory it is possible that you need to run

```bash
source prefix.sh
```

so that the correct Okular instance and libraries are picked up.
Afterwards one can run `okular` inside the shell instance.
The source command is also required to run the tests manually.

As stated above, Okular has various build targets.
Two of them are executables.
You can choose which executable to build by passing a flag to CMake:

```bash
cmake -DCMAKE_INSTALL_PREFIX=/path/to/your/install/dir -DOKULAR_UI=desktop ..
```
Available options are `desktop`, `mobile`, and `both`.

### Automated builds with GitHub Actions (downloadable binaries)

A ready-to-use workflow is provided at `.github/workflows/build.yml`. Once you
push this repository to GitHub, it compiles **32-bit x86 (i686)** binaries for
Linux and Windows on every push/pull request and uploads them as **downloadable
artifacts** from the run's *Summary* page:

- **Linux (i686 / 32-bit)**: built natively inside the official
  [`archlinux32/archlinux32`](https://gitlab.archlinux32.org/archlinux32/docker-images)
  container, which ships maintained i686 packages for Qt6, KDE Frameworks 6,
  ECM, Poppler-Qt6 and friends — no `-m32` hacks needed. Produces
  `okular-linux-i686.tar.gz` (staged install tree) and, best-effort, an
  AppImage (`linuxdeploy` has no official i686 release, so the tarball is the
  guaranteed artifact).
- **Windows (i686 / 32-bit)**: built with the MinGW-w64 `i686-w64-mingw32`
  toolchain from MSYS2 (`msys2/setup-msys2`, `MSYSTEM=MINGW32`). Every
  dependency comes from the multilib `mingw-w64-i686-*` packages installed via
  `pacman -m32` (Qt6, KF6, poppler-qt6, FreeDSPdfium, DjVuLibre, TIFF…), so
  the result is a genuine 32-bit binary — never x64. A verification step
  asserts `__SIZEOF_POINTER__ == 4` and that `_WIN64` is undefined before
  configuring. Produces a portable `okular-windows-i686.zip` prepared with the
  i686 `windeployqt`.
- **Releases**: pushing a tag like `v26.11.70` additionally publishes all
  artifacts as assets of a **GitHub Release**, so anyone can download the
  binaries from the Releases page.

Missing optional libraries (DjVu, EPub, Mobipocket, Markdown, PostScript…) are
downgraded to non-fatal through Okular's own
`FORCE_NOT_REQUIRED_DEPENDENCIES` CMake option, so the configure step never
fails because of them; only the corresponding document formats get disabled.
Poppler is deliberately kept required (it provides PDF support, Okular's core
feature) and is available for both i686 targets above.

How to use it:

```bash
git remote add origin https://github.com/<tu-usuario>/okular.git
git push origin master
# o lanzarlo manualmente: Actions -> "Build Okular" -> "Run workflow"
# (permite elegir Release / RelWithDebInfo / Debug)
```

Then go to the run's *Summary* page and download the artifacts
(`okular-linux-i686`, `okular-windows-i686`). Artifacts from `main`/`master`
pushes and releases are kept indefinitely if you enable artifact retention;
PR artifacts expire after 90 days by default.

> **Note:** the previous version of this workflow used KDAB's
> `kdab/cmake-recipes-github-ci` container and KDE's prebuilt *windows-c++*
> (MSVC x64) bundles. That image tag is no longer publicly pullable (CI failed
> with `pull access denied`) and neither stack offers i686 packages, which is
> why the jobs were rewritten around Arch32 and MSYS2-multilib. For building
> 32-bit binaries locally (e.g. directly on CachyOS with an Arch32 chroot or
> `i686-w64-mingw32`), see the project wiki instructions for
> `FORCE_NOT_REQUIRED_DEPENDENCIES` and cross-compilation flags.

### clang-format

The Okular project uses clang-format to enforce source code formatting.
See [README.clang_format](https://invent.kde.org/graphics/okular/-/blob/master/README.clang-format) for more information.
