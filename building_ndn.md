# Building NDN

Building NDN for Galileo requires us to cross compile the main program on host machine and port it onto Galileo. We can't build `ndn-cxx` and `nfd` on Galileo, not just because it's slow but because Galileo has only 512K memory, while compiling `ndn-cxx` requires 2G memory.

Since `boost` is included in the image and toolchain (if you followed my steps in  [`Building Galileo Image on Debian`](building_galileo_image_on_debian.md)). In this section, we show how to cross compile `libcryptopp`, `ndn-cxx`, and `nfd`. There are alternative ways to compile `libcryptopp` and `boost` (please see remark).

## On SDK machine

If you follow my instructions, your SDK should contain `boost` already. We shall install 3 more things here in SDK machine. Because NFD need a `-pthread` flag in `LDFLAGS`. We need to add it to our environment setup script. Note that if you use `/opt/ndn` or other directories outside your home directory on SDK machine, you may need to issue the following commands as a user with the appropriate permission, such as `root`.
```
cd /opt/ndn/
vim environment-setup-i586-poky-linux-uclibc
```
Append `-pthread` flag in `LDFLAGS`, so that the line becomes
`export LDFLAGS="-Wl,-O1 -Wl,--hash-style=gnu -Wl,--as-needed -pthread"`
And then set up the environment
```
source environment-setup-i586-poky-linux-uclibc
```

#### 1. libcryptopp
```
wget http://prdownloads.sourceforge.net/cryptopp/cryptopp562.zip
mkdir cryptopp
cd cryptopp
unzip ../cryptopp562.zip
```
There is a problem about libcryptopp's Makefile that prevents the make process from setting `-march` correctly. Please delete the following lines in `GNUmakefile`:
```
 39 ifeq ($(UNAME),Darwin)
 40 CXXFLAGS += -arch x86_64 -arch i386
 41 else
 42 CXXFLAGS += -march=native
 43 endif
```
These lines append `-march=native` to `CXXFLAGS` and therefore overwrite `-march=i586` from `environment-setup-i586-poky-linux-uclibc`. After deleting these lines, the compiled libary should work.
```
make dynamic
make install PREFIX=$PKG_CONFIG_SYSROOT_DIR/usr
```
Because NDN requires only the dynamic library, we do `make dynamic` here. The value of `PKG_CONFIG_SYSROOT_DIR` was set when you executed `source environment-setup-i586-poky-linux-uclibc`, the value should be `/opt/ndn/sysroots/i586-poky-linux-uclibc` if you choose the same path as me for the toolchain directory.

#### 2. ndn-cxx
```
cd /opt/ndn/
```
The `ndn-cxx` project use `waf`, which is a python tool, but if you simple type `./waf` in `ndn-cxx`, it will report error because many python modules are missing in the cross compiler environment. Fortuantely, we don't actully need to compile and install python programs, we simply use python to compile `ndn-cxx` (which is a C++ program), so there is no need to use toolchain's python. Let's set the python environment to SDK machine's python.
```
export PYTHONHOME=/usr
```
Then install `ndn-cxx`, and remember to invoke the python on SDK machine
```
git clone https://github.com/named-data/ndn-cxx
cd ndn-cxx
/usr/bin/python waf configure --with-cryptopp=$PKG_CONFIG_SYSROOT_DIR/usr/ --boost-includes=$PKG_CONFIG_SYSROOT_DIR/usr/include/ --boost-libs=$PKG_CONFIG_SYSROOT_DIR/usr/lib/ --prefix=$PKG_CONFIG_SYSROOT_DIR/usr/
/usr/bin/python waf
/usr/bin/python waf install
```

#### 3. nfd
```
cd /opt/ndn/
git clone --recursive https://github.com/named-data/NFD
cd NFD
/usr/bin/python waf configure --boost-includes=$PKG_CONFIG_SYSROOT_DIR/usr/include/ --boost-libs=$PKG_CONFIG_SYSROOT_DIR/usr/lib/ --prefix=$PKG_CONFIG_SYSROOT_DIR/usr/
/usr/bin/python waf
/usr/bin/python waf install
```
If you remember to add the `-pthread` to `LDFLAGS` in `environment-setup-i586-poky-linux-uclibc`, things should compile without error. If you did not follow my steps and encounter errors, the following are some of the possible errors and fixes:
* Boost linkage error: in wscript, delete all tests to boost
* `snprintf` not in `std`: In `face/ethernet-face.cpp`, change `std::snprintf` to `snprintf`, because in the toolchain, `snprintf` is not a member of `std`.
* `to_string` not found: In `tools/ndn-autoconfig/base-dns.cpp to_string` change `std::to_string` to `boost::chrono::to_string`. This is because toolchain's library does not provide `to_string`, a C++11 function (toolchain's gcc does provide C++11 standard, but it just doesn't provide `to_string`).

### Remark
We can compile `libcryptopp` and `boost` natively on Galileo, but the compilation takes a very long time.

* There are three ways to install `boost`:
  1. `boost` is best installed by a recipe from the start (as I mentioned in [`Building Galileo Image on Debian`](building_galileo_image_on_debian.md)), so that the toolchain contains boost from beginning.
  2. `boost` can now be installed natively in Galileo from [Intel's IoT repository](http://iotdk.intel.com/repos/1.1/iotdk/i586/).
  3. Cross compile `boost` in the toolchain. It requires changing some lines in the make script of `boost`. `boost` uses somthing called `bjam`, which reads scripts written with its own language. The error is related to a wrong parameter type in a function call. The type was correct in normal compilation, but in cross compilation, extra flags in `$CXX` causes the error.
  4. Compile `boost` natively on Galileo. Tried it and succeeded. It took me 2.5 days, just waiting for the compilation.

* For `libcryptopp`, there is no recipe, so we can cross compile it or compile it natively. Compile it natively takes only an hour, which isn't too bad.
