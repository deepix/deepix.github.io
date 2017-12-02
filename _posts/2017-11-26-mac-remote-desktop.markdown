---
layout: post
title: Accessing home computer from anywhere
tags: mac remotedesktop ssh tunnel
---

Do you sometimes want to access your home computer from an outside
network?  Maybe you use another system, but you do not trust it and
would prefer your home computer for some workflows?

This post outlines the steps to make such access possible.

<!--more-->

### The Assumptions

Working through this requires intermediate to advanced expertise (or
alternatively, curiosity and persistence) in computers and
troubleshooting.  But the requirements are otherwise minimal:

* Your home computer is on a broadband connection (ADSL).
* Your remote network allows `ssh` into another system.  This is usually
  not a problem.
* Your untrusted computer does not have agents that take screenshots of
  your desktop or monitor your keystrokes.

### The Big Picture

 * We'll set up the WiFi router at home for remote access.
 * We'll use SSH tunnel to access our home computer from remote network.
 * We'll use Mac's "Screen Sharing" to access our home Mac's desktop.

### WiFi Router Setup

The IP address your home computer has, say `192.168.1.10`, is known as
a _private address_.  You cannot access it from the internet.  We will
fix this.

Additionally, your home computer may get a different IP address from
your WiFi router every time you connect.  We will fix this as well.

Finally, your _public address_ itself is on a lease and can change.
To fix this, We will sign up for a service that gives your computer an
easy name such as _yourname.no-ip.com_, that never changes.

Here are the steps.

Log in to your router administration page, usually
[http://192.168.1.1/](http://192.168.1.1/)

Then, consult your manual to do the following (I just browsed through
all the pages on their Web interface until I found what I wanted):

* Reserve an IP address for your computer.  The router will provide a
  way to reserve an IP address, say `192.168.1.5`, to the hardware (MAC)
  address of your home computer.

* Set up the router to forward port `22` to `192.168.1.5`.

* Sign up for a dynamic DNS service (e.g.,
  [no-ip.com](http://no-ip.com/)).  The basic plan is free of charge.

After you set these up, connectivity may still be broken, because we
have not set up the home computer for SSH yet.  We will do that next.

### SSH Setup

We'll set up your home computer for remote access.

Under _System Preferences > Sharing_, enable 'Remote Login'.  This
allows SSH access to your home computer.

No other services are needed.

Next, we will set up the SSH server to allow access only:

* from your remote network computer
* when presented with a long key that's computationally hard to fake

First, on your remote network computer, generate a 4096-bit key:
```
ssh-keygen -b 4096
```

Use a long, complicated passphrase to protect your key.  Then copy
your public key to your home Mac:

```
ssh-copy-id home_user_name@yourname.no-ip.com
```

Afterwards, ssh again.  Note that you are asked only for your key
Passphrase.

We next harden the SSH server on your home computer for enhanced
security.

* Disable password-based access
* Disable system-login authentication
* Enable host-based authentication
* Copy your remote network computer's host key to the home
  computer's known hosts file

The relevant file is `/etc/ssh/sshd_config`.  You can read the
comments above the configuration statements for more information.

The config lines are:

```
# Disable password-based access
PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM no

# Enable host-based authentication
HostbasedAuthentication yes
IgnoreUserKnownHosts yes
IgnoreRhosts yes
```

You can copy your remote network computer's host key in
`/etc/ssh/ssh_host_foo_key.pub` to the known hosts file
`/etc/ssh/ssh_known_hosts` on your home computer.

After making these changes, restart the SSH service: uncheck the box
on _System Preferences > Sharing_, then check it again.

The steps above will restrict who can access your home computer.  It
does not prevent flooding your computer with connection attempts.  To
fix this, we will use the Mac's firewall to limit how quickly
connections may be opened.

You will need the following configuration in `/etc/pf.conf`:

```
table <bruteforce> persist
block quick from <bruteforce>
pass in inet proto tcp to any port ssh \
flags S/SA keep state \
(max-src-conn 5, max-src-conn-rate 5/5, \
    overload <bruteforce> flush global)
```

See this [Mac PF](https://pleiades.ucsc.edu/hyades/PF_on_Mac_OS_X)
page for more information around this.  It is OK if you don't want to
do this now, but mark it for later.

Keep your home computer updated with Security Updates from Apple.

Finally, go to _System Preferences > Energy Saver_, check on 'Prevent
computer from sleeping automatically when the display is off'.
Without this, you may find your home computer inaccessible because it
went to sleep.

### Remote Desktop Sharing

We now come to the exciting part.  We will use the SSH connection to
tunnel remote desktop connections.

Add the following to your `~/.ssh/config` file.  Create it if it does
not exist already.

```
Host some_fancy_name
    Hostname yourname.no-ip.com
    User your_home_mac_user_id
    IdentityFile ~/.ssh/id_rsa_or_something_like_that
    LocalForward 5910 127.0.0.1:5900
    ServerAliveInterval 30
```

Start an SSH session: `ssh some_fancy_name_from_your_config_above`.
Use your key passphrase to authenticate.

Now open _Screen Sharing_, and connect to `127.0.0.1:5910`.  This should
connect to your home computer via SSH, and from there to the remote
desktop server running on your home computer.  You should see the login
screen from your home computer.

I always start an SSH session manually.  I have not automated this.  I
also keep my remote desktop under full screen, which makes it easy to
swipe it away when I need focus or privacy.  I try to use wired
Internet if available.

### Is a Mac needed at home?

Not really.  It is possible to use a Linux system and I've used it in
the same way.  I pay for a Linux VM on
[Linode.com](http://linode.com/); I sometimes use it as a backup.

Be aware that this comes with some limitations: I could only get
TigerVNC to work on it to any degree of satisfaction, and even then,
media was a problem.  It does not work for "bring my own laptop" or
conference call situations.  Besides, in general, Linux can be quirky
and frustrating to use as a primary personal desktop.

Also be aware that you'd still need to harden this VM.  I've seen
reams of login attempts within a day of creating a VM.

### Closing Words

Those are the steps, as an outline.  Some of the topics above merit
separate posts by themselves (SSH tunnelling anyone!?), but again,
I've kept the post focussed on the goal.  I hope it is helpful in that
regard.
