# Server setup

These are my notes tracking how my Raspberry Pi server is setup, for reproduction purposes. The target audience is me, so it may not be easy to read.

# Install Raspbian via NOOBS

I formatted the SD card with Gnome disks (msdos/fat), and extracted [NOOBS](https://www.raspberrypi.org/downloads/noobs/) onto it. The idea was to set up the Pi by my workstation (over WiFi), and then move it home (over ethernet). I booted into it, and installed Raspbian Lite. It can be logged into with pi:raspberry. I configured the Pi via `raspi-config`. I changed the password for `pi`, set the hostname, minimised GPU memory (16), and enabled SSH. I didn't want the pi to connect via WiFi, so I deleted the configuration in `/etc/wpa_supplicant/wpa_supplicant.conf`.

The server needs a local static IP address. Find the router IP on the workstation with `ip -r`. Find the interface on the pi with `ifconfig`. You can use `/etc/resolv.conf` for your ISP's DNS, or you can use Google's (8.8.8.8,8.8.4.4) or Cloudflares (1.1.1.1,1.0.0.1). Edit `/etc/dhcpcd.conf` and add the following:

```
interface <INTERFACE>
static ip_address=<STATICIP>/24
static routers=<ROUTERIP>
static domain_name_servers=<DNSIP (spaces in between IPs)>
```

I was then able to move the Pi to it's home over ethernet, and use it remotely.