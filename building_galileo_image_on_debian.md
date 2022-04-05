# Building Galileo Image on Debian

Firstly, create a directory to do all the work
```
mkdir ~/ndn
cd ~/ndn
```

#### Pre-requisites
Install all the pre-requisite
```
sudo apt-get install vim gcc g++ git make build-essential subversion libcurl4-openssl-dev uuid-dev autoconf texinfo libssl-dev libtool diffstat gawk chrpath openjdk-7-jdk connect-proxy autopoint gettext p7zip-full gcc-multilib vim-common wget git-core iasl
```
Try ` iasl | head ` to see if ACPI 5.0 or up is supported. If you see something like

![Supports ACPI Specification Revision 4.0a](fig1.1.01-acpi4.png)

You will need to manually install it yourself.
#### Manual Installation for iasl
Pre-requisite of iasl:
```
sudo apt-get install libbison-dev flex
```
Getting and compiling iasl:
```
mkdir iasl
cd iasl
git clone https://github.com/acpica/acpica.git
cd acpica
make
```
Starting to use the newly built iasl:
```
sudo make install
```
Again, try ` iasl | head ` to see if ACPI 5.0 or up is supported. It should be now.

![Supports ACPI Specification Revision 5.1](fig1.1.02-acpi5.png)

A possible error may happen during `iasl` compilation, see remark at the end.

#### Getting Recipes
This is the specific meta package I use for my build: `https://github.com/schwannden/Galileo-Runtime`, which is forked from ` https://github.com/01org/Galileo-Runtime on 2015/2/1`. I forked it because I intend to maintain it as a customized meta package for NDN development. So it is suggested you clone from my repository.

```
cd ~/ndn
git clone https://github.com/schwannden/Galileo-Runtime
cd Galileo-Runtime
```

Because we are building an SD card image, we need only the following two folders:

This is the major folder that contains all the recipes for building galileo
```
tar -xzf meta-clanton_v1.0.1.tar.gz
```
This contains all the patches
```
tar -xzf patches_v1.0.4.tar.gz
```
Firstly, we apply the patches. The script will in turn calls the setup.sh in `meta-clanton_v1.0.1` to clone addition meta folders (meta-oe and meta-intel) and apply the patches
```
./patches_v1.0.4/patch.meta-clanton.sh
```

Now, if you do `ls meta-clanton_v1.0.1`, you can see all the meta directories.

![meta-clanton_v1.0.1](fig1.1.03-meta-clanton.png)

