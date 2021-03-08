---
layout: post
title: Log rotation, no code change needed
tags: freebsd syslog logrotate newsyslog
newlink: /posts/2017-01-24-logrotate/
---

This post shows you how to rotate old logs from your application.
There is no change to application code.  There is no specialized
logging library or framework needed.  It works for any language, on
standard Unix platform.

<!--more-->

### What is log rotation?

Log files from applications can fill up disks over time.  To prevent
this, they are periodically truncated at a manageable size.  The older
logs are gradually removed from the system.  This is known as "log
rotation".

For example, a 35 MB app.log can be split as follows:

* 5 MB app.log
* 10 MB app.log.1 (older)
* 10 MB app.log.2 (older)
* 10 MB app.log.3 (oldest)

When app.log grows to 10 MB and log rotation is invoked, app.log.3 is
deleted.  app.log.2 becomes app.log.3, app.log.1 becomes app.log.2,
app.log becomes app.log.1.  An empty app.log is created.  The
application opens this file and writes to it.  This is why the
application needs to be aware of log rotation: because it has to open
the new, empty log file and write to it.

Logging frameworks like _log4j_ (Java) and _poco_ (python) can rotate
logs for your application.

### Log rotation with Unix utilities

We will use _syslog_, _newsyslog_, and _logger_ utilities that come
standard on FreeBSD.  On Linux, instead of _newsyslog_, you'd use
_logrotate_.

Here is the broad idea:

1. Application logs to standard output (terminal).
2. We pipe the output to _syslog_ server.
3. Syslog redirects the output to an application-specific log file.
4. Periodically, we run a command to rotate old logs.

### Logging to a file

I have a utility script, _startproc.sh_, that starts an executable
program.  Here is _startproc.sh_:

```
# usage: startproc.sh <program_name>
PROG=$1
shift
PROGNAME=`basename $PROG`
$PROG $* 2>&1 | logger -i -t $PROGNAME -p local7.info &
```

I invoke it as follows:

```
nohup startproc.sh /usr/local/bin/app
```

The output from the program is piped to **logger**, which streams it
to syslog daemon.  The logs go with a timestamp by default and a tag
of program name.  I use "local7.info" priority; there are other
possibilities.

**syslogd** can redirect the logs to any file.  I have set up my
`/etc/syslog.conf` as follows:

```
!app
*.*     /var/log/app.log
```

After adding the configuration, re-initialize the syslog via:

```
killall -HUP syslogd
```

This causes logs with the tag _app_ to be written to app.log.

Note: If you do not already have an app.log, syslog will create it, as
root.  This means if your application runs under a different user, it
cannot write to the log.  Create the log file yourself before sending
the HUP signal to syslogd.


### Rotating the log files

**newsyslog** is a FreeBSD utility that can rotate logs.  Linux users
can run _logrotate_ instead.  I have the following configuration in
`/etc/newsyslog.conf`:

```
/var/log/app.log        user:group      644     10      10000    *       GZ
```

_newsyslog_ is invoked by the system automatically on an hourly basis.
If you want to know how, you can look at the following `/etc/crontab`
entry:

```
#minute hour    mday    month   wday    who     command
0       *       *       *       *       root    newsyslog
```

It's worth repeating that you do not have to do anything to invoke
_newsyslog_.  It is done by the system automatically.  You only need
to set up newsyslog.conf.

Once rotation is done, you will see log files as shown earlier.  If
you want to test your changes by forcing a rotation, you can simply
say:

```
newsyslog
```

### Advantages

The advantages of this design is that you do not need any external
libraries.  The program can be written in any language, but log
rotation is handled in a uniform way.  You can use syslog to redirect
output to a log file as well as an external centralized logging
server.  You can continue to run the program for debugging purposes
and it will print to the console.

A disadvantage of this design is that you have the configuration
spread across multiple files.  But we use a configuration management
tool, Ansible, and all the steps are contained in a single playbook.
So this is not a problem.

For additional information, see the FreeBSD handbook on
[configuring system logging](https://www.freebsd.org/doc/handbook/configtuning-syslog.html).
