# micronucleus-commandline-arm-builds

Build of the Micronucleus command line tool for ARMv6/ARMv7.

This is an ARM build of version 2.04 of the [Micronucleus command line tool](https://github.com/micronucleus/micronucleus/tree/master/commandline). I built it to help out with getting a modern set of binaries for use with [ATTinyCore](https://github.com/SpenceKonde/ATTinyCore/issues/465). Otherwise not sure what it's good for.

## How it was built

### Dependencies
On Raspberry Pi OS, the only depencendies seem to be `binutils` and `libusb-dev`. The make sure you've got these:

```
$ sudo apt update 
$ sudo apt install -y binutils libusb-dev
```

### Building the commandline tool

To build `micronucleus` for `arm-linux-gnueabihf`:

```
$ git clone https://github.com/micronucleus/micronucleus.git
$ cd micronucleus/commandline
$ git checkout tags/2.04 -b 2.04-release
$ make
```

This will take about 15 seconds and give you a binary for `micronucleus`.
