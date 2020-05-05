# Workstation setup

These are notes for tracking how to reproduce my workstation. The target audience is me, so it may not be easy to read (use the [Debian Installation Guide](https://www.debian.org/releases/stable/amd64/) instead). Applications that don't relate to programming, mathematics, multimedia and the system aren't included in here.

## Install Debian Buster

I used the [Debian 10.3.0 amd64 netinst /w non-free firmware](https://cdimage.debian.org/cdimage/unofficial/non-free/cd-including-firmware/10.3.0+nonfree/amd64/iso-cd/) image for installation. The image was written to a USB stick using Gnome Disks. To avoid graphical bugs, I booted the USB straight from the graphical bios setup to give it a better video buffer. Debian was installed with the Gnome Desktop and LVM+encryption.

## Backported kernel (graphics firmware)

I booted into recovery mode from GRUB, and connected to ethernet with `systemctl start NetworkManager-wait-online`. Then, I edited `/etc/apt/sources.list` to include the following sources

```
# For backported linux kernel
deb <server> buster-backports main
deb-src <server> buster-backports main
```

A backported kernel and firmware was installed with

```
apt update
apt upgrade
apt -t buster-backports install linux-image-amd64 firmware-amd-graphics amd64-microcode
reboot
```

## First steps

I added main users to sudoers with `gpasswd -a brandon sudo`. Then, I installed KeePassXC and Dropbox from Software, so I could log into other services. On Firefox, I installed HTTPS Everywhere, Privacy Badger, Clear Cache, and Grammarly. A simple firewall is set up with

```
sudo apt install ufw
reboot
sudo ufw enable
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

In Tweaks, improve font rendering with Slight Hinting and Standard Antialiasing. Enable Launch new instance. In settings I assigned a static IP and manually set my DNS (google: 8.8.8.8,8.8.4.4, cloudflare: 1.1.1.1,1.0.0.1).

## Install some software

I installed flatpak for software where I wanted the latest version/

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
blender*   # For 2.8
audacity
musescore* # For 3
mpv        # Set this as default Video player
obs
pdfsam basic
hardinfo
wxmaxima
geogebra*  # Run with `flatpak run org.geogebra.GeoGebra`
gnome web
hexchat    # Set the font to Sans 11
filezilla
ghostwriter
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
sudo apt install fonts-hack fonts-firacode fonts-linuxlibertine
sudo apt install irssi
sudo apt install python-pygments # For latex minted package
```

I also downloaded the Desktop version of [Language Tool](https://languagetool.org/), and downloaded the [Hemminway](http://www.hemingwayapp.com/) (with firefox) to help with my writting.

## Development stuff

I don't have any Rust projects going on, so this section is mostly focused on Haskell for now. For Ghcup dependancies, install `sudo apt install libgmp-dev libffi-dev libncurses-dev libtinfo5`. Then I Installed [Nix package manager](https://nixos.org/nix/), [Rust](https://rustup.rs/), [Stack](https://haskellstack.org), and [Ghcup](https://haskell.org/ghcup/). I rebooted here to add everything into my PATH. Then upgraded GHC to latest version with

```
ghcup install latest
ghcup set latest
```

Then downloaded default Stack with `stack setup`, and then install some tools with

```
stack install hoogle
hoogle generate
stack install hlint
stack install brittany
```

I installed the latest version of LLVM+Clang that GHC supports (try running ghc on a simple .hs file with `-fllvm` to see which version it supports). I also installed latex and hugo. We can search avalible versions with

```
nix-env -qa 'llvm.*'
nix-env -qa 'clang.*'
nix-env -qa 'texlive.*'
nix-env -qa hugo
```

Then install them with

```
nix-env -i llvm-9.0.1
nix-env -i clang-wrapper-9.0.1
nix-env -i texlive-combined-full-2019
nix-env -i hugo-0.69.0
```

Finally I installed [vscode](https://code.visualstudio.com/). I installed Code Spell Checker, Todo Tree, Rewrap, Simple GHC, Brittany, Haskell Linter, and markdownlint. My `settings.json` look like this

```
{
    "window.menuBarVisibility": "toggle",
    "todo-tree.tree.showScanModeButton": false,
    "editor.fontFamily": "'Fira Code','Droid Sans Mono', 'monospace', monospace, 'Droid Sans Fallback'",
    "editor.fontLigatures": true,
    "editor.wordWrap": "wordWrapColumn",
    "rewrap.autoWrap.enabled": false,
    "workbench.colorTheme": "Default Light+",
    "editor.minimap.enabled": false,
    "workbench.editor.showTabs": false,
    "git.autofetch": true,
    "editor.rulers": [80],
    "[markdown]":{
        "editor.wordWrap": "wordWrapColumn",
    },
}
```

## Settings related to the server

Since my ISP does not allow hairpinning, we can add the local server IP to `/etc/hosts`. Generate SSh keys with `ssh-keygen -t ed25519`, and install them onto the server with `ssh-copy-id <server>`. Password authentification can then be removed on the server in `/etc/ssh/sshd_config`.