# AFNI for Apple Silicon macOS

Unofficial AFNI binaries for Apple Silicon Macs. The package is built with
Clang and Homebrew libraries and is intended for `arm64` macOS systems using
the `/opt/homebrew` prefix.

> [!IMPORTANT]
> This is a community build, not an official AFNI release. For supported
> installation methods, see the [AFNI macOS Apple Silicon guide][afni-guide].

## TL;DR: Install AFNI

These steps install the current published package, **AFNI 26.2.01**. They also
preserve an existing `~/abin` directory as a timestamped backup.

### 1. Install the runtime requirements

Install [Homebrew][homebrew] first, then run:

```zsh
brew install --cask xquartz
brew install openmotif mesa-glu gsl glib libomp libxpm zlib-ng-compat
```

If your organization manages software installations, ask IT to install
[XQuartz][xquartz] instead of running the first command yourself.

### 2. Download and verify AFNI

```zsh
cd "$HOME/Downloads"
curl -LO https://github.com/cecilyen/MacM2_AFNI/releases/download/v26.2.01-arm64-clang/macos_13_ARM.AFNI_26.2.01.arm64-clang.tgz
echo "aa9c7392198c6bca70922a769c782f7ec6d4f6861f43bbdd6493e5facf633513  macos_13_ARM.AFNI_26.2.01.arm64-clang.tgz" | shasum -a 256 -c -
```

The checksum command must print `OK`. Stop if it does not.

### 3. Install into `~/abin`

```zsh
if [[ -d "$HOME/abin" ]]; then
  mv "$HOME/abin" "$HOME/abin.backup.$(date +%Y%m%d-%H%M%S)"
fi
mkdir -p "$HOME/abin"
tar -xzf "$HOME/Downloads/macos_13_ARM.AFNI_26.2.01.arm64-clang.tgz" \
  -C "$HOME/abin" --strip-components=1

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

The published `v26.2.01-arm64-clang` archive contains:

- AFNI 26.2.01 arm64 programs and GUI plugins
- SUMA programs
- AFNI atlases, templates, refacer data, and support files
- no Python cache files

Download it from the [GitHub release page][release-26]. The previous
[AFNI 25.2.00 package][release-25] remains available.

## Runtime Requirements

The package links to Homebrew libraries supplied by the formulae in the TL;DR,
including OpenMotif, Mesa/GLU, GSL, GLib, OpenMP, X11 libraries, libpng,
`jpeg-turbo`, and `zlib-ng-compat`. It uses the macOS system copies of Expat
and `libiconv`.

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
AFNI/R library set. Core AFNI and SUMA programs do not require it.

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
[release-26]: https://github.com/cecilyen/MacM2_AFNI/releases/tag/v26.2.01-arm64-clang
[release-25]: https://github.com/cecilyen/MacM2_AFNI/releases/tag/v25.2.00-m2clang.2
[zlib-ng]: https://github.com/zlib-ng/zlib-ng
[jpeg-turbo]: https://github.com/libjpeg-turbo/libjpeg-turbo
