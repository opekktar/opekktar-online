---
title: Creating Vultr Instance
date: 2022-03-08T14:59:36.000Z
draft: false
description: Creating Vultr Instance
---

[Matching Blog Post](https://rkey.online/posts/creating_vultr_instance/)

#### Creating instance at Vultr & setting up DNS

Setting up my CAA: I use a Caddy server that auto-magically takes care of my SSL certs for me. I like lazy solutions that work.

```shell
0 issue "letsencrypt.org"
0 iodef "mailto:rkey@rkey.tech"
```

Next is all the mail stuff to work with ProtonMail: In case anyone is wondering those are not the real keys :)

```shell
CNAME protonmail._domainkey protonmail.domainkey.dn39a43a188d9439487409be25ea.domains.proton.ch
CNAME protonmail2._domainkey protonmail2.domainkey.dn39a43a188d9439487409bewida.domains.proton.ch
CNAME protonmail3._domainkey protonmail3.domainkey.dn39a43a188d9439487409bewida.domains.proton.ch
MX mail.protonmail.ch 300 10
MX mailsec.protonmail.ch 300 20
TXT "protonmail-verification=ca39a43a188d9439487409be"
TXT "v=spf1 include:_spf.protonmail.ch mx ~all"
TXT _dmarc "v=DMARC1; p=quarantine; rua=mailto:rkey@rkey.tech; ruf=mailto:rkey@rkey.tech; sp=quarantine; aspf=s; fo=1;"
```

Before changing the DNS pointers I need to migrate my Caddy server at Digital Ocean (DO) over to Vultr.

I do that by replicating my deployment script to mirror sites.

```shell
bsh ➜  cat deploy.sh
rm -rf public/
rm public.tar
HUGO_ENV="production" hugo --gc || exit 1
echo OK, now that stuff is built
rsync -azP --delete public/ titania:~/rkey.tech/ # Caddy server at Vultr
rsync -azP --delete public/ spirit:~/rkey.tech/ # Caddy server at DO
echo OK, now that stuff is uploaded
echo ======================================
echo Done
echo ======================================
```

1. create user and sync ssh keys.

  ```shell
  rsync -a --chown user:group ~/.ssh /home/user/
  ```

2. Make sure the new user can su to root. I use doas.

  ```shell
  🕙[ 13:47:15 ] bsh ➜  doas cat /usr/local/etc/doas.conf
  permit nopass keepenv :wheel
  ```

3. Turn off password authentication and root user login by adding to the end of `/etc/ssh/sshd_config`

  ```shell
  PermitRootLogin no
  PasswordAuthentication no
  ```

  Configure Caddy

```shell
titania.rkey.tech {
        root * /home/user/wtf/
        file_server
        log {
                output file /var/log/caddy/titania.rkey.tech.access.log
                format json
        }
}
```
