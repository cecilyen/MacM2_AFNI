# AFNI for Apple Silicon macOS

Unofficial AFNI binaries for Apple Silicon Macs. The package is built with
Clang and Homebrew libraries and is intended for `arm64` macOS systems using
the `/opt/homebrew` prefix.

> [!IMPORTANT]
> This is a community build, not an official AFNI release. For supported
> installation methods, see the [AFNI macOS Apple Silicon guide][afni-guide].

## TL;DR: Install AFNI

These steps install the updater-compatible **AFNI 26.2.01 revision 2** package.

### 1. Install the runtime requirements

Install [Homebrew][homebrew] first, then run:

```zsh
brew install --cask xquartz
brew install fontconfig glib gsl jpeg-turbo libomp libpng \
  libx11 libxext libxft libxmu libxp libxpm libxt \
  mesa mesa-glu openmotif zlib-ng-compat
```

If your organization manages software installations, ask IT to install
[XQuartz][xquartz] instead of running the first command yourself.

### 2. Download and verify AFNI

```zsh
cd "$HOME/Downloads"
curl -LO https://github.com/cecilyen/MacM2_AFNI/releases/download/v26.2.01-arm64-clang.2/macos_13_ARM.AFNI_26.2.01.arm64-clang.2.tgz
echo "65c5903b0a418ff76b396f782c50672944023ae23e80f19fd451478e6c6f94a9  macos_13_ARM.AFNI_26.2.01.arm64-clang.2.tgz" | shasum -a 256 -c -
```

The checksum command must print `OK`. Stop if it does not.

### 3. Install or update AFNI

If AFNI is already installed in the dedicated `~/abin` directory, use its
updater:

```zsh
"$HOME/abin/@update.afni.binaries" \
  -local_package "$HOME/Downloads/macos_13_ARM.AFNI_26.2.01.arm64-clang.2.tgz" \
  -bindir "$HOME/abin" \
  -make_backup yes \
  -sys_ok \
  -no_recur
```

`-sys_ok` is needed when upgrading the older package because it included a
`python` symlink. Use this command only when `~/abin` is dedicated to AFNI.
The updater preserves the previous files in an `auto_backup.*` directory.

For a first installation, extract the archive directly:

```zsh
mkdir -p "$HOME/abin"
tar -xzf "$HOME/Downloads/macos_13_ARM.AFNI_26.2.01.arm64-clang.2.tgz" \
  -C "$HOME/abin" --strip-components=1
```

The AFNI updater uses `-package` for package names hosted on the AFNI server.
Use `-local_package` for this downloaded GitHub archive.

Add AFNI to the shell path after either method:

```zsh
touch "$HOME/.zshrc"
grep -qxF 'export PATH="$HOME/abin:$PATH"' "$HOME/.zshrc" || \
  echo 'export PATH="$HOME/abin:$PATH"' >> "$HOME/.zshrc"
source "$HOME/.zshrc"
```

### 4. Check and launch AFNI

```zsh
afni -ver
afni_system_check.py -check_all
afni
```

If the AFNI window or plugins fail to open, launch it with the XQuartz
flat-namespace libraries:

```zsh
DYLD_LIBRARY_PATH=/opt/X11/lib/flat_namespace afni
```

## Package Contents

The published `v26.2.01-arm64-clang.2` archive contains:

- AFNI 26.2.01 arm64 programs and GUI plugins
- SUMA programs
- AFNI atlases, templates, refacer data, and support files
- 2,126 regular files under a single updater-compatible top-level directory
- no Python cache files

Download it from the [revision 2 release page][release-26-2]. Earlier packages
remain available from the [releases page][releases].

## Runtime Requirements

The binaries directly link to these Homebrew formulae, all listed explicitly
in the TL;DR installation command:

```text
fontconfig  glib       gsl       jpeg-turbo  libomp   libpng
libx11      libxext    libxft    libxmu      libxp    libxpm
libxt       mesa       mesa-glu  openmotif   zlib-ng-compat
```

Homebrew may install additional transitive dependencies. The package uses the
macOS system copies of Expat and `libiconv`. Build-only tools such as
`autoconf`, `netpbm`, and `pkgconf` are not required at runtime.

XQuartz is required for the AFNI and SUMA graphical interfaces. R and optional
Python modules are needed only by AFNI programs that specifically use them.

This package assumes Apple Silicon Homebrew at `/opt/homebrew`. It is not an
Intel (`x86_64`) build.

## Build Details

AFNI 26.2.01 is compiled as a generic Apple `arm64` build rather than being
tuned for one M-series processor.

Build profile:

```text
Compiler: Apple clang 21.0.0
CFLAGS/CXXFLAGS: -O3 -arch arm64 -flto=thin -pipe
LDFLAGS: -arch arm64 -flto=thin -Wl,-dead_strip
zlib: Homebrew zlib-ng-compat
JPEG: Homebrew jpeg-turbo
Expat/iconv: macOS system libraries
```

The package was stripped with `strip -x`, which retains global plugin symbols.
Validation found:

```text
445/445 Mach-O files: arm64
Missing non-system libraries: 0
AFNI GUI plugins loaded: 49
Plugin load errors: 0
Dataset and JPEG smoke tests: passed
Python cache files: 0
```

The older `R_io.so` is not included because it was built against a different
AFNI/R library set and requires an R 4.3 framework not present on the test
system. Core AFNI and SUMA programs do not require it.

## Official Package Comparison

The archive layout and payload were compared with AFNI's official
[`macos_13_ARM.AFNI_25.3.03.tgz`][official-25]. Common paths had zero file-type
or permission-mode mismatches. The missing portable support file
`funstuff/face_lincoln1.jpg` was added. The only remaining official-only path
is the incompatible `R_io.so` described above.

The revision-2 archive was installed successfully with
`@update.afni.binaries -local_package`, including backup mode and an upgrade
from the older package's `python` symlink. All 2,126 installed files matched
the package manifest.

## Build Choices

- [`zlib-ng-compat`][zlib-ng] provides the zlib-compatible API used by AFNI.
- [`jpeg-turbo`][jpeg-turbo] provides the libjpeg-compatible API used by AFNI.
- Apple Clang avoids a GCC runtime dependency.
- The build targets generic Apple `arm64`; it does not use M2-only CPU flags.

## License and Support

AFNI is developed and distributed by the [AFNI project][afni]. Report problems
specific to these packaged binaries in this repository. Report general AFNI
questions through the official [AFNI support channels][afni-support].

[afni]: https://afni.nimh.nih.gov/
[afni-guide]: https://afni.nimh.nih.gov/pub/dist/doc/htmldoc/background_install/install_instructs/steps_macOS_12_Silicon.html
[afni-support]: https://afni.nimh.nih.gov/pub/dist/doc/htmldoc/discussion/user_forums.html
[homebrew]: https://brew.sh/
[xquartz]: https://www.xquartz.org/
[release-26-2]: https://github.com/cecilyen/MacM2_AFNI/releases/tag/v26.2.01-arm64-clang.2
[releases]: https://github.com/cecilyen/MacM2_AFNI/releases
[official-25]: https://afni.nimh.nih.gov/pub/dist/tgz/macos_13_ARM.AFNI_25.3.03.tgz
[zlib-ng]: https://github.com/zlib-ng/zlib-ng
[jpeg-turbo]: https://github.com/libjpeg-turbo/libjpeg-turbo
