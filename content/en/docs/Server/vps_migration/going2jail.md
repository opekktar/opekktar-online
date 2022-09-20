---
title: Going to jail
date: 2022-03-08T23:26:06.000Z
draft: false
description: ''
---
[Matching Blog Post](https://rkey.online/posts/going2jail/)

# Getting thrown in jail

Install BastilleBSD:
```shell
doas pkg update
doas pkg upgrade
doas pkg install bastille
```
Setup Bastille to start at boot:
```shell
doas sysrc bastille_enable=YES
```
I only have one IP, so I use local loop back:
```shell
doas sysrc cloned_interfaces+=lo1
doas sysrc ifconfig_lo1_name="bastille0"
doas service netif cloneup
```
I use Packet Filter firewall so here is my initial /etc/pf.conf: Before moving into the jails.
```shell
ext_if="vtnet0"

set block-policy return
scrub in on $ext_if all fragment reassemble
set skip on lo

table <jails> persist
nat on $ext_if from <jails> to any -> ($ext_if:0)
rdr-anchor "rdr/*"

block in all
pass out quick keep state
antispoof for $ext_if inet
pass in inet proto tcp from any to any port ssh flags S/SA keep state
pass in inet proto tcp from any to any port 80 flags S/SA keep state
pass in inet proto tcp from any to any port 443 flags S/SA keep state
pass in inet proto udp from any to any port 51820
```
I'm still using the host at this point so I have to allow ports: 80, 443, 22 and port 51820 is my WireGuard VPN to proxy to my NextCloud instance in a jail on a server running in my apartment.

With pf configured set it to auto start on boot and fire it up:
```shell
doas sysrc pf_enable=YES
doas service pf start
```
At this point we get a TCP socket error and our connection drops, so we have to ssh back in.

Now it's time to bootstrap Bastille:
```shell
doas bastille bootstrap 13.0-RELEASE update
```
I will be moving Caddy into a jail and also running NextCloud in a jail on my VPS instead of at home and also I will have a jail for my database server that NextCloud uses.
```shell
doas bastille create database 13.0-RELEASE 10.10.10.2
doas bastille create nextcloud 13.0-RELEASE 10.10.10.3
doas bastille create caddy 13.0-RELEASE 10.10.10.4
```
Here is what I have so far:
```shell
bsh ➜  doas bastille list
 JID             IP Address      Hostname                      Path
 caddy           10.10.10.4      caddy                         /usr/local/bastille/jails/caddy/root
 database        10.10.10.2      database                      /usr/local/bastille/jails/database/root
 nextcloud       10.10.10.3      nextcloud                     /usr/local/bastille/jails/nextcloud/root
 ```
 I'm going to do some cheating here just to show ***how awesome jails are.*** Instead of building from scratch I will simply tar my database and nextcloud from my home server to the cloud and then I will simply tar all the websites and the Caddyfile from the host to the jail. Then modify pf and ssh reboot and everything will come up in jails on the VPS. This method which I did not use the first time worked so well I did not even have to turn off my up time monitor and there was no downtime beyond a second for the reboot. ***This is so freaking awesome so here we go:***

 First I will tar up my database and nextcloud jails on my home server
 ```shell
 doas su
 cd /usr/local/bastille/jails/
 tar cvf ~/database.tar database
 tar cvf ~/nextcloud.tar nextcloud
 ```
 You should shut down each jail before taring up the jails. I just turned off syncing on my iPhone and paused DAVX on my android to limit any activity. I also logged out of all web instances I had on NextCloud. (I wanted to see if I could do this with no down time, obviously on a business system I would have maintenance window and shut things down). I moved these files to regular user and change permissions.
I then just scp the files to the VPS instance.
```shell
cd
scp database.tar titania:~/
scp nextcloud.tar titania:~/
```
ssh into cloud instance and reverse the process.
```shell
ssh titania
doas su
bastille stop database
bastille stop nextcloud
cd /usr/local/bastille/jails/
tar xvf /home/user/database.tar
tar xvf /home/user/nextcloud.tar
```
Create regular user in caddy jail and setup SSH for deploying websites from workstation.
```shell
doas bastille console caddy
pkg update
pkg upgrade
pkg install caddy
sysrc caddy_enable="YES"
adduser
exit
```
Enable ssh for caddy jail
```shell
doas bastille sysrc caddy sshd_enable="YES"
```
It will start when we reboot we do not need to start it yet.
We want to copy the web sites and the Caddyfile into the jail also we need our SSH key.
```shell
doas rsync /home/user/ /usr/local/bastille/jails/caddy/root/home/user/
doas rsync /usr/local/etc/caddy/ /usr/local/bastille/jails/caddy/root/usr/local/etc/caddy/
```
The file permissions for the regular user in caddy will need to be fixed
```shell
doas bastille console caddy
chown -R user:group /home/user
```
Also since I am no longer using the WireGuard VPN to hit the Nextcloud instance at home I will need to change the Caddy File in the caddy jail to reflect this.
```shell
vi /usr/local/etc/caddy/Caddyfile
```
Change proxy for nc.rkey.tech
From:
```shell
nc.rkey.tech {
        reverse_proxy 10.20.10.2:80
        header {
        Strict-Transport-Security "max-age=15768000;includeSubDomains;"
        }

        redir /.well-known/carddav /remote.php/dav 301
        redir /.well-known/caldav /remote.php/dav 301

        log {
                output file /var/log/caddy/nc.rkey.tech.access.log
                format json
        }
}
```
To:
```shell
nc.rkey.tech {
        reverse_proxy 10.10.10.3:80
        header {
        Strict-Transport-Security "max-age=15768000;includeSubDomains;"
        }

        redir /.well-known/carddav /remote.php/dav 301
        redir /.well-known/caldav /remote.php/dav 301

        log {
                output file /var/log/caddy/nc.rkey.tech.access.log
                format json
        }
}
```
10.20.10.2:80 was the WireGuard to home and 10.10.10.3:80 is now the NextCloud jail locally.
For ease of use and habit I'm keeping the jail at the standard SSH port for deployments and so I will need to change the host port.  So back on the host I make the following changes.
I change the /etc/ssh/sshd_config file.
From:
```
#Port 22
```
To:
```
Port 5000
```
On the host I also need to keep Caddy from stating at boot.
```shell
doas sysrc caddy_enable="NO"
```
Then in the /etc/pf.conf file I need to remove the following by commenting out or deleting:
```shell
#pass in inet proto tcp from any to any port ssh flags S/SA keep state
#pass in inet proto tcp from any to any port 80 flags S/SA keep state
#pass in inet proto tcp from any to any port 443 flags S/SA keep state
#pass in inet proto udp from any to any port 51820
```
Because the Caddy server is now in a jail and because my host SSH port has changed I need to add the following:
```shell
nat on $ext_if from <jails> to any -> ($ext_if:0)
rdr pass inet proto tcp from any to any port {80, 443, 22} -> 10.10.10.4
rdr-anchor "rdr/*"

pass in inet proto tcp from any to any port 5000 flags S/SA keep state
```
At this point I say "Hold onto your butts" and reboot the VPS instance.

Within about a second I browse to <a href="https://rkey.tech/" target="_blank">rkey.tech</a>, <a href="https://rkey.online/" target="_blank">rkey.online</a> and <a href="https://nc.rkey.tech/" target="_blank">nc.rkey.tech</a> and all 3 sites were up and operational. I then ssh into the host ```ssh -p 5000 titania``` and then ssh into the jail ```ssh caddy``` and this all worked as well.