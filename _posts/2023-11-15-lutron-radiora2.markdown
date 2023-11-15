---
layout: post
title:  "Lutron RadioRA2 Recovery"
date:   2023-11-15 15:55:04
categories: lutron, home-automation
---
Our home came with a Lutron RadioRA 2 system controlling much of the lights and thermostats. While it's a nice
integrated system, it was designed before the era of easy, consumer-configurable smart home setups, and is a
huge beast. We've had a few issues with the system, and the installer has gone out of business, and our rural
location means there are limited options.

The issue that really broke the setup was when we upgraded to a wired GigE setup, and put in a higher-tier wired router
that was configured with a different default subnet. I could see the Lutron devices broadcasting ARP, but the router
didn't reply and assign it an IP, and at this point, I didn't want to reconfigure the entire network just for 1 device.
Without this network connection, none of the remote management options worked, and I couldn't reprogram the
thermostats as needed to save energy when we're gone.

There are two network-connected devices that are part of this setup, the Main Repeater (which acts as an ethernet to
their custom ~400MHz wireless band), and the Connect Bridge, which then allows for remote access, and to tie in
integrations with the app, and other smart home devices like Alexa. You can put the Connect Bridge back into DHCP mode
by holding the black button on the device for 20s, this allowed me to then add the home via the mobile app. However,
once added, the app would never connect, because the Connect Bridge and the Main Repeater were unable to communicate.

I used an old router to configure a network with the old subnet, then the Main Repeater was able to come up. I went
through the entire Level 1 training program during a long flight delay, so I have access to the software, but only
the bottom-tier "Essentials", which cannot open projects (or configure repeaters) that were installed by "Inclusive"
professionals. As Lutron installation is not my main goal in life, I was stuck with a project file that I could not
open.

Eventually I realized I could telnet to the Main Repeater with the credentials `lutron` / `integration`. I found
[this](https://assets.lutron.com/a/documents/040249.pdf) guide that explained the interface. By connecting and
running the `?ETHERNET,0` command I was able to see the IP address, `?ETHERNET,1` is the gateway, and `?ETHERNET,2`
the subnet mask. All I needed to do was change the IP address, so I was able to do it with `#ETHERNET,0,192.168.001.XYZ`. Rebooting the repeater and plugging it back into the main network worked, it connected to the bridge and I could
access all the information and controls I needed from the app.
