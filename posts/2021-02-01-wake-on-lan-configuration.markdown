---
title: Enabling Wake On Lan Feature on Ubuntu
---

Recently, I decided to make an old desktop machine to a server like resource, 
which meant to remove any input or output peripherals except for the 
internet connectivity means (Ethernet cable) as to be able to access it via `ssh`.

Of course, a 24/7 available machine is not a very cost-effective and 
environment-friendly solution in a home setup, so the **Wake-On-Lan** feature 
was the right choice. This feature enables you to turn on your machine using 
the local network. (_https://en.wikipedia.org/wiki/Wake-on-LAN_)

The desktop at hand runs **Ubuntu 20.04.1 LTS (GNU/Linux 5.8.0-40-generic x86_64)** as OS
(not a server oriented one) and has an **ASUS P5Q-SE** parentboard with an integrated network 
interface controller (**NIC**). The network interface controller must support 
the **Wake-On-Lan** feature in order to enable it.

As a zero step, I installed the `openssh-server` in order to be able to access the machine
after the configuration.

To check if the `ssh` server is installed and/or active I issued the below command:

`sudo service ssh status`

If the `ssh` server is not in place, one can download it by issuing the below command:

`sudo apt update && sudo apt install openssh-server`

After `ssh` server was in place and active, I performed an `ssh` connection as a health check 
(in order to avoid plugging/unplugging input/output devices for debugging).

Next step was to enter the **BIOS** and find the much-desired flag(s).
In the aforementioned parentboard I had to enter the **BIOS**, go under **Power** section,
next to **APM Configuration** menu and enable both **Power by PCI Devices** and 
**Power by PCIE Devices**. For reasons unknown to me, both flags should be enabled in order to
work.

(_to be continued_)
