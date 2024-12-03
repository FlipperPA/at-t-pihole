# Setting Up Pi Hole with AT&T Fiber's BGW320-505 Router

AT&T Fiber internet service is one of the worst I've encountered, and I'm from the home of Comcast's Xfinity, so that's saying something. They engage in more dark patterns than any ISP I have seen:

* They force you to use their equipment, which is some of the worst on the market;
* They don't let you easily change your DNS provider;
* They monitor your web-surfing and sell the data to data brokers;
* They don't allow you basic white and black listing of websites, unless you pay an additional monthly fee for their awful app;
* Their default configuration of IPv6 is broken, and prevent devices such as the Meta Quest from properly connecting over WiFi until disabled completely.

I could go on, but you get the point.

## A Partial Solution: the Pi Hole!

Most people are familiar with installing an ad blocker in your web browser or on your phone, such as uBlock or AdBlock Plus. Pi Hole does largely the same thing, but for your entire home network. That means any device that hooks up to your home WiFi (a laptop, an iPad, a phone, even a TV device like Roku) is protected. It also provides a web-based interface where you can block any website you want. This is very handy if you have kids! It blocks ads on YouTube, in the interface for Roku and Amazon Fire, and also blocks these devices logging everything you watch, so they don't get sold to data brokers.

This tutorial will walk you through how to setup a Pi Hole on AT&T Fiber with the standard BGW320-505 router.

## What You Need to Build a Pi Hole

The total cost should be about $45.

* A Raspberry Pi kit, I recommend the affordable [Raspberry Pi Zero W Basic Starter Kit](https://www.amazon.com/dp/B0748MPQT4)
* [A MicroUSB ethernet port](https://www.amazon.com/dp/B08VRXJGYK)

## Building the Pi Hole

* [Follow This Tutorial](https://www.raspberrypi.com/tutorials/running-pi-hole-on-a-raspberry-pi/)
* The tutorial will instruct you to install the operating system using this page.
    * Be sure to choose "Configure Settings" during the installation
    * I choose to activate SSH with username and password
    * Save the username and password you create during the installation

Follow the instructions to update the operating system and install Pi Hole using the tutorial. After you have installed the operating system on the MicroSD card, and assembled the Raspberry Pi, be sure to use the ethernet cable to connect it to the router. Then continue below.

## Customization For AT&T After Initial Installation

To make this work, we need to configure your AT&T router to pass DHCP through to the Pi Hole server, and set the Pi Hole with a fixed IP so everything will still work after a power cycle.

### Setting a Fixed IP Address

The AT&T BGW320-505 router uses IP address `192.168.1.254` as the gateway address; in this example, I'll use `192.168.1.250` for the Pi Hole. First, enter your router's configuration at http://192.168.1.254 and:

* Click `Home Network`, then `IP Allocation`
* You should see an entry for `192.168.1.### / pi-hole` on the list; click the `Allocate` button on that entry.
* From the `New allocation` list below, select `Private fixed: 192.168.1.250`
* Click `Save`

Open an SSH terminal to your Pi Hole again, and bring up the NetworkManager configuration tool:

```
sudo nmtui
```

* In the interface, use the cursor arrows and the enter key to select `Edit a connection`, and select `Wired connection 1`.
* Make note of the MAC address in the `Device` field; it appears as six two-character pieces separated by colons. (`00:##:##:##:##:##`)
* Under `IPv4 CONFIGURATION` (click `<Show>` if necessary):
    * Addresses: `192.168.1.250/24`
    * Gateway: `192.168.1.254`
    * DNS Server: `1.1.1.1` and `1.0.0.1`
* Click `OK`

```
sudo systemctl restart NetworkManager
```

https://www.reddit.com/r/pihole/comments/q2nfwk/is_it_possible_to_use_pihole_with_att_fiber/



This is what I would do if I were you and wanted to continue using the AT&T router.

1- Assign the pihole with a static IP, 192.168.1.10 (assuming nothing is using that address) subnet 255.255.255.0 and gateway 192.168.1.254
2- Confirm you can access the pihole web interface and login
3- Disable DHCP on the AT&T router 4- Enable DCP on the pihole and make sure the DNS server is is pushing out is 192.168.1.10 or whichever IP you used as static. 5- Reboot your clients or wait for their existing DHCP lease to expire
No need to set any DHCP reservations on the AT&T router since your pihole is static and DHCP is no longer running on the AT&T router.
Don't forget to set your upstream DNS severs within pihole (google DNS, quad DNS, custom, etc...
