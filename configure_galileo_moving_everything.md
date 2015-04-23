# Configure Galileo: Moving Everything


### Moving everything onto Galileo




```
cd $PKG_CONFIG_SYSROOT_DIR
```
Suppose the IP address of your Galileo is `10.0.0.2`

`export GalileoIP=10.0.0.2`

1. `libcryptopp` library and headers
```
scp -r include/cryptopp root@$GalileoIP:/include
scp lib/libcryptopp.so root@$GalileoIP:/lib
```
2. `ndn-cxx` libraries and headers
```
scp -r  usr/include/ndn-cxx root@$GalileoIP:/usr/include
scp usr/lib/libndn-cxx.a root@$GalileoIP:/usr/lib
scp -r  usr/include/ndn-cxx
```
3. `nfd` executable
```
scp -r usr/bin/nfd* root@$GalileoIP:/bin
scp -r usr/bin/ndn* root@$GalileoIP:/bin
```
4. `ndn` configuration files
```
scp -r usr/etc/ndn root@$GalileoIP:/opt/ndn/sysroots/i586-poky-linux-uclibc/usr/etc/
```

### Changng the configuration files
In Galileo, make sure configuration files have the right names
```
cd /opt/ndn/sysroots/i586-poky-linux-uclibc/usr/etc/ndn
cp client.conf.sample client.conf
cp nfd.conf.sample nfd.conf
```
Then some edit to `nfd-start` script is required
```
cd /bin
vim nfd-start
```
Delete everything related to `sudo` and `pgrep`.