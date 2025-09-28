# Kali Linux VM Setup Guide

- Key Components that needed:
  1. QEMU/KVM, QEMU is stands for Quick Emulator and KVM is Kernel-based virtual machines
  2. libvirt, a managment layer for virtualization
  3. virt-install, command line tool for creating new VMs
  4. virt-manager, GUI tool for managing VMs
  5. dnsmasq, lightweight DHCP and DNS server for providing network services to VMs
  6. iptables-nft, firewall tool for managing network traffic
  7. nftables, packet filtering framework for Linux
  8. cloud-utils, tools for managing cloud instances (not necessary so not installing)

## Installation Steps

1. `sudo pacman -S --needed qemu-full libvirt virt-install virt-manager dnsmasq iptables-nft nftables` to install the necessary packages.
2. `sudo systemctl enable --now libvirtd` to enable and start the libvirt service.
3. `sudo usermod -aG libvirt,kvm $USER` to add your user to the libvirt and kvm groups for permissions.
4. `newgrp libvirt` to apply the new group membership without logging out and back in.
5. Using `virt-manager` to create and manage Kali Linux VM.

- Download the Kali Linux ISO file
- Sent OS type to Debian 12
- Allocate resources (2 vCPU, 4096MB RAM, 20GB disk)
- Network: I want to access internet from the VM, so I am using NAT mode.
- Install the OS using the ISO file.
- Storage location: `/var/lib/libvirt/images/kali-2025-T3.qcow2`

## Update the Firewall Rules by switching from nftables to iptables-nft

- Write a script to set up the firewal rules

```bash
#!/bin/bash
# filepath: /usr/local/bin/firewall-setup-fixed.sh

echo "Setting up corrected firewall rules..."

# Enable IP forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward

# Clear existing rules
iptables-nft -F
iptables-nft -t nat -F
iptables-nft -X

# Set correct default policies
iptables-nft -P INPUT DROP
iptables-nft -P FORWARD DROP
iptables-nft -P OUTPUT ACCEPT  # THIS MUST BE ACCEPT!

# === INPUT CHAIN ===
iptables-nft -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables-nft -A INPUT -m state --state INVALID -j DROP
iptables-nft -A INPUT -i lo -j ACCEPT
iptables-nft -A INPUT -p icmp -j ACCEPT
iptables-nft -A INPUT -p tcp --dport 22 -j ACCEPT

# DHCP/DNS on eno1 (LAN)
iptables-nft -A INPUT -i eno1 -p udp --dport 67 -j ACCEPT
iptables-nft -A INPUT -i eno1 -p udp --dport 68 -j ACCEPT
iptables-nft -A INPUT -i eno1 -p udp --dport 53 -j ACCEPT
iptables-nft -A INPUT -i eno1 -p tcp --dport 53 -j ACCEPT

# DHCP/DNS on virbr0 (VMs)
iptables-nft -A INPUT -i virbr0 -p udp --dport 67 -j ACCEPT
iptables-nft -A INPUT -i virbr0 -p udp --dport 68 -j ACCEPT
iptables-nft -A INPUT -i virbr0 -p udp --dport 53 -j ACCEPT
iptables-nft -A INPUT -i virbr0 -p tcp --dport 53 -j ACCEPT

# Tailscale
iptables-nft -A INPUT -i tailscale0 -p tcp --dport 21112 -j ACCEPT

# === FORWARD CHAIN ===
iptables-nft -A FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables-nft -A FORWARD -i virbr0 -o wlp1s0 -j ACCEPT  # VM internet
iptables-nft -A FORWARD -i eno1 -o wlp1s0 -j ACCEPT    # LAN internet
iptables-nft -A FORWARD -i virbr0 -o virbr0 -j ACCEPT  # VM to VM

# === NAT RULES (CRITICAL!) ===
# NAT for LAN devices
iptables-nft -t nat -A POSTROUTING -s 192.168.50.0/24 -o wlp1s0 -j MASQUERADE

# NAT for VMs
iptables-nft -t nat -A POSTROUTING -s 192.168.122.0/24 -o wlp1s0 -j MASQUERADE

echo "Firewall rules applied successfully!"

# Test connectivity
echo "Testing host internet..."
ping -c 2 8.8.8.8

echo "=== Current Rules ==="
iptables-nft -L -n -v --line-numbers
echo "=== NAT Table ==="
iptables-nft -t nat -L -n -v
```

### Debug Commands

- `sudo iptables-nft -L -n -v --line-numbers` to list current firewall rules.
- `sudo iptables-nft -t nat -L -n -v` to list NAT table rules.
- `sudo systemctl restart libvirtd` to restart the libvirt service if needed.
