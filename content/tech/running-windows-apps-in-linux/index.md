+++
date = '2025-06-22T09:03:21+06:00'
title = 'Running Windows Apps in Linux'
draft = true
tags = []
+++
https://blog.oddbit.com/post/2018-03-12-using-docker-macvlan-networks/#host-access
```bash
sudo nmcli connection add type macvlan \
    con-name vlan-shim \
    ifname vlan-shim \
    dev wlp0s20f0u6 \
    mode bridge \
    ip4 192.168.0.223/32 \
    ipv4.method manual

sudo nmcli connection modify vlan \
    +ipv4.routes "192.168.0.192/27"

sudo nmcli connection modify vlan connection.autoconnect yes
sudo nmcli connection up vlan
```

```bash
sudo nmcli connection down vlan
sudo nmcli connection delete vlan
```
