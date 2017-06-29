
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

## Adding VNC server to remote connection
We found useful to remote connect to the utility server, so the following was done:
* install vnc server:
```
$ sudo apt-get update
$ sudo apt-get install xfce4 xfce4-goodies tightvncserver
$ vncserver
```
After setting the password the X desktop was brownutility:1 so the hostname and first display.

To configure the vnc server:
```
mv ~/.vnc/xstartup ~/.vnc/xstartup.bak
vi xstartup
sudo chmod +x ~/.vnc/xstartup
```

The content of the xstartup file specifies the server to launch XFCE.
```
#!/bin/bash
xrdb $HOME/.Xresources
startxfce4 &
```

Then to help starting the server as service, first add a script `vncserver` under `/etc/init.d`.

```
#!/bin/bash
PATH="$PATH:/usr/bin/"
export USER="brownuser"
DISPLAY="1"
DEPTH="24"
GEOMETRY="1024x768"
OPTIONS="-depth ${DEPTH} -geometry ${GEOMETRY} :${DISPLAY} -localhost"
. /lib/lsb/init-functions
case "$1" in
start)
log_action_begin_msg "Starting vncserver for user '${USER}' on localhost:${DISPLAY}"
su ${USER} -c "/usr/bin/vncserver ${OPTIONS}"
;;
stop)
log_action_begin_msg "Stopping vncserver for user '${USER}' on localhost:${DISPLAY}"
su ${USER} -c "/usr/bin/vncserver -kill :${DISPLAY}"
;;
restart)
$0 stop
$0 start
;;
esac
exit 0
```

Then we need to authorize SSL connection for the localhost port 5901 (5900 +$DISPLAY) to the same port number for the public IP address.
```
$ssh -L 5901:127.0.0.1:5901 -N -f -l brownuser 172.16.50.9
```

Finally add the vncserver as a service so it can start after boot. For that add a file `/etc/systemd/system/vncserver@.service `
```
[Unit]
Description=Start TightVNC server at startup
After=syslog.target network.target

[Service]
Type=forking
User=brownuser
PAMName=login
PIDFile=/home/brownuser/.vnc/%H:%i.pid
ExecStartPre=-/usr/bin/vncserver -kill :%i > /dev/null 2>&1
ExecStart=/usr/bin/vncserver -depth 24 -geometry 1280x800 :%i
ExecStop=/usr/bin/vncserver -kill :%i

[Install]
WantedBy=multi-user.target
```
