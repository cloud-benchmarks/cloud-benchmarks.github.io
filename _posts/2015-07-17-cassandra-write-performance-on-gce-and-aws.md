---
title: Comparing Cassandra Write Performance on Google Compute Engine and AWS
author: Adam Israel
layout: post
comments: true
---

*... or how we learned to create cross-cloud repeatable benchmarks for everybody.*

tl;dr - We achieved better Cassandra performance on GCE vs. Amazon, at half the cost.

Benchmarking and performance are interesting problems, especially in today’s growing [microservices](https://en.wikipedia.org/wiki/Microservices) scene. It used to be a question of “how does this hardware compare to that hardware,” but as computing and service-oriented architectures grow the question has evolved. How does my [cloud and application](http://12factor.net/) stack handle this? It’s no longer enough to run your favorite machine benchmark tool on your web server and call it a day.

Measuring every microservice in your stack, from backend to frontend, is a complex task. We started thinking about how you would model a system to benchmark all of these services. It’s not just a matter of measuring the performance of one service, but also its interactions with other services. Now multiply that by every config option for every service, like Cassandra, which has hundreds of options that can affect performance. And let’s not forget everything that you need to get Cassandra running, all those things have knobs too.

So, to start this process, we decided to benchmark Cassandra on different public clouds. Over the next few months we’ll continue to expand for larger sizes and eventually we plan on rerunning the famous [1 million writes in Cassandra](http://googlecloudplatform.blogspot.com/2014/03/cassandra-hits-one-million-writes-per-second-on-google-compute-engine.html) across some public clouds. Let’s get started!

#The Setup

Every iteration of benchmark used the following constants:

Apache Cassandra 2.1
- OpenJDK 7 and Oracle Java SE 8u51
- Ubuntu 14.04.1 with Juju 1.24

We used the following hardware configurations on AWS:

- 2 cores with 8G RAM (r3.large)
- 4 cores with 16G RAM (r3.xlarge)
- 8 cores with 61G RAM (r3.2xlarge)
- 16 cores with 122G RAM (r3.4xlarge)

We ran cassandra-stress with the following configurations:

```
cassandra-stress write n=2000000 cl=LOCAL_ONE -mode native cql3 -schema keyspace=Keyspace1 -log -node 52.3.185.174
```

#GCE Results

[Insert Marco’s data here]

#AWS Results

The best results were were able to achieve on AWS was with a three-node Cassandra cluster on r3.4xlarge instances. With that, we were able to achieve 100k writes/second with a 99th percentile latency of 4.8ms.

A follow-up test using i2.4xlarge instances achieved similar results. We’ll be looking at how to best benchmark between instance types in a future blog post.

#Lessons Learned

## Building benchmark charms
We started by adding benchmarking to the Cassandra charm. This sounded ideal; the charm comes packaged with a benchmark. We quickly realized, however, that running `cassandra-stress` on the nodes created a significant resource contention.

We created a new charm to run `cassandra-stress` on its own instance, allowing us to wail on the Cassandra cluster without skewing the results.

## Scaling the benchmark
As we scaled from three to six nodes in the Cassandra cluster, we unexpectedly saw a slight decrease in performance. We are likely hitting a ceiling with regard to how many ops/sec we can achieve with a single `cassandra-stress` instance.

The next phase of benchmarking Cassandra will look at creating a cluster of `cassandra-stress` machines to simultaneously stress the cluster, as well as determining the optimal number of `cassanda-stress` instances to Cassandra nodes.

## Potential bottlenecks
There was a negligible differences in performance between r3.2xlarge and r3.4xlarge instances. My suspicion is that we were hitting a bottleneck somewhere in our environment, and we’ll need to isolate that during the next benchmark cycle.

##A tale of two JDKS: Open JDK vs. Oracle

By default, the Cassandra charm installs OpenJDK 7. We also tested against Oracle’s Java SE 8u51. We did not, however, do any tuning of java parameters. Recommendations from seasoned Cassandra experts would be welcome in this area.

[Marco, I think you had better numbers with this. I didn’t see much difference between the two on AWS but you did on GCE?]

But these are just the initial findings, as we ramp up towards a million writes per second we will continue to do a few things:
Keep adding recommendations for experienced Cassandra users to squeeze the most out of these benchmarks (PRs accepted!)
Keep adding recommendations to other parts of the stack that can be optimized.
Keep improving the tools so that people can submit their own benchmarks to throw into the pile.

[1] Link to these charms so people can click right into the benchmark actions
[Add link to cassandra-stress in marco’s namespace]
