
# rasbian as a network controler
some links as references

* http://oversimple.fr/point-dacces-wifi-avec-un-raspberry-pi/
* http://www.themagpi.com/issue/issue-11/article/turn-your-raspberry-pi-into-a-wireless-access-point/
* http://sirlagz.net/2012/08/09/how-to-use-the-raspberry-pi-as-a-wireless-access-pointrouter-part-1/
* http://elinux.org/RPI-Wireless-Hotspot
* http://blog.chaucery.com/2013/02/raspberry-pi-wireless-bridge.html
----
## install nedeed tools

    $ sudo apt-get install rfkill hostapd hostap-utils iw dnsmasq bridge-utils 

## Configure the interface

#### Setup a static bridge interface

you need to edit your /etc/network/interfaces


    auto lo
    iface lo inet loopback
    iface eth0 inet manual
    iface wlan0 inet manual

    auto br0
    iface br0 inet static
    	address 192.168.5.1
    	gateway 192.168.5.2
    	netmask 255.255.255.0
    	bridge-waitport 5
    	bridge-ports eth0
    	bridge-stp off
    	bridge-fd 0
    	#bridge-waitport 2


Then restart your interface using:

    $ sudo ifdown br0
    $ sudo ifup br0

First up, I had to modify the file /etc/default/hostapd. The DAEMON_CONF variable was not configured, so I pointed it to a configuration file that I was about to create.
DAEMON_CONF="/etc/hostapd/hostapd.conf"

#### Now configure hostap. Edit /etc/hostapd/hostapd.conf (it may not already exist but this will create it anyway) to look like this:


    interface=wlan0
    ctrl_interface=/var/run/hostapd
    driver=nl80211
    bridge=br0
    ssid=ola-pi
    channel=8
    

The settings are pretty obvious, ‘driver’ being the only exception. Just leave it as it is but change the other values as you see fit, though these should do for an initial test. With the “bridge=br0″ statement, Hostapd will bridge the wlan0 interface to br0 automatically. One thing to bear in mind is that hostapd seems to be very literal about reading its configuration file so make sure you have no trailing spaces on the end of any lines! Restart the hostapd service with:


    $ sudo hostapd -d  /etc/hostapd/hostapd.conf 

just for test, if everything is OK then ^C to stop it and


    $ sudo service hostapd restart


All being well you should now see ‘ola-pi’ as a wireless network, though a little more work needs to be done to connect to it yet.

#### Configuring the DHCP server

The final step is to configure dnsmasq so you can obtain an IP address from your new Pi-point. Edit your /etc/dnsmasq.conf file to look like this:

    # Never forward plain names (without a #dot or domain part)
    domain-needed

    # Only listen for DHCP on wlan0
    interface=br0

    # create a domain if you want, comment #it out otherwise
    #domain=

    # Create a dhcp range on your /24 br0 network with 12 hour lease time
    dhcp-range=192.168.5.100,192.168.5.200, 255.255.255.0,12h

Remember to change the dhcp-range option for the network IP range you’re using.
For the configuration to take effect, restart dnsmasq by typing:

    $ sudo service dnsmasq restart

Now you should be able to connect to your new Pi-point and get a proper IP address from it.


-----------------------------------
Configuring the wireless interface

Your wireless interface must be set statically for hostap. Therefore you need to edit your /etc/network/interfaces file to look like this:


    auto lo
    iface lo inet loopback
    iface eth0 inet dhcp
    iface wlan0 inet static
    address 192.168.5.1
    netmask 255.255.255.0


Then restart your wireless interface using:

    $ sudo ifdown wlan0
    $ sudo ifup wlan0

This should hopefully go smoothly without errors. The address of 192.168.5.1 (pi, get it!?) should not be the same as the network connected to eth0. My main LAN is 192.168.0.0/24 and the wireless interface should be a different network. This adds a simple layer of security in case you’re configuring an open access point (as I was) and allows you to firewall one network from the other far more easily.


First up, I had to modify /etc/default/hostapd. The DAEMON_CONF variable was not configured, so I pointed it to a configuration file that I was about to create.
DAEMON_CONF="/etc/hostapd/hostapd.conf"


Now to configure hostap. Edit /etc/hostapd/hostapd.conf (it may not already exist but this will create it anyway) to look like this:


    interface=wlan0
    ctrl_interface=/var/run/hostapd
    driver=nl80211
    ssid=ola-pi
    channel=8


The settings are pretty obvious, ‘driver’ being the only exception. Just leave it as it is but change the other values as you see fit, though these should do for an initial test. One thing to bear in mind is that hostapd seems to be very literal about reading its configuration file so make sure you have no trailing spaces on the end of any lines! Restart the hostapd service with:

    $ sudo hostapd -d  /etc/hostapd/hostapd.conf 

just for test, if everything is OK then ^C to stop it and

    $ sudo service hostapd restart

All being well you should now see ‘ola-pi’ as a wireless network, though a little more work needs to be done to connect to it yet.

Configuring the DHCP server

The final step is to configure dnsmasq so you can obtain an IP address from your new Pi-point. Edit your /etc/dnsmasq.conf file to look like this:

    # Never forward plain names (without a #dot or domain part)
    domain-needed

    # Only listen for DHCP on wlan0
    interface=wlan0

    # create a domain if you want, comment #it out otherwise
    #domain=

    # Create a dhcp range on your /24 wlan0 #network with 12 hour lease time
    dhcp-range=192.168.5.100,192.168.5.200, 255.255.255.0,12h

Remember to change the dhcp-range option for the network IP range you’re using.
For the configuration to take effect, restart dnsmasq by typing:

    $ sudo service dnsmasq restart

Now you should be able to connect to your new Pi-point and get a proper IP address from it.

add 

    # add wlan0 to bridge
    brctl addif br0 wlan0

at the end of file /etc/rc.local to be bridged at startup

voir aussi http://bastienceriani.fr/?p=53 pour une connection automatique


config d'une liason wifi en mode client

$ sudo wpa_supplicant -iwlan1 -c/etc/wpa_supplicant/wpa_supplicant.conf -d
$ wpa_cli (ajouter les commandes pour se connecter)
$ sudo dhclient wlan1

$ sudo route add default gw 192.168.0.9
