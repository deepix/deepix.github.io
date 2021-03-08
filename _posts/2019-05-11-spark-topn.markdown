---
layout: post
title: Getting top-N elements in Spark
tags: spark
newlink: /posts/2019-05-11-spark-topn/
---

The documentation for pyspark `top()` function has this warning:

```
This method should only be used if the resulting array is expected
to be small, as all the data is loaded into the driver's memory.
```

This piqued my interest: why would you need to bring all the data to
the driver, if all you need is a few top elements?

The answer is: it does *not* load *all* the data into the driver's
memory.

<!--more-->

Let's look at the [source
code](https://spark.apache.org/docs/2.3.1/api/python/_modules/pyspark/rdd.html#RDD.top).

{% highlight python %}
def top(self, num, key=None):
    def topIterator(iterator):
        yield heapq.nlargest(num, iterator, key=key)

    def merge(a, b):
        return heapq.nlargest(num, a + b, key=key)

    return self.mapPartitions(topIterator).reduce(merge)
{% endhighlight %}

There is a lot happening in those few lines.

* The idea is to split up the data (_mapPartitions_).  The number of
splits depends on the data source and cluster configuration.
* On each slice, we get the top N elements (_heapq.nlargest_).  This
part needs no data movement across nodes.
* Next, we find the top N within _those_ (_.reduce(merge)_) elements.
To see exactly how, let's first look at its source:

{% highlight python %}
def reduce(self, f):
    # ...
    def func(iterator):
        iterator = iter(iterator)
        try:
            initial = next(iterator)
        except StopIteration:
            return
        yield reduce(f, iterator, initial)

    vals = self.mapPartitions(func).collect()
    if vals:
        return reduce(f, vals)
{% endhighlight python %}

We collect the top-N elements from each partition! We then run
[_reduce()_](https://docs.python.org/3/library/functools.html) from
Python's functools package on that.  This runs on the driver, and it
gets us the top-N elements overall.

So yes, we do bring more than "top N" elements to the driver, but
definitely not "all the data".  If you're only collecting say top 100
elements, there is no cause for concern.
