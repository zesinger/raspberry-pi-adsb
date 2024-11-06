Raspberry Pi ADS-B Base Station for OpenSky Network
===================================================

# Introduction

This guide explains how to assemble a simple ADS-B base station using Raspberry Pi and cheap off-the-shelf components
and how to connect your station to the [OpenSky Network](https://opensky-network.org/).

The proposed base station will be able to receive and decode transponder signals from planes in the radius of up to 200-300 km.

![DUMP1090 Screenshot](images/dump1090.png)

You will also be able to feed your data into [OpenSky Network](https://opensky-network.org/) and other networks and use it in local applications like [PlanePlotter](http://www.coaa.co.uk/planeplotter.htm).

![OpenSky Network Screenshot](images/osn.png)

Instructions below are based on the [ADS-B using dump1090 for the Raspberry Pi](http://www.satsignal.eu/raspberry-pi/dump1090.html) guide.

This guide assumes installation in a home network with internet connection via router like FRITZ!Box.

# Prerequisites

* The most important prerequisite is place with clear view of the sky where you could place the antenna.
* If you want to feed your data into [OpenSky Network](https://opensky-network.org/), you must be able to open and forward ports on your router.
* [OpenSky Network](https://opensky-network.org/) needs a static host name or IP address to connect to your base station.
If you don't have a static IP address (you normally don't), your router must support dynamic DNS using a provider like [DuckDNS.org](https://duckdns.org) or [No-IP.com](http://www.noip.com/).
* Your router must support WiFi so that you could connect Raspberry Pi to the network wirelessly.

# Shopping list

![Shopping list](images/parts-labeled.jpg)

* Raspberry Pi set
  * (1) Raspberry Pi 2 Model B [amazon.de](https://www.amazon.de/dp/B01CPGZY3O)
  * (2) Memory Card 32GB MicroSDHC Class 10, SD adapter [amazon.de](https://www.amazon.de/dp/B01CPGZY3O)
  * (3) WLAN Adapter [amazon.de](https://www.amazon.de/gp/product/B00416Q5KI)
  * (4) USB extension cable [amazon.de](https://www.amazon.de/dp/B000NWS55M)
  * (5) Power supply, heatsinks, case [amazon.de](https://www.amazon.de/gp/product/B00UCSO9G6)
* Receiver
  * (6) ADS-B Antenna with *N Female* Connector [ebay.de](http://www.ebay.de/itm/Antenna-ads-b-collinear-great-gain-for-usb-dongle-flightbox-/291800588117)
  * (7) Cable *N Male - SMA Male*  [amazon.de](https://www.amazon.de/gp/product/B0152WWEUO)
  * (8) (Optional) Band-pass Filter *SMA Female - SMA Male* [amazon.de](https://www.amazon.de/gp/product/B010GBQXK8)
  * (9) (Optional) Cable *SMA Female - MCX Male* [amazon.de](https://www.amazon.de/gp/product/B00V4PS1L0)
  * (10) USB RTL-SDR Receiver [amazon.de](https://www.amazon.de/gp/product/B00VZ1AWQA)

We'll assume that you already have a PC with microSD or SD card reader, monitor with HDMI connection and USB keyboard and mouse needed for installation and that you don't have to order the additionally.
  
## Notes on the shopping list

* You will need a 2A power supply for Raspberry Pi.
* Make sure memory card package includes the SD adapter so that you could use this card in a normal PC (which usually has a SD card reader but not the microSD.
* MicroSDHC Class 10 (high speed class) is highly recommended.
* If you connect the receiver to one of the USB port on Raspberry Pi directly, it will cover other ports. You can avoid this using the USB extension cable.
* You'll need a coaxial cable to connect your antenna to your USB RTL-SDR receiver. The shopping list above assumes an antenna with the N Female connector. The recommended [NooElec NESDR Mini 2+](https://www.amazon.de/gp/product/B00VZ1AWQA) receiver has an MCX Female socket, so you basically need to connect MCX Female to N Female. Such cables (*MCX Male - N Male*) are quite rare, a much better option is a *N Male - SMA Male* plus *SMA Female - MCX Male* combination.
* The *N Male - SMA Male* plus *SMA Female - MCX Male* combination also using the [FlightAware band-pass filter](https://www.amazon.de/gp/product/B010GBQXK8) which reduces the noise and increases the number of the processed ADS-B messages. This filter has the *SMA Female* input and *SMA Male* output so it can be inserted between to cables (from antenna and to to receiver). If you want to spare the filter, just connect two cables directly.
* The [band-pass filter](https://www.amazon.de/gp/product/B010GBQXK8) is optional, but it is highly recommended. It drastically increases the number of messages.
* The [NooElec NESDR Mini 2+](https://www.amazon.de/gp/product/B00VZ1AWQA) includes a small *SMA Female - MCX Male* adapter, so you don't necessarily need the *SMA Female - MCX Male* cable. However, if you use the [FlightAware band-pass filter](https://www.amazon.de/gp/product/B010GBQXK8), it is highly recommended to get *SMA Female - MCX Male* cable as well to add flexible connection between the filter and the receiver. Otherwise the filter is connected rigidly into the small MCX socket and as a potential to break it.

# Overview of the installation steps

* Assemble the Raspberry Pi
* Connect the components
* Setup the Raspberry Pi
  * Prepare the SD card with the Raspbian operating system
  * Perform the Raspbian installation
  * Build and install drivers for the RTL-SDR receiver
  * Build and install the `dump1090` decoder
* Connect your base station to the OpenSky Network
  * Set up dynamic DNS for your Raspberry Pi
  * Make the port `30005` port accessible from the internet
  * Create an OpenSky Network account
  * Configure a new sensor in OpenSky Network

# Assemble the Raspberry Pi

* Attach heatsinks to chips
* Fix the Raspberry Pi to the case using screws, close the case

# Connect the components

![Assembled components](images/assembled.jpg)

Antenna:

* Antenna *N Female*
* Cable *N Male - SMA Male*
* (Optional) Band-pass filter *SMA Female - SMA Male* 
* (Optional) Cable *SMA Female - MCX Male*, alternatively use the *SMA Female - MCX Male* adapter
* USB RTL-SDR Receiver
* USB extension cable
* ... to Raspberry Pi

WLAN Adapter:

* WLAN Adaper
* USB extension cable
* ... to Raspberry Pi

Also connect keyboard and mouse.

# Set up the Raspberry Pi

Here is the easiest way to install the OpenSky feeder on a Raspberry Pi 2B+ with Windows.
This is based on the OpenSky receiver kit readme available here https://github.com/openskynetwork/receiver-kit.

- Create an account on https://opensky-network.org/
- Erase all the partitions from a SD card (>32Go) using windows disk manager (Windows key + R, type "diskmgmt.msc" + Enter). Be sure to modify the SD card!
- Download and install Win32DiskImager
- Download the last image available here https://github.com/openskynetwork/receiver-kit?tab=readme-ov-file#updating-your-receiver
- Use Win32DiskImager to write it on the SD card
- Edit the file opensky\config.txt from the Fat32 partition, set the latitude, longitude and altitude of the device, remove the "#" before "Username" and type your opensky-network username
- Put the SD card in the Raspberry Pi, connect a keyboard, a Wifi USB key and a screen
- Start it, wait for the installation to complete
- When prompted, enter the login "pi" and the password "oskydefault" (caution: QWERTY keyboard)
- Change this password typing "passwd"
- Type "sudo touch ssh"
- Type "sync"
- Type "nmtui" and configure the Wifi connection
- Type "reboot"

It should work...


# Credits

* This guide is largely based on the [ADS-B using dump1090 for the Raspberry Pi](http://www.satsignal.eu/raspberry-pi/dump1090.html) guide
* This guide uses RTL-SDR drivers from the [osmocomSDR](http://sdr.osmocom.org/trac/wiki/rtl-sdr) project
* We use the [Malcolm Robb's fork](https://github.com/MalcolmRobb/dump1090) of the [`dump1090`](https://github.com/antirez/dump1090) software

# License

This guide is published under the [MIT license](LICENSE).
