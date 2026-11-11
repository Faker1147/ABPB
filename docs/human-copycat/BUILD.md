# Building ABPB 4.3.3

The source snapshot vendors its dependencies. Requirements: **CMake 3.22+**, a **C++20 compiler**, **Python 3**, and **Ninja** or Make. Building the DLL does not need a running Metamod or AMXX installation.

All build options use the `ABPB_` prefix.

## Native fake-engine tests

```sh
cmake -S . -B build/test -G Ninja \
  -DCMAKE_BUILD_TYPE=Debug -DABPB_NATIVE=ON -DABPB_64BIT=ON \
  -DABPB_BUILD_TESTS=ON -DABPB_WITH_TLS=OFF \
  -DABPB_WITH_AMXX=OFF -DABPB_LTO=OFF
cmake --build build/test --parallel 4
ctest --test-dir build/test --output-on-failure --timeout 90 --parallel 2
```

The synthetic fixture feeds 1200 records through the aim pipeline and checks pitch, yaw and fire decisions. Angles allow a 0.001-degree floating-point tolerance; fire decisions must match exactly. Host tests are not a real game match.

## Windows x86 DLL with MinGW

```sh
cmake -S . -B build/windows -G Ninja \
  -DCMAKE_TOOLCHAIN_FILE=cmake/toolchains/windows-x86-mingw.cmake \
  -DCMAKE_BUILD_TYPE=Release \
  -DABPB_WITH_TLS=ON -DABPB_WITH_AMXX=OFF -DABPB_LTO=OFF
cmake --build build/windows --parallel 4
python tools/check_dll.py --arch i386 build/windows/ABPB.dll
```

This needs `i686-w64-mingw32-g++-posix`, binutils and windres on `PATH`. A manually extracted toolchain may need `CMAKE_RC_FLAGS` with:

```text
--preprocessor=/path/i686-w64-mingw32-gcc-posix
--preprocessor-arg=-E --preprocessor-arg=-xc --preprocessor-arg=-DRC_INVOKED
```

## Windows x86 DLL with Zig

The release package was built with Zig 0.17.1 (`x86-windows-gnu`, Clang) plus CMake and Ninja, using `cmake/toolchains/windows-x86-clang.cmake` with `-DABPB_64BIT=OFF`. Zig builds link against libc++ and libc++abi so the Human Copycat `std::__1::` symbols resolve; see `cmake/ABPBFlags.cmake`.

When configuring with that toolchain on Windows, point `CMAKE_AR` and `CMAKE_RANLIB` at small wrappers that call `zig ar` and `zig ranlib`, and make sure `windres` is on `PATH`. The MSVC and MinGW recipes above are the simplest supported build paths.

## Windows x86 DLL with MSVC

```powershell
cmake -S . -B build/windows -A Win32 `
  -DABPB_WITH_TLS=ON -DABPB_WITH_AMXX=OFF -DABPB_LTO=OFF
cmake --build build/windows --config Release --parallel 4
python tools/check_dll.py --arch i386 build/windows/Release/ABPB.dll
```

The GitHub Actions workflow runs the Linux regression tests and a Windows DLL build/ABI check. See the Actions tab for the current status.

## AMXX module

```sh
cmake -S . -B build/windows-x86 -G Ninja \
  -DCMAKE_TOOLCHAIN_FILE=toolchains/zig-x86-windows-gnu.cmake \
  -DCMAKE_BUILD_TYPE=Release -DABPB_WITH_AMXX=ON
cmake --build build/windows-x86 --target abpb_amxx --parallel 4
```

The module is 32-bit only and is not needed unless an AMXX plugin calls the `abpb_*` natives.

## Portable package

```sh
python tools/package_abpb.py \
  --binary build/windows-x86/ABPB.dll \
  --amxx build/windows-x86/abpb_amxx.dll \
  --output dist/ABPB-4.3.3-Windows-x86.zip
```

The packaging script copies the DLL, `conf/`, the Dust2 graph, licenses and the optional AMXX module/include/sample, then writes `SHA256SUMS.txt`.

## License and attribution

The combined fork is GPL-2.0-or-later. Component attribution and graph notices are in `NOTICE-ABPB.md` and `cfg/addons/ABPB/licenses/`. Build output, logs and ZIP files are ignored by `.gitignore`.
