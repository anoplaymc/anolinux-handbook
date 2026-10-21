# Welcome to the AnoLinux Handbook
## By this handbook you can install AnoLinux manually
### Stage 1: Booting
Download an Arch Linux Live ISO image from https://archlinux.org/ and write it on flash drive or on Virtual Machine and boot from iit
### Stage 2: Partitioning disk (for uefi)
##### If you need to do custom partition - do it yourself
##### First, run `lsblk` to see your disk name (e.g., `sda`, `vda`, or `nvme0n1`), and set it to a variable ("$DISK")
#### Create GPT label and partitions (EFI 512MiB + Root rest)
parted $DISK --script mklabel gpt

parted $DISK --script mkpart ESP fat32 1MiB 512MiB

parted $DISK --script set 1 boot on

parted $DISK --script mkpart primary ext4 512MiB 100%
#### Format partitions (Adjust partition numbers if your disk uses p-suffixes like nvme0n1p1)
mkfs.fat -F32 ${DISK}1

mkfs.ext4 -F ${DISK}2
#### Mount root and boot
mount ${DISK}2 /mnt

mkdir -p /mnt/boot

mount ${DISK}1 /mnt/boot
### Stage 3: Installing base
#### Installing base system
pacstrap /mnt base base-devel linux linux-firmware networkmanager sudo
#### Generating fstab file
genfstab -U /mnt >> /mnt/etc/fstab
#### Entering "chroot" to set up the system
arch-chroot /mnt
#### Setting locale and timezone
ln -sf /usr/share/zoneinfo/UTC /etc/localtime

hwclock --systohc

echo "en_US.UTF-8 UTF-8" > /etc/locale.gen

locale-gen

echo "LANG=en_US.UTF-8" > /etc/locale.conf
#### Setting hostname
echo "anolinux" > /etc/hostname
echo -e "127.0.0.1\tlocalhost\n::1\tlocalhost\n127.0.1.1\tanolinux.localdomain\tanolinux" > /etc/hosts
#### Setting users and passwords (change "myuser" to your username)
passwd

useradd -m -G wheel -s /bin/bash myuser

passwd myuser

sed -i 's/# %wheel ALL=(ALL:ALL) ALL/%wheel ALL=(ALL:ALL) ALL/' /etc/sudoers
#### Enabling NetworkManager
systemctl enable NetworkManager
#### Installing GRUB (for uefi)
pacman -S --noconfirm grub efibootmgr

grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=AnoLinux --recheck

grub-mkconfig -o /boot/grub/grub.cfg
#### Stage 4: Installing AnoLinux system
bash <(curl -sSL https://raw.githubusercontent.com/anoplaymc/anolinux-handbook-files/refs/heads/main/stage-4.sh)
#### Stage 5: Configuring AnoLinux packages & drivers
bash <(curl -sSL https://raw.githubusercontent.com/anoplaymc/anolinux-handbook-files/refs/heads/main/stage-5.sh)
#### Final stage 6: Exit from "chroot", umount and reboot to the system
exit

umount -R /mnt

reboot
