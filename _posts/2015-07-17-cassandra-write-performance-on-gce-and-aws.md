---
title: Comparing Cassandra Write Performance on Google Compute Engine and AWS
author: Adam Israel
layout: post
comments: true
published: true
---

*... or how we created cross-cloud tunable, repeatable benchmarks for everybody.*

tl;dr - We achieved better Cassandra performance on GCE vs. Amazon, at close to half the cost.

When we decided to start the process of benchmarking across different clouds, Cassandra stood out as a great "hello world" for the way different clouds and tuning strategies can impact performance. We'll continue to explore different configurations and cluster sizes over the coming weeks, and plan on eventually re-running the infamous [1 million writes in Cassandra](http://googlecloudplatform.blogspot.com/2014/03/cassandra-hits-one-million-writes-per-second-on-google-compute-engine.html) across public clouds. Cassandra is just the start though, we plan on highlighting numerous benchmarks for a wide range of services. With that, let’s get started!

#The Setup

Every iteration of benchmark used the following constants:

- Apache Cassandra 2.1 with OpenJDK 7 or Oracle Java SE 8u51
- Ubuntu 14.04 LTS with Juju 1.24.2

We used the following hardware configuration on AWS:

- 8 cores with 30G RAM (m3.2xlarge)

and comparable hardware on GCE:

- 8 cores with 30G RAM (n1-standard-8)

## Juju

Leveraging Juju to model the infrastructure, we're able to condense the installation, tuning, and benchmarking of Cassandra into composable, repeatable components which can be executed against any architecture or cloud. This allowed us to turn this:

- Create cloud instances
- Read the [Cassandra docs](https://wiki.apache.org/cassandra/GettingStarted)
- Install and configure Cassandra
- [Tune Cassandra](http://wiki.apache.org/cassandra/PerformanceTuning)
- Run cassandra-stress:
```
cassandra-stress write n=2000000 cl=LOCAL_ONE -mode native cql3 -schema keyspace=Keyspace1 -log -node 52.3.185.174
```

into this:

```
juju deploy cassandra -n3
juju deploy cassandra-stress
juju add-relation cassandra-stress cassandra
juju action do cassandra-stress/0 stress operations=2000000
```

##A tale of two JDKs: Open JDK vs. Oracle

Due to licensing issues, the Cassandra charm installs OpenJDK 7. However, there are configuration options to use another JRE, such as Oracle's Java JRE. We tested both versions to see if there was discernable difference and our results are outlined below. Max heap sizes and heap new sizes are set automatically by the charm and tuned depending on instance size. Recommendations from seasoned Cassandra experts would be welcome in this area.

On AWS, when running Oracle Java 8 JRE vs OpenJDK 7 there was a 32% increase in performance when using the Oracle Java over OpenJDK.

On GCE, Cassandra would crash with a core dump when using OpenJDK but Oracle Java 8 JRE had no such issue.

Because of these results, going forward all benchmarks were run with Oracle Java 8u51 Server Edition.

#Results

##AWS

The best result were were able to achieve on AWS was with a three-node Cassandra cluster on m3.2xlarge instances and a c3.4xlarge instance for `cassandra-stress`.

With that, we were able to achieve [90,748 writes/second](http://cloud-benchmarks.org/submissions/4) with a 99th percentile latency of 4.0ms.

Follow-up tests using r3.4xlarge and i2.4xlarge instances were hit slightly higher numbers but at greater cost. We’ll be looking at how to best benchmark between instance types in a future blog post.

##GCE

The best result we were able to achieve on GCE was [111,394 ops/s](http://cloud-benchmarks.org/submissions/1) and 3ms latency (99th percentile) with a three-node Cassandra cluster using n1-standard-8 instances and a n1-highcpu-16 for `cassandra-stress`.

The next two best results came in at 109,583 ops/s and 101,886 ops/s.


## Summary

With Google's average hourly cost of $0.304 and Amazon's cost of $0.532, Google clearly outperforms it's AWS counterpart at both ops/sec and from a cost/resource standpoint.

We've [submitted](http://cloud-benchmarks.org/services/cassandra) the data we produced during these benchmark runs and invite anyone to [reproduce these results](http://blog.cloud-benchmarks.org/about/submit.html) either on AWS, GCE, Azure, Joyent, HP Cloud, Digital Ocean, or most any other public/private cloud!

#Lessons Learned

## Building benchmark charms

We started by [adding benchmarking](https://jujucharms.com/docs/devel/authors-charm-benchmarks) to the Cassandra charm. This sounded ideal; the charm comes packaged with the tools to benchmark the service making it easy to deploy. We quickly learned, however, that running `cassandra-stress` on the nodes it was benchmarking created significant resource contention.

We created a new [cassandra-stress](https://jujucharms.com/u/marcoceppi/cassandra-stress/trusty/1) charm to run `cassandra-stress` from its own instance, allowing us to wail on the Cassandra cluster without skewing the results.

## Scaling the benchmark
As we scaled from three to six nodes in the Cassandra cluster, we unexpectedly saw a slight decrease in performance. We are likely hitting a ceiling with regard to how many ops/sec we can achieve with a single `cassandra-stress` instance.

The next phase of benchmarking Cassandra will look at creating a cluster of `cassandra-stress` machines to simultaneously stress the cluster, as well as determining the optimal number of `cassanda-stress` instances to Cassandra nodes.

## Potential bottlenecks
There was a negligible differences in performance between r3.2xlarge and r3.4xlarge instances. My suspicion is that we were hitting a bottleneck somewhere between in our environment (network latency, disk i/o, kernel, etc), and we’ll need to isolate that during the next benchmark cycle.

# What's next

But these are just the initial findings, as we ramp up towards a million writes per second we will continue to do a few things:

 - Keep adding recommendations for experienced Cassandra users to squeeze the most out of these benchmarks (PRs accepted!)
 - Keep adding recommendations to other parts of the stack that can be optimized.
 - Keep improving the tools so that people can submit their own benchmarks to throw into the pile.
