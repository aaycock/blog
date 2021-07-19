---
title:  "Enabling dynamic PSK (DPSK) on hostapd"
date:   2021-04-24
categories: [pi, wifi]
tags: [pi, wifi]
---

Enabling DPSK on a Raspberry Pi with hostapd required a little more work and troubleshooting than I expected.

The good news is that if you follow the steps in my post on [enabling RADIUS-based MAC auth][mac-auth], you will be 95% done with the work needed, unless you need to micro-segment [traffic on individual VLANs][dvlan]. 

Once you have completed all the steps to enable RADIUS-based MAC auth, there are only a few configuration changes that need to be made. 

On the RADIUS server, you will need to return `Tunnel-Password` attribute in the accept response. The details for this attribute can be found in [RFC 2868][rfc]

On the rXg, it looks like this:
<img src="/images/tunnel-password.png">

On the Pi, you will need to add a line to your hostapd.conf:
```
# Optionally, WPA passphrase can be received from RADIUS authentication server
# This requires macaddr_acl to be set to 2 (RADIUS)
# 0 = disabled (default)
# 1 = optional; use default passphrase/psk if RADIUS server does not include
#	Tunnel-Password
# 2 = required; reject authentication if RADIUS server does not include
#	Tunnel-Password
wpa_psk_radius=1
```

Once that is done, start up hostapd:
```
hostapd -dd /etc/hostapd/hostapd.conf
```

Now connect to your new DPSK-enabled WLAN.

**Note:** Be sure to turn off 'Private Address' or 'MAC Randomization' on your device. Otherwise, you may be passing a MAC address that has no corresponding account (and therefore no dynamic PSK set).

On an iPhone, it should look like this for your WLAN settings:

<img src="/images/random-mac.jpg" width="400">

[mac-auth]: /2021/hostapd-macacl/
[dvlan]: /2021/hostapd-dvlan/
[rfc]: https://datatracker.ietf.org/doc/html/rfc2868#section-3.5