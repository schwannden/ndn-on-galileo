# Boot From SD Card

Intel Galileo Gen 2 can boot from a micro SD card. This section explains how to write a Galileo linux image to SD card and boot from SD card.

![](sd3.png)

## Getting Image
You can get the Galileo's linux image by following [Building Galileo Image on Debian](building_galileo_image_from_debian.md) or download it [here](http://sourceforge.net/projects/ndn-in-one/files/image.tar.gz), built by the authors.

## Preparing SD Card
If you don't have a micro SD card reader, get a micro to SD card converter.
![](sd2.png)

We need to format the SD card and make sure it has correct partition before we copy kernel images onto it.
* SD card should have partition type `Master Boot Record` (MBR)
* SD card should have one partition of type `FAT32`

After SD card is prepared, simply copy all five files/folder in the image to SD card
1. boot/
2. bzImage
3. core-image-minimal-initramfs-clanton.cpio.gz
4. grub.efi
5. image-full-galileo-clanton.ext3

If, for some reason, networking is not availale, you can [use a FTDI232 TTL 3.3V cable](https://communities.intel.com/message/247258) with an appropriate terminal software, such as [PuTTY](http://www.putty.org/) in Windows, to connect to the serial console of Galileo. When Galileo boots up, you can see the options

![](fig4.4.01-boot.png)

select items on the grub menu

![](fig4.4.02-boot-menu.png)

and go through the whole booting process to see if there is any issue.

![](fig4.4.03-boot-finish.png)

## Example: Debian 7.8
To demontrate how it is done on Debian 7.8, let's use a GUI Disk utility: `Applications/Accessories/Dist Utility`

![](deb0.png)

1. Select Unmount Volume, then Format Drive, and format the SD card into `Master Boot Record`. If you get an error during this process, try first using the Format Volume option before proceeding with the remainder of the steps.
![](deb1.png)
![](deb2.png)

2. Select Add Partition, then add a `FAT` partition
![](deb3.png)
![](deb4.png)

3. Now you can copy everything to SD card
```
cp -rL boot/ bzImage core-image-minimal-initramfs-clanton.cpio.gz grub.efi image-full-galileo-clanton.ext3 /media/clantonCF/
```

# Example: Mac Yosemite
1. Insert your SD card
![](sd1.png)

2. Open Disk Utility
![](mac1.png)

3. Format the SD card
![](mac2.png)

4. Copy files to SD card
![](mac3.png)