Because NDN project requires Boost library, which is not provided by default, we need to get it at [Open Embedded Metadata Index](http://layers.openembedded.org/layerindex/branch/master/layer/openembedded-core/). Go to [Open Embedded Metadata Index](http://layers.openembedded.org/layerindex/branch/master/layer/openembedded-core/), one can see `boost` is provided. Go to [boost index](http://layers.openembedded.org/layerindex/recipe/5268/), at the time of writing, they provide `boost 1.57`, which is not suitable for NDN development. We can find an older version of `boost 1.55` [here](http://cgit.openembedded.org/cgit.cgi/openembedded-core/commit/?id=e0bc74e14f7ad67ff85959ce7c0a111d05ac7f2f), in the download section. Download and extract [this file](http://cgit.openembedded.org/openembedded-core/snapshot/openembedded-core-e0bc74e14f7ad67ff85959ce7c0a111d05ac7f2f.tar.gz):
```
wget http://cgit.openembedded.org/openembedded-core/snapshot/openembedded-core-e0bc74e14f7ad67ff85959ce7c0a111d05ac7f2f.tar.gz
tar -xzf openembedded-core-e0bc74e14f7ad67ff85959ce7c0a111d05ac7f2f.tar.gz
```
All we need is boost's recipe, so copy the `boost` folder into one of the working directory's meta-* folder. Since this is provided by [Open Embedded](http://layers.openembedded.org/layerindex/branch/master/layer/openembedded-core/), let's copy it into `meta-oe` folder.
```
cd openembedded-core-e0bc74e14f7ad67ff85959ce7c0a111d05ac7f2f
cd meta/recipes-support
cp -r boost ~/ndn/Galileo-Runtime/meta-clanton_v1.0.1/meta-oe/meta-oe/recipes-support
```
Now we have all we need for building Galileo image!

#### Applying Changes
```
cd ~/ndn/Galileo-Runtime/meta-clanton_v1.0.1
```
First we need to edit the image recipe to make sure all relevent packges are in Galileo.  Edit `meta-clanton-distro/recipes-core/images/image-full.bb`
1. Increase the image filesystem ten times to around 3 G:
    `IMAGE_ROOTFS_SIZE = "3072000"`
2. Add features:
    `IMAGE_FEATURES += "package-management dev-pkgs"`
    The added option `dev-pkgs` means install the development packages (headers and extra library links) for all packages installed in a given image.
3. Add packages
    `IMAGE_INSTALL += "autoconf automake binutils binutils-symlinks cpp cpp-symlinks gcc gcc-symlinks g++ g++-symlinks gettext make libstdc++ libstdc++-dev file coreutils boost"`

Then edit `meta-oe/meta-oe/recipes-support/boost/boost.inc` to add `chrono` and `random` libraries. These two libraries are not installed by default but are needed for NDN development. (One can find out what boost libraries are required in `.waf-tools/boost.py`.)

```
  7 BOOST_LIBS = "\
  8         date_time \
  9         chrono \
 10         random \
 11         filesystem \
 12         graph \
 13         iostreams \
 14         program_options \
 15         regex \
 16         serialization \
 17         signals \
 18         system \
 19         test \
 20         thread \
 21         "
```

There is a known removed repository. Edit `meta-clanton-bsp/recipes-bsp/grub/grub_0.97.bb` and you have the following two options:
1. Use `git://github.com/mjg59/grub-fedora.git` by changing the line of SRC_URI as
```
SRC_URI = "git://github.com/mjg59/grub-fedora.git"
```

2. Use `git://github.com/architech-backup/grub-fedora.git` by changing the line of SRC_URI as
```
SRC_URI = "git://github.com/architech-backup/grub-fedora.git"
```
and the line of SECREV as
```
SRCREV = "f72d4d83931f7a6427771b480101e251a57ac1b8"
```

There are a number of changes that need to be made to URLs which call repositories:
1. In file `meta-clanton_v1.0.1/meta-oe/meta-oe/recipes-multimedia/x264/x264_git.bb`, on what should be line 11:
    - Change: `SRC_URI = "git://git.videolan.org/x264.git \` ...
    - To: `SRC_URI = "git://gitorious.org/gxk-media/x264.git;protocol=http \` ...        
    - Comment out the existing `SRCREV` line and add a new line: `SRCREV = "${AUTOREV}"`
2. In file `openembedded-core-e0bc74e14f7ad67ff85959ce7c0a111d05ac7f2f/meta/recipes-multimedia/x264/x264_git.bb` on what should be line 10:
    - Change: `SRC_URI = "git://git.videolan.org/x264.git \` ...
    - To: `SRC_URI = "git://gitorious.org/gxk-media/x264.git;protocol=http \` ...
3. In file `meta-clanton_v1.0.1/meta-clanton-bsp/recipes-bsp/grub/grub_0.97.bb` on what should be line 19:
    - Change: `SRC_URI = "git://github.com/mjg59/grub-fedora.git"`
    - To: `SRC_URI = "git://github.com/mjg59/grub-fedora.git;protocol=https"`
4. In file `meta-clanton_v1.0.1/meta-oe/meta-oe/recipes-graphics/tslib/tslib_git.bb` on what should be line 19:
    - Change: `SRC_URI = "git://github.com/kergoth/tslib.git;protocol=git \`
    - To: `SRC_URI = "git://github.com/kergoth/tslib.git;protocol=https \`
5. In file `meta-clanton_v1.0.1/meta-oe/meta-oe/recipes-support/yasm/yasm_1.2.0.bb` on what should be lines 3 **and** 7:
    - Change `HOMEPAGE` to: `HOMEPAGE = "https://www.tortall.net/projects/yasm/"`
    - Change `SRC_URI` to: `SRC_URI = "https://www.tortall.net/projects/yasm/releases/yasm-${PV}.tar.gz"`
6. In file `meta-clanton_v1.0.1/meta-oe/meta-oe/recipes-support/yasm/yasm_1.2.0.bb` on what should be lines 2 **and** 9:
    - Change `HOMEPAGE` to: `HOMEPAGE = "https://ftp.osuosl.org/pub/blfs/conglomeration/libsndfile"`
    - Change `SRC_URI` to: `SRC_URI = "https://ftp.osuosl.org/pub/blfs/conglomeration/libsndfile/libsndfile-${PV}.tar.gz"`
7. In file `meta-clanton_v1.0.1/meta-oe/meta-oe/recipes-support/yasm/yasm_1.2.0.bb` on what should be lines 3 **and** 4:
    - Change `HOMEPAGE` to: `HOMEPAGE = "https://flac.sourceforge.net/"`
    - Change `BUGTRACKER` to `BUGTRACKER = "https://sourceforge.net/tracker/?group_id=13478&atid=113478"`
8. In file `meta-clanton_v1.0.1/poky/meta/recipes-connectivity/libpcap/libpcap.inc` on what should be lines 5, 6 **and** 19:
    - Change `HOMEPAGE` to: `HOMEPAGE = "https://www.tcpdump.org/"`
    - Change `BUGTRACKER` to `BUGTRACKER = "https://sourceforge.net/tracker/?group_id=53067&atid=469577"`
    - Change `SRC_URI` to `SRC_URI = "https://www.tcpdump.org/release/libpcap-${PV}.tar.gz"`
9. In file `meta-clanton_v1.0.1/poky/meta/recipes-multimedia/libtheora/libtheora_1.1.1.bb` on what should be lines 3 **and** 10:
    - Change `HOMEPAGE` to: `HOMEPAGE = "https://ftp.osuosl.org/"`
    - Change `SRC_URI` to `SRC_URI = "https://ftp.osuosl.org/pub/xiph/releases/theora/libtheora-${PV}.tar.bz2 \`
10. In file `meta-clanton_v1.0.1/meta-oe/meta-oe/recipes-multimedia/webm/libvpx.inc` on what should be line 5:
    - Change `SRC_URI` to `SRC_URI = "https://src.fedoraproject.org/lookaside/pkgs/libvpx/libvpx-v0.9.5.tar.bz2/4bf2f2c76700202c1fe9201fcb0680e3/libvpx-v${PV}.tar.bz2"`
11. In file `meta-clanton_v1.0.1/poky/meta/recipes-multimedia/libvorbis/libvorbis_1.3.3.bb` on what should be lines 5 **and** 16:
    - Change `HOMEPAGE` to: `HOMEPAGE = "https://ftp.osuosl.org"`
    - Change `SRC_URI` to `SRC_URI = "https://ftp.osuosl.org/pub/xiph/releases/vorbis/libvorbis-${PV}.tar.gz \`
12. In file `meta-clanton_v1.0.1/poky/meta/recipes-multimedia/liba52/liba52_0.7.4.bb` on what should be lines 2 **and** 11:
    - Change `HOMEPAGE` to: `HOMEPAGE = "https://ftp.osuosl.org/"`
    - Change `SRC_URI` to `SRC_URI = "https://ftp.osuosl.org/pub/blfs/conglomeration/a52dec/a52dec-${PV}.tar.gz \`
13. In file `meta-clanton_v1.0.1/poky/meta/recipes-graphics/libsdl/libsdl_1.2.15.bb` on what should be lines 5, 6 **and** 24:
    - Change `HOMEPAGE` to: `HOMEPAGE = "https://www.libsdl.org"`
    - Change `BUGTRACKER` to: `BUGTRACKER = "https://bugzilla.libsdl.org/"`
    - Change `SRC_URI` to `SRC_URI = "https://www.libsdl.org/release/SDL-${PV}.tar.gz \`
14. In file `meta-clanton_v1.0.1/poky/meta/recipes-devtools/perl/perl-native_5.14.3.bb` on what should be lines 2 **and** 13:
    - Change `HOMEPAGE` to: `HOMEPAGE = "https://www.perl.org/"`
    - Change `SRC_URI` to `SRC_URI = "https://www.cpan.org/src/5.0/perl-${PV}.tar.gz \`
15. In file `meta-clanton_v1.0.1/poky/meta/recipes-multimedia/libsamplerate/libsamplerate0_0.1.8.bb` on what should be line 9:
    - Change `SRC_URI` to `SRC_URI = "https://ftp.osuosl.org/pub/blfs/conglomeration/libsamplerate/libsamplerate-${PV}.tar.gz"`
16. Run `find . -type f -name "*.bb" -print0 | xargs -0 sed -i '' -e 's/http\:\/\//https\:\/\//g'` to convert all http URLs to use https
        

#### Building Images
```
cd ~/ndn/Galileo-Runtime/meta-clanton_v1.0.1
```
First, set up the environment. `oe-init-build-env` will set up the environment variables for Yocto build system and automatically change directory to the build directory. The default is `build/`, but here we specify `yocto_build/`.

```
source poky/oe-init-build-env yocto_build
```
![Shell environment set up for builds](fig1.1.04-env-setup.png)

Now you should be in `yocto_build/` directory, simply build the image by using:
```
bitbake image-full-galileo
```
It takes a long time, probably a couple of hours, to finish. On my Intel i5 it takes 2 hours. Do not panic if you see warnings and errors. Sometimes the network in not stable, or the required repository is temporarily unavailable. The process consequently gets a time-out when fetching some packages. Just relaunch the building process repeatedly. In case a repository is really dead, add an alternative live repository (this is more complicated, see [Debug Yocto](debug_yocto.md)).

If the building process succeeds, the image will be created in directory `yocto_build/tmp/deploy/images/`.

If you do `tree tmp/deploy/images`, you should see

![Image files in yocto_build/tmp/deploy/images/](fig1.1.05-image.png)

#### Copying images to SD card
Galileo uses micro SD card, so a machine that reads micro SD card is needed for this step. I use a micro SD to SD card adapter for convenience. In Debian, we can simply use the `Disk Utility` in `Applications/Accessories/Dist Utility` to format the (micro) SD card. First format the SD card into the `Master Boot Record` (MBR) scheme, then add a `FAT` partition, and copy the following 5 files from `yocto_build/tmp/deploy/images/` onto the card.
1. boot/
2. bzImage
3. core-image-minimal-initramfs-clanton.cpio.gz
4. grub.efi
5. image-full-galileo-clanton.ext3

If the way with which you use to copy perserves the symbolic links, you may need to change the behavior, such as adding `-L` or `--dereference` to `cp`. If you would like to directly copy the actual files, renaming some files is required, because Yocto inserts the date-time as part of the file name on each build. As you might see, for example, `image-full-galileo-clanton-20150426053734.rootfs.ext3`.

Now, insert the micro SD card into Galileo, and you are all set for booting your Galileo image. By default, the image has the ssh server daemon starting on boot with the standard ssh port (22) and user `root` with no password. If, for some reason, networking is not availale, you can [use a FTDI232 TTL 3.3V cable](https://communities.intel.com/message/247258) with an appropriate terminal software, such as [PuTTY](http://www.putty.org/) in Windows, to connect to the serial console of Galileo for troubleshooting.

* See [Boot From SD Card](boot_from_sd_card.md) for more detail.
* See [Debug Galileo](debug_galileo.md) for some hacks on debugging and connecting to Galileo.

#### Remark
* `iasl` bug

At the time of writing (before `acpica commit 8e2118`), the Makefile for `iasl` treats warning messages as fatal, causing the compilation to fail due to line 660 of `source/components/events/evgpe.c` with usage of an uninitialized variable `Status`. Just initialize `Status` to 0 at line 635 and `make` it.


