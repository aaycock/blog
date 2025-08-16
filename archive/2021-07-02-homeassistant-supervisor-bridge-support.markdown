---
title:  "Modifying HomeAssistant Supervisor to Support A Managed Bridge Device"
date:   2021-07-02
categories: [pi, wifi, iot, homeassistant]
tags: [pi, wifi, iot, homeassistant]
---

There are several scenarios where I find it helpful to establish a network bridge device that is managed by NetworkManager. For example, when running in managed mode to allow hostapd to automatically bring up the wlan interface, bridged to ```eth0```.

This works great in a hostapd-only environment. However, if you plan to run [HomeAssistant][homeassistant] with the supervisor docker container, you will run into issues because HomeAssistant Supervisor does not officially support the bridge network device (yet).

I've been working on a fix that I plan to PR in the homeassistant code base. Here are the details if you'd like to try.

For this example, I'll assume you already have a working HomeAssistant install in place with Supervisor installed. This example is taken from a Kali-based linux distro, but the docker steps should be consistent across platforms.

SSH to instance
```
ssh <username>@hostname
sudo -i
```

Exec into the docker container and cd to the src directory
```
┌──(root💀kali)-[~]
└─# docker exec -it hassio_supervisor /bin/bash
bash-5.1# cd /usr/src/supervisor/supervisor/
```

Files that you'll need to modify:

``` python
# supervisor/dbus/const.py (line 118)
    VLAN = 11
    TUN = 16
    VETH = 20
    # Add
    BRIDGE = 13
```

``` python
# supervisor/dbus/network/__init__.py (line 122)
  DeviceType.ETHERNET,
  DeviceType.WIRELESS,
  DeviceType.VLAN,
  # Add
  DeviceType.BRIDGE,
```

``` python
# supervisor/host/const.py (line 16)
    ETHERNET = "ethernet"
    WIRELESS = "wireless"
    VLAN = "vlan"
    # Add
    BRIDGE = "bridge"
```

``` python
# supervisor/host/network.py (line 266)
    # Modify
    if inet.settings
    and inet.settings.ipv4
    and inet.connection
    and inet.connection.ipv4
```

``` python
# supervisor/host/network.py (line 294)
    # Modify
    if inet.settings
    and inet.settings.ipv6
    and inet.connection
    and inet.connection.ipv6
```

``` python
# supervisor/host/network.py (line 334)
  DeviceType.ETHERNET: InterfaceType.ETHERNET,
  DeviceType.WIRELESS: InterfaceType.WIRELESS,
  DeviceType.VLAN: InterfaceType.VLAN,
  # Add
  DeviceType.BRIDGE: InterfaceType.BRIDGE,
```

Once you have made these changes in line, you will need to restart HomeAssistant Supervisor. 

```
exit
ha supervisor restart
```

Now, reload ```http://<HOST>:<PORT>/hassio/dashboard```

And you should see a working supervisor UI:
![working-supervisor](/images/supervisor-working.png)

[homeassistant]: https://www.home-assistant.io/