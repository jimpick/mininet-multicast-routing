# mininet-multicast-routing

This is a mininet setup based on:

* https://fojta.wordpress.com/2014/09/24/troubleshooting-multicast-with-linux/
* https://github.com/troglobit/smcroute
* https://github.com/mininet/mininet/blob/master/examples/linuxrouter.py

# Requirements

* Ubuntu 16.04 LTS
* apt install mininet
* sudo apt-get install openvswitch-testcontroller
* sudo ln -s /usr/bin/ovs-testcontroller /usr/bin/controller

(see https://stackoverflow.com/questions/21687357/mininet-ovs-controller-can-t-be-loaded-and-run)

Install smcroute from source:

```
./configure --prefix=/opt/smcroute --sysconfdir=/etc --localstatedir=/var
make
sudo make install
```

# Running demo

```
$ sudo ./multicastdemo.py
*** Creating network
*** Adding controller
*** Adding hosts:
h1 h2 h3 r0
*** Adding switches:
s1 s2 s3
*** Adding links:
(h1, s1) (h2, s2) (h3, s3) (s1, r0) (s2, r0) (s3, r0)
*** Configuring hosts
h1 h2 h3 r0
*** Starting controller
c0
*** Starting 3 switches
s1 s2 s3 ...
*** Routing Table on Router:
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
10.0.0.0        *               255.0.0.0       U     0      0        0 r0-eth3
172.16.0.0      *               255.240.0.0     U     0      0        0 r0-eth2
192.168.1.0     *               255.255.255.0   U     0      0        0 r0-eth1
*** Starting CLI:
mininet> xterm r0 h1 h2 h3
mininet>
```

In r0 xterm:

```
g# /opt/smcroute/sbin/smcroutectl -I smcroute-r0 show
ROUTE (S,G)                        INBOUND             PACKETS      BYTES OUTBOUND                                     
(*, 239.0.0.3)                     r0-eth3          0000000000 0000000000  r0-eth1 r0-eth2
(*, 239.0.0.2)                     r0-eth2          0000000000 0000000000  r0-eth1 r0-eth3
(*, 239.0.0.1)                     r0-eth1          0000000000 0000000000  r0-eth2 r0-eth3
```

In h1 xterm:

```
# ping -t 2 239.0.0.1
PING 239.0.0.1 (239.0.0.1) 56(84) bytes of data.
64 bytes from 192.168.1.100: icmp_seq=1 ttl=64 time=0.009 ms
64 bytes from 10.0.0.100: icmp_seq=1 ttl=63 time=9.36 ms (DUP!)
64 bytes from 172.16.0.100: icmp_seq=1 ttl=63 time=9.70 ms (DUP!)
64 bytes from 192.168.1.100: icmp_seq=2 ttl=64 time=0.018 ms
64 bytes from 10.0.0.100: icmp_seq=2 ttl=63 time=1.68 ms (DUP!)
64 bytes from 172.16.0.100: icmp_seq=2 ttl=63 time=2.14 ms (DUP!)
# ip maddr
# /opt/smcroute/sbin/smcroutectl -I smcroute-h1 join h1-eth0 239.0.0.1
# /opt/smcroute/sbin/smcroutectl -I smcroute-h1 leave h1-eth0 239.0.0.1
# /opt/smcroute/sbin/smcroutectl -I smcroute-h1 show
```

Hosts h2 and h3 are configured similarly

* 239.0.0.1 is set up for sending from h1
* 239.0.0.2 is set up for sending from h2
* 239.0.0.3 is set up for sending from h3

