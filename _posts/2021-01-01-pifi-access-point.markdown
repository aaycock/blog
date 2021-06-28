---
title:  "Pi-Fi Access Points Using Kali Linux"
date:   2021-01-01 08:09:23
categories: [pi, wifi]
tags: [pi, wifi]
---
To make the raspberry pi a Wi-Fi access point, we will use hostapd. 

Steps:

Install the hostapd package:

```
sudo apt-get install hostapd
```


Modify NetworkManager.conf:

```
sudo vi /etc/NetworkManager/NetworkManager.conf
```

Remove `type:bridge`

Then restart the NetworkManager service:

```
sudo service NetworkManager restart
```


Add a bridge for eth0
```
sudo nmcli con add ifname br0 type bridge con-name br0
sudo nmcli con add type bridge-slave ifname eth0 master br0

nmcli con modify br0 bridge.stp no

sudo nmcli con up br0
sudo nmcli con down "Wired connection 1"
```

Unmask hostapd
```
systemctl unmask hostapd
```

Mask wpa_supplicant
```
systemctl mask wpa_supplicant.service
```

Setup forwarding
```
sysctl net.ipv4.conf.all.forwarding=1
iptables -P FORWARD ACCEPT
```

Restart
```
sudo shutdown -r now
```

Test a simple configuration

Now we will create a config file for our new PiFi WLAN:

```
vi /etc/hostapd/hostapd.conf
```

You can use the following example config file for a simple WPA2 WLAN:
```
bridge=br0
country_code=US
interface=wlan0
ssid=PiFi
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
```

Start hostapd
```
hostapd -dd /etc/hostapd/hostapd.conf
```

Now connect to your new Pi-Fi WLAN. Once the 4-way handshake is complete, you are looking for the following in the output of hostapd:
```
wlan0: STA f0:18:98:17:1c:8e WPA: pairwise key handshake completed (RSN)
```

![Pi-Fi WLAN Details](/images/Pi-fi.png)

In order to see this extra detail on a Mac, hold down the **⌥ Option** key when clicking on the Wi-Fi symbol in top menu bar.
