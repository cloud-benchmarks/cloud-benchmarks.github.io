---
title: Introduction
author: Adam Israel
layout: post
comments: true
---

[Who we are, what this is, and why we're doing it.]

While we may intimately know a few services, we’re by no means the experts. We’ve created benchmarks for some of popular services[1] in the charm store, such as mongodb, cassandra, mysql and siege, in order to provide a basic set of examples. Now we’re looking for community experts who are interested in benchmarking in order to fill the gap of knowledge. We’re excited about performance and how Juju can be used to model performance validation. We need more expertise on how to stress a service or workload to measure that performance.


#Running your own benchmarks and submitting them to the community

We’ve built a site so anybody can submit these benchmarks and then take them, modify them, run them, recombine them in crazy combinations, whatever people want to do. We feel that crowdsourcing benchmarking combinations of services on public clouds is a great way for users to investigate performance, price, and scalability before they make platform decisions.

As an example, monitoring the effect of adjusting the cache in nginx is a solved problem. What we’re going after is what happens when you adjust any service in your stack in relation to every other service. Turn every knob programmatically and measure it at any scale, on any cloud. Where exactly will you get the best performance: your application, the cache layer, or the backend database? Which configuration of that database stack performs best? Which microservice benefits from faster disk I/O? These are the kinds of questions we want answered.
