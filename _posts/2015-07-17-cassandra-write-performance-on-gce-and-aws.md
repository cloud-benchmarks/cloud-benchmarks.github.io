---
title: Comparing Cassandra Write Performance on Google Compute Engine and AWS
author: Adam Israel
layout: post
comments: true
---

*... or how we created cross-cloud tunable, repeatable benchmarks for everybody.*

tl;dr - We achieved better Cassandra performance on GCE vs. Amazon, at close to half the cost.

Benchmarking and performance are interesting problems, especially in today’s growing [microservices](https://en.wikipedia.org/wiki/Microservices) scene. It used to be a question of “how does this hardware compare to that hardware,” but as computing and service-oriented architectures grow the question has evolved. How does my [cloud and application](http://12factor.net/) stack handle this? It’s no longer enough to run your favorite machine benchmark tool on your web server and call it a day.

Measuring every microservice in your stack, from backend to frontend, is a complex task. We started thinking about how you would model a system to benchmark all of these services. It’s not just a matter of measuring the performance of one service, but also its interactions with other services. Now multiply that by every config option for every service, like Cassandra, which has hundreds of options that can affect performance. And let’s not forget everything that you need to get Cassandra running, all those things have knobs too.

So, to start this process, we decided to benchmark Cassandra on different public clouds. Over the next few months we’ll continue to expand for larger sizes and eventually we plan on rerunning the famous [1 million writes in Cassandra](http://googlecloudplatform.blogspot.com/2014/03/cassandra-hits-one-million-writes-per-second-on-google-compute-engine.html) across some public clouds. Let’s get started!

#The Setup

Every iteration of benchmark used the following constants:

- Apache Cassandra 2.1 with OpenJDK 7 and Oracle Java SE 8u51
- Ubuntu 14.04.1 with Juju 1.24

We used the following hardware configuration on AWS:

- 8 cores with 30G RAM (m3.2xlarge)

and comparable hardware on GCE:

- 8 cores with 30G RAM (n1-standard-8)

## Juju
Leveraging Juju to model the infrastructure, we're able to condense the installation, tuning, and benchmarking of any service into a composable, repeatable component to execute against any architecture or substrate. This allowed us to turn this:

```
cassandra-stress write n=2000000 cl=LOCAL_ONE -mode native cql3 -schema keyspace=Keyspace1 -log -node 52.3.185.174
```

into this:
```
juju action do cassandra-stress/0 stress
```

We ran cassandra-stress with the following configurations:

##A tale of two JDKS: Open JDK vs. Oracle

By default, the Cassandra charm installs OpenJDK 7. We also tested against Oracle’s Java SE 8u51. We did not, however, do any tuning of java parameters. Recommendations from seasoned Cassandra experts would be welcome in this area.

[Marco, I think you had better numbers with this change. I didn’t see much difference between the two on AWS but you did on GCE?]


#Results
##AWS

The best result were were able to achieve on AWS was with a three-node Cassandra cluster on m3.2xlarge instances and a c3.4xlarge instance for `cassandra-stress`.

With that, we were able to achieve 90,748 writes/second with a 99th percentile latency of 4.0ms.

Follow-up tests using r3.4xlarge and i2.4xlarge instances were hit slightly higher numbers but at greater cost. We’ll be looking at how to best benchmark between instance types in a future blog post.

##GCE

The best result we were able to achieve on GCE was 111,394 ops/s and 3ms latency (99th percentile) with a three-node Cassandra cluster using n1-standard-8 instances and a n1-highcpu-16 for `cassandra-stress`.

The next two best results came in at 109,583 ops/s and 101,886 ops/s.

## Summary

With Google's average hourly cost of $0.304 and Amazon's cost of $0.532, Google clearly outperforms it's AWS counterpart at almost half the cost.

All benchmark results for Cassandra are available for comparison [here](http://cloud-benchmarks.org/services/cassandra).

#Lessons Learned

## Building benchmark charms
We started by adding benchmarking to the Cassandra charm. This sounded ideal; the charm comes packaged with a benchmark. We quickly realized, however, that running `cassandra-stress` on the nodes created a significant resource contention.

We created a new [cassandra-stress](https://jujucharms.com/u/marcoceppi/cassandra-stress/trusty/1) charm to run `cassandra-stress` from its own instance, allowing us to wail on the Cassandra cluster without skewing the results.

## Scaling the benchmark
As we scaled from three to six nodes in the Cassandra cluster, we unexpectedly saw a slight decrease in performance. We are likely hitting a ceiling with regard to how many ops/sec we can achieve with a single `cassandra-stress` instance.

The next phase of benchmarking Cassandra will look at creating a cluster of `cassandra-stress` machines to simultaneously stress the cluster, as well as determining the optimal number of `cassanda-stress` instances to Cassandra nodes.

## Potential bottlenecks
There was a negligible differences in performance between r3.2xlarge and r3.4xlarge instances. My suspicion is that we were hitting a bottleneck somewhere between in our environment (network latency, disk i/o, kernel, etc), and we’ll need to isolate that during the next benchmark cycle.



But these are just the initial findings, as we ramp up towards a million writes per second we will continue to do a few things:
Keep adding recommendations for experienced Cassandra users to squeeze the most out of these benchmarks (PRs accepted!)
Keep adding recommendations to other parts of the stack that can be optimized.
Keep improving the tools so that people can submit their own benchmarks to throw into the pile.
