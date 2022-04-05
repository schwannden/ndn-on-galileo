# All In One
## Getting Image
```
git clone https://github.com/schwannden/ndn-in-one
cd ndn-in-one
./sdk2Galileo.sh -m get image
tar -xzvf image.tar.gz
cd image
cp -r * /media/SDcard
```
* The image is the end result of [Building Galileo Image on Debian](building_galileo_image_on_debian.md).
* /media/SDcard should be the path where you mount your SD card.
* See [Boot From SD Card](boot_from_sd_card.md) for more detail on how to boot Galileo from SD card.

## Setting up NDN
To copy everything to Galileo, and configure Galileo, use the script `sdk2Galileo.sh` in `ndn-in-one` folder.
```
cd ndn-in-one
./sdk2Galileo.sh -a
./sdk2Galileo.sh -m configure
```
* Visit [ndn-in-one](https://github.com/schwannden/ndn-in-one) for more tutorial on using the script
* Visit [The Whole Story](chapter2.md) for the details in each steps
