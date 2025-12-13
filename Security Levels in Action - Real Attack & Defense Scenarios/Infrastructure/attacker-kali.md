# Kali Linux – Attacker Machine Setup

## Purpose
Dedicated attacker workstation for red team / adversary simulation scenarios.
Runs on bare metal to avoid resource contention with the main workstation.

---

## Hardware
- CPU: Intel i5-2320
- RAM: 8 GB DDR3
- Storage: SATA HDD 500GB
- GPU: NVIDIA GTX 1660 SUPER

---

## Installation
- OS: Kali Linux (Installer)
- Firmware: Legacy BIOS (Award)
- Disk layout: Single partition, full disk
- Bootloader: GRUB installed to /dev/sda

---

## Post-install Verification
The following checks were performed to validate system state:

- System and kernel information
- Disk layout
- Network configuration
- Package updates completed successfully

### System overview
![Kali fastfetch](../../assets/infrastructure/kali-attacker/fastfetch+others.png)

### First system upgrade
![Kali first upgrade](../../assets/infrastructure/kali-attacker/first-time-upgrade.png)

---

## System Configuration
- Timezone set correctly
- Core attacker tooling available
- OpenSSH server installed and enabled for:
  - Secure file transfer (SCP / SFTP)
  - Remote management

---

## System Protection
- Timeshift configured for system snapshots
- Snapshot created after base configuration
- Automatic schedule disabled; manual snapshots at major milestones

---

## Status
System is fully operational and ready for attack scenario execution.
