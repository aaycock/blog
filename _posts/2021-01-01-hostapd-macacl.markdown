---
title:  "Enabling RADIUS-based MAC ACL on hostapd"
date:   2021-01-01 08:09:23
categories: [pi, wifi]
tags: [pi, wifi]
---

I ran into an interesting problem when trying to enable RADIUS-based MAC authorization on a Raspberry Pi running hostapd. It took a little experimentation, but I was able to eventually make it work. 

The bad news is that I was not able to get and RADIUS-based MAC auth working with the onboard radio of the Raspberry Pi 4 (yet). You will need an external radio to support this. 

Here are a few models in my lab that are **working**:

* [AR9271 802.11n (Atheros)][atheros]
* [Alfa Long-Range Dual-Band AC1200][alfa]
* [Panda Wireless PAU09 N600][panda]

Here are a few models that **do not work**:

* Onboard radio for Raspberry Pi 3b+
* Onboard radio for Raspberry Pi 4
* [Comfast CF-912AC][comfast]


**Security caveat:** you should know that MAC-based auth is not considered secure because it is very easy to spoof a client MAC address. My ultimate goal is to support dynamic PSK (DPSK), but I'll be starting with MAC auth to confirm simple RADIUS requests.

Here are the steps:

First, you will need to [install and configure hostapd][install-hostapd] on your raspberry pi. On my lab devices, I'm running Kali linux.

Next you will need a working RADIUS server. For my lab, I'm running an RGNets rXg with a few accounts populated. 

I find it helpful to test requests from the command line before I start to integrate more layers (like hostapd, etc.)

For testing, I use radtest, which can be installed on Kali with the _freeradius_ package.

To test a simple auth request from the command line:
```
radtest <username> <userpassword> <server_host> <port> <radius_secret>
radtest testuser testpass radius.example.com 1812 mysupersecret
```

For MAC auth, you can simply pass your MAC address as your username and password:
```
radtest 'a0:fb:c5:d7:42:4e' 'a0:fb:c5:d7:42:4e' radius.example.com 1812 mysupersecret
```

If you pass along the correct credentials and everything is working right, you should see:
```
Received Access-Accept Id 154 from x.x.x.x:1812 to y.y.y.y:33397 length 117
```

If you pass incorrect credentials, you will be denied with the following message:
```
Received Access-Reject Id 74 from x.x.x.x:1812 to y.y.y.y:55076 length 35
```

If you don't get either, and instead see 3 attempts with no response, you may need to troubleshoot your RADIUS server config or network path:

Troubleshooting
If you are having trouble connecting to your RADIUS server to begin with, it may be helpful to watch the traffic to port 1812 with tcpdump. Run this on the server while you run the command line test above, substituting your actual interface.
```
tcpdump -vnes0 -i igb0 port 1812
```


 

Edit your hostapd.conf file
```
bridge=br0
country_code=US
interface=wlan1 # External radio
ssid=Pi-Fi-MAC
hw_mode=g
channel=11
ignore_broadcast_ssid=0
# WPA stuff
wpa=2
auth_algs=1
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
wpa_pairwise=CCMP
wpa_passphrase=pifipass
wmm_enabled=0

# RADIUS MAC
macaddr_acl=2 # This will call RADIUS
auth_server_addr=10.10.10.161
auth_server_port=1812
auth_server_shared_secret=mysupersecret
```

The key items in the hostapd.conf file are:
* interface=wlan1 - Uses the external radio (e.g. Atheros) 
* macaddr_acl=2 - Makes a call to RADIUS, passing MAC as username
* auth_server_x - Config settings for your RADIUS server

**Note:** The auth_server_addr must be an IP address. I found passing a hostname did not work.

Test
Now launch hostapd and connect to your new RADIUS enabled WLAN:
```
hostapd -dd /etc/hostapd/hostapd.conf
```


**Update:** I now have a [working DPSK configuration][config-dpsk] on the Pi.

[install-hostapd]: /blog/2021/pifi-access-point/
[config-dpsk]: /blog/2021/hostapd-dpsk/
[atheros]: https://www.amazon.com/dp/B07FVRKCZJ?psc=1&ref=ppx_yo2_dt_b_product_details
[alfa]: https://www.amazon.com/dp/B00MX57AO4?psc=1&ref=ppx_yo2_dt_b_product_details
[comfast]: https://www.amazon.com/Comfast-CF-912AC-1200MBPS-Realtek-Network/dp/B01KX1M436/ref=sr_1_3?dchild=1&keywords=Comfast+CF-912AC&qid=1624857028&s=electronics&sr=1-3
[panda]: https://www.amazon.com/dp/B01LY35HGO?psc=1&ref=ppx_yo2_dt_b_product_details