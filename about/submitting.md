---
title: Submitting to Cloud Benchmarking
layout: post
permalink: /about/submitting.html
date: 2015-01-01 00:00:00

---

The kind of comparative analysis that cloud-benchmarking.org hopes to foster depends on you the user to submit your benchmarking results. To do that, we've released a `benchmark-gui` charm in order to facilitate that.

The first step is to deploy the workload you wish to benchmark. It can be as simple or complex as you'd like. For example, if you wanted to test a simple Cassandra cluster, you could:

```
juju deploy -n3 cs:trusty/cassandra
```

Add the `benchmark-gui` to your environment:

```
juju deploy cs:~marcoceppi/trusty/benchmark-gui
juju set benchmark-gui juju-pass=asdfadsf
```
And relate the two:

```
juju add-relation cassandra benchmark-gui
```
At this point, you are ready to start benchmarking.

```
juju action do cassandra/0 stress n=2000000
```
run benchmarks (CLI-only)
use benchmark-gui to select and submit results


For the brave, submitting directly via API
