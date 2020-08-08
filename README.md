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

You must decide on two IP address subnets for your ATAK network. One subnet will be for the "wlan0" short range wifi network, and IP addresses will be assigned by the Pi to your end user devices (phone, tablet, etc). The other subnet will be for the "tnc0" long range radio link and you must manually specify this IP address on each Pi. Each Pi should be given it's own unique IP address for tnc0. In the provided files the IP range of 10.10.10.x is used for the wlan0 interface. The IP range of 10.99.99.x is used for the tnc0 interface.


#
