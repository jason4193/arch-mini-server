# Display Setup

- GUI is essential. And ricing would be fun.
- I choose to use `Wayland` as the display server protocol rather than `X11`.
- `KDE Plasma` is my current decision for the desktop environment. From the research, it seems to be more stable and user-friendly compared to `Hyprland` and `Sway`.
- I am going to use `SDDM` as the display manager, which recommended by KDE Plasma.

## Current Setup

- Display Server Protocol: `Wayland`
- Desktop Environment: `KDE Plasma` (use `plasma-meta` instead of `plasma` to avoid installing unnecessary packages)
- Display Manager: `SDDM`
- File Manager: `Dolphin` the default file manager for KDE Plasma
- Terminal `Kitty` which is popular and highly customizable
- Text Editor: `Neovim` already installed

## Installation

1. `pacman -S --needed wayland` to install Wayland.
2. `pacman -S --needed sddm` to install SDDM. (need to choose ttf-font)

- We choose `noto-fonts` for the font.

3. `pacman -S --needed plasma-meta` to install KDE Plasma.
4. `pacman -S --needed dolphin kitty konsole kate gwenview okular spectacle ark print-manager`

- `dolphin`: the default file manager for KDE Plasma
- `kitty`: a fast, feature-rich, GPU-based terminal emulator
- `konsole`: the default terminal emulator for KDE Plasma
- `kate`: a powerful text editor with advanced features (will use `neovim` instead)
- `gwenview`: an image viewer for KDE Plasma
- `okular`: a document viewer for KDE Plasma
- `spectacle`: a screenshot utility for KDE Plasma
- `ark`: an archive manager for KDE Plasma
- `print-manager`: a print management utility for KDE Plasma
- Require dependencies of `pipewire-jack`(for audio support),`qt6-multimedia-ffmpeg` (for multimedia support)

5. `pacman -S --needed noto-fonts noto-fonts-cjk noto-fonts-emoji` to install Noto fonts.
6. `pacman -S --needed pipewire wireplumber xdg-desktop-portal-kde networkmanager bluez bluez-utils tuned-ppd` to install additional packages for audio, network, and power management.
   - `pipewire`: a multimedia server for handling audio and video streams
   - `wireplumber`: a session and policy manager for PipeWire
   - `xdg-desktop-portal-kde`: provides a portal interface for KDE applications
   - `networkmanager`: a network management service
   - `bluez` and `bluez-utils`: Bluetooth support
   - `tuned-ppd`: a power management daemon for tuning system performance
7. `pacman -S --needed discover packagekit-qt5` to install additional package management tools.

   - `discover`: a software center for KDE Plasma
   - `packagekit-qt5`: a Qt5 interface for PackageKit, allowing easy installation and management of software packages

8. Enable services

   - `systemctl enable sddm` to enable SDDM as the display manager.
   - `systemctl enable NetworkManager` to enable NetworkManager for network management.
   - `systemctl enable bluetooth` to enable Bluetooth support. ()
   - `systemctl enable pipewire pipewire-pulse wireplumber` to enable PipeWire for audio management.
   - `systemctl enable tuned-ppd` to enable the power management daemon.

9. Create a user account

   - `useradd -m -G wheel -s /bin/bash <username>` to create a new user account with sudo privileges.
   - Set a password for the new user with `passwd <username>`.
   - Enable sudo privileges for the user by editing the sudoers file with `visudo` and uncommenting the line `%wheel ALL=(ALL) ALL`.

10. Reboot the system
    - After completing the installation and configuration, reboot the system to start using KDE Plasma with Wayland.

## Post-Installation

- I want to customise the SDDM login screen.
- I update the config file at `/etc/sddm.conf` to set the theme to `breeze` (the default theme for KDE Plasma).
