---
title:  "Monitor Management Frames on a Raspberry Pi"
date:   2021-01-01 08:09:23
categories: [pi, security, wifi]
tags: [pi, security, wifi]
---

For an IoT project I'm working on, I recently needed to put a Raspberry Pi into monitor mode to capture management frames, specifically probe-requests. This had to be done while running as an access point with connected clients.

As you learn early on in CWNA curriculum, a Wi-Fi radio can only be in a single mode at any time, so in order to monitor for management frames without affecting connected clients, you will need at least 2 radios. 

In this scenario, I'll be using an external radio (Atheros chipset), and will be using the Pi's onboard radio to monitor. Keep in mind that not all OS distros and radios support monitor mode. In order to confirm that your radio supports monitor mode, you can use `iw` to confirm.

```
iw list
```

What you are looking for is `monitor` in the supported interface modes for your adapter (in this case `phy0`): 
```
Supported interface modes:
		 * IBSS
		 * managed
		 * AP
		 * monitor
		 * P2P-client
		 * P2P-GO
		 * P2P-device
```

Once you have confirmed support, you can put the interface into monitor mode with the following:
```
iw phy phy0 interface add mon0 type monitor
ifconfig mon0 up
iw dev mon0 set channel <channel_number>
```

If you are in monitor mode, you should be able to capture and analyze 802.11 packets. Here is the command to capture management frames, including probe requests:
```
tcpdump -i mon0 -e -s 0 -l type mgt
```

There are some interesting things you can do when capturing management frames, including [cracking a PSK with a dictionary attack][aircrack].

[aircrack]: /blog/2021/aircrack/
