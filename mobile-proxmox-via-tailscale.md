1. install tailscale
2. setup tailscale
```
iptables -P INPUT DROP && iptables -P FORWARD DROP && iptables -P OUTPUT DROP && iptables -F INPUT && iptables -F FORWARD && iptables -F OUTPUT && iptables -A INPUT -i lo -j ACCEPT && iptables -A OUTPUT -o lo -j ACCEPT && iptables -A INPUT -i tailscale0 -j ACCEPT && iptables -A OUTPUT -o tailscale0 -j ACCEPT && iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT && iptables -A OUTPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```
4. setup tailscale subnetting
```
tailscale up --reset --hostname=pve --advertise-routes=10.200.0.0/24
```
5. setup new virtual interface `vmbr1`in proxmox (`Linux Bridge`>IPv4:`10.200.0.1/24`>ok)
6. setup proxmox dhcp
```
