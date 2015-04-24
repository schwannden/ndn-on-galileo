# Boot From SD Card

## Getting image
You can get the Galileo's linux image by following [Building Galileo Image on Debian](building_galileo_image_from_debian.md), or download it [here]('http://sourceforge.net/projects/ndn-in-one/files/image.tar.gz), they are the same image.

## Preparing SD card
If you don't have a micro SD card reader, get a micro to SD card converter. We need to format the SD card and make sure it has correct partition before we copy kernel images on it.
* SD card should have partition type `master boot record`
* SD card should have one partition of type `FAT 32`

After SD card is prepared, simply copy all five files in the image to SD card


```
cp -r boot/ bzImage core-image-minimal-initramfs-clanton.cpio.gz grub.efi image-full-galileo-clanton.ext3 /media/clantonCF/
```