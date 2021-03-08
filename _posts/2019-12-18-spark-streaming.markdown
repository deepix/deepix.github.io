---
layout: post
title: Notes on Spark Streaming app development
tags: spark streaming kafka
newlink: /posts/2019-12-18-spark-streaming/
---

This post contains various notes from the second half of this
year.  It was a lot of learning trying to get a streaming model
working and ready in production.  We used Spark Structured Streaming,
and wrote the code in Scala.  Our model was stateful.  Our source and
sink were both Kafka.

<!--more-->

### Match Spark parallelism with Kafka

If your Kafka topic has N partitions, use N tasks in Spark.  Spark
Structured Streaming uses Kafka streaming API with 1:1 correspondence
between Kafka partitions and Spark tasks.  Set these in your app,
while building the Spark session, to match the number of Kafka
partitions:

```
spark.sql.shuffle.partitions
spark.default.parallelism
```

### Know your parallelism

Spark allows you to run multiple queries within your application.
Each query runs independently.  If you have 2 queries that can run
concurrently, you will 2_N_ cores.  If you have _M_ queries, you'll
need _M_ x _N_ cores.

In fact, it's more than that.  If you write to Kafka, there is a
KafkaProducer thread shared among all cores within an executor.  So
you'll need _P_ cores extra, if _P_ is the number of your executors.
Spark uses a different Kafka consumer group for each query.

Suppose we had an application that ran 5 queries, and our source Kafka
topic had 8 partitions.  Say we decided to have 4 executors.  So we
would need: (8 * 5) + 4 cores in all.

If there is not enough parallelism, the streaming job will sometimes
show excessive delays.  In our case, we saw a delay of 40 seconds, or
multiples thereof.  This corresponds to Kafka `read.timeout.ms`.

What happened was: the Spark task made a request to fetch data from
Kafka, but after that, never got around to run.  (There were many
other threads contending to run, because we were severely
under-provisioned.)  Eventually, the read times out, the connection is
reset, and another request is made.  If it comes back soon enough,
you're lucky, else you'll wait another 40 seconds, and so on.

### You can run only one action, and that's the streaming action

Yes, most developers are aware of this, but practically this means you
can't do things like _count()_, or even _take()_.  How do you get
visibility into what's going on?  You'll have to use _map()_, with logs
interspersed at key places.  Note that these logs will run on
executors, not on the driver.

### Use accumulators for counting

If you do need to count, you can use Spark accumulators for this
purpose.  You'd need a _map()_ call where you increment the accumulator.
Once done, you can check its value to get the count.

### Testing can be a pain, even if it uses a SparkContext

Streaming jobs have a number of limitations that a regular Spark job
does not.  Although using a SparkContext is better than nothing, there
is no substitute to actually running a Spark streaming job and
verifying that it works.

Case in point: you might have 2 actions in your code, and it'll
happily pass the tests that use SparkContext, but fail when run as a
streaming job.

### Jobs can run out of memory, or be very slow

Be sure to add a SparkListener to monitor job execution times.  Also
enable verbose GC logs, and heap dumps upon out of memory.  Java
allows you to do these things.

Be especially careful about how much state you're using.  If you're
appending to a list, check if you also have a timeout, or otherwise
limit the size of the list.

Run Spark at `TRACE` log-level, if something is amiss.  This will give
more information to narrow down problems.  I found this especially
useful when debugging KafkaConsumer problems.

### State processing should be the last aggregation

This applies to update-style aggregations (e.g. user session updates,
keyed by user).  State processing should be the last aggregation.  So
if you want to take a group-by count at output time, you're out of
luck.

### State processing function takes, and returns, an iterator of events

If you have a consume-only query, you can return an empty iterator.
If you're using _duplicate()_ to process the iterator twice, heed to the
Scala doc: you _cannot_ reuse the original iterator after calling it.

### State cannot be shared across queries

The state that Spark provides by default is backed by HDFS.  The path
is keyed by the query ID.  This means you have a problem whenever one
of your queries wants to access state from another query, even if it's
in the same application.  In our case, one was a producer query,
another was a consumer query.  No, didn't work!

The solution to this quandary is to use a single query, perhaps by
doing a `df.union()` if you read from two different sources.  Then,
you should use a single state-processing function, which filters the
rows and processes each topic row separately.

It's messy and introduces tight coupling in what should be really
independent modules, but that's what works for now.

### "Task not serializable"

At some point, you will see the notorious _TaskNotSerializable_
exception.  When that happens, read the exception very carefully: it
contains additional information on what could not be serialized.

It's frustrating if you make a massive change and then see this
exception.  Always break up your changes and run tests often, so that
you can catch this exception sooner and faster.

### Use rate limiting

Always limit how much data you can read, just in case someone decides
to blast away at your source.  For Kafka, this is easy to set via
`maxOffsetsPerTrigger` in the reader.

OK, that was a long list!  But each of them represents something
learned the hard way.  Let's hope this will help someone starting off
with Spark Structured Streaming.
