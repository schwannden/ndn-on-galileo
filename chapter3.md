# The Lazy Ride

Download this [allInOne]( https://www.dropbox.com/sh/v24bbmqymbkw3v9/AADdVbMV57T6U9WhRNxrGDVta?dl=0) folder.

The image is the end result of [Building Galileo Image on Debian](building_galileo_image_on_debian.md)
```
tar -xzvf image.tar.gz
cd image
cp * /media/SDcard
```

Tool chain: https://github.com/schwannden/galileoToolchain

`This tool chain is only verified on Debian 7.8, but should work on most 64-bit linux system`

This tool chain is the result of [Building Cross Compile Toolchain](building_cross_compile_toolchain.md). Other than above mentioned packages, it has in addition installed
```
libcryptopp ndn-cxx NFD
```

To copy everything to Galileo, usse the script `sdk2Galileo.sh` in [allInOne]( https://www.dropbox.com/sh/v24bbmqymbkw3v9/AADdVbMV57T6U9WhRNxrGDVta?dl=0) folder.
```
./sdk2Galileo -a
./sdk2Galileo -m configure
```