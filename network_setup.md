# Network Setup
Example topology

```

              ==============================    ====================================    ===================
              |     router at my home      |    |      My PC: Fedora 20            |    |   Intel Galileo |
              ==============================    ====================================    ===================
Internet----->|<-Face A    Face B--------->|<-->|<-Face wlp0s29u1u3  Face p34p1--->|<-->|<-Face eth0      |   
              |  x.x.x.x   ip:192.168.1.1  |    |  ip: 192.168.1.32  ip: 10.0.0.1  |    |  ip:  10.0.0.2  |
              |                            |    |         gw: 192.168.1.1          |    |  gw:  10.0.0.1  |
              |                            |    |    runs DHCP server on eth0      |    |                 |   
              ==============================    ==============================+=====    ===================
              
```

Example DHCP configuration file for Debian
```
# the nameservers taht will be responded to DHCP client.
option domain-name-servers 8.8.8.8, 8.8.4.4;
# define a subnet with no ip range, because all ip are
# bind to a specific host as spesified in host statement
subnet 10.0.0.0 netmask 255.255.255.0 { 
  option routers 10.0.0.1; # default route for DHCP client
}
# bind the only machine to ip address 10.0.0.2
host clanton {
  hardware ethernet 98:4F:EE:01:A1:EC;
  fixed-address 10.0.0.2; #ip for DHCP client with above MAC
}
```

Getting Galileo on Internet
```
# "IP forwarding" is a synonym for "routing." It is called
# "kernel IP forwarding" because it is a feature of the Linux kernel.
# A router has multiple network interfaces. If traffic comes in on 
# one interface that matches a subnet of another network interface, 
# a router then forwards that traffic to the other network interface.
# So for example, ip_forward will allow Galileo to ping wlp0s29u1u3
sysctl -w net.ipv4.ip_forward=1 
# turn on NAT on wlp0s29u1u3 since this is the face that willbe relaying traffic from 10.0.0.0/24
# after truning on NAT, Galileo should be able to ping all live ips from the Internet
iptables -t nat -A POSTROUTING -o wlp0s29u1u2 -j MASQUERADE

```
