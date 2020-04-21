# Install Debian Buster

I used the [Debian 10.3.0 amd64 netinst /w non-free firmware](https://cdimage.debian.org/cdimage/unofficial/non-free/cd-including-firmware/10.3.0+nonfree/amd64/iso-cd/) image for installation. The image was written to a USB stick using Gnome Disks. To avoid graphical bugs, I booted the USB straight from the graphical bios setup to give it a better video buffer. Debian was installed with the Gnome Desktop and LVM+encryption.

# Ryzen 2400G Graphics

I booted into recovery mode from GRUB, and connected to ethernet with `systemctl start NetworkManager-wait-online`. Then, I edited `/etc/apt/sources.list` to include the following sources
```
# For backported linux kernel
deb <server> buster-backports main
deb-src <server> buster-backports main
```
A backported kernel and firmware was installed
```
apt update
apt -t buster-backports install linux-image-amd64 firmware-amd-graphics amd64-microcode

```

# First Things

I added main users to sudoers with `gpasswd -a brandon sudo`.

I installed KeePassXC and Dropbox so I could log into other services. A simple firewall is set up.
```
apt install ufw
ufw enable
ufw default deny incoming
ufw default allow outgoing
```