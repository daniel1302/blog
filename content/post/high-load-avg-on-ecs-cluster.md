---
title: My ECS cluster is burning. High load on ECS cluster!
date: 2020-01-05T10:00:00Z
draft: false
Summary: High load avg on the ECS cluster? This article will show how to start tracking the issue. The problem is quite common. It is not related only to the ECS. The same problem you can find on the K8S cluster or any other system.
---

### Introduction

For the last 2/3 weeks, I observe some performance issue on our cluster. On the ECS cluster, We have a few dozens of application containers. When We are running several applications, it's difficult to tell which process causes the issue. The Load avg per cluster instance is higher than 2, which is not bad at all, because applications are running smoothly. The problem starts when it's growing to 4 and higher.

![Cluster load avg](/static/cluster-load-avg-1.png)

### On the trail of the problem

My ex-boss told me all the time when I did not know what to do "Do not do random things, find a way to figure out what is wrong!".

I use CloudWatch to monitor all our services. AWS monitoring does not give us accurate pieces of information per instance by default. You can turn it on, but the CloudWatch metrics agent uses more CPU than I want to allocate for it. The second important thing you should know, AWS calculates the cost per CloudWatch request(metric push, get, etc...), which is expensive for a longer-term. 

### Monitoring system

I have designed monitoring system using Prometheus and Grafana. It looks like below on the image.

![Monitoring](/static/monitoring.png)

#### System components

- Docker containers: Containers with real applications running under ECS.
- [cAdvisor](https://github.com/google/cadvisor): Analyzes resource usage and performance characteristics of running containers.
- [Node Exporter](https://github.com/prometheus/node_exporter): Exporter for machine metrics.
- [Prometheus](https://prometheus.io/): Monitoring system and time series database.
- [Grafana](https://grafana.com): Monitoring and analytics platform that works with every database.

### Understand the problem

The first thing before you start tracking the problem is to understand the problem. High avg load is not only a CPU problem, how people can thinking. The CPU is very idle when the load average is high.

![CPU utilization](/static/cpu-usage.png)

To understand the problem read the following article: https://www.tecmint.com/understand-linux-load-averages-and-monitor-performance/

### Preparing for tracking

The first thing that I did was prepare the application stress test. I achieved it with the Apache JMeter for few applications. 

### Root of the issue

After I have run a stress test, CPU utilization looks horrible.

![Load avg after stress test](/static/load-avg-after-stress-test.png)

Then I started looking for a correlation between the load avg and something else. I noticed the disk utilization is very high, which you can see on the following image.

![Disk utilization after stress test](/static/disk-utilization-after-stress-test.png)

Then I have started looking for disk usage per container. I found one specific application with too high disk load. Once I have found the guilty app, it was much easier to fix the source of the problem.