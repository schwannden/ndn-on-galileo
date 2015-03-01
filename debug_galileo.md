# Debug Galileo

Mostly it is related to connection problem.

### DHCP Server Mis-configuration
Just connect a normal device (your cell phone, laptop) to your DHCP server. If it works, your DHCP server is fine.

### Network Problem
If you think maybe it is the network problem, the routing process is wrong, try assign Galileo a static IP to make things simpler. Mount the image's file system on your host by `mount -t ext3 -o loop image-full-galileo-clanton.ext3 /tmp/sdcard` and edit `/etc/network/interfaces` to assign `eth0` a static IP and gateway. They manually set up path from your host by adding static arp entry `arp -s static-ip-of-galileo mac-of-galileo`. If all these does not work, your image could be corrupted.

### Corrupted Image
1. Are you using partition type `master boot record` with a `FAT32` partition?
2. Did you rename the files corrected? Maybe you delete one more character?
3. Did you copy the images from the same build? For example, every build updates `image-full-galileo-clanton.ext3`, and `core-image-minimal-initramfs-clanton.cpio.gz`. Combining two files from different build might not work.