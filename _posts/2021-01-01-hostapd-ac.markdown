---
title:  "Adding 802.11ac support to hostapd"
date:   2021-01-01 08:09:23
categories: [pi, wifi]
tags: [pi, wifi]
---

The standard hostapd package installed by default using apt-get did not include 802.11ac support. I had to compile with a modified make .config file to enable it.

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
CONFIG_IEEE80211AC=y
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


Test

First, you'll need to update your hostapd.conf file to enable an 802.11ac WLAN:
```
bridge=br0
country_code=US
interface=wlan1
driver=nl80211
ssid=pies
ignore_broadcast_ssid=0
ieee80211d=1
hw_mode=a
ieee80211n=1
require_ht=1
ieee80211ac=1
require_vht=1
vht_oper_chwidth=1
channel=36
vht_oper_centr_freq_seg0_idx=42
ht_capab=[HT40+][SHORT-GI-20][SHORT-GI-40][DSSS_CCK-40][MAX-AMSDU-3839]
vht_capab=[MAX-MPDU-3895][SHORT-GI-80][SU-BEAMFORMEE]
# WPA stuff
wpa=2
auth_algs=1
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
wpa_pairwise=CCMP
wpa_passphrase=piespies
wmm_enabled=1
```

Now start your WLAN and check the details.

