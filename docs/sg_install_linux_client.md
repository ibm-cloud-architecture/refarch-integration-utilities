
## Configuring Static IP Address

Basic network configuration and hostname on a Ubuntu system are stored in several files which must be edited to create a working configuration:
* /etc/hosts
* /etc/hostname  was set to brownutility
* /etc/network/interface: This file describes the network interfaces available on your system and how to activate them. Below is the original file
```
source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto ens160
iface ens160 inet dhcp
```
To set static IP here are some example of settings
```
# The primary network interface
auto ens160
iface ens160 inet static
   address 172.16.50.9
   netmask 255.255.0.0
   network 172.16.0.0
   dns-nameservers 172.16.0.11 172.16.0.17
   dns-domain csplab.local
   dns-search csplab.local
   gateway 172.16.255.250
```


```
$ ifconfig -a
```
