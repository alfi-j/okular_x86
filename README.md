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
push this repository to GitHub, it will compile Okular on every push/pull
request and upload the resulting **binaries as downloadable artifacts**:

- **Linux (x86_64)**: built inside the KDAB `cmake-recipes` CI container
  (Qt 6.9 + KDE Frameworks 6.13 preinstalled). Produces
  `okular-linux-x86_64.tar.gz` (staged install tree) and, best-effort, an
  **AppImage**.
- **Windows (MSVC x64)**: Qt is installed with `jurplel/install-qt-action`
  and KDE Frameworks + third-party deps (Poppler, TIFF, DjVu…) are fetched
  from KDE's prebuilt *windows-c++* bundles on
  `download.kde.org/stable/frameworks/<KF_VERSION>/windows/`. Produces a
  portable `okular-windows-x86_64.zip` prepared with `windeployqt`.
- **Releases**: pushing a tag like `v26.11.70` additionally publishes all
  artifacts as assets of a **GitHub Release**, so anyone can download the
  binaries from the Releases page.

How to use it:

```bash
git remote add origin https://github.com/<tu-usuario>/okular.git
git push origin master
# o lanzarlo manualmente: Actions -> "Build Okular" -> "Run workflow"
# (permite elegir Release / RelWithDebInfo / Debug)
```

Then go to the run's *Summary* page and download the artifacts
(`okular-linux-x86_64`, `okular-windows-x86_64`). Artifacts from `main`/`master`
pushes and releases are kept indefinitely if you enable artifact retention;
PR artifacts expire after 90 days by default.

> **Note:** the KDE windows-c++ bundle file names change between releases; if
> the download step fails, open the printed index URL and adjust the pattern
> in the "Download KDE binary frameworks" step. The 32-bit (i686) builds are
> **not** automated here because no maintained i686 dependency stack exists on
> GitHub runners — see the next section to build them locally.

### Building for 32-bit x86 (i686), cross-compiling for Windows and Linux

By default Okular is built for the native architecture of your machine (usually
x86-64). It is also possible to build 32-bit (`i686`) binaries, both natively on
Linux and cross-compiled from Linux to Windows. This section describes how to do
it from an Arch-based distribution such as **CachyOS**.

> **Important:** Okular requires Qt 6.8+ and KDE Frameworks 6.13+. These must be
> available *for the target architecture* as well, which is usually the hardest
> part of a 32-bit build: you need a complete 32-bit dependency stack
> (Qt6, KF6, KUserFeedback, Poppler/other generator libraries, libtiff, etc.).

#### 1. Native 32-bit Linux build (i686 chroot / multilib)

On CachyOS/Arch there are no official `multilib` (32-bit) packages like in Debian,
so the recommended approach is a clean 32-bit chroot (or container) where all
dependencies are built/installed for `i686`:

```bash
# Create an Arch32 chroot (the arch32-tools / pacman-ng approach)
sudo pacman -S arch-install-scripts
sudo mkdir -p /srv/arch32
# Install the archlinux32 keyring and a 32-bit pacman into the chroot, then:
sudo pacman -Syyg --arch i686 \
     --dbpath /srv/arch32/var/lib/pacman \
     --cachedir /srv/arch32/var/cache/pacman/pkg \
     -r /srv/arch32 base-devel

# Inside the chroot, install the 32-bit dependency stack, e.g.:
sudo pacman -S qt6-base qt6-svg qt6-declarative extra-cmake-modules \
               kf6-* poppler libtiff djvulibre ...

# Then build Okular forcing the i686 ABI:
cmake -B build -G Ninja \
      -DCMAKE_C_FLAGS="-m32" \
      -DCMAKE_CXX_FLAGS="-m32" \
      -DCMAKE_EXE_LINKER_FLAGS="-m32" \
      -DCMAKE_SHARED_LINKER_FLAGS="-m32" \
      -DCMAKE_INSTALL_PREFIX=/usr ..
ninja -C build
```

Alternatively, pass `-DCMAKE_SYSTEM_NAME=Linux -DCMAKE_SYSTEM_PROCESSOR=i686`
together with the flags above so CMake's try-compile tests run under
`linux32`/`setarch i686` if the host shell is 64-bit.

#### 2. Cross-compiling a 32-bit Windows build (from CachyOS)

Use **Mingw-w64** with the `i686-w64-mingw32` target. You will need a full
32-bit Windows toolchain plus Qt6 and KDE Frameworks built for
`i686-w64-mingw32` (either from the [MSYS2](https://www.msys2.org) `mingw32`
environment, using [KDAB's CMake recipes](https://github.com/KDAB/cmake-recipes),
or by building the dependency stack yourself with a tool such as
[aqt/Qt-installer](https://doc.qt.io/qtinstallerframework/ifw-overview.html) or
KDE's [`kde-cli-tools` / craftmaster](https://community.kde.org/Craft)).

```bash
# Install the cross toolchain on the host
sudo pacman -S mingw-w64-gcc cmake ninja

# Configure Okular for the Windows 32-bit target
cmake -B build-win32 -G Ninja \
      -DCMAKE_TOOLCHAIN_FILE=i686-w64-mingw32.toolchain.cmake \
      -DCMAKE_SYSTEM_NAME=Windows \
      -DCMAKE_SYSTEM_PROCESSOR=i686 \
      -DCMAKE_INSTALL_PREFIX=/opt/okular-win32 \
      -DOKULAR_UI=desktop ..
ninja -C build-win32
```

A minimal `i686-w64-mingw32.toolchain.cmake` looks like this:

```cmake
set(CMAKE_SYSTEM_NAME Windows)
set(CMAKE_SYSTEM_PROCESSOR i686)
set(CMAKE_C_COMPILER   i686-w64-mingw32-gcc)
set(CMAKE_CXX_COMPILER i686-w64-mingw32-g++)
set(CMAKE_RC_COMPILER  i686-w64-mingw32-windres)
set(CMAKE_FIND_ROOT_PATH /opt/qt6-i686-windows /opt/kf6-i686-windows)
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)
```

Make sure `Qt6_DIR`, `KF6_DIR` and the ECM modules point at the *32-bit Windows*
installations (e.g. `-DQt6_DIR=/opt/qt6-i686-windows/lib/cmake/Qt6`).

#### 3. Multiarch builds in one tree

CMake supports multiple independent build directories for different
architectures/targets from the same source checkout — just repeat the configure
step with a different `-B` directory and toolchain/flags, for example:

```bash
cmake -B build-native          ..                                  # x86-64 Linux
cmake -B build-linux-i686 -DCMAKE_C_FLAGS=-m32 -DCMAKE_CXX_FLAGS=-m32 ..   # 32-bit Linux
cmake -B build-win-i686   -DCMAKE_TOOLCHAIN_FILE=i686-w64-mingw32.toolchain.cmake .. # 32-bit Windows
```

Do not mix object files between build trees, and remember that every dependency
(Poppler, TIFF, DjVuLibre, …) must exist in each target architecture.

Note: 32-bit (i686) support is not continuously tested upstream; expect to fix
occasional format/pointer-size warnings (`%lld` vs `%zd`, `quintptr` casts, etc.)
and please contribute those fixes back via merge requests.

### clang-format

The Okular project uses clang-format to enforce source code formatting.
See [README.clang_format](https://invent.kde.org/graphics/okular/-/blob/master/README.clang-format) for more information.
