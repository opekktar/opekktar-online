---
title: "Ghost Blog"
date: 2022-04-06T11:29:41-07:00
draft: true
description: "Setup Ghost Blog In FreeBSD Jail"
weight: 6
---

[Matching Blog Post](https://rkey.online/posts/ghostblog/)

#### Create new jail to install Ghost Blog into:
```shell
$ doas bastille create ghost 13.0-RELEASE 10.10.10.5
```
#### Create database user for the Ghost Blog
```shell
bsh ➜  doas bastille console database
[database]:
```
Log into the database server
```shell
mysql -u root -p
Enter password: **********
```
Create database user
```shell
[(none)]> create user 'ghost'@'10.17.63.5' identified by '**********************';
```
Create the database
```shell
[(none)]> create database ghostdb;
```
Grant all privileges on the database to the user
```shell
[(none)]> grant all on `ghostdb`.* to 'ghost'@'10.17.63.5';
```
Make it so the user can log in over jails network
```shell
[(none)]> alter user 'ghost'@'10.17.63.5' identified with mysql_native_password;
```
After the above step I always have to enter the password for the user again.  It's been decades since I worked as DBA, so maybe I'm just doing things out of step here. 
```shell
[(none)]> alter user 'ghost'@'10.17.63.5' identified by '**********************';
```
Flush privileges and exit
```shell
[(none)]> flush privileges;
[(none)]> quit
Bye
```
Log into Ghost jail and install and configure Ghost. ***Note*** initially I tried latest npm-node package, but kept backing down the version until I found a version that worked which is npm-node14. Doas is just for my convenience and not required if you prefer to ```su -``` instead. And of course I need to the MySQL client to connect to the database in the database jail. 
```shell
doas bastille console ghost
# pkg update
# pkg upgrade
# pkg install doas npm npm-node14 openssl curl mysql80-client
```
Create ghost user
```shell
# adduser
```
Create startup script so the Ghost Blog will start on reboot. 
```shell
# vi /usr/local/etc/rc.d/ghost
```
Example of script below
```shell
#!/bin/sh

# PROVIDE: ghost
# REQUIRE: mysql
# KEYWORD: shutdown

. /etc/rc.subr

name="ghost"
rcvar="ghost_enable"
extra_commands="status"

load_rc_config ghost

start_cmd="ghost_start"
stop_cmd="ghost_stop"
restart_cmd="ghost_restart"
status_cmd="ghost_status"

PATH=/bin:/usr/bin:/usr/local/bin:/home/ghost/.bin

ghost_start()
{
    su ghost -c "/usr/local/bin/ghost start -d /usr/local/www/msrobota.online"
}
ghost_stop()
{
    su ghost -c "/usr/local/bin/ghost stop -d /usr/local/www/msrobota.online"
}

ghost_restart()
{
    ghost_stop;
    ghost_start;
}

ghost_status()
{
    su ghost -c "/usr/local/bin/ghost status -d /usr/local/www/msrobota.online"
}

run_rc_command "$1"
```
Make the script executable.
```shell
# chmod +x /usr/local/etc/rc.d/ghost
```
Create directory for the blog to be installed into and run from.
```shell
# mkdir -p /usr/local/www/msrobota.online
```
Make user ghost own the directory
```shell
# chown -R ghost:ghost /usr/local/www/msrobota.online
```
Install ghost command line tools
```shell
# npm install ghost-cli@latest -g
```
change to ghost user and install ghost
```shell
# su - ghost
$ cd /usr/local/www/msrobota.online
$ ghost install
```
During the install you will get errors about unsupported system, since everyone who writes software nowadays assumes we have all been assimilated into the Linux collective. 

The configuration file built from the answers provided by the install will look like the file below. 
```shell
$ cat config.production.json
{
  "url": "https://msrobota.online",
  "server": {
    "port": 2368,
    "host": "127.0.0.1"
  },
  "database": {
    "client": "mysql",
    "connection": {
      "host": "10.17.63.2",
      "user": "ghost",
      "password": "***************************",
      "database": "ghostdb"
    }
  },
  "mail": {
    "transport": "Direct"
  },
  "logging": {
    "transports": [
      "file",
      "stdout"
    ]
  },
  "process": "local",
  "paths": {
    "contentPath": "/usr/local/www/msrobota.online/content"
  }
}
```
You can start ghost from the install directory with ```$ ghost start``` or by using the rc script created earlier with ```$ doas service ghost start```
I had already setup a DNS zone file as shown earlier for [Creating Vultr Instance](/docs/vps_migration/creating_vultr_instance/) for the Ghost Blog's domain of ```https://msrobota.online```
So we just need to add that domain to the caddy server jail configuration
```shell
bsh ➜  doas bastille console caddy
# vi /usr/local/etc/caddy/Caddyfile
```
And add
```shell
msrobota.online {
        reverse_proxy 10.17.63.5:2368
        log {
                output file /var/log/caddy/msrobota.online.access.log
                format json
        }
}
```
Then restart caddy with ```# service caddy restart```
You will not want to wast time and create your user and secure that account because until you do anyone can do so. I used to create firewall rules and allow only access from my workstation or rather ISP account.  But that triples the amount of time.  So I just make sure as soon as I restart caddy I'm setting up the owner account at ```https://msrobota.online/ghost/```
I used to then go in and configure the server to host dark only themes, open external links in new windows/tabs and remove the Ghost branding by modifying the ```/usr/local/www/msrobota.online/current/content/themes/casper/default.hbs``` file, but with version 3 you can configure the server to default to dark mode at ```Settings --> design --> site wide --> Color scheme``` and for simplicity I now just use code injection.
To remove the Ghost Branding I inject the following code in the footer
```shell
<style>
  /* The footer links to ghost keep coming back */
  .site-footer a[href^="https://ghost.org"] { display: none; }
</style>
```
And to ensure external links open in new tabs/windows I inject the following code also in the footer
```shell
<script>
  const anchors = document.querySelectorAll('a');

  for (x = 0, l = anchors.length; x < l; x++) {
    const regex = new RegExp('/' + window.location.host + '/');
    
    if (!regex.test(anchors[x].href)) {
      anchors[x].setAttribute('target', '_blank');
      anchors[x].setAttribute('rel', 'noopener');
    }
  }
</script>
```
That's it now all that remains is to begin blogging, which is the most difficult part me.
