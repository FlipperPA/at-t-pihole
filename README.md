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

The total cost should be about $50.

* A Raspberry Pi kit, I recommend the affordable [Raspberry Pi Zero W Basic Starter Kit](https://www.amazon.com/dp/B0748MPQT4)
* [A MicroUSB ethernet port](https://www.amazon.com/dp/B08VRXJGYK)
* [A MicroSD card](https://www.amazon.com/SanDisk-Ultra-microSDXC-Memory-Adapter/dp/B073JWXGNT) (and an adapter to read it on your computer; on my Mac, I use the provided MiniSD SanDisk Adapter)
* Comfort with a computer command-line interface terminal; enough to open a terminal, SSH, and issue some basic commands.

## Building the Pi Hole

You can follow along with more [detailed instructions here](https://www.raspberrypi.com/tutorials/running-pi-hole-on-a-raspberry-pi/), but here are the key steps.

* [Download the Raspberry Pi Imager](https://www.raspberrypi.com/software/) and install it on your Computer.
* Insert the MicroSD card into your computer with an appropriate adapter.
* Click `Choose Device`, and select `Raspberry Pi Zero`
* Click `Choose OS`, and select `Raspberry Pi OS (other)`, and then `Raspberry Pi OS Lite (32-bit)`
* Click `Storage` and select the MicroSD disk.
* Click `Next`, and `EDIT SETTINGS`. In the `General` tab:
    * Set hostname to `pi-hole`
    * Set a username and password; **you’ll need these later to SSH to the Pi-hole**
    * Uncheck the box next to `Configure wireless LAN`, unless you don't have the MicroUSB ethernet port
        * If using WiFi, enter your network SSID (name) and password
* In the `Services` tab, check the box next to `Enable SSH`, and select `Use password authentication`
    * Make note of the username and password you create, we'll need them to SSH to the Pi-hole!
* Click `Save`, then `Yes` to apply the OS customization settings. Then click `Yes` again to create the disk image.

Once this completes, build your Raspberry Pi-hole:

* Put the SD card into the Raspberry Pi board
* Put the board into the plastic enclosure, and put the top on the enclosure.
* Plug the power into the power port (closest to the edge), and the MicroUSB ethernet dongle into the other (towards to center).
* Plug the ethernet cable from your router into the Raspberry Pi, plug in the power cable, and power it on.

Be patient here! It can take up to ten minutes to boot up the first time. Then SSH to the Pi-hole from your terminal using the username and password you selected during `Configure Settings`; be sure to replace `[username]` with the one you created:

```bash
$ ssh [username]@pi-hole
```

If that doesn't prompt you for the password, you can try `ssh username@pi-hole.local`, or find the IP address from the router web interface in the next section, and try that. Once you are logged in, these next few steps will take a few minutes each. From the terminal, issue the following commands:

```bash
$ sudo apt update
$ sudo apt full-upgrade
$ sudo reboot
```

## Customization of the AT&T Router: Setting a Fixed IP Address

To make this work, we need to configure your AT&T router to pass DHCP through to the Pi Hole server, and set the Pi Hole with a fixed IP so everything will still work after a power cycle. The AT&T BGW320-505 router uses IP address `192.168.1.254` as the gateway address, and we can configure a fixed IP address for the Pi Hole through the web interface at [http://192.168.1.254](http://192.168.1.254).

1. Click `Home Network`, then `IP Allocation`
2. You should see an entry for `192.168.1.### / pi-hole` on the list; click the `Allocate` button on that entry, and note the IP address number!
3. From the `New allocation` list below, select the IP address noted in Step 2: `Private fixed: 192.168.1.###`
4. Click `Save`
5. Click `Home Network`, `IPv6`
6. Turn `IPv6` to `Off`. Their implementation is broken, and will often interfere with devices like Pi-hole. You may also find other devices such as the Meta Quest can now connect.
7. Open an SSH terminal to your Pi Hole again
8. Bring up the NetworkManager configuration by entering the command `sudo nmtui`
9. In the interface, use the cursor arrows and the enter key to select `Edit a connection`, and select `Wired connection 1`.
10. Under `IPv4 CONFIGURATION` (click `<Show>` if necessary):
    * Addresses: `192.168.1.###` (enter the IP address noted in Step 2)
    * Gateway: `192.168.1.254`
    * DNS Server: `1.1.1.1` and `1.0.0.1`
* Click `OK`
* Restart the NetworkManager by entering the command: `sudo systemctl restart NetworkManager`

## Pi-hole Installation

From the Pi-hole command line, install Pi-hole with the command: `curl -sSL https://install.pi-hole.net | bash`

This will take a few minutes to setup, then bring up an interface for installation:

* When warned about needing a static IP address, click Continue to proceed; we already did this above.
* When prompted to choose an upstream DNS provider, choose OpenDNS
* Include StevenBlack’s Unified Hosts List
* Install the Admin Web Interface
* Install lighttpd and the required PHP modules to run the Admin Web Interface
* Enable query logging
* When prompted to choose a privacy level, choose Anonymous mode
* When installation is complete, be sure to note the password for the web interface!

## Activate the DHCP Server on the Pi-hole, and Configure the AT&T Router to Use It

Visit http://pi-hole/admin/ and log in with the password you just noted for the web interface.

**On MacOS Sequoia and above, you'll need to enable your browser to have access to the local network. In Settings, navigate to `Privacy & Security > Local Network`, and be sure the toggle is ON for your browser(s).**

* Click on `Settings`, `DHCP`.
* Check `DHCP server enabled`
* Enter IP range From `192.168.1.2` to `192.168.1.150` (feel free to tweak these values if you know what you're doing)
* Ensure the `Router (gateway) IP address` is `192.168.1.254`

Navigate to http://192.168.1.254 and log in to the router configuration interface.

* Click on `Firewall`, `IP Passthrough`
* Under `Allocation Mode`, click `Passthrough`
* Select the `pi-hole` from the `Passthrough Fixed MAC Address` device list, which will automatically fill in the proper MAC address.
* Click `Save` at the bottom.
* Go to the Home Network tab.
* Navigate to Subnets & DHCP.
* Set DHCP Server to Off.
* Click `Save` at the bottom
* Restart the router by navigating to `Device`, `Restart Device`

## Adding a Firewall on the Raspberry Pi-Hole

For additional protection beyond NAT, you can add a firewall on the Pi-Hole, called `ufw` (`u`ncomplicated `f`ire`w`all). From the Pi-hole command line, you can run these commands to install the firewall and enable SSH, HTTP, DHCP, and DNS.

```bash
sudo apt install ufw
sudo ufw allow ssh comment 'Allow SSH'
sudo ufw allow http 'Allow web HTTP'
sudo ufw allow bootps comment 'Allow 67/UDP'
sudo ufw allow bootpc comment 'Allow 68/UDP'
sudo ufw allow dns comment 'Allow DNS'
sudo ufw enable
```

## You're done!

Congrats! After the router reboots after a few minutes, you should be active. You can navigate to the Raspberry Pi web interface at http://pi-hole/admin/ to see a dashboard of what it is blocking and configure more lists. It is a good idea to test a power outage: turn off both the router and Raspberry Pi for a minute, and then turn them both back on. After five minutes, ensure your devices can still connect to the internet.

If anything goes wrong, you may have to hit the factory reset button on the AT&T router. Ask me how I know. :)

Good luck!
 
