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

Now you can log in with `ssh <server/alias>`. Also note that the local computer name and ip address can be set within gnome settings.

## First steps

Then, I installed KeePassXC and Dropbox from Software, so I could log into other services. On Firefox, I installed HTTPS Everywhere, Privacy Badger, and Grammarly. A simple firewall is set up with gufw from the pop store. In settings I assigned a static IP and manually set my DNS (google: 8.8.8.8,8.8.4.4, cloudflare: 1.1.1.1,1.0.0.1). Also, up the Blank screen time to 15 minutes in Power.

## Install some software

Then I installed the following from Pop Shop:

```
octave
scribus
krita
gimp
inkscape
blender
audacity
cheese
musescore
manuskript
thunderbird
devhelp
mpv
vlc
obs
pdf arranger
hardinfo
wxmaxima
geogebra
gnome web
filezilla
builder
glade
chromium
vscode
virtualbox
lmms
godot
sonic pi
todo
onlyoffice
libresprite
```

The following from apt:

```
texlive-full
python-pygments
racket
gnome-shell-extensions
gimp
ssh
tmux
fonts-hack fonts-firacode fonts-linuxlibertine
bashtop
mesa-utils
cmus
ubuntu-restricted-extras
espeak-ng mbrola mbrola-en* mbrola-us*
```

The following from the internet: 

```
[OnlyOffice](https://www.onlyoffice.com/)
[Language Tool](https://languagetool.org/)
[Hemminway Editor](http://www.hemingwayapp.com/) (filefox download + installed as app with Gnome Web)
[RStudo](https://rstudio.com/products/rstudio/)]
```

## Windows VM

Here's what I put on my Virtual Machine for right now.
[Windows 10 Pro](https://www.microsoft.com/en-ca/software-download/windows10ISO),
[Visual Studio](https://visualstudio.microsoft.com/vs/community/), and
[VSCELicence](https://github.com/beatcracker/VSCELicense).

## Development stuff

Zero-configuration development envrionments are used like any other app. For example, LaTeX, RStudio, Octave, Godot, and Racket. Complex development environments (and their vscode extentions) are isolated in seperate `$HOME` directories to avoid contamination.

In vscode, I installed Vim, Code Spell Checker, Todo Tree, Rewrap, LaTeX workshop, and markdownlint to vscode. My `settings.json` look like this

```
{
    "window.menuBarVisibility": "toggle",
    "workbench.editor.showTabs": false,
    "todo-tree.tree.showScanModeButton": false,
    "editor.fontFamily": "'Fira Code','Droid Sans Mono', 'monospace', monospace, 'Droid Sans Fallback'",
    "editor.fontLigatures": true,
    "editor.wordWrap": "wordWrapColumn",
    "rewrap.autoWrap.enabled": true,
    "rewrap.wrappingColumn": 80,
    //"editor.rulers": [80],
    "workbench.activityBar.visible": true,
    "editor.renderIndentGuides": false
}

### Generic Development Envrionments

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