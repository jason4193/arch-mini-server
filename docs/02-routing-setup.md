# Setup the router system

- I need to set up the router system on the Arch Linux system to share the internet connection with other devices.
- Network Traffic Design:

```
Internet
  |
PC as Router (Arch Linux)
  | <- Ethernet to LAN port
Physical Router (TP-Link Router)
  |
  +- LAN ports -> Windows Laptop
  |
  +- WiFi -> MacBook Air
```

- Avoid connecting the Arch Linux system to the physical router's WAN port, as it will create a double NAT situation.
- Instead, connect the Arch Linux system to the LAN port of the physical router.

1. Identify the network interface

```
nmcli device status
> wlp1s0 wifi connected eduroam
> eno1 ethernet
```

2. Create Shared Ethernet Interface

```
nmcli connection modify "Wired connection 1" \
  ipv4.addresses 192.168.50.1/24 \
  ipv4.method shared
  ipv6.method ignore
nmcli connection up shared-ethernet
```

- failed and after reading the logs, I found that the `shared` method requires `dnsmasq` to be installed.
- Install `dnsmasq` with `pacman -S dnsmasq`

3. Physical Router Configuration

- Configure the IP adrress to `192.168.50.2` and reboot
- Disable the DHCP server on the physical router
- Connect the Arch Linux system to the LAN port of the physical router.

4. Check can ping external IPs

- Window laptop: `192.168.50.109`
- Arch Linux system: `192.168.50.1`
- Physical router: `192.168.50.2`
- Windows laptop connected to the physical router
- `ping 192.168.50.1` to check connectivity between the Arch Linux system and the Windows laptop

- Facing issues with the windows laptop not being able to access the internet.
  - Check the default getway on the Windows laptop is still set to the physical router's IP address.
  - Unplug the connection between the Windows laptop and the physical router, then plug it back in.
  - Default gateway should now be set to the Arch Linux system's IP address `192.168.50.1`
  - Check internet connectivity on the Windows laptop by pinging an external IP, e.g., `ping google.com`

5. Verify Physical Wifi Connection

- MacBook Air connected to the physical router's WiFi
- Check the IP address assigned to the MacBook Air
- `ifconfig`
- ping to external website to verify internet connectivity

6. Set up Arch Linux system routing for both MacBook Air and Windows laptop to have a fixed IP address

- Realise that `ipv4.method shared` is not working as expected, so I will set up the routing manually using `ipv4.method manual`

  - `ipv4.method shared` auto spawns a dnsmasq service, which not designed for adding persistent custom dhcp-host reservations.
  - `ipv4.method manual` allows me to set a static IP address for the Arch Linux system and configure the DHCP server manually.

- Understand what is dnsmasq
  - a lightweight DNS forwarder and DHCP server
  - relies on a cofiguration file, which we might need to text editor.

## Set up a Text Editor

- My final target is to use `LazyVim` but now just use `neovim` first.
- Install `neovim` with `pacman -S neovim`
- calling `nvim` will open the neovim text editor

## Setup the router system cont

7. Change the network connection to manual mode

```bash
nmcli connection modify "Wired connection 1"\
  ipv4.method manual \
  ipv4.addresses 192.168.50.1/24 \
  ipv4.never-default yes ipv6.method ignore
nmcli connection down "Wired connection 1" || true
nmcli connection up "Wired connection 1"
```

8. Enable IP forwarding

- This allows the Arch Linux system to forward packets between interfaces, enabling routing.

```bash
echo 'net.ipv4.ip_forward=1' | sudo tee /etc/sysctl.d/30-ipforward.conf
sysctl -p /etc/sysctl.d/30-ipforward.conf
```

- Verify IP forwarding is enabled

```bash
sysctl net.ipv4.ip_forward
> net.ipv4.ip_forward = 1
```

9. Install `nftables` for firewall rules

- `pacman -S nftables`

10. Create dnsmasq configuration file

- `dnsmasq.d` directory is where dnsmasq looks for additional configuration files.
- `/etc/dnsmasq.d/arch-router.conf` is a custom configuration file for the Arch Linux router.

```bash
sudo mkdir -p /etc/dnsmasq.d
sudo nvim /etc/dnsmasq.d/arch-router.conf
```

10. Add the following content to the `arch-router.conf` file:

```
interface=eno1
bind-interfaces
listen-address=192.168.50.1
domain=arch-router
dhcp-authoritative
log-dhcp
# Dynmic pool
dhcp-range=192.168.50.100,192.168.50.200,12h
# Static IP reservations
dhcp-host=<Windows Laptop MAC>,192.168.50.10,windows-laptop,12h
dhcp-host=<MacBook Air MAC>,192.168.50.11,macbook-air,12h
# Hand out gateway and DNS server
dhcp-option=option:router,192.168.50.1
dhcp-option=option:dns-server,192.168.50.1,1.1.1.1
server=1.1.1.1
```

- Replace `<Windows Laptop MAC>` and `<MacBook Air MAC>` with the actual MAC addresses of the devices. using

```bash
Mac: ifconfig en0
Windows PowerShell: Get-NetAdapter
```

- This configuration sets up a DHCP server on the `eno1` interface, providing IP addresses in the range `192.168.50.100` to `192.168.50.200` with a lease time of 12 hours which means the devices will keep the IP address for 12 hours before needing to renew it.
- It also reserves static IP addresses for the Windows laptop and MacBook Air, ensuring they always receive the same IP address when they connect to the network. Providing a lease time of 12 hours for the static IP reservations as well.
- Given a domain name `arch-router` for the DHCP server, which can be used by devices to resolve the router's hostname.

11. Set nftables NAT + forwarding rules

```bash
nvim /etc/nftables.conf
```

12. Add the following content to the `nftables.conf` file:

```
table ip nat {
  chain postrouting {
    type nat hook postrouting priority 100;
    oif "wlp1s0" masquerade
  }
}
table inet filter {
  chain forward {
    type filter hook forward priority 0; policy drop;
    ct state established,related accept
    iif "eno1" oif "wlp1s0" accept
  }
}
```

- This configuration sets up NAT (Network Address Translation) for the `wlp1s0` interface and allows forwarding of packets from the `eno1` interface to the `wlp1s0` interface.

- Enable and start the `dnsmasq` and `nftables` services

```bash
sudo systemctl enable --now dnsmasq
sudo systemctl enable --now nftables
```

- Verify the status of the services

```bash
sudo systemctl status dnsmasq
sudo systemctl status nftables
```

- Troubleshooting: Identify issues of `dnsmasq` service on, but not assigning IP addresses to devices. Using `journalctl -u dnsmasq` to check the logs. Using `tcpdump` to monitor network traffic (Observe the DHCP Discover packet but no DHCP Offer packet from the Arch Linux system).
  1. Identify that the `nftables` configuration is not setting correctly
  2. The input chain is not allowing DNS requests to pass throught
  3. Add the following rule to the `nftables.conf` file to allow DNS requests:

```
chain input {
  type filter hook input priority 0; policy accept;
  # Allow DHCP
  udp dport 67 iif "eno1" accept
  udp dport 68 iif "eno1" accept
  # Allow DNS
  udp dport 53 iif "eno1" accept
  tcp dport 53 iif "eno1" accept
}
```

4. Reload the `nftables` configuration
5. Restart the `dnsmasq` service

## Current installation list

1. Base system: `base`, `linux`, `linux-firmware`
2. Bootloader: `grub`, `efibootmgr`
3. Network management: `networkmanager`
4. DNS and DHCP server: `dnsmasq`
5. Text editor: `neovim`
6. Firewall and NAT: `nftables`
