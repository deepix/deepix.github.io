---
layout: post
title: Socket timeout
tags: hdfs java
---

Have you run into a problem that happens only once in a while, and you never seem to have enough information to figure it out?  We all have.  This is one such tale: except that it has a happier ending.

### EPIPE

We run Hadoop for its distributed file system, HDFS.  We have a program that reads data over the network, and writes it out to HDFS.  How exactly does it write to HDFS?  This is by connecting to the local HDFS "DataNode" process over a TCP socket, and issuing [RPC calls](https://wiki.apache.org/hadoop/HadoopRpc).

Now, this is a newly written program (in Go) and let's call it Writer.  We gave it out for testing, and it would just die seemingly at random.  In one instance, it had exited with a message like this:

```
2016/06/27 19:29:23 Writer.go:102: error writing to hdfs:
    write tcp 10.106.76.14:60008->10.106.76.14:50010: write: broken pipe
```

(I have broken lines where they are too long to fit display.)

Note the message.  "Broken pipe" comes from the Unix error code EPIPE, which means we're trying to write on a connection that has gone away.

On the HDFS DataNode, there was an exception logged as well:

```
2016-06-27 19:25:32,484 INFO org.apache.hadoop.hdfs.server.datanode.DataNode:
    Exception for BP-1801848907-10.106.76.13-1466980424005:blk_1073741857_1088
    java.net.SocketTimeoutException: 60000 millis timeout while waiting for channel
    to be ready for read. ch : java.nio.channels.SocketChannel[connected
    local=/10.106.76.14:50010 remote=/10.106.76.14:60008]
at org.apache.hadoop.net.SocketIOWithTimeout.doIO(SocketIOWithTimeout.java:164)
at org.apache.hadoop.net.SocketInputStream.read(SocketInputStream.java:161)
at org.apache.hadoop.net.SocketInputStream.read(SocketInputStream.java:131)
at java.io.BufferedInputStream.fill(BufferedInputStream.java:235)
at java.io.BufferedInputStream.read1(BufferedInputStream.java:275)
at java.io.BufferedInputStream.read(BufferedInputStream.java:334)
at java.io.DataInputStream.read(DataInputStream.java:149)
at org.apache.hadoop.io.IOUtils.readFully(IOUtils.java:192)
at org.apache.hadoop.hdfs.protocol.datatransfer.PacketReceiver.doReadFully(PacketReceiver.java:213)
at org.apache.hadoop.hdfs.protocol.datatransfer.PacketReceiver.doRead(PacketReceiver.java:134)
at org.apache.hadoop.hdfs.protocol.datatransfer.PacketReceiver.receiveNextPacket(PacketReceiver.java:109)
at org.apache.hadoop.hdfs.server.datanode.BlockReceiver.receivePacket(BlockReceiver.java:467)
at org.apache.hadoop.hdfs.server.datanode.BlockReceiver.receiveBlock(BlockReceiver.java:781)
at org.apache.hadoop.hdfs.server.datanode.DataXceiver.writeBlock(DataXceiver.java:761)
at org.apache.hadoop.hdfs.protocol.datatransfer.Receiver.opWriteBlock(Receiver.java:137)
at org.apache.hadoop.hdfs.protocol.datatransfer.Receiver.processOp(Receiver.java:74)
at org.apache.hadoop.hdfs.server.datanode.DataXceiver.run(DataXceiver.java:237)
at java.lang.Thread.run(Thread.java:722)
```

But this message was logged almost 4 minutes before the Writer tried to write.  There cannot be any network issues or clock skew, because both processes are running on the same system.

### Hadoop thread timeout

I looked at Hadoop sources and found something interesting.  When you ask Hadoop to write to HDFS over RPC, it is handled by a 'worker' thread listening on a socket.  This thread simply [puts itself to sleep waiting for data](https://github.com/apache/hadoop/blob/release-2.6.3/hadoop-common-project/hadoop-common/src/main/java/org/apache/hadoop/net/SocketIOWithTimeout.java#L125) to come in ('blocking read').  It also sets a timeout for 60 seconds, and if no data comes in within this period, it will just terminate.

Accordingly, the log continued:

```
2016-06-27 19:25:32,511 INFO org.apache.hadoop.hdfs.server.datanode.DataNode:
    PacketResponder: BP-1801848907-10.106.76.13-1466980424005:blk_1073741857_1088,
    type=LAST_IN_PIPELINE, downstreams=0:[]: Thread is interrupted.
2016-06-27 19:25:32,511 INFO org.apache.hadoop.hdfs.server.datanode.DataNode:
    PacketResponder: BP-1801848907-10.106.76.13-1466980424005:blk_1073741857_1088,
    type=LAST_IN_PIPELINE, downstreams=0:[] terminating
```

So, as per design, the thread went away at 19:25.  Four minutes later, we tried to write to it and of course, it failed.  The question is: what caused this timeout?  Why did we stop writing to HDFS and go idle, and what caused us to wake up after four minutes?

Our first theory was that there was a network issue.  But both processes were running on the same system.  Maybe there was a problem connecting to the HDFS NameNode, which was running on a different system?  I found that hard to believe.  The log clearly does not indicate any such problem.

Another theory I had in mind was if the thread could not be run because the system was busy.  But it's hard to believe that a thread won't get CPU time for 60 seconds either.  What about that villain that always gets blamed: Garbage Collector?  Sixty seconds for the light load we have?  No way!

Time to go back to our logs and see if we can dig out anything more.

### Idle or not?

Our Writer logs all writes, and there were successful writes logged up until before the crash.  This led me up the wrong alley for a long time, because I thought the Writer was not really idle.  After a lot of wasted effort, I looked closer and asked: What if the writes are on _other_ connections?  What if this particular connection was idle for a long time?  And this is what the log showed:

```
2016/06/27 19:24:46 Writer.go:116: <connection_id> <page_no>
```

It checks out: this particular connection received its data last at 19:24:46.  It went idle after that for a few minutes.  The DataNode thread exited 46 seconds later.  Its timeout is set for 60 seconds per the log, so I'm not sure why it went down before that.  One theory I have is that the Writer never sent that data out on the wire: maybe it was buffered in the networking layer of the operating system to be sent out later.  Perhaps its last data transmission was really at 19:24:32?

So the DataNode thread is gone, and now data starts flowing in again, 4 minutes later.  Let me show the log message again:

```
2016/06/27 19:29:23 Writer.go:102: error writing to hdfs:
    write tcp 10.106.76.14:60008->10.106.76.14:50010: write: broken pipe
```

It seems obvious that we are using a stale socket handle.  OK, we can fix that.  There is still a mystery on what happened to the data.  Four minutes seems like the time it takes to restart a system, and we can verify that from the source system logs.

### Sit back, relax and restart

This bit was the easiest.  After factoring in the time difference, the data source system's clock was ahead of our system by almost 11 hours:

Our system:

```
Writer$ date
Wed Jun 29 12:23:53 UTC 2016
```

Origin system (UTC-4):

```
Source$ date
Wed Jun 29 20:19:43 AST 2016
```

So if I had to look at its logs, it would be 10 hours 56 minutes after June 27 7:24 PM, which is somewhere around June 28, 6:20 AM on our system.

This is what I found:

```
Jun 28 06:24:59 <user.crit> <hostname> reboot: rebooted by root
Jun 28 06:25:02 <syslog.err> <hostname> syslogd: exiting on signal 15
```

So the system was restarted around the time we expect.  To be sure, it is 4 minutes later than what we expect, but again, I'll blame that to clock skew - these systems are not wired to NTP yet.  The log also showed that the restart took all of 4 minutes, and once it was back up, data started flowing in.

So there we have it.  We did not drop any data, but the source stopped sending it.  By the time it came back and started sending again, our receiver had already kicked us out.  Our program tried to use this dead connection and got itself shot.
