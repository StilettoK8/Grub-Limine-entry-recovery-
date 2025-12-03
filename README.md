# Omarchy Limine Boot Fix Script

This repository contains a simple helper script to (re)install and prioritize the **Limine** bootloader on a UEFI system.

The script:

- Deploys Limine to the target disk
- Copies `limine.conf` to EFI locations
- Removes existing `systemd-boot` UEFI entries (if present)
- Creates a new UEFI boot entry for Limine
- Sets Limine as the **first** boot option in UEFI

> ⚠️ **Warning:** This script modifies your system's UEFI boot entries and targets a specific disk (`/dev/nvme1n1`).  
> Use only if you know what you’re doing and adjust paths to match your system.

> A forced BIOS/firmware update from Windows wiped my Limine UEFI entries.
> This script is meant for **quick recovery** and to save me time when that
> happens again.
>
> It is more or less *vibe-coded with intent* not designed to be universal,
> production-safe, or particularly elegant.
>
> Use it as inspiration, not as a drop-in solution unless your setup closely
> matches mine :)


---

## Script

```bash
#!/usr/bin/bash
sudo limine-deploy /dev/nvme1n1 && \
sudo cp /boot/limine.conf /boot/EFI/limine/limine.conf && \
sudo cp /boot/limine.conf /boot/EFI/BOOT/limine.conf && \
sudo efibootmgr -q --delete-bootnum --bootnum $(efibootmgr | grep -i systemd | awk '{print $1}' | cut -c5-8) 2>/dev/null; \
sudo efibootmgr --create --disk /dev/nvme1n1 --part 1 --loader '\EFI\limine\limine_x64.efi' --label 'Omarchy (Limine)' --unicode --quiet && \
sudo efibootmgr --bootorder $(efibootmgr | grep 'Omarchy (Limine)' | awk '{print $1}' | cut -c5-8),0001,0004 && \
echo "Limine boot fixed and set as first entry"
