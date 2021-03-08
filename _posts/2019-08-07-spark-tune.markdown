---
layout: post
title: The Spark tunable that gave me 8X speedup
tags: spark performance
newlink: /posts/2019-08-07-spark-tune/
---

There are many configuration tunables in Spark.  However, if you have
time for only one, set this one.  It made a streaming application we
run process data 8X faster.  That's 800% improvement, no code change
needed!

<!--more-->
### The tunables

Here are the tunables:

```
spark.sql.shuffle.partitions
```

and its sister:

```
spark.default.parallelism
```

The first setting applies to dataframes.  The second setting applies
to RDDs, which show up when you do calls like _map()_ on dataframes.

By default, both are set to 200.  This can be a lot, unless it is a
production cluster with 20 nodes and 10 cores each!

Here is how I set this value on my Spark cluster managed by YARN.  You
need to know:

* how many worker nodes you have in the cluster, say _W_
* how many cores you have on each worker node, say _C_

The available parallelism is _W_ * _C_ and that is the number you want
to use.  This is why I said "20" and "10" above.  Some people make an
"allowance" of 2X on top of this, but I'm not convinced about it yet.

You can set these either in spark-defaults.conf (preferred, helps
everyone on the cluster), or within your application, while
constructing _SparkConf_.

### What happens if you don't set this value?

There are two problems.

1. Your Spark (batch) application will take longer to finish, because
   Spark can only run _W_ * _C_ tasks at any given time.  The other
   tasks have to wait.

2. Each task may get only a very small chunk of data, and it takes
   longer to set it up and run it, than for it to do any data
   munching.

It is very easy to see this when you look at the timeline view of a
job under Spark's UI.  You can also see this when Spark reports "200
tasks" in any stage.

### Bad for you?

Is there a reason not to reduce this value?  Yes: say your data is
very large (or you make them large during processing), and having 10
chunks takes up much more memory than having 200 chunks.  Then, your
workers can run out of memory and the job can fail.  However, this
still means that you probably need beefier worker nodes, or leaner
processing logic!
