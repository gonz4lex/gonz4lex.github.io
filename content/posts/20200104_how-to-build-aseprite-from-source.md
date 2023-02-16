title: How to build Aseprite from source in Ubuntu
date: 2020-1-4 00:33
category: game dev, pixel art
tags: aseprite
slug: how-to-build-aseprite-from-source
author: Alex Gonzalez
summary: Aseprite is an amazing pixel art software and its source code is open. This is how you can build and test it, or help with its development.
status: published

> __Disclaimer:__
> Aseprite is a paid software and you should buy it to support its developers. It's 15$ and can be purchased for Windows, macOS and Linux, as well as Steam. You can use this method to inspect the source code and try to help with development, or to try the software. There is also a free trial in their website.

## Building Aseprite from Github

[Aseprite](https://www.aseprite.org) is the best pixel art tool I have come across, and has a very affordable price if you just want to test the waters of pixel art. However, should you want to help tackle some of the [issues](https://github.com/aseprite/aseprite/issues) on the project's repository, you need to compile the program to inspect your changes.

I went through this process on my Windows and Linux machines, and I must say that the experience was _wildly_ different, even if the process is very similar. I am currently using Ubuntu 18.04 LTS.

## Get started

Before beginning, please note that the full [install instructions](https://github.com/aseprite/aseprite/blob/master/INSTALL.md) can be found in the project's repo.

First, you should clone the repository in a directory of your choice, such as `~/`:

```bash
git clone --recursive https://github.com/aseprite/aseprite.git
```

This will create an `aseprite` directory. You can later update this clone with new releases by running:

```bash
cd aseprite
git pull
git submodule update --init --recursive
```

## Aseprite dependencies

There are three main dependencies for Aseprite:

- latest [CMake](cmake.org)
- the [Ninja](ninja-build.org) build system
- a compiled version of [Skia](https://skia.org/)

Note that for Windows, you need Windows 10 and [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/), as well as a [C++ SDK] for desktop development. The latter is an item to check in your [Visual Studio installer](https://imgur.com/4Pq2Cbv), so make sure you have selected it.

Also, installing CMake on Windows was HORRIBLE and took quite a long time. In Ubuntu, however, I was able to install all dependencies with a single command:

```bash
sudo apt-get install -y g++ cmake ninja-build libx11-dev libxcursor-dev libgl1-mesa-dev libfontconfig1-dev
```

Building Skia is also quite trivial on Linux. I was able to run all of the following commands without any errors or missing dependencies, unlike in Windows:

```bash
mkdir $HOME/deps
cd $HOME/deps
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
git clone -b aseprite-m71 https://github.com/aseprite/skia.git
export PATH="${PWD}/depot_tools:${PATH}"
cd skia
python tools/git-sync-deps
gn gen out/Release --args="is_debug=false is_official_build=true skia_use_system_expat=false skia_use_system_icu=false skia_use_system_libjpeg_turbo=false skia_use_system_libpng=false skia_use_system_libwebp=false skia_use_system_zlib=false"
ninja -C out/Release skia
```

These commands can also be found in the Aseprite documentation, so head there if you get stuck.

## Compiling the code

After getting all the dependecies, create a new folder `build` inside your `aseprite` directory, and `cd` into it. Afterwards, run `cmake`:

```bash
cd ~/aseprite
mkdir build
cd build
```
> In this last command, `cmake` needs a different setup based on your OS. I will just outline the Linux process since I still get nightmares trying to install Clang, Google depot tools, CMake and Visual Studio in Windows. Feel free to look up the other ones in the [installation guide](https://github.com/aseprite/aseprite/blob/master/INSTALL.md#compiling).

Run this command on your Linux machine to build the source:

```bash
cmake \
  -DCMAKE_BUILD_TYPE=RelWithDebInfo \
  -DLAF_OS_BACKEND=skia \
  -DSKIA_DIR=$HOME/deps/skia \
  -DSKIA_OUT_DIR=$HOME/deps/skia/out/Release \
  -G Ninja \
  ..
```

When your terminal finishes the previous task, just run Ninja inside the `build` directory and you will have your executable in `aseprite/build/bin/aseprite`:

```bash
ninja aseprite
```

## Start pixeling!

That should be it! Run your Aseprite executable and you should be greeted by the clean interface of the software.
Remember to actually purchase Aseprite if you are going to use it commercially or professionally.





