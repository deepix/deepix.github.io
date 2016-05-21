# "You got too many open files"

I'm trying to get Ansible 2.0 working on a custom FreeBSD 8.4 that's running custom Python 2.6. That's like fitting satellite navigation on a vintage car, but it's a different story for another day.

### Errno 23

Intermittently, Ansible would complain like so:

`Unexpected Exception: [Errno 23] Too many open files in system`

The full trace is below:

```
Traceback (most recent call last):
File "/var/python/lib/python2.6/site-packages/ansible-2.0.0.1-py2.6.egg/EGG-INFO/scripts/ansible-playbook", line 85, in <module>
sys.exit(cli.run())
File "/var/python/lib/python2.6/site-packages/ansible-2.0.0.1-py2.6.egg/ansible/cli/playbook.py", line 150, in run
results = pbex.run()
File "/var/python/lib/python2.6/site-packages/ansible-2.0.0.1-py2.6.egg/ansible/executor/playbook_executor.py", line 148, in run
result = self._tqm.run(play=play)
File "/var/python/lib/python2.6/site-packages/ansible-2.0.0.1-py2.6.egg/ansible/executor/task_queue_manager.py", line 203, in run
self._initialize_processes(min(contenders))
File "/var/python/lib/python2.6/site-packages/ansible-2.0.0.1-py2.6.egg/ansible/executor/task_queue_manager.py", line 103, in _initialize_processes
main_q = multiprocessing.Queue()
File "/var/python/lib/python2.6/multiprocessing/__init__.py", line 213, in Queue
return Queue(maxsize)
File "/var/python/lib/python2.6/multiprocessing/queues.py", line 42, in __init__
self._wlock = Lock()
File "/var/python/lib/python2.6/multiprocessing/synchronize.py", line 117, in __init__
SemLock.__init__(self, SEMAPHORE, 1, 1)
File "/var/python/lib/python2.6/multiprocessing/synchronize.py", line 49, in __init__
sl = self._semlock = _multiprocessing.SemLock(kind, value, maxvalue)
OSError: [Errno 23] Too many open files in system
```
Hmmm. That looks an easy one to fix.

* I set `ulimit -n 50000`. No help.
* `sysctl kern.maxfiles=50000`. Nope.
* `sysctl kern.maxprocfiles=50000`. Didn't work either.

I had a demo coming up where I would deploy a cluster with Ansible.  This was starting to give trouble.  I had to get to the bottom of this.

### Entering python

I started by looking in the python library `/var/python/lib/python2.6` for `_multiprocessing` module sources, but couldn't find it. There was a match in dynamic libraries, however:

```
$ grep _multiprocessing *
Binary file lib-dynload matches
Binary file test matches
```

That means it must be in the sources for the python interpreter: that's how it ended up being a compiled, binary library.

I opened up python-2.6 sources. A grep showed matches in `Modules/_multiprocessing/semaphore.c`. It was making a system call: `sem_open()`.

Hmmm. So `sem_open()` is failing. I looked up `errno` man page for the number 23 and it was, symbolically, `ENFILE`.

```
23 ENFILE	Too many open files in system.  Maximum number of file descrip-
			tors allowable on the system has been reached and a requests for
			an open cannot be satisfied until at least one has been closed.
```

Maybe there is a different tuning parameter for IPC file descriptors. That's why the previous commands didn't work?

### Kernel of truth

Down the rabbit hole: now I opened the FreeBSD kernel sources. I first simply tried to list files that had *sem* in their filename. This led me to `kern/sysv_sem.c` and a bunch of weird `sysctl` tunables like `kern.ipc.semmnu`.  But these tunables already had reasonably high values.  Dead end.

Of course, there's a better way than that. Why not just grep for `sem_open`? Besides, what I wanted was *POSIX semaphores*, not System-V!  I remembered now: Python `configure` script for 2.6 on FreeBSD 8.4 doesn't enable POSIX IPC by default, because it warns, the platform support code is experimental.  I had read it, uncommented it, and then forgot all about it.  Did I run into this problem?

OK, progress at last. The following was interesting:

```
kern/syscalls.c: "ksem_open", /* 405 = ksem_open */
kern/uipc_sem.c:ksem_open(struct thread *td, struct ksem_open_args *uap)
sys/semaphore.h:sem_t *sem_open(const char *, int, …);
sys/syscall.h:#define SYS_ksem_open 405
```

So it seems `sem_open()` is implemented via `ksem_open()` in `kern/uipc_sem.c`.  I opened up `uipc_sem.c` and sure enough, found a tunable:

```
if (nsems == p31b_getcfg(CTL_P1003_1B_SEM_NSEMS_MAX) || ksem_dead) {
				mtx_unlock(&ksem_count_lock);
				return (NULL);
}
```

### Thirty ought to be enough

Turns out `CTL_P1003_1B_SEM_NSEMS_MAX` corresponds to the sysctl `sem_nsems_max`.  And guess what its default value is?  30!  You read that right, it's 30!

```
#ifndef SEM_MAX
#define SEM_MAX 30
#endif
```

That sounds like a very low value.  I set it to 300:
```
sysctl sem.nsems_max=300
```

Magic!  I stopped getting the "too many open files" error.  I tried a few more times and never ran into the problem again.  I still haven't had a chance to see why `sem.nsems_max` defaults to 30, but I will stop here for now.

### Afterthought

Looking back at this, I'm amazed at how far away the problem is from the actual cause.  Saying "Too many open files" is certainly not the same as saying "Your configured maximum semaphores limit is too low".  A different errno would have helped.  But then, I was warned the code is experimental.

When I first wrote the Ansible playbooks, it was on Linux.  We never thought of running it on FreeBSD, much less its 8.4 version.

What really helped a lot here is the open source nature of the entire stack: I had access to python sources, as well as the FreeBSD kernel sources.  All of that was sitting checked in to a single source repository.  It was a matter of couple of hours to get to the bottom of it.  I don't know if I could have pulled this off if I had to call a Customer Support line and open a ticket!
