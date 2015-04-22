# The Lazy Ride

Galileo image: https://github.com/schwannden/galileoImage

The image is the end result of [Building Galileo Image on Debian](building_galileo_image_on_debian.md), which has a 3.1 GB file system, and contains
```
autoconf automake binutils binutils-symlinks cpp cpp-symlinks 
gcc gcc-symlinks g++ g++-symlinks gettext make 
libstdc++ libstdc++-dev file coreutils boost
```
Note that due to file size contraint, I have to compress and split the file system before pushing to GitHub
```
tar -czf image-full-galileo-clanton.ext3.tar.gz image-full-galileo-clanton.ext3
split -n 2 image-full-galileo-clanton.ext3.tar.gz
```
You can combine the file system by
```
cat xaa xab >  image-full-galileo-clanton.ext3.tar.gz
```
then de-compress by
```
tar -xzvf image-full-galileo-clanton.ext3.tar.gz
```

Tool chain: https://github.com/schwannden/galileoYoctoToolcahin

`This tool chain is only verified on Debian 7.8, but should work on most 64-bit linux system`

This tool chain is the result of [Building Cross Compile Toolchain](building_cross_compile_toolchain.md). Other than above mentioned packages, it has in addition installed
```
libcryptopp ndn-cxx NFD
```

Now following instructions in [Building NDN](building_ndn.md), suppose the IP address of your Galileo is `10.0.0.2`
```
export GalileoIP=10.0.0.2
```
1. `libcryptopp` library and headers
```
scp -r include/cryptopp root@$GalileoIP:/include
scp lib/libcryptopp.so root@$GalileoIP:/lib
```
2. `ndn-cxx` libraries and headers
```
scp -r  usr/include/ndn-cxx root@$GalileoIP:/usr/include
scp usr/lib/libndn-cxx.a root@$GalileoIP:/usr/lib
scp -r  usr/include/ndn-cxx 
```
3. `nfd` executable
```
scp -r usr/bin/nfd* root@$GalileoIP:/bin
scp -r usr/bin/ndn* root@$GalileoIP:/bin
```
4. `ndn` configuration files
```
scp -r usr/etc/ndn root@$GalileoIP:/opt/ndn/sysroots/i586-poky-linux-uclibc/usr/etc/
```