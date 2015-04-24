# Building Cross Compile Toolchain

The easiness of building a cross compilation environment is one of the great feature of Yocto. First let's clarify some terms:
* Host machine: The machine on which you are running Yocto.
* SDK machine: Same as host machine.
* Target machine: The machine on which you want to run cross compiled target code (in this case, Intel Galileo).


```
cd ~/ndn/Galileo-Runtime/meta-clanton_v1.0.1/yocto_build
```

Simply type
```
bitbake  image-full -c populate_sdk
```
The `-c` flag tells `bitbake` to execute a specific procedure of the recipe. The SDK installation file will be put into `yacto_build/tmp/deploy/sdk`. Execute the SDK installation file
```
cd tmp/deploy/sdk
./clanton-tiny-uclibc-x86_64-i586-toolchain-1.4.2.sh
```
This will first ask where you want to install the cross compilation tool chain, and install the cross compile tool chain there. I use `/opt/ndn` as my target (see remark 1). After the installation is successful, go to the tool chain directory and set up the environment.
```
cd /opt/ndn
source environment-setup-i586-poky-linux-uclibc
```
Do look at `environment-setup-i586-poky-linux-uclibc` to see what variables are set. For example, your compiler `CC` is set to `i586-poky-linux-uclibc-gcc -m32 -march=i586 --sysroot=/opt/ndn/sysroots/i586-poky-linux-uclibc`.

Now you can start cross compile everything.

### Remark
1. Why should we install cross compile tool chain under a root directory like `/opt`?

Installing the tool chain under `/opt` has the advantage of flexibility. Sometimes a cross compiled target code does not depends on other code, or depends on other code in a relative path. These cases are fine. But if it depends on other code in terms of absolute path, it is hard to move everything to another machine. For example, in `nfd-start`, it execute `nfd` in terms of absolute path. So if I install the tool chain in my home directory `/home/foo/ndn/`, and I compiled `nfd` in this tool chain, the `nfd-start` then calls `/home/foo/ndn/sysroots/....../usr/bin`, this means if you want to move the whole thing to your Galileo, you need to create a user `foo` and copy everything there, and then set permission to allow other users to use it and other stuff.
