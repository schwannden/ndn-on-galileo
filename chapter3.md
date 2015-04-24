# The Lazy Ride
```
git clone https://github.com/schwannden/ndn-in-one
cd ndn-in-one
./sdk2Galileo -m get
tar -xzvf image.tar.gz
cd image
cp -r * /media/SDcard
```
* The image is the end result of [Building Galileo Image on Debian](building_galileo_image_on_debian.md).
* /media/SDcard should be the path where you mount your SD card.
  * SD card should have partition type `master boot record`
  * SD card should have one partition of type `FAT 32`

To copy everything to Galileo, and configure Galileo, use the script `sdk2Galileo.sh` in `ndn-in-one` folder.
```
cd ndn-in-one
./sdk2Galileo -a
./sdk2Galileo -m configure
```