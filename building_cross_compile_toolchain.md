# Building Cross Compiler Toolchain

The easiness of building a cross compilation environment is one of the great features of Yocto. First let's clarify some terms:
* Host machine: The machine on which you are running Yocto.
* SDK machine: Same as host machine.
* Target machine: The machine on which you want to run the cross compiled target code (in this case, Intel Galileo).


```
cd ~/ndn/Galileo-Runtime/meta-clanton_v1.0.1
source poky/oe-init-build-env yocto_build
```

Simply type
```
bitbake image-full -c populate_sdk
```
The `-c` flag tells `bitbake` to execute a specific procedure of the recipe. This process might take a couple of hours to finish. The SDK installation file will be put into `yocto_build/tmp/deploy/sdk/`. Execute the SDK installation file
```
cd tmp/deploy/sdk
./clanton-tiny-uclibc-x86_64-i586-toolchain-1.4.2.sh
```
You will be first asked where you want to install the cross compiler toolchain, and then the cross compiler toolchain will be installed there. I use `/opt/ndn` as my destination directory (see remark 1). After installing the SDK, you may go to the toolchain directory and set up the environment for building NDN.
```
cd /opt/ndn
source environment-setup-i586-poky-linux-uclibc
```
Do look at `environment-setup-i586-poky-linux-uclibc` to see what variables are set. For example, your compiler `CC` is set to `i586-poky-linux-uclibc-gcc -m32 -march=i586 --sysroot=/opt/ndn/sysroots/i586-poky-linux-uclibc`.

Now you can start to cross compile everything.

### Remark
1. Why should we install cross compile toolchain under a root directory like `/opt`?

Installing the toolchain under `/opt` has the advantage of flexibility. Sometimes the cross compiled target code does not depends on other code or depends on other code in a relative path. These cases are fine. However, if it depends on other code in terms of an absolute path, it is hard to move everything to another machine. For example, in `nfd-start`, it executes `nfd` in terms of an absolute path. So if I install the toolchain in my home directory, e.g., `/home/foo/ndn/`, and I compile `nfd` in this toolchain, `nfd-start` will call `/home/foo/ndn/sysroots/....../usr/bin`. This means if you want to move the whole thing to Galileo, you need to create a user `foo`, copy everything over, and then set the appropriate permission to allow other users to use it and other stuff.
