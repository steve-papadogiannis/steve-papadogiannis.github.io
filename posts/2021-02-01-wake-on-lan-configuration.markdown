---
title: Enabling Wake On Lan Feature on Ubuntu
---

## Intro

Recently, I decided to make an old desktop machine to a server like resource, 
which meant to remove any input or output peripherals except for the 
internet connectivity means (Ethernet cable) as to be able to access it via `ssh`.

Of course, a 24/7 available machine is not a very cost-effective and 
environment-friendly solution in a home setup, so the **Wake-On-Lan** feature 
was the right choice. This feature enables you to turn on your machine using 
the local network. (_[Wake-on-Lan](https://en.wikipedia.org/wiki/Wake-on-LAN)_)

The desktop at hand runs **Ubuntu 20.04.1 LTS (GNU/Linux 5.8.0-40-generic x86_64)** as OS
(not a server oriented one) and has an **ASUS P5Q-SE** parentboard with an integrated network 
interface controller (**NIC**). The network interface controller must support 
the **Wake-On-Lan** feature in order to enable it.

## Install ssh server

As a zero step, I installed the `openssh-server` in order to be able to access the machine
after the configuration.

To check if the `ssh` server is installed and/or active I issued the below command:

`sudo service ssh status`

If the `ssh` server is not in place, one can download it by issuing the below command:

`sudo apt update && sudo apt install openssh-server`

After `ssh` server was in place and active, I performed an `ssh` connection as a health check 
(in order to avoid plugging/unplugging input/output devices for debugging).

## Enable WOL in BIOS

A good guide for the steps can be found at:
[WakeOnLan Ubuntu Help](https://help.ubuntu.com/community/WakeOnLan#:~:text=To%20enable%20WoL%20in%20the,Save%20your%20settings%20and%20reboot.)

Next step was to enter the **BIOS** and find the much-desired flag(s).
In the aforementioned parentboard I had to enter the **BIOS**, go under **Power** section,
next to **APM Configuration** menu and enable both **Power by PCI Devices** and 
**Power by PCIE Devices**. For reasons unknown to me, both flags should be enabled in order to
work.

## Enable WOL in software

After that, I had to get the name of my **NIC** in order to check if it supports and has enabled the 
**WOL** feature.
Issuing `ifconfig` gave me the info I needed. In my case the **NIC** was called `enp2s0`

Next command in line was `sudo ethtool enp2s0` which revealed its secrets:

```
Supports Wake-on: pg

...

Wake-on: d
```

The `g` in `Supports Wake-on` meant that I was lucky and my machine actually supported the **WOL** feature
and the `d` in `Wake-on` meant that the feature was disabled at the time.

One command kept me away from fulfilling the configuration (or so I thought..).

I issued an `sudo ethtool -s enp2s0 wol g` to enable the feature.

After that I got a reassuring:

```
Wake-on: g
```

when I issued the `sudo ethtool enp2s0` command again.

Then, I shut down the machine.

## Wake-On-Lan Client program

In order to perform the **Wake-On-Lan**s to this machine I got a portable **Wake-On-Land** program
for my other **Windows** machine (which would be the client for the desktop machine).

The program fitted to my needs was **WakeMeOnLan** for **Windows** which can be
downloaded at:

[Nirsoft WakeMeOnLan](https://www.nirsoft.net/utils/wake_on_lan.html)

It is a simple program to hold and awake your **Wake-On-Lan** machines.

## Persisting setting

So, I stored my machine and performed an **awakening**.
All played well, until I shut down again the server after the session and tried to **awake** it
again.

No response.

The cause was (maybe for security reasons, maybe not) that the **enable** flag
was reset after shutdown. It was not **persisted** across boots.

Solution to this incident, was brought by this site:
[Wake-on-LAN ArchLinux Wiki](https://wiki.archlinux.org/index.php/Wake-on-LAN)

What I read and did was to 
make a new file at `/etc/systemd/system/wol@.service`
and append the below content:

```
[Unit]
Description=Wake-on-LAN for %i
Requires=network.target
After=network.target

[Service]
ExecStart=/usr/bin/ethtool -s %i wol g
Type=oneshot

[Install]
WantedBy=multi-user.target
```

What this script does is to enable the **Wake-On-Lan** feature in every boot
after the network initializes. This is done by `systemd` automatically, so you end
up with a **persistent** setting of the feature.

## Conclusion

Lines above, documented a walkthrough for enabling **Wake-On-Lan** feature for the specific
system. Many parts may vary when enabling the feature to another machine, but the
concept remains as described and summarized to the next list:

* Install **ssh-server**
* Enable **WOL** in **BIOS**
* Enable **WOL** in software
* **Wake-On-Lan Client** program
* (_Optional_) Persisting setting 
