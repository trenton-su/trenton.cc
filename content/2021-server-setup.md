date: 2021-09-28T14:33:58Z
template: posts
title: Server Setup
urlstub: server-setup

# Ubuntu 20.04 basic server setup

Once your VPS is up and running, there are a few basic configuration changes necessary to 

a) make the server more secure and 
b) keep it secure over time.

We'll cover:

- creating a user account
- configuring SSH access
- configuring automatic updates
- enabling the firewall

## Create a user account

You should use an unprivileged account for most of your day-to-day activity. You may want to replace the username `joe` with something more meaningful for you.

`root@<server># adduser joe`

You may want to add this new user to the `sudo` group for sysadmin tasks

`root@<server># usermod -aG sudo joe`

Now, try logging in to your new user over SSH

`you@local$ ssh joe@<server>`

If all went well, you're now logged into the server and see

`joe@<server>$`

## Configure SSH access

Leaving SSH access open to the `root` user is considered a security hole. Let's shut that down by making the following changes

```
File: /etc/ssh/sshd_config
--
PermitRootLogin no 
```

## Configure Automatic Updates

Keeping the server up to date with security patches is a good idea. This configuration also sends out periodic emails to you, containing info about package upgrades.

`joe@<server>$ sudo apt install unattended-upgrades`

To enable unattended-upgrades:

`joe@<server>$ sudo dpkg-reconfigure -plow unattended-upgrades`

You'll want to hit `Yes` for automatic upgrades.

### Email status updates

Install the `mailx` package:

`joe@<server>$ sudo apt install bsd-mailx`

You'll need to configure the `mailx` package - just pick the `Internet` option and then enter your domain name

Sending email updates for `unattended-upgrades` is simple!

```
File: /etc/apt/apt.conf.d/50unattended-upgrades
--
Unattended-Upgrade::Mail "your@emailaddress.com";
```

To make sure that `bsd-mailx` is configured correctly, `/etc/mailname` should contain a domain name that resolves to an IP address. This is important since some mailservers will check the origin address to ensure that the server actually exists. Gandi does:

```
File: /var/log/mail.log
--
Dec 21 21:54:51 <server> postfix/smtp[18154]: 9E0443E949: to=<sysadmin@example.com>, 
relay=spool.mail.gandi.net[217.70.178.1]:25, 
delay=0.35, 
delays=0.02/0.01/0.26/0.07, 
dsn=5.1.8, 
status=bounced (host spool.mail.gandi.net[217.70.178.1] said: 550 5.1.8 <root@<server>>: Sender address rejected: Domain not found (in reply to RCPT TO command))
```

## Firewall

Setting up the `ufw` firewall is quick and simple!

`joe@<server>$ sudo ufw allow ssh`
`joe@<server>$ sudo ufw allow 'Postfix SMTPS'`
`joe@<server>$ sudo ufw allow 'Postfix Submission'`
`joe@<server>$ sudo ufw allow Postfix`
`joe@<server>$ sudo ufw enable`


## Login alerts



This also gets triggered on, e.g. `scp`


```
File: /etc/ssh/sshrc
from: https://askubuntu.com/a/179924
---
ip=`echo $SSH_CONNECTION | cut -d " " -f 1`

logger -t ssh-wrapper $USER login from $ip
echo "User $USER just logged in from $ip" | mailx -s "SSH Login" -r sysadmin@joesserver.com joe@gmail.com &
```