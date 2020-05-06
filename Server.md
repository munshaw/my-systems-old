# Server setup

These are notes for tracking how to reproduce my server. The target audience is me, so it may not be easy to read.

## Install Raspbian via NOOBS

I formatted the SD card with Gnome disks (msdos/fat), and extracted [NOOBS](https://www.raspberrypi.org/downloads/noobs/) onto it. The idea was to set up the Pi by my workstation (over WiFi), and then move it home (over ethernet). I booted into it, and installed Raspbian Lite. It can be logged into with pi:raspberry. I configured the Pi via `raspi-config`. I changed the password for `pi`, set the hostname, minimised GPU memory (16), and enabled SSH. I didn't want the pi to connect via WiFi, so I deleted the configuration in `/etc/wpa_supplicant/wpa_supplicant.conf`.

The server needs a local static IP address. Find the router IP on the workstation with `ip -r`. Find the interface on the pi with `ifconfig`. You can use `/etc/resolv.conf` for your ISP's DNS, or you can use Google's (8.8.8.8,8.8.4.4) or Cloudflares (1.1.1.1,1.0.0.1). Edit `/etc/dhcpcd.conf` and add the following:

```
interface <INTERFACE>
static ip_address=<STATICIP>/24
static routers=<ROUTERIP>
static domain_name_servers=<DNSIP (spaces in between IPs)>
```

I was then able to move the Pi to it's home over ethernet, and use it remotely.

## Some security

After SSH keys are generated for the workstation, disable password authentification in `/etc/ssh/sshd_config`. I set up the following simple firewall for the server.

```
sudo su
apt install ufw
ufw reset
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh
ufw limit ssh
ufw allow http
ufw allow https
ufw enable
```

We should also require a password for root access. Edit `/etc/sudoers.d/010_pi-nopasswd`, and change the last line to `pi ALL=(ALL) PASSWD: ALL`. Make sure to forward these ports through the router (22, 80, 443).

## no-ip

My DDNS is provided by [no-ip](https://www.noip.com/). After the hostname is set up, install the client to keep the IP updated.

```
wget http://www.no-ip.com/client/linux/noip-duc-linux.tar.gz
tar vzxf noip-duc-linux.tar.gz
cd noip-*/
make
sudo make install
sudo noip2
```

Make this autostart with crontab with `sudo crontab -e`, and add `@reboot sudo -u root noip2` to the end of the file.

## Nginx and Let's Encrypt

Nginx is primarliy used to host the Hugo blog, as a reverse proxy for Gitea, and serving static files. The TLS certificate is managed by Let's Encrypt. First, I install nginx and the certbot with `sudo apt install nginx certbot python-certbot-nginx`. The certificate is generated with `sudo certbot --nginx -d domain.com`. To make sure the cert is updated if it expires within 30 days weekly, add `13 2 * * 1 /usr/bin/certbot renew --quiet` to `sudo crontab -e`. Randomise the time to prevent server load.

We can now configure the server by editing `/etc/nginx/sites-available/default`. Comment out the location statements in the default server and ssl server. We first try to display static pages for the blog. If that fails, we try to connect to Gitea, which will run on port 3000. Add the following to the default server and ssl server


```
location / {
    try_files $uri $uri/ @gitea;
}

location @gitea {
    proxy_pass http://localhost:3000;
}
```


Files can be shared by adding the following example location to the default and ssl server. Since root is defaulted to `/var/www/html`, this would be the directory `/var/www/html/my-pictures`. To make user pi able to edit these files directly, use `sudo chown -R pi /var/www/html`. For convience, statically link this to the home directory with `ln -s /var/www/html ~/html`. If you need a random directory to put stuff in, you can use `openssl rand 32 | base64` to generate some characters.

```
location /my-pictures {
    # show directory listing?
    autoindex on;

    # First attempt to serve request as file, then
    # as directory, then fall back to displaying a 404.
    try_files $uri $uri/ =404;
}
```



Run `sudo systemctl enable nginx` and `sudo systemctl start nginx` to start the web service. You can also use `sudo nginx -t` to test your nginx configuration.

## Gitea

[Gitea](https://gitea.io/en-us/) is the primary reason for this servers existance. First we install SQLite and Git with `sudo apt install git sqlite`. We find the latest version of Gitea [here](https://dl.gitea.io/gitea/). Installing is simply a matter of downloading the binary `sudo wget -O /usr/local/bin/gitea https://dl.gitea.io/gitea/1.9.6/gitea-1.9.6-linux-arm-6` and setting the executable bit `sudo chmod +x /usr/local/bin/gitea`. We now add a user for Gitea to run in

```
sudo adduser \
     --system \
     --shell /bin/bash \
     --gecos 'Git Version Control' \
     --group \
     --disabled-password \
     --home /home/git \
     git
```

And create some directories for it to work in

```
sudo su
mkdir -p /var/lib/gitea/{custom,data,log}
chown -R git:git /var/lib/gitea/
chmod -R 750 /var/lib/gitea/
mkdir /etc/gitea
chown root:git /etc/gitea
chmod 770 /etc/gitea
```

The systemd service is installed with `sudo wget -O /etc/systemd/system/gitea.service https://raw.githubusercontent.com/go-gitea/gitea/master/contrib/systemd/gitea.service`. We can now enable Gitea with `sudo systemctl enable gitea` and start it with `sudo systemctl start gitea`. You should be able to log into it with `http://<server-ip>:3000/` if you temporarily unblock port 3000. Once the reverse proxy is set up, so should also be able to log in with `http://<server-ip>/`, `http://<hostname>/`, or `https://<hostname>/`. Set up Gitea to your preference (select SQLite as your database). After the configuration is set, lock it with

```
sudo chmod 750 /etc/gitea
sudo chmod 640 /etc/gitea/app.ini
```

## IRC

I installed the IRC bouncer znc with `sudo apt install znc`. Add the znc-admin user and use it to generate the configuration with

```
sudo adduser znc-admin
sudo su
su znc-admin
znc --makeconf
```

I choose to run znc on port 7000, and keep the rest at the defaults. znc can be accessed via SSH tunnel with `ssh -L 7000:localhost:7000 <user>@<server>`. Log into the web interface at `http://localhost:7000`, or log into `localhost:7000` via irc client.

## Backup

The Gitea server can be backed up with

```
sudo su
cd /var/lib/gitea/
su git
gitea dump -c /etc/gitea/app.ini
exit
mv git*.zip /home/pi
chown pi /home/pi/git*.zip
exit
```

Restoring the backup after unzipping looks something like

```
sudo su
mv app.ini /etc/gitea/app.ini
unzip gitea-repo.zip
mv gitea-repositories /home/git/gitea-repositories
sqlite3 /var/lib/gitea/data/gitea.db <gitea-db.sql
chown -R git:git /etc/gitea /home/git/gitea-repositories /var/lib/gitea/data/gitea.db
chmod 750 /etc/gitea
chmod 640 /etc/gitea/app.ini
```