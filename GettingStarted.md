# Getting Started with Debian on the i.MX8M

## Check out the repository

We've stored our code in Gerrit, and like the Android developers before us, we
use repo to manage the projects in our Gerrit repositories.

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
new directory where you'd like it to live, and do the following:

```
repo init -u sso://aiyprojects/manifest -g default,imx
repo sync -j$(nproc)
```

After a short wait, you'll be ready to go for making changes to the repository
and building images.


## Build the tree

To build the tree, first you need to prepare your environment:

```
source build/setup.sh
```

At this point you'll have a couple of extra functions available to navigate
around the tree and build things in part or in whole.

We assume you're running Debian or one of its derivatives and on a relatively
recent Intel CPU. Before you actually try to build the system, you'll want to
make sure you have all of the prerequisites installed. Since we assume
Debian, you can just do the following to install them:

```
m prereqs
```

Now that you have the prerequisites installed, we can build:
```
m -j$(nproc)
```


## Flash a device

After the above is complete, you should be able to flash the device:
```
j build
flash.sh
```

Most of this build relies on cached versions of a Debian bootstrap.
To rebuild this, remove any copies of debootstrap.tgz from your cache directory
and run:
```
mm debootstrap make-bootstrap-tarball
```

To build the world for an sdcard, build the `sdcard` target:

```
m sdcard
```

## Quick Explanation of the Build System

The build system is relatively straightforward. It's a selection of split
out multi-target makefiles defined in the `build/` directory. Each makefile is
designed to be as standalone as possible so that each can be built individually
using the `mm` command, or collectively using `m`.

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

This is actually parsed by the `mm` function to help with tab completion. It's
relatively robust in the face of spaces, but you must separate your targetname
with a dash (`-`) from the description, or the completion routine will likely
break.

If you follow the conventions in the template, `mm` will autocomplete your
module and it's targets listed in the `targets` make target.

## Quick Tour of setup.sh Functions

### m -- Make everything from the toplevel

The `m` function can be called from any directory in the filesystem to build the
entire tree from the ground up. Effectively the command does a pushd to the root
of the tree and issues a make command. You can specify any options or targets
you'd like to build, including the `targets` command to have a look around in
the build system.

### mm -- Make an individual Module

The `mm` function can, like the `m` function, be called from any directory in
the filesystem to build an individual module. It has autocompletion functions,
so simply typing `mm` and then hitting TAB will autocomplete the list of modules
available. Additionally, if you press TAB again, after you've autocompleted a
module name, it will autocomplete the list of targets available in that module,
as returned by the `targets` target.

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

#### imx-gpu-viv/
Contains the Vivante upstream binary drivers for the GPU. These are required to
get the displays working. Also clonsed from the AndroidThings repository.

#### linux-imx/
Contains the source code for the Linux kernel.

#### out/
The ephemeral directory that contains both local host binaries and tools, as
well as build artifacts and the final images produced. It has a specific
structure documented elsewhere.

#### packages/
Debian package sources used to build the packages installed to make the
distribution more functional for the Enterprise board.

#### toolchain/
Contains the prebuilt toolchains to build for an aarch64 target.

#### tools/
Contains miscellaneous tools to support the image. Bpt and img2simg are forked
from the Android Things tree, and imx-mkimage from NXP.

#### uboot-imx/
Contains the u-boot bootloader, with support for i.MX8 devices.
