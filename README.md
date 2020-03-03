# eSpeak NG - CMake script

A [CMake](https://cmake.org) script
to build [eSpeak NG](https://github.com/espeak-ng/espeak-ng)
targetting the [MSYS2](https://www.msys2.org) MinGW64 subsystem.


## Build instructions

Building eSpeak NG using this CMake script
can be done as follows, roughly:

  * Start MSYS2 MinGW64
    with [GCC](https://packages.msys2.org/package/mingw-w64-i686-gcc),
    [SDL2](https://packages.msys2.org/package/mingw-w64-x86_64-SDL2) and
    [CMake](https://packages.msys2.org/package/mingw-w64-x86_64-cmake)
    installed.
  * Compile in debug build from source
    and install (into the same install root)
    each of the following libraries (before compiling the next):
    * [FAudio](https://github.com/FNA-XNA/FAudio)
    * [modified](https://github.com/BoniLindsley/pcaudiolib) source of
      [Portable C Audio Library](https://github.com/espeak-ng/pcaudiolib)
    * and eSpeak NG.

Known issues:

  * Packaging eSpeak NG requires copying `espeak-ng-data` and `SDL2.dll`
    to the same directory as the binary.
  * There are stutters in audio output, probably due to buffering issues.
  * Extra build options enabling Klatt, MBROLA support, SONIC
    and asynchronous commands produce unusable binaries.
    * Disabling `pcaudiolib` seems to be fine.


## Purpose

Windows support in eSpeak NG is given
by providing a Visual Studio solution.
This is not usable for targetting MSYS2 MinGW64.
It might be possible to tweak Autotools to make it work
(but I have little experience in using it).

A bigger issue is that building eSpeak NG with audio output
uses the Portable C Audio Library.
(I was not able to make it behave as expected without audio output.
It compiles, but it gets stuck in an infinite loop.)
The library in turn depends on
[XAudio2](https://docs.microsoft.com/windows/win32/xaudio2/) in Windows.
This cannot be easily made to target MSYS2 MinGW64.
Alternative dependencies include ALSA, OSS and PulseAudio,
but none of them are available in MSYS2 MinGW64 at the time of writing.

The solution chosen is to replace XAudio2 by FAudio
-- an open-source reimplmentation of XAudio2
that depends only on [SDL2](https://www.libsdl.org/download-2.0.php) --
that uses CMake for its build by default and has a binary
[package](https://packages.msys2.org/package/mingw-w64-x86_64-FAudio)
provided in the default MSYS2 package repository.
The Portable C Audio Library is made to link to FAudio
by slightly its source and adding CMake support to it.
The CMake build script in this repository
adds CMake support for eSpeak NG as well,
and to use the modified `pcaudiolib` library.
