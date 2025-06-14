

## Install tailscape on OpenWRT

On OpenWRT SSH:
```
opkg install iptables-nft ip6tables-nft tailscape
/etc/init.d/tailscale start
```

On OpenWRT LUCI:

In Network > Firewall > General Settings > Add (Under "Zones"). Set the following:

![image](https://github.com/user-attachments/assets/0e4b398e-1c0c-432f-a9f5-53e1c26b3588)
![image](https://github.com/user-attachments/assets/ae4d80c8-6bf9-4b06-bb21-92ad1e4caa2a)


On OpenWRT SSH:
```
tailscale up --netfilter-mode=off --advertise-routes=192.168.1.0/24 --advertise-exit-node
```

Then Login to tailscape using the link provided by the command.


(Based on: https://www.reddit.com/r/Tailscale/comments/yrqbwb/how_to_using_openwrt_as_a_tailscale_exit_node/)

