# The Whole Story
### Summary
This chapter is split into three parts

1. How to compile a customized linux kernel for Galileo using Yocto (customized in the sense that you can install ndn-cxx and nfd on to it.).
2. How to build the cross compilation SDK for this customized linux (In case you want to play more with Galileo).
3. How to cross compile ndn-cxx and nfd using the above SDK.

### What is Yocto?

Yocto is not a linux distribution, and Yocto is not a kernel. Yocto is a set of tools that facilitate the compilation of a customized linux kernel on various embedded machine. We choose Yocto for its flexibility on building our own kernel, and also because Intel offers a complete BSP layer (will talk more about this) for building a customized kernel onto Intel Galileo.

Since Galileo's BSP layer uses an out-dated version of Yocto (1.6), many version of operating system won't work directly. In Fedora 20 I was able to get it working with many patches. In Mac OS, it is even harder to get it working.

So far I've had better memory with Ubuntu 12.04 and Debian 7.8, while debian 7.8 is the best and most recommended platform!


