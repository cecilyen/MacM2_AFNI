# AFNI macOS ARM64 Clang Build

This is an unofficial AFNI macOS ARM64 build made for the Apple Silicon Mac where the standard binary/install path was not ideal. It uses Apple clang, M2-targeted optimization, Homebrew `jpeg-turbo`, Homebrew `zlib-ng-compat`, and Apple system `expat`/`libiconv`.

## Why

This build was created because:

- The available AFNI macOS binary was not the right fit for my Apple Silicon Macbook.
- AFNI's Apple Silicon installation path depends on Homebrew/XQuartz/Mesa behavior. AFNI's own macOS ARM setup script currently pins Mesa through an AFNI tap for SUMA support.
- The goal was a reproducible arm64 build using Apple clang and M2-aware flags, without Homebrew GCC.

The result keeps practical optimization while preserving AFNI's macOS plugin ABI. ThinLTO and `dead_strip` were not included because they broke runtime plugin loading.

## Tested System

- Hardware: 2023 MacBook Pro, Apple M2 Max
- OS: macOS 26.5.1, arm64
- Compiler: `/usr/bin/clang`
- AFNI source: `afni_src`
- Output tree: `afni_src/macos_13_ARM`
- X11: XQuartz with `/opt/X11/lib/flat_namespace`
- Homebrew prefix: `/opt/homebrew`

## Build Choices

Dependency choices:

- `zlib-ng-compat` replaces zlib through the zlib-compatible `libz` ABI. zlib-ng documents a zlib-compatible API and CPU-intrinsic acceleration paths.
- `jpeg-turbo` replaces libjpeg for JPEG support. libjpeg-turbo documents SIMD acceleration and compatibility with the traditional libjpeg API.
- Apple system `expat` and `libiconv` reduce Homebrew runtime coupling.

Key Makefile settings:

```make
LOCAL_CC_PATH ?= /usr/bin/clang
CCMIN  = $(LOCAL_CC_PATH) -arch arm64 -DDARWIN -DARM_M1
OPTFLAGS = -O3 -mcpu=apple-m2 -pipe
LDFLAGS_OPT = -arch arm64

ZLIB_NG_ROOT = /opt/homebrew/opt/zlib-ng-compat
LZLIB = $(ZLIB_NG_ROOT)/lib/libz.dylib

JPEG_TURBO_ROOT = /opt/homebrew
XLIBS = -lXm $(JPEG_TURBO_ROOT)/lib/libjpeg.dylib -lXt

MACOS_SDKROOT ?= $(shell xcrun --show-sdk-path)
LGIFTI = $(MACOS_SDKROOT)/usr/lib/libexpat.tbd

AFNI_LDFLAGS = -Wl,-export_dynamic
PLUGIN_LFLAGS = -m64 -bundle -flat_namespace -undefined suppress -Wl,-x $(LDFLAGS_OPT)
```

Avoid these flags for this AFNI plugin build:

```text
-flto=thin
-Wl,-dead_strip
```

They can be valid for other projects, but here they hid or stripped symbols needed by AFNI's flat-namespace plugin bundles.

## Requirements

Homebrew packages used by this build:

```zsh
brew install libpng jpeg-turbo zlib-ng-compat freetype fontconfig openmotif libomp libxt gsl glib pkgconf autoconf mesa mesa-glu libxpm netpbm cmake
```

R is needed for AFNI's R-based programs, but not for the core C/C++ tools to build and run.

## Build

From the workspace root:

```zsh
/afni_src
export PATH="/opt/homebrew/bin:/opt/homebrew/opt/pkgconf/bin:$PATH"
export SDKROOT="$(xcrun --show-sdk-path)"
make -f Makefile.macos_13_ARM clean
make -f Makefile.macos_13_ARM vastness
```

Output:

```text
/afni_src/macos_13_ARM
```

## Validation

Local version:

```text
Precompiled binary macos_13_ARM: Jun 17 2026 (Version AFNI_25.2.00 'Gordian I')
```

Architecture checks:

```text
afni:   Mach-O 64-bit executable arm64
3dinfo: Mach-O 64-bit executable arm64
suma:   Mach-O 64-bit executable arm64
```

Dependency checks confirmed:

- `libz.1.dylib` from `/opt/homebrew/opt/zlib-ng-compat`
- `libjpeg.8.dylib` from `/opt/homebrew/opt/jpeg-turbo`
- `libiconv.2.dylib` from `/usr/lib`
- `libexpat.1.dylib` from `/usr/lib`
- no GCC-14 runtime, `libgomp`, or `gfortran` runtime in checked binaries

Plugin checks confirmed that `afni` exports:

```text
_first_plugin_check
_AFNI_driver_register
_PLUTO_set_sequence
_PLUTO_prefix_ok
_PLUTO_popup_worker
```

GUI/plugin smoke test:

```text
plug_3dsvm.so loaded successfully
PLUG_get_many_plugins -- found 49 plugins
```

Program-help validation:

```text
found 623 good progs and 20 issues out of 643
```

The remaining 20 issues were R-dependent AFNI programs or wrappers.

Functional smoke test:

```zsh
printf '0 0 0 7\n1 1 1 3\n' > vox.1D
3dUndump -overwrite -dimen 2 2 2 -ijk -datum short -prefix vox vox.1D
3dcalc -overwrite -a vox+orig -expr 'a*2' -prefix vox2
3dBrickStat -sum vox2+orig
```

Expected result:

```text
20
```

## Package

Packaged artifact:

```text
dist/macos_13_ARM.AFNI_25.2.00.m2clang.tgz
```

Size:

```text
359 MB
```

SHA-256:

```text
3a96ef3ecea2c600487cf833308473fdafd958d695de32ea139d730c4dbc660e
```

Checksum file:

```text
dist/macos_13_ARM.AFNI_25.2.00.m2clang.tgz.sha256
```

The archive contains a top-level `macos_13_ARM/` directory. It was extracted into `/private/tmp` and re-tested successfully.

## Install From The Package

Example user install:

```zsh
mkdir -p "$HOME/abin"
tar -xzf macos_13_ARM.AFNI_25.2.00.m2clang.tgz -C "$HOME/abin" --strip-components=1
echo 'export PATH="$HOME/abin:$PATH"' >> "$HOME/.zshrc"
echo 'export DYLD_LIBRARY_PATH="/opt/X11/lib/flat_namespace:${DYLD_LIBRARY_PATH}"' >> "$HOME/.zshrc"
```

Open a new terminal and check:

```zsh
afni -ver
afni_system_check.py -check_all
```

## References

[^afni-arm-guide]: AFNI documentation, "macOS 12+ (Apple Silicon/ARM processor/chip: M1, M2, ...)." https://afni.nimh.nih.gov/pub/dist/doc/htmldoc/background_install/install_instructs/steps_macOS_12_Silicon.html
[^afni-arm-script]: AFNI `OS_notes.macos_12_ARM_a_admin_pt1.zsh`, including Homebrew dependencies and `mesa@25.3.4` tap. https://raw.githubusercontent.com/afni/afni/master/src/other_builds/OS_notes.macos_12_ARM_a_admin_pt1.zsh
[^zlib-ng]: zlib-ng project README, zlib-compatible API and CPU-intrinsic optimization notes. https://github.com/zlib-ng/zlib-ng
[^jpeg-turbo]: libjpeg-turbo project README, SIMD acceleration and traditional libjpeg API support. https://github.com/libjpeg-turbo/libjpeg-turbo
