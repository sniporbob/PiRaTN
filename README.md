# PiATAK
Services, timers, and configuration for PiATAK

# Required Software

Download direwolf from github, then compile and install

Download tncattach from github, then compile and install

Download and install hostapd via apt

Download and install dnsmasq via apt

Download and install socat via apt

# Installation

**SERVICES FOLDER**

  Copy all .service and .timer files to /etc/systemd/system/

**CONFIG FOLDER**

Copy sdr.conf to /home/pi/

Copy dhcpcd.conf to /etc/

Copy dnsmasq.conf to /etc/

Copy hostapd.conf to /etc/hostapd/

# Configuration

You must decide on two IP address subnets for your ATAK network. One subnet will be for the "wlan0" short range wifi network, and IP addresses in this range will be assigned by the Pi to your end user devices (phone, tablet, etc). The other subnet will be for the "tnc0" long range radio link and you must manually specify this IP address on each Pi. Each Pi should be given it's own unique IP address for tnc0. In the provided files the IP range of 10.10.10.x is used for the wlan0 interface, and the IP range of 10.99.99.x is used for the tnc0 interface. You may use these or change them according to your preference. You may use the same wlan0 IP addresses across all Pis but you must edit the tnc0 interface IP address and assign a specific address for each individual Pi.

Example setup:

Pi#1: wlan0 with IP range 10.10.10.x, tnc0 with static IP address 10.99.99.1

Pi#2: wlan0 with IP range 10.10.10.x, tnc0 with static IP address 10.99.99.2

Following this readme without changing any IP addresses will set up Pi#1 exactly as this example. To set up your second Pi following this readme, just replace all instances of 10.99.99.1 with 10.99.99.2

--------
**WLAN0 CONFIGURATION**

====Run the following command to edit hostapd.conf====

    sudo nano /etc/hostapd/hostapd.conf

Edit the following values as appropriate:

country_code: set to your country code

ssid: set to whatever you want your wireless network on this Pi to be called

channel: set to whatever wifi channel you want. Recommend only using 1, 6, and 11 as these are the non-overlapping channels

wpa_passphrase: set to your desired wifi password

====Run the following command to edit dhcpcd.conf====

    sudo nano /etc/dhcpcd.conf
    
Edit the following IP address under "interface wlan0" as appropriate (or leave alone if you want keep the default range of 10.10.10.x):

static ip_address=10.10.10.1/24

Note this IP address can be exactly the same on each Pi.

====Run the following command to edit dnsmasq.conf====

    sudo nano /etc/dnsmasq.conf
    
Edit the first two IP addresss as appropriate to fall within the range specified in dhcpcd.conf

dhcp-range=10.10.10.10,10.10.10.200,255.255.255.0,24h

In this example the Pi will assign IP addresses between the values of 10.10.10.10 and 10.10.10.200.

**DIREWOLF CONFIGURATION**

====Run the following command to edit sdr.conf====

    nano ~/sdr.conf
    
Edit the following line to be your amateur radio callsign. By default this will not be transmitted. See direwolf documentation
if you desire to enable this for use on the ham bands.

MYCALL AB1CDE-1

Edit the following line to match your sound device.

ADEVICE plughw:1,0

Note that the assigned number for the sound device can change between restarts. When the Pi is plugged in to an HDMI monitor with speaker, typically the monitor/speaker is assigned plughw:1,0 and the USB sound device is assigned plughw:2,0. When there is no HDMI plugged in the USB sound device is typically plughw:1,0.

To list your current sound devices you can run the command: aplay -l (lowercase L)

DWAIT, TXDELAY, and TXTAIL have been adjusted for use with Baofengs. Higher quality radios will be able to use tighter timing (smaller numbers).
See direwolf documentation for details on these settings.

**TNC0 CONFIGURATION**

====Run the following command to edit tncattach.service====

    sudo nano /etc/systemd/system/tncattach.service
    
Edit the following IP address to be within the IP address range you plan to use for the tnc0 long range radio link.

--ipv4 10.99.99.1/24

Note THIS MUST BE UNIQUE FOR EACH Pi. For example Pi#2 could have 10.99.99.2/24, Pi#3 could have 10.99.99.3/24, etc.

#
