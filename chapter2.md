# The Commands

* WARNING: This Section May Contain Outdated Information! For latest instruction, visit [Chapter 1: The Whole Story](chapter1.md)

* WARNING: This page is a sequence of commands without explainations. By simply following this command, you might encounter bugs unexpected depending on your system. [The Whole Story](chapter1.md) presents a more detailed account and actions to take on different senario.

### `On Debian 7.8`
```
mkdir ~/ndn
cd ndn
sudo apt-get install vim gcc g++ git make build-essential subversion libcurl4-openssl-dev uuid-dev autoconf texinfo libssl-dev libtool diffstat gawk chrpath openjdk-7-jdk connect-proxy autopoint gettext p7zip-full gcc-multilib vim-common wget git-core
sudo  apt-get install libbison-dev flex
mkdir iasl;
cd iasl;
git clone git://github.com/acpica/acpica.git
cd acpica
make
sudo ln -s /home/schwannden/ndn/iasl/acpica/generate/unix/bin/iasl /usr/bin/iasl

cd ~/ndn
git clone https://github.com/schwannden/Galileo-Runtime
cd Galileo-Runtime
tar -xzf meta-clanton_v1.0.1.tar.gz
tar -xzf patches_v1.0.4.tar.gz
./patches_v1.0.4/patch.meta-clanton.sh

wget http://cgit.openembedded.org/openembedded-core/snapshot/openembedded-core-e0bc74e14f7ad67ff85959ce7c0a111d05ac7f2f.tar.gz
tar -xzvf openembedded-core-e0bc74e14f7ad67ff85959ce7c0a111d05ac7f2f.tar.gz
cd openembedded-core-e0bc74e14f7ad67ff85959ce7c0a111d05ac7f2f
cd meta/recipes-support
cp -r boost ~/ndn/Galileo-Runtime/meta-clanton_v1.0.1/meta-oe/meta-oe/recipes-support

cd ~/ndn/Galileo-Runtime/meta-clanton_v1.0.1
```

Edit image recipe `meta-clanton-distro/recipes-core/images/image-full.bb`
```
IMAGE_ROOTFS_SIZE = "3072000"
IMAGE_FEATURES += "package-management dev-pkgs"
IMAGE_INSTALL += "autoconf automake binutils binutils-symlinks cpp cpp-symlinks gcc gcc-symlinks g++ g++-symlinks gettext make libstdc++ libstdc++-dev file coreutils boost"
```

Edit `meta-oe/meta-oe/recipes-support/boost/boost.inc` to add `chrono` and `random` libraries.

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

```
cd ~/ndn/Galileo-Runtime/meta-clanton_v1.0.1
source poky/oe-init-build-env yocto_build
bitbake image-full-galileo
yocto_build/tmp/deploy/images
```

Format the SD card into `master boot record`, then add a `FAT32` partition, and copy the following 5 files from `yocto_build/tmp/deploy/images` onto the SD card.
1. boot/
2. grub.efi
3. bzImage
4. core-image-minimal-initramfs-clanton.cpio.gz
5. image-full-galileo-clanton.ext3


```
cd ~/ndn/Galileo-Runtime/meta-clanton_v1.0.1/yocto_build
bitbake  image-full -c populate_sdk
cd tmp/deploy/sdk
./clanton-tiny-uclibc-x86_64-i586-toolchain-1.4.2.sh
```
Enter `/opt/ndn` when prompted.
```
cd /opt
```
Change the owner and group of `/opt/ndn` to your name and group.
```
cd ndn
source environment-setup-i586-poky-linux-uclibc
wget http://prdownloads.sourceforge.net/cryptopp/cryptopp562.zip
mkdir cryptopp
cd cryptopp
unzip ../cryptopp562.zip
make dynamic
make install PREFIX=$PKG_CONFIG_SYSROOT_DIR

cd /opt/ndn/
git clone https://github.com/named-data/ndn-cxx
cd ndn-cxx
/usr/bin/python waf configure --with-cryptopp=$PKG_CONFIG_SYSROOT_DIR --boost-includes=$PKG_CONFIG_SYSROOT_DIR/usr/include/ --boost-libs=$PKG_CONFIG_SYSROOT_DIR/usr/lib/ --prefix=$PKG_CONFIG_SYSROOT_DIR/usr/
/usr/bin/python waf
/usr/bin/python waf install

cd /opt/ndn/
git clone --recursive https://github.com/named-data/NFD
cd NFD
```
* In `face/ethernet-face.cpp`, change `std::snprintf` to `snprintf`.
* In `tools/ndn-autoconfig/base-dns.cpp to_string` change `std::to_string` to `boost::chrono::to_string`.

```
/usr/bin/python waf configure --boost-includes=$PKG_CONFIG_SYSROOT_DIR/usr/include/ --boost-libs=$PKG_CONFIG_SYSROOT_DIR/usr/lib/ --prefix=$PKG_CONFIG_SYSROOT_DIR/usr/
/usr/bin/python waf
/usr/bin/python waf install
```

`End of Debian 7.8`

## `On Galileo`

Edit `/etc/opkg/base-feeds.conf` to add repository. Add following lines to the file:
```
src/gz all     http://repo.opkg.net/galileo/repo/all
src/gz clanton http://repo.opkg.net/galileo/repo/clanton
src/gz i586    http://repo.opkg.net/galileo/repo/i586
```

```
opkg update
opkg install --force-overwrite uclibc
opkg install pkgconfig openssl sqlite3
mkdir -p /opt/ndn/sysroots/i586-poky-linux-uclibc
```

`End of Galileo`
### `On Debian 7.8`
find out your Galileo IP and export to environment
```
export GalileoIP=10.0.0.2
```

```
cd $PKG_CONFIG_SYSROOT_DIR
scp -r include/cryptopp root@$GalileoIP:/include
scp lib/libcryptopp.so root@$GalileoIP:/lib
scp -r  usr/include/ndn-cxx root@$GalileoIP:/usr/include
scp usr/lib/libndn-cxx.a root@$GalileoIP:/usr/lib
scp -r  usr/include/ndn-cxx root@$GalileoIP:/opt/ndn/sysroots/i586-poky-linux-uclibc/usr/include
scp usr/lib/libndn-cxx.a root@$GalileoIP:/opt/ndn/sysroots/i586-poky-linux-uclibc/usr/lib
scp -r usr/bin/nfd* root@$GalileoIP:/bin
scp -r usr/bin/ndn* root@$GalileoIP:/bin
scp -r usr/bin/nfd* root@$GalileoIP:/opt/ndn/sysroots/i586-poky-linux-uclibc/usr/bin
scp -r usr/bin/ndn* root@$GalileoIP:/opt/ndn/sysroots/i586-poky-linux-uclibc/usr/bin
scp -r usr/etc/ndn root@$GalileoIP:/opt/ndn/sysroots/i586-poky-linux-uclibc/usr/etc/
```
