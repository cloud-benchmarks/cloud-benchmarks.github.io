---
title: Submitting to Cloud Benchmarking
layout: post
permalink: /about/submit.html
published: true
date: 2017-07-17 00:00:00
highlight: submit
---

The kind of comparative analysis that cloud-benchmarking.org hopes to foster depends on you the user to submit your benchmarking results. To do that, we've released a `benchmark-gui` charm in order to facilitate that. Check out the [Getting Started](https://jujucharms.com/get-started) page to set up Juju on the cloud you want to test on. 

The first step is to deploy the workload you wish to benchmark. It can be as simple or complex as you'd like. For example, if you wanted to test a simple Cassandra cluster, you could:


    juju deploy -n3 cs:trusty/cassandra
    juju deploy cs:~marcoceppi/trusty/cassandra-stress


Add the `benchmark-gui` to your environment:


    juju deploy cs:~marcoceppi/trusty/benchmark-gui
    juju set benchmark-gui juju-pass=$(grep "password" ~/.juju/environments/$(juju switch).jenv | awk '{print $2}')


And relate the two:


    juju add-relation cassandra benchmark-gui
    juju add-relation cassandra cassandra-stress


At this point, you are ready to start benchmarking. You can read more about writing and running benchmarks in the [Charm Authors Guide to Benchmarking](https://jujucharms.com/docs/stable/authors-charm-benchmarks).


    juju action do cassandra-stress/0 stress operations=2000000


Open the benchmark-gui in your web browser by browsing to `http://unit-ipaddress/`. From there, you can select the relevant results and submit them by clicking "Publish to loud-benchmarking.org"
