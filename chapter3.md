# All In One
## Getting Image
```
git clone https://github.com/schwannden/ndn-in-one
cd ndn-in-one
./sdk2Galileo -m get
tar -xzvf image.tar.gz
```
* The image is the end result of [Building Galileo Image on Debian](building_galileo_image_on_debian.md).
* /media/SDcard should be the path where you mount your SD card.
* See [Boot From SD Card](boot_from_sd_card.md) for more detail on how to boot Galileo from SD card.

## Setting up NDN
To copy everything to Galileo, and configure Galileo, use the script `sdk2Galileo.sh` in `ndn-in-one` folder.
```
cd ndn-in-one
./sdk2Galileo -a
./sdk2Galileo -m configure
```
