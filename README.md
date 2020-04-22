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

A backported kernel and firmware was installed with

```
apt update
apt -t buster-backports install linux-image-amd64 firmware-amd-graphics amd64-microcode
```

I rebooted the system with `reboot`.

## First steps

I added main users to sudoers with `gpasswd -a brandon sudo`.

I installed KeePassXC and Dropbox from Software, so I could log into other services. A simple firewall is set up with

```
sudo apt install ufw
reboot
sudo ufw enable
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

In Tweaks, improve font rendering with Slight Hinting and Standard Antialiasing. Enable Launch new instance.

## Install some software

I installed flatpak for software that seems abandoned by Debian with

```
sudo apt install flatpak gnome-software-plugin-flatpak
sudo flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
reboot
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
polari
games*
```

And the follwoing from the command line

```
sudo apt install build-essential
sudo apt install net-tools
sudo apt install curl
sudo apt install git
sudo apt install tmux
sudo apt install vim
sudo apt install mc
sudo apt install texlive-full python-pygments
sudo apt install pandoc
sudo apt install fonts-hack fonts-firacode fonts-linuxlibertine
sudo apt install sgt-puzzles
sudo apt install irssi
```

## External Development tools

I Installed [Nix package manager](https://nixos.org/nix/), [Rust](https://rustup.rs/), [Stack](https://haskellstack.org), and [Ghcup](https://haskell.org/ghcup/) (ghcup requires you to run `sudo apt install libgmp-dev libffi-dev libncurses-dev libtinfo5`).

I rebooted here to add everything into my PATH.

Then upgraded GHC to latest version with

```
ghcup install latest
ghcup set latest
```

Then downloaded default Stack with `stack setup`, and then install these Haskell tools with

```
stack install hoogle
hoogle generate
stack install hlint
stack install brittany
```

I installed the latest version of LLVM+Clang that GHC supports. First, searched for avalible versions with

```
nix-env -qa 'llvm.*'
nix-env -qa 'clang.*'
```

Then installed them with

```
nix-env -i llvm-9.0.1
nix-env -i clang-wrapper-9.0.1
```