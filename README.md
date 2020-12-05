# micronucleus-commandline-builds

Build of the Micronucleus command line tool for various platforms.

## Why this exists

I was trying to install the latest version of the excellent [ATTinyCore](https://github.com/SpenceKonde/ATTinyCore) on my Raspberry Pi, and it failed due to the fact that as of version 1.4.0, ATTinycore depends on the [`micronucleus` command line](https://github.com/micronucleus/micronucleus/tree/master/commandline). Unfortunately, the build of the tool referenced is not only an old alpha build (version 2.0a4), it only had been built for Windows, MacOS, and i686/x86_64 Linux builds.

After looking around, I came across this [issue](https://github.com/SpenceKonde/ATTinyCore/issues/465). There was a lot of questions about the current version of the `micronucleus` CLI tool, and best to build it. Long story short, that's what I'm doing here.

### Updated Makefile for static binaries

To build `micronucleus` for better portability when using it as tooling with ATTinyCore, I've updated the Makefile to produce static binaries on all platforms. I've got these updates in [my mirconucleus fork](https://github.com/stonehippo/micronucleus). The only diff is in the Makefile, which I may PR upstream at some point. I'll try to keep my repo largely in line with the upstream.

If you want to build for any platform below and not make the Makefile edits yourself, I recommend using my fork for now.

## What's here

I've managed to build or borrow the latest version of the `micronucleus` CLI tool (version 2.04) for multiple platforms:

- MacOS (64 bit)
- ~Windows (32 bit + 64 bit)~
- ~Windows (64 bit)~
- Linux x32 (32 bit)
- Linux x64 (64 bit)
- Linux ARM v6/v7 (32 bit)
- Linux ARM aarch (64 bit)

## MacOS

I built the MacOS binary on MacOS Big Sur. I didn't specify a minimum OS version number, but it does require a 64 bit version of MacOS at the moment.

### MacOS Dependencies

You'll need to make sure you've got the `libusb` and `libusb-compat` libraries installed. The easist way to do this is Homebrew:

```
$ brew install libusb libusb-compat
```

### Building the commandline tool for MacOS

First:

```
$ git clone https://github.com/micronucleus/micronucleus.git
$ cd micronucleus/commandline
$ git checkout tags/2.04 -b 2.04-release
```

The build will work as-is, but it links `libusb` dynamically by default. Since we want this to be portable, we need to modify the default Makefile. The following changes do the trick:

```
diff --git a/commandline/Makefile b/commandline/Makefile
index 050f205..77da05e 100644
--- a/commandline/Makefile
+++ b/commandline/Makefile
@@ -14,10 +14,11 @@ else ifeq ($(shell uname), Darwin)
        EXE_SUFFIX =
        OSFLAG = -D MAC_OS
        # Uncomment these to create a static binary:
-       # USBLIBS = /opt/local/lib/libusb-legacy/libusb-legacy.a
+       USBLIBS = /usr/local/opt/libusb-compat/lib/libusb.a
+       USBLIBS += /usr/local/opt/libusb/lib/libusb-1.0.a
        # USBLIBS += -mmacosx-version-min=10.5
-       # USBLIBS += -framework CoreFoundation
-       # USBLIBS += -framework IOKit
+       USBLIBS += -framework CoreFoundation
+       USBLIBS += -framework IOKit
        # Uncomment these to create a dual architecture binary:
        # OSFLAG += -arch x86_64 -arch i386
 else ifeq ($(shell uname), OpenBSD)
```

Once you've got those changes made, it's pretty simple:

```
$ make
```

And that's it, you've now got a working, more-or-less portable `micronucleus` CLI tool for MacOS.

## Windows

TK

## Linux 32 bit/64 bit

I built the Linux x32 and x64 binareis on DigitalOcean Droplets running Ubuntu 16.04 and 20.04, respectively.

### Linux Dependencies

At least on Ubuntu, I needed to install, the following to get things to compile:

```
$ sudo apt update
$ sudo apt install -y build-essentials binutils libusb-dev
```

### Building the commandline tool for Linux

This works fine:

```
$ git clone https://github.com/micronucleus/micronucleus.git
$ cd micronucleus/commandline
$ git checkout tags/2.04 -b 2.04-release
$ make
``

For the most part, the standard build file is fine. The main problem is that it doesn't build a static binary with the current config. This might not normally be an issue, but since I'm building the binary to support tooling in ATTinyCore, avoiding having to install `libusb-dev` is a good thing. So, I updated the micronucleus build file like this:

```diff
index 050f205..cf4a607 100644                                                                                           --- a/commandline/Makefile                                                                                              +++ b/commandline/Makefile                                                                                              @@ -5,7 +5,7 @@ CC=gcc                                                                                                                                                                                                                           ifeq ($(shell uname), Linux)                                                                                                   USBFLAGS=$(shell libusb-config --cflags)                                                                        -       USBLIBS=$(shell libusb-config --libs)                                                                           +       USBLIBS= -static $(shell libusb-config --libs)                                                                          EXE_SUFFIX =                                                                                                            OSFLAG = -D LINUX                                                                                                else ifeq ($(shell uname), Darwin) 
```

After that, the build process was pretty much the normal one:

```
$ make clean && make
```


## Linux ARM 32 bit/64 bit

I built the Linux ARM version of the tool on a Raspberry Pi 3 running Raspberry Pi OS Buster (for 32 bit) and Ubuntu Server 20.04 LTS (for 64 bit). The build steps were the same on both.

### Linux ARM Dependencies

On Rasperry Pi OS, the only dependencies seem to be `binutils` and `libusb-dev`. The make sure you've got these:

```
$ sudo apt update 
$ sudo apt install -y binutils libusb-dev
```

On Ubuntu, you also need:

```
$ sudo apt install -y build-essentials
```

### Building the commandline tool for Linux ARM

```
$ git clone https://github.com/micronucleus/micronucleus.git
$ cd micronucleus/commandline
$ git checkout tags/2.04 -b 2.04-release
$ make
```

This will take about 15 seconds and give you a binary for `micronucleus`.
