# Setting Up Pi Hole with AT&T Fiber's BGW320-505 Router

AT&T Fiber internet service is one of the worst I've encountered, and I'm from the home of Comcast's Xfinity, so that's saying something. They force a lot of bad, [dark patterns](https://en.wikipedia.org/wiki/Dark_pattern) on their users:

* They force you to use their equipment, which is some of the worst on the market;
* They don't let you easily [change your DNS server](https://www.howtogeek.com/897514/why-you-should-change-your-dns-server-today/);
* They monitor your web-surfing through their DNS servers and [sell the data to data brokers](https://www.howtogeek.com/887360/att-sells-your-data-by-default-heres-how-to-opt-out/);
* They don't allow you basic white and black listing of websites, [unless you pay an additional $7.99 monthly fee for their awful app](https://www.att.com/wireless/parental-controls);
* Their default configuration of IPv6 is broken, and prevent devices such as the Meta Quest from properly connecting over WiFi until disabled completely.

I could go on, but you get the point.

## A Partial Solution: the Pi Hole!

Most people are familiar with installing an ad blocker in your web browser or on your phone, such as uBlock or AdBlock Plus. Pi Hole does largely the same thing, but for your entire home network. That means any device that hooks up to your home WiFi (a laptop, an iPad, a phone, even a TV device like Roku) is protected. It also provides a web-based interface where you can block any website you want. This is very handy if you have kids! It blocks ads on YouTube, in the interface for Roku and Amazon Fire, and also blocks these devices logging everything you watch, so they don't get sold to data brokers.

This tutorial will walk you through how to setup a Pi Hole on AT&T Fiber with the standard BGW320-505 router.

## What You Need to Build a Pi Hole

The total cost should be about $45.

* A Raspberry Pi kit, I recommend the affordable [Raspberry Pi Zero W Basic Starter Kit](https://www.amazon.com/dp/B0748MPQT4)
* [A MicroUSB ethernet port](https://www.amazon.com/dp/B08VRXJGYK)
* A MicroSD card (and an adapter to read it on your computer)

## Building the Pi Hole

You can follow along with more [detailed instructions here](https://www.raspberrypi.com/tutorials/running-pi-hole-on-a-raspberry-pi/), but here are the key steps.

* [Download the Raspberry Pi Imager](https://www.raspberrypi.com/software/)
* Click `Choose Device`, and select `Raspberry Pi Zero`
* Click `Choose OS`, and select `Raspberry Pi OS (other)`, and then `Raspberry Pi OS Lite (32-bit)`
* Be sure to choose `Configure Settings` during the installation, and enter these settings:
    * Enter `pi-hole` as the hostname
    * Enter a username and password; you’ll need these later to SSH to the Pi-hole
    * Uncheck the box next to `Configure wireless LAN`, unless you don't have the MicroUSB ethernet port
        * If using WiFi, enter your network SSID (name) and password
* Check the box next to `Enable SSH`, and activate SSH with username and password
    * Make note of the username and password you create, we'll need them to SSH to the Pi-hole!

Once this completes, put the SD card into the Raspberry Pi, and build it. Plug the power into the correct port, and the MicroUSB ethernet dongle into the other. Plug the ethernet cable from your router into the Raspberry Pi, and power it on. Detailed instructions are available at the "details instructions here" link above. Once it boots we can continue:

* SSH to the Pi-hole from your terminal using the username and password you selected during `Configure Settings`.

Follow the instructions to update the operating system and install Pi Hole using the tutorial. After you have installed the operating system on the MicroSD card, and assembled the Raspberry Pi, be sure to use the ethernet cable to connect it to the router. Then continue below.

## Customization for AT&T After Raspberry Pi OS Installation

To make this work, we need to configure your AT&T router to pass DHCP through to the Pi Hole server, and set the Pi Hole with a fixed IP so everything will still work after a power cycle.

### Setting a Fixed IP Address

The AT&T BGW320-505 router uses IP address `192.168.1.254` as the gateway address, and we can configure a fixed IP address for the Pi Hole through the web interface at [http://192.168.1.254](http://192.168.1.254).

1. Click `Home Network`, then `IP Allocation`
2. You should see an entry for `192.168.1.### / pi-hole` on the list; click the `Allocate` button on that entry, and note the IP address number!
3. From the `New allocation` list below, select the IP address noted in Step 2: `Private fixed: 192.168.1.###`
4. Click `Save`
5. Open an SSH terminal to your Pi Hole again, and bring up the NetworkManager configuration tool:
6. Enter the command `sudo nmtui`
7. In the interface, use the cursor arrows and the enter key to select `Edit a connection`, and select `Wired connection 1`.
8. Under `IPv4 CONFIGURATION` (click `<Show>` if necessary):
    * Addresses: `192.168.1.###` (enter the IP address noted in Step 2)
    * Gateway: `192.168.1.254`
    * DNS Server: `1.1.1.1` and `1.0.0.1`
* Click `OK`

```
sudo systemctl restart NetworkManager
```

## Pi-hole Installation

SSH to the Pi-hole system, and run the following command:

```
curl -sSL https://install.pi-hole.net | bash
```

This will take a few minutes to setup, then bring up an interface for installation:

* When warned about needing a static IP address, click Continue to proceed; we’ll deal with this later
* When prompted to choose an upstream DNS provider, choose OpenDNS
* Include StevenBlack’s Unified Hosts List
* Install the Admin Web Interface
* Install lighttpd and the required PHP modules to run the Admin Web Interface
* Enable query logging
* When prompted to choose a privacy level, choose Anonymous mode
* When installation is complete, be sure to note the password for the web interface!

## Activate the DHCP Server on the Pi Hole

Visit http://pi-hole/admin/ and log in with the password for the web interface.

* Click on `Settings`, `DHCP`.
* Check `DHCP server enabled`
* Enter IP range From `192.168.1.2` to `192.168.1.150`
* Ensure the `Router (gateway) IP address` is `192.168.1.254`

Navigate to http://192.168.1.254 a log in to the router configuration.

* Click on `Firewall`, `IP Passthrough`
* Under `Allocation Mode`, click `Passthrough`
* Select the `pi-hole` from the `Passthrough Fixed MAC Address` device list, to enter the MAC address
* Click `Save` at the bottom.
* Go to the Home Network tab.
* Navigate to Subnets & DHCP.
* Set DHCP Server to Off.
* Click `Save` at the bottom
* Restart the router
