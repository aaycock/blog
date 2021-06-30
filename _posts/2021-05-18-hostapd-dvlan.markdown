---
title:  "Adding Dynamic VLAN (DVLAN) support to hostapd"
date:   2021-05-18
categories: [pi, wifi]
tags: [pi, wifi]
---

The standard hostapd package installed by default using apt-get did not include dynamic VLAN support. I had to compile with a modified make config file to enable it.

Support for this feature can be enabled by compiling hostapd with the appropriate feature flag enabled. Here are the steps:

Get required supporting libraries
```
sudo apt-get install ccache libnl-3-dev libssl-dev libnl-genl-3-dev
```

Clone repo
```
cd /usr/src
git clone https://w1.fi/cgit/hostap
```

Update config

In the hostap/hostapd directory, prepare your .config file:
```
cp defconfig .config
```

In order to support 802.11ac mode, you'll need to enable this flag:
```
CONFIG_FULL_DYNAMIC_VLAN=y
```

Make
```
make -j4
```

Copy files
```
cp /usr/sbin/hostapd /usr/sbin/hostapd.orig
cp ./hostapd /usr/sbin/hostapd
```

To configure hostapd to assign static VLANs, follow these steps:

First, define your vlans in a hostapd.vlan file:
```
wlan1.100 eth0
wlan1.101 eth0
wlan1.102 eth0
```

Then, create a static mapping to the VLANs in your accept.conf file:
```
<MAC 1> 100
<MAC 2> 101
<MAC 3> 102
```
 
Next, you'll need to update your hostapd.conf file to enable VLAN support:
```
bridge=br0 
country_code=US 
interface=wlan1 
ssid=vlantest
hw_mode=g 
channel=11 
ignore_broadcast_ssid=0 
# WPA stuff                                                                              
wpa=2 
auth_algs=1 
wpa_key_mgmt=WPA-PSK 
rsn_pairwise=CCMP 
wpa_pairwise=CCMP 
wpa_passphrase=vlantest
wmm_enabled=0 
macaddr_acl=1 
accept_mac_file=/etc/hostapd/accept.conf 
vlan_file=/etc/hostapd/hostapd.vlan 
vlan_tagged_interface=eth0 
per_sta_vif=1 
dynamic_vlan=0
```

Start your WLAN and confirm proper VLAN tagging.


**Note:** If your Pi is connected via switch, be sure to enable the VLANs appropriately.


Finally, to enable dynamic VLANs, set `dynamic_vlan=1`, `macaddr_acl=2` and be sure that RADIUS returns `Tunnel-Private-Group-ID` which will determine the VLAN ID.


