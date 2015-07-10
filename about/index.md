---
title: Welcome to Cloud Benchmarking
layout: post
permalink: /about/index.html
date: 2015-01-01 00:00:00

---

We've all read the "Database vendor gets X,XXX,XXXX writes on this cloud" posts that are popular these days. However, wouldn't it be nice if you could build those types of benchmarks and then run them on the cloud yourself? Turn the knobs, see what changes, and then replicate it on any cloud.

![Go fast](/images/speedometer.jpg)

That's what this project is about, we give people the tools and means to grab any set of services, benchmark them, and publish the results. Anyone can then take those bits and replicate the tests in multiple clouds.

## How we got here

Benchmarking and performance are interesting problems, especially in today’s growing cloud-based microservice scene. It used to be a question of “how does this hardware compare to that hardware,” but as computing and service-oriented architectures grow the question has evolved. How does my cloud and application stack handle this? It’s no longer enough to run a benchmark on one machine and call it a day.

Measuring every microservice in your stack, from backend to frontend, is a complex task. We started thinking about how you would model a system to benchmark all of these services. It’s not just a matter of measuring the performance of one service, but also its interactions with other services. Now multiply that by every config option for every service, like PostgreSQL, which has hundreds of options that can affect performance.
 
As an example, measuring the effect of upgrading hardware is a solved problem. Generally speaking, buying all new gear leads to better performance. What we’re going after is what happens when you adjust any service in your stack in relation to every other service. In other words, measuring improvemens on what you already have. Turn every knob programmatically and measure it at any scale, on any cloud. Where exactly will you get the best performance: your application, the cache layer, or the backend database? Which configuration of that database stack performs best? Which microservice benefits from faster disk I/O? Make it so we can measure overall performance, but also measure each layer of the stack individually, and independantly of each other.

These are the kinds of questions we want answered.

## So are you trying to tell me what cloud to use? 

While we may intimately know a few services, we’re by no means _the_ experts on everything. We’ve created benchmarks for some popular services, such as mongodb, cassandra, hadoop, sql servers, and web applications, in order to provide a basic set of examples. Now we’re looking for community experts who are interested in benchmarking in order to fill the gap of knowledge.  Is your software being used in the cloud? You can now give users the means to compare and contrast your product on their clouds with optimal configuration instead of relying on a third party to get it right. 

This site does not endorse or care about individual cloud providers, or which piece of technology performs better than another; all we care about is providing tools and reusable bits of expertise for people to draw their own conclusions.

Go get em!

## How to contribute to this blog

This blog is open to anyone who is interested in sharing their experience in performance benchmarking, especially those doing workloads in clouds. 

- You can submit a PR on this repository with an entry, it's a simple [Github Pages](https://pages.github.com/) blog, just write something up in Markdown and send it in! 

