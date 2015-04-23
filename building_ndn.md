# Building NDN

Building NDN for Galileo requires we cross compile the main program on host machine, and port it onto Galileo. We can't build `ndn-cxx` and `nfd` on Galileo, not just because its slow, but because Galileo only has 512K memory, while compiling `ndn-cxx` requires 2G.

Since `boost` is included in the image and toolchain ( if you followed my steps in  [`Building Galileo Image on Debian`](building_galileo_image_on_debian.md)). In this chapter, we show how to cross compile `libcryptopp`, `ndn-cxx`, `nfd`. There are alternative ways to compile `libcryptopp` and `boost`, please see remark.


### On SDK machine

If you follow my instructions, your SDK should contain `boost` already. We shall install 3 more things here in SDk machine. Because NFD need a `-pthread` flag in `LDFLAGS`. We need to add it to our environment set-up script.
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

1. libcryptopp

```
wget http://prdownloads.sourceforge.net/cryptopp/cryptopp562.zip
mkdir cryptopp
cd cryptopp
unzip ../cryptopp562.zip
```
There is a problem about libcryptopp's make file that prevents making process from setting `-march` correctly. Please delete the following lines in `GNUmakefile`
```
 39 ifeq ($(UNAME),Darwin)
 40 CXXFLAGS += -arch x86_64 -arch i386
 41 else
 42 CXXFLAGS += -march=native
 43 endif
```
These lines appends `-march=native` to `CXXFLAGS`, overwriting `-march=i586` from `environment-setup-i586-poky-linux-uclibc`. After deleting these lines, the compiled libary should work correctly.
```
make dynamic
make install PREFIX=$PKG_CONFIG_SYSROOT_DIR
```
Note because we NDN only requires dynamic library, some we only make the dynamic library. The value of `PKG_CONFIG_SYSROOT_DIR` was set when you executed `source environment-setup-i586-poky-linux-uclibc`, the value should be `/opt/ndn/sysroots/i586-poky-linux-uclibc` if you choose the same path as me for the tool chain directory.

2. ndn-cxx
```
cd /opt/ndn/
```
The `ndn-cxx` project use waf, which is a python tool, but if you simple type `./waf` in `ndn-cxx`, it will report error because many python modules are missing in the cross compilation environment. Fortuantely, we don't actully need to compiler and install python programs, we simply use python to compile `ndn-cxx` (which is C++ program), so there is no need to use tool chain's python. Let's set the python environment to the sdk machine's python.
```
export PYTHONHOME=/usr
```
Then install `ndn-cxx`, and remember to execute SDK machine's python
```
git clone https://github.com/named-data/ndn-cxx
cd ndn-cxx
/usr/bin/python waf configure --with-cryptopp=$PKG_CONFIG_SYSROOT_DIR --boost-includes=$PKG_CONFIG_SYSROOT_DIR/usr/include/ --boost-libs=$PKG_CONFIG_SYSROOT_DIR/usr/lib/ --prefix=$PKG_CONFIG_SYSROOT_DIR/usr/
/usr/bin/python waf
/usr/bin/python waf install
```

3. nfd
```
cd /opt/ndn/
git clone --recursive https://github.com/named-data/NFD
cd NFD
/usr/bin/python waf configure --boost-includes=$PKG_CONFIG_SYSROOT_DIR/usr/include/ --boost-libs=$PKG_CONFIG_SYSROOT_DIR/usr/lib/ --prefix=$PKG_CONFIG_SYSROOT_DIR/usr/
/usr/bin/python waf
/usr/bin/python waf install
```
At the time of writing, I encountered two errors. They are easily fixed by looking at error message, nevertheless I shall list the errors and modifications I made.

* In wscript, delete akk test to boost
* In `face/ethernet-face.cpp`, change `std::snprintf` to `snprintf`, because in the tool chain, `snprintf` is not a member of `std`.
* In `tools/ndn-autoconfig/base-dns.cpp to_string` change `std::to_string` to `boost::chrono::to_string`. This is because tool chain's library does not provide `to_string`, a C++11 function (tool chain's gcc does provide C++11 standard, it just doesn't provide `to_string`).

### Remark
We can, compile `libcryptopp` and `boost` natively on Galileo, but it takes very long.

* There are three ways to install `boost`:
  1. `boost` is best installed by a recipe from the start (as I mentioned in [`Building Galileo Image on Debian`](building_galileo_image_on_debian.md)), so that the tool chain contains boost from beginning.
  2. `boost` can now be installed natively in Galileo from [Intel's IoT repository](http://iotdk.intel.com/repos/1.1/iotdk/i586/).
  3. Cross compile `boost` in the tool chain. It requires changing some lines in `boost`'s make script. `boost` uses somthing called bjam, which read script written by its own language. The error is related to a wrong parameter type in a function call. The type was correct in normal compilation, but in cross compilation, extra flags in `$CXX` causes the error.
  4. Compile `boost` natively on Galileo. Tried it and succeeded, took me 2.5 days, just waiting for the compilation.

* For `libcryptopp`, there is no recipe, so we can cross compile it or compile it natively. Compile it natively takes only an hours, which isn't too bad.
