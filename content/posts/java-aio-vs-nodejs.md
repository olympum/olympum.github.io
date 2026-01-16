---
title: Java AIO (NIO.2) vs NodeJS
date: 2011-01-30T09:13:32+0000
categories:
- Java
author: Bruno Fernandez-Ruiz
aliases:
- /java/java-aio-vs-nodejs/
---

I just installed OpenJDK 7 on Ubuntu 10.04 LTS under VirtualBox (one core, 256 MB). I wanted to run a quick test to see how the new JDK7 Async channel APIs were performing in comparison with node. A simple test of a hello world running on a single core shows that the JVM truly has what it takes to be the best runtime for network servers. Most notably, compare the distribution of response times, especially at 99%. The advantage the JVM is showing might be enough to comfortably fit Rhino onto it.

<p>Here are the numbers from Apache benchmark. First, Java AIO (build b127):</p>
<pre><code>brunofr@ubuntu-vm:~$ ab -n 30000 -c 300 http://127.0.0.1:8080/
Concurrency Level:      300
Time taken for tests:   4.908 seconds
Complete requests:      30000
Requests per second:    6112.65 [#/sec] (mean)
Time per request:       49.079 [ms] (mean)
Transfer rate:          567.26 [Kbytes/sec] received

Percentage of the requests served within a certain time (ms)
  50%      8
  66%      8
  75%      9
  80%     11
  90%     12
  95%     13
  98%     14
  99%     18
 100%   4553 (longest request)
</code></pre>
<p>And then nodejs (from Ryan's branch):</p>
<pre><code>brunofr@ubuntu-vm:~$ ab -n 30000 -c 300 http://127.0.0.1:8124/
Concurrency Level:      300
Time taken for tests:   8.140 seconds
Complete requests:      30000
Requests per second:    3685.69 [#/sec] (mean)
Time per request:       81.396 [ms] (mean)
Transfer rate:          273.55 [Kbytes/sec] received

Percentage of the requests served within a certain time (ms)
  50%     35
  66%     44
  75%     50
  80%     53
  90%     60
  95%     65
  98%     75
  99%     86
 100%   3285 (longest request)
</code></pre>
<p>Note that right now AsynchronousChannel only has platform support on Linux and Solaris (you'll get a java.lang.InternalError: platform not recognized running on OSX).</p>
