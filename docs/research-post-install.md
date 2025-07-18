# Research Post-install

## Research

- **What is Arch Linux?**:
  - a linux distribution know for simplicity as without unnecessary additions or modifications.
  - It is in rolling release model, meaning if there are updates, they are applied continuously.
  - It is without a system-wide GUI, so we need to use CLI at first, and find a good GUI package later.
- **Why Arch Linux?**
  - I want to challenge myself to deeply understand the linux system and get experience on justify the choices I make.
- **Hardware Compatibility:** Ensure the mini-PC hardware is compatible with Arch Linux.
  - I already have a mini-PC which on Windows 10 with the following specs:
    - Intel I5-6600T with 4 cores and 2.7GHz
    - 8GB RAM
    - 256GB SSD
    - 2TB HDD (Externally connected via USB 3.0)

### Things to know before starting

- **System Administration:**
  - Users and Groups, common linux concepts that manage access and control. Common commands for listing users and groups are `cat /etc/passwd` and `cat /etc/group`.
  - Service Management, Arch Linus uses `systemd` for service managment.
    - Common commands include:
      - `systemctl start <service>`: Start a service.
      - `systemctl stop <service>`: Stop a service.
      - `systemctl enable <service>`: Enable a service to start at boot.
      - `systemctl disable <service>`: Disable a service from starting at boot.
      - `systemctl status <service>`: Check the status of a service.
  - System Maintenance, use `systemctl --failed` to check for failed services. A proper Backup and Recovery program is needed.
- **Package Management**, Arch Linux use `pacman` as its package manager, need more research on how to use it.

- **GUI**

  - Surely we need a GUI to manage the server, in [Arch Wiki/GUI](https://wiki.archlinux.org/title/Category:Graphical_user_interfaces) have more resources.

- **More Aspects that need to be considered:**
  - too many things to research now, so TODO :)
  1. Booting
  2. GUI (important for myself)
  3. Power Management
  4. Networking (important for the server)
  5. Multimedia
  6. Input Devices
  7. System services
  8. Appearance (important for myself)
  9. Optimisation
  10. Security

## References

- [Arch Linux General Recommendations](https://wiki.archlinux.org/title/General_recommendations)
- [Arch Linux Installation Guide](https://wiki.archlinux.org/title/Installation_guide)
