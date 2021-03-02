# Workstation setup

These are notes for tracking how to reproduce my workstation. The target audience is me, so it may not be easy to read. Applications that don't relate to programming, mathematics, multimedia and the system aren't included in here.

## Install Pop!_OS

First I installed Windows, as some software I need for school cannot be ran in a virtual machine. Then, I installed [Pop!_OS 20.10](https://pop.system76.com/), with a btrfs filesystem.

## Timeshift

The first thing I did with the system was set up backups with timeshift. I installed Timeshift from the Pop store, and created the `@` and `@home` subvolumes.

```
sudo btrfs sub snapshot / /@
sudo btrfs sub cr /@home
sudo vi /etc/fstab
# Added `subvol=@ option` to `/` mount point.
# Created `/home` mount point with option `subvol=@home`.
sudo vi /boot/efi/loader/entries/Pop_OS-current.conf
# Added `rootflags=subvol=@` to end of options.
sudo vi /etc/kernelstub/configuration
# Added `rootflags=subvol=@` to kernel options.
```

Upon rebooting, press Ctrl-Alt-F2 and create a `/home/<username>` directory. Now we can create backups in timeshift.

## Settings related to the server

Since my ISP does not allow hairpinning, we can add the local server IP to `/etc/hosts`. Generate SSh keys with `ssh-keygen -t ed25519`, and install them onto the server with `ssh-copy-id <server>`. Password authentification can then be removed on the server in `/etc/ssh/sshd_config`. To make sure ssh logs into the user `pi` automatically, add the following to `~/.ssh/config`

```
Host <server/alias>
    HostName <server>
    User pi
```

Create an ssh-agent with `ssh-agent bash`, and add the key `ssh-add ~/.ssh/id_ed25519`. Now you can log in with `ssh <server/alias>`. Also note that the local computer name and ip address can be set within gnome settings.

## First steps

Then, I installed KeePassXC and Dropbox from Software, so I could log into other services. On Firefox, I installed HTTPS Everywhere, Privacy Badger, and Grammarly. A simple firewall is set up with gufw from the pop store. In settings I assigned a static IP and manually set my DNS (google: 8.8.8.8,8.8.4.4, cloudflare: 1.1.1.1,1.0.0.1). Also, up the Blank screen time to Never in Power.

Create a new bashrc, and set up a simple prompt

```
touch ~/.bashrc
vi ~/.bashrc
```

And add the following

```
export PS1="\[\033[44m\]\u@\h:\w\[\033[00m\]\n\$ "
```

## Install some software

Then I installed the following from Pop! Shop:

```
krita
gimp
inkscape
audacity
cheese
musescore
thunderbird
mpv
vlc
obs
hardinfo
wxmaxima
gnome web
filezilla
chromium
lmms
sonic pi
libresprite
rhythembox
```

The following from apt:

```
gnome-shell-extensions
gimp
ssh
tmux
bashtop
mesa-utils
ubuntu-restricted-extras
traceroute
```

## Office Software

I installed the following from the internet:

* [Language Tool](https://github.com/languagetool-org/languagetool)
* [Hemminway Editor](http://www.hemingwayapp.com/) (filefox download + installed as app with Gnome Web)

The following was installed via apt:

```
libreoffice libreoffice-sdbc-hsqldb
texlive-full
python-pygments
fonts-hack fonts-firacode ubuntustudio-fonts
espeak-ng mbrola mbrola-en* mbrola-us*
```

The following was installed via Pop! Shop:

```
scribus
manuskript
gnome todo
pdf arranger
```

## Blender

I installed Blender from the Pop! Shop, and installed the [UvSquares](https://github.com/Radivarig/UvSquares) plugin.

## Windows VM

I installed VirtualBox from Pop! Shop. Here's what I put on my Virtual Machine for right now.

* [Windows 10 Pro](https://www.microsoft.com/en-ca/software-download/windows10ISO)
* [Visual Studio](https://visualstudio.microsoft.com/vs/community/)
* [VSCELicence](https://github.com/beatcracker/VSCELicense)

## Mathematics

I installed [RStudo](https://rstudio.com/products/rstudio/) from the internet.

From Pop! Shop, I installed:

```
Geogebra
Octave
```

## Development stuff

I installed the following from apt
```
racket
godot3
hugo
npm nodejs
```

From Pop! Shop I installed:

```
devhelp
glade
builder
neovim
```

I installed [Emacs](https://www.gnu.org/software/emacs/) from source, and [Spacemacs](https://www.spacemacs.org/).

Top install Emacs from source, run:
```
apt build-dep emacs
apt install libgnutls28-dev libwebkit2gtk-4.0-dev mailutils libgccjit-10-dev
git clone git://git.savannah.gnu.org/emacs.git -b feature/native-comp
./autogen
./configure --with-native-compilation --with-x-toolkit=gtk3 --with-cairo --with-xwidgets
make
make install
git clone -b develop https://github.com/syl20bnr/spacemacs ~/.emacs.d
```

In Visual Studio I installed the Vim and Terminal extensions.

### TODO: Create Rust and Haskell Generic Development Environments.

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

I installed the latest version of LLVM+Clang that GHC supports (try running ghc on a simple .hs file with `-fllvm` to see which version it supports). I also installed valgrind, latex, and hugo. We can search avalible versions with

```
nix-env -qa 'llvm.*'
nix-env -qa 'clang.*'
nix-env -qa 'texlive.*'
nix-env -qa valgrind
nix-env -qa hugo
```

Then install them with

```
nix-env -i llvm-9.0.1
nix-env -i clang-wrapper-9.0.1
nix-env -i texlive-combined-full-2019
nix-env -i hugo-0.69.0
```