# 在windows下编译

1. 下载`https://github.com/KhronosGroup/OpenCL-SDK`中最新的Release版本，比如说：[OpenCL-SDK-v2022.05.18-Win-x64.zip](https://github.com/KhronosGroup/OpenCL-SDK/releases/download/v2022.05.18/OpenCL-SDK-v2022.05.18-Win-x64.zip)，解压缩后放置到某个文件夹，比如说`G:\OpenCL-SDK-2022.5.18-win32`。最新的OpenCL-SDK的Release，请查看：[https://github.com/KhronosGroup/OpenCL-SDK/releases](https://github.com/KhronosGroup/OpenCL-SDK/releases)
2. 编译clinfo
```bash
$ %comspec% /k "C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Auxiliary\Build\vcvars64.bat"
$ make.cmd /D OPENCLDIR=G:\OpenCL-SDK-2022.5.18-win32
```

**使用CMake编译**

可以解决不同OpenCL SDK发布版本中的差异，比如说是`libOpenCL.lib`还是`OpenCL.lib`等问题。

```bash
$ cmake .. -G "Visual Studio 15 2017 Win64" -DCMAKE_INSTALL_PREFIX=../dist/clinfo -DOPENCL_SDK_ROOT=G:\OpenCL-SDK-2022.5.18-win32
$ %comspec% /k "C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Auxiliary\Build\vcvars64.bat"
$ msbuild /maxcpucount:4 /p:Configuration=Release /p:PreferredToolArchitecture=x64 ALL_BUILD.vcxproj -t:rebuild
$ .\Release\clinfo.exe -l
```

##  参考

- [Compiler Warning (level 1) C4819](https://docs.microsoft.com/en-us/cpp/error-messages/compiler-warnings/compiler-warning-level-1-c4819?view=msvc-170)



# What is this?

clinfo is a simple command-line application that enumerates all possible
(known) properties of the OpenCL platform and devices available on the
system.

Inspired by AMD's program of the same name, it is coded in pure C and it
tries to output all possible information, including those provided by
platform-specific extensions, trying not to crash on unsupported
properties (e.g. 1.2 properties on 1.1 platforms).

# Usage

    clinfo [options...]

Common used options are `-l` to show a synthetic summary of the
available devices (without properties), and `-a`, to try and show
properties even if `clinfo` would otherwise think they aren't supported
by the platform or device.

Refer to the man page for further information.

## Use cases

* verify that your OpenCL environment is set up correctly;
  if `clinfo` cannot find any platform or devices (or fails to load
  the OpenCL dispatcher library), chances are high no other OpenCL
  application will run;
* verify that your OpenCL _development_ environment is set up
  correctly: if `clinfo` fails to build, chances are high no
  other OpenCL application will build;
* explore/report the actual properties of the available device(s).

## Segmentation faults

Some faulty OpenCL platforms may cause `clinfo` to crash. There isn't
much `clinfo` itself can do about it, but you can try and isolate the
platform responsible for this. On POSIX systems, you can generally find
the platform responsible for the fault with the following one-liner:

    find /etc/OpenCL/vendors/ -name '*.icd' | while read OPENCL_VENDOR_PATH ; do clinfo -l > /dev/null ; echo "$? ${OPENCL_VENDOR_PATH}" ; done

## Missing information

If you know of device properties that are exposed in OpenCL (either as core
properties or as extensions), but are not shown by `clinfo`, please [open
an issue](https://github.com/Oblomov/clinfo/issues) providing as much
information as you can. Patches and pull requests accepted too.


# Building

<img
src='https://api.travis-ci.org/Oblomov/clinfo.svg?branch=master'
alt='Build status on Travis'
style='float: right'>

Building requires an OpenCL SDK (or at least OpenCL headers and
development files), and the standard build environment for the platform.
No special build system is used (autotools, CMake, meson, ninja, etc),
as I feel adding more dependencies for such a simple program would be
excessive. Simply running `make` at the project root should work.

## Android support

### Local build via Termux

One way to build the application on Android, pioneered by
[truboxl][truboxl] and described [here][issue46], requires the
installation of [Termux][termux], that can be installed via Google Play
as well as via F-Droid.

[truboxl]: https://github.com/truboxl
[issue46]: https://github.com/Oblomov/clinfo/issues/46
[termux]: https://termux.com/

Inside Termux, you will first need to install some common tools:

	pkg install git make clang -y


You will also need to clone the `clinfo` repository, and fetch the
OpenCL headers (we'll use the official `KhronosGroup/OpenCL-Headers`
repository for that):

	git clone https://github.com/Oblomov/clinfo
	git clone https://github.com/KhronosGroup/OpenCL-Headers

(I prefer doing this from a `src` directory I have created for
development, but as long as `clinfo` and `OpenCL-Headers` are sibling
directories, the headers will be found. If not, you will have to
override `CPPFLAGS` with e.g. `export CPPFLAGS=-I/path/to/where/headers/are`
before running `make`.
Of course `/path/to/where/headers/are` should be replaced with the actual
path to which the `OpenCL-Headers` repository was cloned.)

You can then `cd clinfo` and build the application. You can try simply
running `make` since Android should be autodetected now, buf it
this fails you can also force the detectio with

	make OS=Android

If linking fails due to a missing `libOpenCL.so`, then your Android
machine probably doesn't support OpenCL. Otherwise, you should have a
working `clinfo` you can run. You will most probably need to set
`LD_LIBRARY_PATH` to let the program know where the OpenCL library is at
runtime: you will need at least `${ANDROID_ROOT}/vendor/lib64`, but on
some machine the OpenCL library actually maps to a different library
(e.g., on one of my systems, it maps to the GLES library, which is in a
different subdirectory).

Due to this requirement, on Android the actual binary is now called
`clinfo.real`, and the produced `clinfo` is just a shell script that
will run the actual binary after setting `LD_LIBRARY_PATH`. If this
is not sufficient on your installation, please open an issue and we'll
try to improve the shell script to cover your use case as well.

## MacOS support

clinfo should build without issues out of the box on most macOS installations
(starting from OS X v10.6).
In contrast to most other operating systems,
the macOS system OpenCL library only supports Apple's own OpenCL platform.

To use other platforms such as [PoCL](https://portablecl.org),
it is necessary to install an alternative OpenCL library that works as an ICD loader,
such as [Homebrew](https://brew.sh)'s [ocl-icd](https://formulae.brew.sh/formula/ocl-icd).

To build `clinfo` using the Homebrew OpenCL library instead of the macOS system library,
you can use

    make OS=Homebrew


## Windows support

The application can usually be built in Windows too (support for which
required way more time than I should have spent, really, but I digress),
by running `make` in a Developer Command Prompt for Visual Studio,
provided an OpenCL SDK (such as the Intel or AMD one) is installed.

Precompiled Windows executable are available as artefacts of the
AppVeyor CI.

<table style='margin: 1em auto; width: 100%; max-width: 33em'>
<tr><th>Build status</th><th colspan=2>Windows binaries</th></tr>
<tr>
<td><a href='https://ci.appveyor.com/project/Oblomov/clinfo/'><img
src='https://ci.appveyor.com/api/projects/status/github/Oblomov/clinfo?svg=true'
alt='Build status on AppVeyor'></a></td>
<td><a href='https://ci.appveyor.com/api/projects/oblomov/clinfo/artifacts/clinfo.exe?job=platform%3a+x86'>32-bit</a></td>
<td><a href='https://ci.appveyor.com/api/projects/oblomov/clinfo/artifacts/clinfo.exe?job=platform%3a+x64'>64-bit</a></td>
</tr>
</table>
