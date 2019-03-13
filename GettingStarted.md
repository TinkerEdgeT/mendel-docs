# Getting Started with Mendel Linux

## Check out the repository

We've stored our code in Gerrit, and like the Android developers before us, we
use `repo` to manage the projects in our Gerrit repositories.

To get started, first, you're going to need to pull down a copy of the `repo`
tool from our public facing sites and add it to your path:

```
mkdir -p bin
export PATH=$PATH:$HOME/bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

Make sure you've initialized git with your name and email address, and have
configured it properly for fetching the sources:

```
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

Once you've done this, you're actually ready to check out the sources. Make a
new directory where you'd like it to live. Depending on which board you will be
building for determines which manifest you need in your init. Each XML file is
named after the board's codename.

Our default board type is the Coral Dev Board, also known by the codename
`enterprise`, which is chosen by specifying the `enterprise.xml` manifest file.

```
repo init -u sso://aiyprojects/manifest -m enterprise.xml
repo sync -j$(nproc)
```

A current list of boards that are supported is available in the [manifest
project](https://aiyprojects.googlesource.com/manifest/+/refs/heads/master),
listed by codename.

Note that some boards require specific groups to be enabled to pull down the
correct packages. Contact the authors for more information on what groups are
needed for your specific board and situation.

After a short wait, you'll be ready to go for making changes to the repository
and building images.

## Prepare your build environment

### Supported Operating Systems

Currently the Mendel Linux build system assumes that you are building on a
Debian-based system that uses `apt-get` as the primary packaging tool. Building
on other platforms may be possible, but is currently unsupported.

### Docker

Further, to isolate the build from the host system, we build our packages using
pbuilder inside of Docker. We recommend configuring Docker so that the user that
you normally use can use it. You can find out how to install Docker on your
machine by following [Docker's official installation instructions for Docker
CE](https://docs.docker.com/install/).

Googlers have specific security requirements on internal workstations and must
follow the [install instructions here](http://go/installdocker).

### Fastboot

Mendel boards typically use the `fastboot` protocol for flashing boards, much
like an Android device. Once you've built, you will need to install `fastboot`
either via the [Android SDK
Manager](https://developer.android.com/studio/#downloads) or through your
operating system's package manager.

Once you have `fastboot` installed, you'll want to go sudo-less to flash your
device by making a new udev rules file:

```
sudo sh -c "echo 'SUBSYSTEM==\"usb\", ATTR{idVendor}==\"0525\", MODE=\"0664\", \
GROUP=\"plugdev\", TAG+=\"uaccess\"' >> /etc/udev/rules.d/65-android-local.rules"
sudo udevadm control --reload-rules && udevadm trigger
```

In some rare cases, it you may need a reboot to ensure udev is appropriately
reloaded with the new rule. Now you can check that you can use `fastboot` without
root privileges by running the following with a fastboot-compatible device
connected:

```
fastboot devices
```

If you see output from the tool, you know it's working fine.

## Build the tree

To build the tree, first you need to prepare your environment:

```
source build/setup.sh
```

At this point you'll have a couple of extra functions available to navigate
around the tree and build things in part or in whole. See below for more
information, or simply [read the source
here](https://aiyprojects.googlesource.com/build/+/refs/heads/master/setup.sh).

Currently the build system is configured to make use of Google's internal
continuous build system to speed up the build. Unfortunately, these tools are
not available publicly. This will likely change in the future, but for now,
you'll want to disable the use of this functionality by setting the following
environment variables:

```
export IS_EXTERNAL=true
```

The next step is to install the prerequisite packages by running:

```
m prereqs
```

Now you can build the tree by running:

```
m
```

The first time you'll be asked to set a pbuilder mirror site. Use
http://deb.debian.org/debian.

If you have not modified any packages, and would like to speed your install:

```
FETCH_PACKAGES=true m
```

This will cause packages to be fetched from Aptitude.

## Artifact caching

There are a few build artifacts that generally don't change, and can be cached
to provide a build speedup.

#### cache/base.tgz
Filesystem tarball for pbuilder. Set `FETCH_PBUILDER_DIRECTORY` in your
environment to the folder containing a copy of the file to use an already built
version.

#### rootdir_ARCH.raw.img
Base filesystem tarball made by multistrap. Set `ROOTFS_RAW_CACHE_DIRECTORY` in
your environment to the folder containing a copy of the file to use an already
built version.

#### cache/aiy-board-builder.tar
Docker container that the build happens in. Set `PREBUILT_DOCKER_ROOT` in your
environment to the folder containing a copy of the file to use an already built
version.

## Flash a device

After the above is complete, you should be able to flash the device:

```
flash.sh
```

To build the world for an sdcard, build the `sdcard` target:

```
m docker-sdcard
```

If the device was flashed properly you should be able to login to it:

```
mdt shell
```

## Quick Explanation of the Build System

The build system is relatively straightforward. It's a selection of split
out multi-target makefiles defined in the `build/` directory.

Adding modules is also relatively straightforward, assuming a decent
understanding of GNU Make rules. An example of how to get started is available
in the `build/template.mk` script. Simply copy this file to a new filename, and
then add the following line to the list of includes in the toplevel
`build/Makefile` file:

```
include $(ROOTDIR)/build/your-module.mk
```

Note that `targets` and `clean` are special partial targets. The rules must be
defined with a double colon (`::`) and not added to your module's `.PHONY`
target. Additionally, the output of the `targets` command is a list of strings
of the form:

```
targetname - some short description of your target
```

## Quick Tour of setup.sh Functions

### m -- Make everything from the toplevel

The `m` function can be called from any directory in the filesystem to build the
entire tree from the ground up. You can specify a target to build with the
command as well, just like you would do with make.

Effectively the command does a pushd to the root of the tree and issues a make
command. You can specify any options or targets you'd like to build, including
the `targets` command to have a look around in the build system.

### j -- Jump around the directory tree

This is a kind of switchboard of sorts that lets you jump around the tree
quickly. Many of the paths are arcane because of historical reasons, so this
tool was written to make things a bit easier.

If you issue the `j` command with no arguments, it will automatically change
your current working directory to the root of the tree. If, however, you issue
`j help`, it will list a list of target names you can "jump" to. You can open up
`build/setup.sh` to see which ones are currently defined, or just jump around
and explore.

## Quick Tour of the Directory Layout

#### build/
Contains build scripts, the setup.sh script and rootfs overlay, as well as some
additional tooling.

#### cache/
Contains the debootstrap tarball you must build before starting any other build
lives here. This directory is ephemeral and doesn't exist on a first clone, but
will also not be removed when the `clean` target is run. Bear in mind: if you
update the package list in `build/debootstrap.mk`, you'll need to re-reun `mm
deboostrap make-debootstrap-tarball` to regenerate this.

#### docs/
Contains this documentation, as well as other documentation about the build
system and working in the tree.

#### imx-firmware/
Contains the atheros and qualcomm Wifi firmware blobs. Cloned from the
AndroidThings repository and will likely migrate out as time goes on and bom
selections change.

#### linux-imx/
Contains the source code for the Linux kernel.

#### out/
The ephemeral directory that contains both local host binaries and tools, as
well as build artifacts and the final images produced. It has a specific
structure documented elsewhere.

#### packages/
Debian package sources used to build the packages installed to make the
distribution more functional for the Enterprise board.

#### tools/
Contains miscellaneous tools to support the image. Bpt and img2simg are forked
from the Android Things tree, and imx-mkimage from NXP.

#### uboot-imx/
Contains the u-boot bootloader, with support for i.MX8 devices.
