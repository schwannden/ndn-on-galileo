# Configure Galileo: Moving Everything
It is recommended that you follow the lazy steps here:
```
git clone https://github.com/schwannden/ndn-in-one
cd ndn-in-one
```
Moving libcryptopp, ndn-cxx, nfd, ndn configuration file to Galileo
```
./sdk2Galileo.sh -a
```
Moving `opkg` configuration file to Galileo, configure `opkg`, install required packages in Galileo. 
```
./sdk2Galileo.sh -m configure
```
Overwriting `nfd-start` and `nfd-stop` by customized version
```
scp nfd-start root@$GalileoIP:/bin
scp nfd-stop  root@$GalileoIP:/bin
```

## Commands explained

## 1. `./sdk2Galileo.sh -a`
This moves all libraries to Galileo. To do this manually

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

## 2. `./sdk2Galileo.sh -m configure`
This configures Galileo. To do this manually:

Because Yocto's Galileo image uses `opkg` as its package management utility, we need to set up `opkg` first. The following tutorial is based on [LJ Chen's tutorial](https://sites.google.com/site/cclljj/resources/notes_galileo), which in turn based on [AlexT's repository](http://alextgalileo.altervista.org/package-repo-configuration-instructions.html).

Edit `/etc/opkg/base-feeds.conf` to add repository. Add following lines to the file:
```
src/gz all     http://repo.opkg.net/galileo/repo/all
src/gz clanton http://repo.opkg.net/galileo/repo/clanton
src/gz i586    http://repo.opkg.net/galileo/repo/i586
```

Update and configure `opkg`
```
opkg update
opkg install --force-overwrite uclibc
```

Now we are in position to install packages on Galileo
```
opkg install vim git
opkg install pkgconfig openssl sqlite3
```
The installation of `vim` and `git` is not necessary, it's just that I can't survive without them.

## 3. Changng the configuration files
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