---
layout: post
title: Rotating the log makes it empty
tags: logrotate syslog freebsd
---

Our logs were getting too big, and it was time to have them "rotated":
phase out older entries and delete the oldest of them all.  I'd
already found out about [newsyslog.conf](https://www.freebsd.org/cgi/man.cgi?query=newsyslog.conf&apropos=0&sektion=5&manpath=FreeBSD+8.4-RELEASE&arch=default&format=html), and I added a line there to
rotate my log file.

OK, the log did get rotated, but wait - there's nothing being logged
now!  This is what I saw in the new log file:

```
Jul 30 06:00:00 myhost newsyslog[14536]: logfile turned over due to size>1000K
# ... empty ...
```

What just happened here?

### Empty log

Turns out it's not as simple as it appears.  When a log rotation
happens, the rotating program replaces the file with a new one.  Your
application will continue to write to a handle referring to the old
file, which of course, is now gone!

To make this clear, we can do a "dry run" of newsyslog.  Take a look
at the output below:

```
$ newsyslog -n
# ... snip ...
Start new log...
	mktemp /var/log/myapp.log.zXXXXXX
	chown 1002:4294967295 /var/log/myapp.log.zXXXXXX
	chmod 644 /var/log/myapp.log.zXXXXXX
	mv /var/log/myapp.log.zXXXXXX /var/log/myapp.log
```

Notice the last line, where it creates a new file to replace the log
file!

Now if you had written a script to log output like this:

```
myapp >> /var/log/myapp.log
```

It won't any more, because the system would continue to write to a
stale file handle.

### Ghost files

Does it sound bizarre?  It's got to do with how Unix works.  When you
open a file, you get back a handle, known as a "file descriptor".
This descriptor points to an open file.  When you write to the file,
you actually write to what the file descriptor is pointing to.

This is the surprising part: when the replacement file is created, the
system notices that the old file is still open (for writing), so it
will not free up its disk blocks.  It postpones this until all the
readers and writers are finished.  Quote the [unlink](https://www.freebsd.org/cgi/man.cgi?query=unlink&apropos=0&sektion=2&manpath=FreeBSD+8.4-RELEASE&arch=default&format=html) manual:

> If one or more process have the file open when the last link is
removed, the link is removed, but the removal of the file is delayed
until all references to it have been closed.

This is how I ended up in the weird situation.  Myapp was writing to
some file, but that file was not really accessible, and in its place
was sitting an empty file that the app never wrote to.  What's worse,
disk usage would also grow until I killed the app.

OK, what's the solution to this mess?  Syslog!

### Syslog to the rescue

The correct way to solve the above problem is to use
[syslogd](https://www.freebsd.org/cgi/man.cgi?query=syslogd&apropos=0&sektion=8&manpath=FreeBSD+8.4-RELEASE&arch=default&format=html).
You should replace the redirect with a pipe to
[logger](https://www.freebsd.org/cgi/man.cgi?query=logger&apropos=0&sektion=0&manpath=FreeBSD+8.4-RELEASE&arch=default&format=html)
as follows.

```
myapp | logger
```

In our case, our application was a daemon that would never exit.  The
exact command sequence turns out to be:

```
nohup myapp | nohup logger -i -t myapp &
```

In addition, there also needs to be two lines in `/etc/syslog.conf`,
so that syslogd will route the logs to the correct log file:

```
!myapp
*.*     /var/log/myapp.log
```

If the lines above look weird, you should consult the
[syslog.conf](https://www.freebsd.org/cgi/man.cgi?query=syslog.conf&apropos=0&sektion=5&manpath=FreeBSD+8.4-RELEASE&arch=default&format=html)
manual.

Finally, the log rotation itself is done via an entry in
`/etc/newsyslog.conf` like this:

```
# logfilename          [owner:group]    mode count size when  flags [/pid_file] [sig_num]
/var/log/myapp.log        appuser:      644     10      1000    *       GZ
```

For additional information, see the FreeBSD handbook on
[configuring system logging](https://www.freebsd.org/doc/handbook/configtuning-syslog.html).
