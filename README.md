# PiATAK
Services, timers, and configuration for PiATAK.

This setup will allow position updates, broadcast map markers, and geochat messages to be sent over radio.

# Overview Of Components

**HARDWARE**

1x Pi Zero W

1x dual 18650 battery holder

1x USB hub

1x USB sound card

1x PTT circuit (custom made, see the hardware folder for additional info)

2x 3.5mm audio jacks

1x snap on ferrite

1x 3.5mm trs to 3.5mm trs cable

1x 2.5mm trs to 3.5mm trs cable

**SCRIPTS**

direwolf.service: starts direwolf which converts data into AFSK and sends it out the audio device, and also listens to the mic input for AFSK audio to convert back to data.

tncattach.service: starts tncattach which makes direwolf available as a tcp/ip network interface.

multicastwlan0.service: adds the routing info for multicast packets on the wlan0 network interface.

multicasttnc0.service: adds the routing info for multicast packets on the tnc0 network interface.

socatChatTx.service: Listens to the wlan0 interface for multicast packets created by geochat messages and forwards them out the tnc0 interface.

socatChatRx.service: Listens to the tnc0 interface for multicast packets from geochat messages and forwards them out the wlan0 interface.

socatPositTx.service: Listens to the wlan0 interface for multicast packets created by position updates or broadcast map markers, and forwards them out the tnc0 interface.

socatPositRx.service: Listens to the tnc0 interface for multicast packets from position updates or broadcast map markers, and forwards them out the wlan0 interface.

iptablesBlockPing.service: With SA MULTICAST enabled in ATAK a periodic (approx every 40 seconds) message is sent containing no useful information whatsoever. This is a firewall rule to prevent needlessly cluttering the network with these useless messages.

# Required Software

Download and install hostapd via apt

    sudo apt install hostapd

Download and install dnsmasq via apt

    sudo apt install dnsmasq

Download and install socat via apt

    sudo apt-get install socat

Download and install libasound2-dev via apt

    sudo apt-get install libasound2-dev

Download direwolf from github, then compile and install

    git clone https://www.github.com/wb2osz/direwolf.git
    
    cd direwolf
    
    make
    
    sudo make install
    
    make install-conf
    
    make install-rpi
    
    cd ~
    
    cp sdr.conf sdr_orig.conf

Download tncattach from github, then compile and install

    git clone https://www.github.com/markqvist/tncattach.git
    
    cd tncattach
    
    make
    
    sudo make install
    
    cd ~

# Installation

**SERVICES FOLDER**

  Copy all .service and .timer files to /etc/systemd/system/

**CONFIG FOLDER**

Copy sdr.conf to /home/pi/

Copy dhcpcd.conf to /etc/

Copy dnsmasq.conf to /etc/

Copy hostapd.conf to /etc/hostapd/

# Configuration

You must decide on two IP address subnets for your ATAK network. One subnet will be for the "wlan0" short range wifi network, and IP addresses in this range will be assigned by the Pi to your end user devices (phone, tablet, etc). The other subnet will be for the "tnc0" long range radio link and you must manually specify this IP address on each Pi. Each Pi should be given it's own unique IP address for tnc0. In the provided files the IP range of 10.10.10.x is used for the wlan0 interface, and the IP range of 10.99.99.x is used for the tnc0 interface. You may use these or change them according to your preference. You may use the exact same wlan0 IP addresses across every Pi but you must edit the tnc0 interface IP address and assign a unique address for each individual Pi.

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

**SERVICE CONFIGURATIONS**

If you have left the IP address subnets as their defaults then you only need to edit the socatChatRx.service file and the socatPositRx.service file.

    sudo nano /etc/systemd/system/socatChatRx.Service
    
    sudo nano /etc/systemd/system/socatPositRx.Service
    
Find the following section and adjust the second IP address (the one after the colon : ) to match the IP address you specified in the tncattach.service file. This will be unique for each Pi.

ip-add-membership=224.10.10.1:10.99.99.1

If you have changed from the default IP address subnets you must edit all four of the socat .service files and change the IP addresses to match what you have specified. Do not change any of the IP address beginning with 224 or 239.

# Automatic Startup

Lets make everything start by default when the Pi boots up.

    sudo rfkill unblock wlan
    
    sudo systemctl unmask hostapd
    
    sudo systemctl enable hostapd
    
    sudo systemctl enable direwolf.timer
    
    sudo systemctl enable tncattach.timer
    
    sudo systemctl enable iptablesBlockPing.timer
    
    sudo systemctl enable multicastwlan0.timer
    
    sudo systemctl enable multicasttnc0.timer
    
    sudo systemctl enable socatChatTx.timer
    
    sudo systemctl enable socatPositTx.timer
    
    sudo systemctl enable socatChatRx.timer
    
    sudo systemctl enable socatPositRx.timer
    
The timers cause all of the services to start a specific number of seconds after boot. The first service to start is direwolf which launches approx 10 seconds after a Pi Zero W finishes booting to the desktop. A faster Pi would probably boot faster and therefore the OnBootSec could be decreased. These timings are very slow and should give a Pi Zero W plenty of time to start up. In total it takes roughly 3 minutes from power on until all the services are started.

The services are started in the following order with the following boot timing:

1) direwolf @ 80sec

2) tncattach @ 95sec

3) multicastwlan0 @ 100sec

4) socatChatTx @115 sec

5) socatPositTx @ 120sec

6) multicasttnc0 @ 124sec

7) socatChatRx @ 130sec

8) socatPositRx @ 140sec

9) iptablesBlockPing @140sec

If a radio is turned on and hooked up it will transmit a fair bit of AFSK data at step two, then a smaller burst of AFSK data at step 7 and at step 8. If there are users already on ATAK using the network it is advisable to keep the radio powered off or unhooked until after all services have started (approx 3 minutes) to avoid unnecessary radio traffic.

# ATAK App Configuration

The frequency of position updates must be reduced. Go to:

Settings > Show All Preferences > Device Preferences > Reporting Preferences

Change the Dynamic Reporting Rate for all (Unreliable) categories to much larger values. Due to limited network bandwidth the Maximum rate should probably be no less than 60, with correspondingly larger numbers for the Minimum and Stationary rate.

# Known Issues

Only data broadcast over UDP will be sent over the radio. Anything without a "broadcast" option is unlikely to be supported. Direct chat is not supported but the All Chat Rooms does function correctly.

It is possible for messages to be lost if multiple users transmit simultaneously, or if one user attempts to rapidly transmit a significant number of items. Direwolf attempts to avoid transmitting over other radio traffic but seems to throw away packets if it cannot transmit them within a fairly short window.

ATAK ignores the maximum reporting rate setting when large changes in speed or direction are involved, for example a car accelerating, breaking, or going around a corner. In these situations ATAK will send multiple rapid position updates. A single moving vehicle can completely saturate the radio link and prevent any other nodes from sending position or chat information.
#
