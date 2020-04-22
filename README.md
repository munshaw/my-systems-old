# Workstation setup

These are my notes tracking how my workstation is setup, for reproduction purposes. The target audience is me, so it may not be easy to read.

## Install Debian Buster

I used the [Debian 10.3.0 amd64 netinst /w non-free firmware](https://cdimage.debian.org/cdimage/unofficial/non-free/cd-including-firmware/10.3.0+nonfree/amd64/iso-cd/) image for installation. The image was written to a USB stick using Gnome Disks. To avoid graphical bugs, I booted the USB straight from the graphical bios setup to give it a better video buffer. Debian was installed with the Gnome Desktop and LVM+encryption.

## Ryzen 2400G graphics

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

## First steps

I added main users to sudoers with `gpasswd -a brandon sudo`.

I installed KeePassXC and Dropbox from Software, so I could log into other services. A simple firewall is set up with

```
sudo apt install ufw
# I needed to restart here.
sudo ufw enable
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

In Tweaks, improve font rendering with Slight Hinting and Standard Antialiasing. Enable Launch new instance.

## Install some apps

I installed flatpak for installed newer versions of some apps.

```
sudo apt install flatpak gnome-software-plugin-flatpak
sudo flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

Then I installed the following from Software (\* = flatpak)

```
krita
gimp
inkscape
blender*
audacity
musescore*
mpv
obs
pdfsam basic
anki
hardinfo
wxmaxima
geogebra*
gnome web
```
