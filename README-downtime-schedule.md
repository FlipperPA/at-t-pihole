# Pihole Downtime Schedule

The Pihole isn't set up to have schedules, but with a little bit of a hack, we can make it do so. Here's the scenario:
our 13-year-old kiddo would get up in the middle of the night and use their school computer to get on Scratch. While
the laptop we own is covered by Apple's parental controls, we needed a way to keep her from using the school computer
in the middle of the night (ironic, I know). Here's how.

## Step 1: Create a Group of Devices

You'll need to know the hardware MAC addresses or IP addresses of the machines you want to schedule blocks for.

* Log into Pihole's web interface and go to `GROUPS`.
* Add a new group with a name (`School-Night-Hours`) and description (`Only allows access from 3pm - 9pm.`)
* Go to `CLIENTS`, and add the clients you need to the group with a comment.
    * You can use an IP address (`192.168.1.100`) or a MAC address (`80:30:49:86:C3:D9`).
    * Some devices rotate MAC addresses, so an IP address may be more reliable. Pihole's DHCP server tends to keep devices on the same address.

## Step 2: Add the Block Rules

You can customize the rules you want for this group to be blocked on a schedule. If you don't want to block everything,
you can block certain sites, but in this example, we're doing a total blackout.

* Go to `DOMAINS` and add the filters you need; to block everything, use the `Regex filter`: `^.+$`, and `Add to denied domains`

## Step 3: Schedule Enforcement

You'll need to understand cron to customize these rules.

* Log into your Pihole via SSH.
* Type `crontab -e` and copy/paste the schedules below.
* The first crontab schedule item below disables the group we just created at 3pm. This allows access to everything on the internet.
* The second crontab schedule item below enables the group by just created at 9pm. This disables access to everything on the internet.

The upshot of this is, the kiddo can ONLY use their school laptop on our home network from 3pm - 9pm. She's in for a big disappointment
the next time she tries to wake up at 2am to play Scratch.

```
# Disable the group at 3pm, allowing access
0 15 * * * sudo /usr/bin/pihole-FTL sqlite3 /etc/pihole/gravity.db "UPDATE 'group' SET enabled = 0 WHERE name = 'School-Night-Hours'" && sudo pihole reloadlists
# Enable the group at 9pm, disabling access
0 21 * * * sudo /usr/bin/pihole-FTL sqlite3 /etc/pihole/gravity.db "UPDATE 'group' SET enabled = 1 WHERE name = 'School-Night-Hours'" && sudo pihole reloadlists
```

## Further Reading for Customization

To find a device's IP address of MAC address, you should be able to search the web for something like "chromebook ip address" or "windows wifi mac address". More help:

* Pihole's Tutorial for regex patterns for blocking domains: https://docs.pi-hole.net/regex/tutorial/
* crontab tutorial: how to customize your enable / disable schedule: https://tecadmin.net/crontab-in-linux-with-20-examples-of-cron-schedule/
