We will use dnsmasq package for this purpose because it is combined DHCP and DNS server and also easy to configure.

If you want something a little more 'heavyweight', you can use the  isc-dhcp-server and bind9 packages for DHCP and DNS respectively, but for our purposes, dnsmasq works just fine.

`sudo apt-get install dnsmasq`

We need to configure interfaces. We will assign a static IP address to eth0 which will be used as gateway. 
Open the interfaces file:

`sudo nano /etc/network/interfaces`

Edit the eth0 section like this:

```
allow-hotplug eth0  
iface eth0 inet static  
    address 192.168.2.1
    netmask 255.255.255.0
    network 192.168.2.0
    broadcast 192.168.2.255
```
Next, we will configure dnsmasq. The shipped dnsmasq config file contains a lot of information on how to use it. 
So, I will advise to move it and create a new one.

`sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig`

`sudo nano /etc/dnsmasq.conf`

Paste the following into the new file:
```
interface=eth0      # Use interface eth0  
listen-address=192.168.2.1 # listen on  
# Bind to the interface to make sure we aren't sending things 
# elsewhere  
bind-interfaces
server=8.8.8.8       # Forward DNS requests to Google DNS  
domain-needed        # Don't forward short names  
# Never forward addresses in the non-routed address spaces.
bogus-priv
# Assign IP addresses between 192.168.2.2 and 192.168.2.100 with a
# 12 hour lease time
dhcp-range=192.168.2.2,192.168.2.100,12h
```
Edit the /etc/sysctl.conf file to enable packet forwarding.

`sudo nano /etc/sysctl.conf`

Remove the # from the beginning of the line containing `net.ipv4.ip_forward=1`.
This will enable packet forwarding on next reboot. But if you want to try it right now without reboot then do this.

`sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"`

We also need to share RPi’s internet connection with the devices connected over Wi-Fi. 
We will configure a NAT between eth0 and wlan0:
```
sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE  
sudo iptables -A FORWARD -i wlan0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT  
sudo iptables -A FORWARD -i eth0 -o wlan0 -j ACCEPT
```
However, we need these rules to be applied every time we reboot the Pi, so run:

`sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"`

to save the rules to the file /etc/iptables.ipv4.nat. Now we need to run this after each reboot, 
so open the /etc/rc.local file with:

`sudo nano /etc/rc.local`

and just above the line exit 0, add the following line:
`iptables-restore < /etc/iptables.ipv4.nat`

And that’s all! Now just Reboot your RPi and you will be able to access Internet

`sudo reboot`
