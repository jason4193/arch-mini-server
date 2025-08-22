## Installation Media & Verification (DONE)

Goal: Obtain authentic ISO file, verify its integrity, and create bootable media.

1. Download the Arch Linux ISO from the official website. (link)[https://mirror.aarnet.edu.au/archlinux/iso/latest/]
2. Verify the ISO file by checking its SHA256 checksum against the one provided on the Arch Linux website.

```powershell
$sha256sum = "expected_sha256_checksum_here"
$sha256sum -eq (Get-FileHash -Path "path\to\archlinux.iso" -Algorithm SHA256).
Hash
# expected to return True if the checksum matches
```

3. Verify the GPG signature of the ISO file using the Arch Linux keys using Gpg4win

4. Create a bootable USB drive using a tool Rufus

- Boot selection: Disk or ISO image (Please select) `archlinux-2025.08.01-x86_64.iso`
- Persistent partition size: 0 MB (why? because we don't need persistence for installation)
- Partition scheme: MBR (for compatibility with most systems, not using GPT as is not necessary for a single boot Arch Linux installation and our 2TB HDD is MBR partitioned will reset that after installation)
- Target system: BIOS (or UEFI-CSM) (for compatibility with most systems)
- File system: FAT32 only for the EFI System Partition (EFI stand for Extensible Firmware Interface, a modern replacement for BIOS)

## Disk Partitioning & Filesystems

Goal: Simple partitioning scheme for both SSD and HDD. Good to have snapshot capabilities.

### Filesystme Decision Matrix

- ext4: Very stable, minimal maintenance, No native snapshots
- btrfs: Supports snapshots, but requires more maintenance and knowledge.
- xfs: Great for large sequential file and big directories, but no native snapshots.
- zfs: Advance filesysem not consider for this project
- FAT32: Good for boot partition, but not suitable for root or home partitions.z

### Partitioning Scheme

- SSD (256GB):
  - `/boot` (512MB, FAT32): For boot files, compatible with both BIOS and UEFI.
  - `/` (rest of the SSD, ext4): Root filesystem
- HDD (2TB):
  - `/home` (2TB, ext4): User data and home directories, no

## Bootloader & Microcode

## Kernel & Modules

## Booting

## Package Management

## Update Strategy & Mirrors / AUR

## System Services

## Logging & Journaling

## Users, Groups & Shell Environment

## Time, Locale & Fonts

## Networking

## Remote Access & File Sharing

## Firewall & Security Hardening

## Encryption / LUKS / Secrets

## GUI

## Appearance

## Input Devices

## Power Management

## Hardware Acceleration & Drivers

## Multimedia

## Printing & Scanning

## Virtualization & Containers

## Development Tools

## Monitoring & Diagnostics

## Backup & Snapshots

## Automation (Cron / systemd timers)

## Accessibility

## Sync & Cloud

## Gaming / Optional Extras

## Documentation & Notes

## Recovery / Rescue

## Optimisation
