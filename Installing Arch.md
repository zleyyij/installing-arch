#linux 
#documentation 

See offical documentation [here](https://wiki.archlinux.org/title/installation_guide).
### Preperation
**Prerequisites**
Check if system is in UEFI mode with:
	`ls /sys/firmware/efi/efivars`
If it lists the directory without issue then it's in EFI mode, if `no 	directory exists` is returned than EFI is disabled.

**Partitioning Disks**
List disks with:
`fdisk -l`
Select disk to modify with:
`fdisk /dev/sdx` (where x is the selected disk)
In the *fdisk* environment, use:
- `n` to create a new partition
- `p` or `e` to select primary or extended, primary is fine for all partitions on this install.
- `+[size][scale]` EG: `+512M` or `+1g` create a partition of that size.
- `w` to write changes to the disk.

Arch requires: 
- 1 *EFi system partition* of at least **300MiB** mounted to **/mnt/boot** (Only for UEFI systems)
- 1 *root partition* using **all leftover space** mounted at **/mnt**
- Optionally a *swap partition* **more than 512MiB**, but ideally equal to the total ram.(Note: the swap partition is not mounted, but is instead initialized with `mkswap /dev/swap_partition`)

**Formatting Partitions**
- Format the root partition as ext4
```bash
mkfs.ext4 /dev/root_partition
```
- Format the EFI system partition as Fat32
```bash
mkfs.fat -F 32 /dev/efi_system_partition
```
- Initialize the swap partition
```bash
swapon /dev/_swap_partition_
```
**Mounting Partitions**
Mount these partitions with `mount /dev/sdxI /mounting_location`, for this purpose `/mnt`.
You will need to:
- Mount root partition(/)
```bash
mount /dev/[root_partition] /mnt
```
- Mount EFI System Partiton
(boot may need to be made with `mkdir`)
```bash
mount /dev/[efi_system_partition] /mnt/boot
```
- If a swap partition was made, initialize it with:
```bash
swapon /dev/[swap_partition]
``` 
**Update Mirrors**
		Use `reflector` to update mirrors to the optimal servers.
### Installation
Use `pacstrap` to install necessary packages into `/mnt`
```bash
pacstrap /mnt base linux linux-firmware dhcpcd nano
```
Note: if you wish to configure a static IP you can remove `dhcpcd`from the package list and use `ip address add [IP]/[subnet] broadcast + dev [interface]`, where subnet is usually `/24` and the interface can be found with `ip link`. You will need to add a route, which can be done with `ip route add default via [gateway address]`.
**Configuring the system**
Generate an `fstab` file(shows how partitions should translate into disk space)
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```
**Chroot into the new system** with `arch-chroot`
```bash
arch-chroot /mnt
```
Optionally set time zone(See installation wiki)

Edit `/etc/locale.gen` to set locales. For English uncomment `en_US.UTF-8 UTF-8`.
If needed, generate locales with `locale-gen`.

Start and enable internet
```bash
systemctl start dhcpcd
systemctl enable dhcpcd
```
**User Account Management**
Set a root password with `passwd`.
Install `sudo` 
```bash
pacman -S sudo
```

Add a user 
```bash
useradd -m [user]
```

Give the user a password:
```bash
passwd [user]
```

Add the user to the sudoers group(you will need to uncomment a line from /etc/sudoers)
```bash
usermod -aG wheel [user]
```

Reboot, unmount the .iso, and login to the user just created.

Install either `amd-ucode` or `intel-ucode` depending on the system processor
```bash
sudo pacman -S [needed package]
```

**Beyond the install**
Install `grub` and`efibootmgr` with pacman
```bash
pacman -S grub
pacman -S efibootmgr
```

Create the directory for grub
```bash
mkdir /boot/EFI/GRUB
```

Install `grub` to the efi partition.
```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub
```

Generate the grub config
```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

Install `xorg-server`
```bash
sudo pacman -S xorg-server
```

Install `plasma`
```bash
sudo pacman -S plasma
```

Install `sddm`
```bash
sudo pacman -S plasma
```

Start sddm
```bash
sudo systemctl start sddm.service
```
If the desktop works as intended, enable sddm to launch at boot
```bash
sudo systemctl enable sddm.service
```