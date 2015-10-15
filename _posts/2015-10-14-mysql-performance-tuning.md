---
title: MySQL performance tuning between disk types
author: aisrael
layout: post
comments: true
published: false
---

# Introduction

We sat down with the intent to benchmark various flavors of MySQL (Oracle MySQL, MariaDB, and Percona) on Amazon EC2 in order to measure the difference between the three on identical hardware, backed by spinning rust and SSD storage. The wheels on that bus came off quickly, though, as we realized there was no way to run these tests in a repeatable fashion.

The [MySQL charm](https://jujucharms.com/mysql/) -- a small collection of code that models how software should be installed and configured -- didn’t have the ability to move MySQL’s data directory away from the default `/var/lib/mysql`, and in the instance type we were testing with, the SSD was allocated to `/mnt`.

Our focus shifted to benchmarking the _model_, rather than the hardware. We can scale up the hardware, but it’s not all that useful unless the software is capable of taking advantage of it.

# The Setup

Cloud: Amazon EC2

Instance type: m3.2xlarge

 - 8 cores
 - 2600 CPU power
 - 30720M ram
 - 32G root-disk
 - EBS vs SSD

We chose to run the OLTP.lua benchmark included with [sysbench](https://www.percona.com/docs/wiki/benchmark:sysbench:olpt.lua) provided by Percona.

This benchmark is a more realistic test of database performance, testing a workload across multiple tables. For our purposes, we chose to allocate 25 tables with 1M rows each before each series of tests.

# Hiccups

When originally trying to benchmark different flavours of MySQL (Oracle MySQL, Percona, and MariaDB) I noticed that things that should have made a difference in performance showed negligible increase. Things like switching from spinning rust to SSD made seemingly no difference, so I dug into what the Juju charm’s [tuning-level](http://bazaar.launchpad.net/~charmers/charms/trusty/mysql/trunk/view/head:/hooks/config-changed#L155) was doing. 

# “Safest” vs “Fast”

It’s often easier (and cheaper) to throw hardware at a bottleneck and call it a day. In my experience, though, this is a losing battle. Bottlenecks shift. Fix one logjam and another appears. Poorly tuned software always trumps hardware.

With that in mind, we looked at how we could test various configurations in a repeatable fashion. Luckily, the charm offers three built-in levels of tuning: safest, fast, and unsafe. The idea behind this is to give people larger knobs to twiddle with without inundating with configuration options.
We decided to skip ‘unsafe’ and focus on ‘safest’ and ‘fast’. Switching from ‘safest’ (the default) to ‘fast’ breaks full ACID compliance -- changing `innodb_flush_log_at_trx_commit` from 1 to 2, and `sync_binlog` from 1 to 0 -- potentially losing up to a second worth of transaction if the operating system were to crash or the machine loses power.

This is key; there are plenty of options you could tweak for speed but you probably shouldn't. The “fast” option is a compromise on data safety, and should only be used if you understand the risks. Still, it’s worth comparing the two so the performance impact can be understood.

# Storage

By default, the MySQL datadir is located in `/var/lib/mysql`. When using an instance type that includes SSD block storage, though, you need to relocate the datadir under `/mnt`. In order to do that, I added a configuration option to the MySQL charm to support this relocation. The modified charm is [available here](https://code.launchpad.net/~aisrael/charms/trusty/mysql/trunk). 

# Innodb and disk space, oh my.

Sysbench's usual pattern is to prepare (iow: insert a bunch of test data), run, and cleanup (drop the tables). The first iterations of my testing prepared and cleaned up the test data between every run iteration. I feel like this provides a better test story, but it presented two issues. First, the time spent to create and drop the test data is significantly longer than running the test itself. Second, I ran out of disk space, unrecoverable, thanks to `/var/lib/mysql/ibdata1`. This may suggesting an issue with sysbench itself not properly cleaning up. To work around the issue, though, I added options to the mysql-benchmark benchmark’s oltp action, so that the test data can be prepared and cleaned up once instead of on every iteration.

The Results

## EBS

| Threads  | Tuning Level | Avg. Time per request (ms) | 95th Percentile Latency (ms) | Transactions/sec |
|----------|--------------|----------------------------|------------------------------|------------------|
| 16 | Safest | 43.13  | 47.57  | 406.84 | 
| 16 | Fast   | 45.42  | 21.12  | 350.94 |
| 32 | Safest | 72.99  | 69.03  | 357.02 |
| 32 | Fast   | 85.54  | 70.85  | 377.25 |
| 64 | Safest | 182.37 | 321.28 | 349.00 |
| 64 | Fast   | 194.34 | 398.50 | 333.37 |

## SSD

| Threads  | Tuning Level | Avg. Time per request (ms) | 95th Percentile Latency (ms) | Transactions/sec |
|----------|--------------|----------------------------|------------------------------|------------------|
| 16 | Safest | 70.67  | 96.48  | 226.96  |
| 16 | Fast   | 15.24  | 15.81  | 1057.48 |
| 32 | Safest | 133.85 | 182.97 | 239.77  |
| 32 | Fast   | 23.63  | 36.92  | 1347.88 |
| 64 | Safest | 259.57 | 339.11 | 245.91  |
| 64 | Fast   | 52.89  | 67.02  | 1191.11 |

## IOPS

| Threads  | Tuning Level | Avg. Time per request (ms) | 95th Percentile Latency (ms) | Transactions/sec |
|----------|--------------|----------------------------|------------------------------|------------------|
| 16 | Safest | 75.76  | 129.64 | 211.04  |
| 16 | Fast   | 10.55  | 16.62  | 1503.85 |
| 32 | Safest | 123.19 | 153.89 | 259.40  |
| 32 | Fast   | 23.28  | 25.36  | 1372.31 |
| 64 | Safest | 259.39 | 360.86 | 245.97  |
| 64 | Fast   | 50.02  | 63.28  | 1274.63 |

# Magical Unicorns chasing rainbows

We’ve seen that MySQL performance relies heavily on disk throughput. I can’t think of a better test of that raw number than running `dd` against the block devices themselves. These raw numbers represent the peak of what the block device is capable of.


|EBS | SSD | IOPS |
|----|-----|------|
| 22.8 MB/s | 131 MB/s | 386 MB/s |


We see almost a 575% increase in performance by switching from EBS to SSD, and a 294% increase from SSD to IOPS (a whopping 1,692% improvement over EBS). That’s awesome. 

These numbers should represent the peak throughput of each device, so our ideal benchmark of MySQL (or any service) could be compared to this as a comparison of efficiency.

It’s hard to draw a conclusion about how this translates to performance in MySQL, but we do see a decreases in the average time per request between EBS and SSD, and very little change between SSD and IOPS.
# Caveat

The m2 instance automatically formatted and mounted the SSD block storage to `/mnt`, but the i2 instance did not. See this [SO question](http://stackoverflow.com/questions/23092436/ssd-drives-not-accessible-in-ec2-ubuntu-instances) for more information. 

# Conclusions

We saw impressive improvements upgrading from EBS to SSD. We expected a similar jump in performance when upgrading from SSD to IOPS, based on the results of `dd`, but that didn’t happen, suggesting we moved disk io and we’re hitting other bottlenecks in the OS, MySQL or its configuration.

While these results are Amazon EC2-specific, our code can be run in any cloud, including your own hardware. We encourage you to try these benchmarks on Microsoft Azure, Google Cloud Platform, Rackspace, OVH, or Scaleway. The permutations of clouds and disk options becomes quick complex, which is why we’re taking the time to make these sorts of benchmarks generic and consumable by everyone. For more information on how to submit these tests, see [this page](http://blog.cloud-benchmarks.org/about/submit.html). 
