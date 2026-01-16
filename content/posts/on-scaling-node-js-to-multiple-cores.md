---
title: On Scaling node.js to Multiple Cores
date: 2010-07-10T15:29:11+0100
categories:
- http
author: Bruno Fernandez-Ruiz
aliases:
- /http/on-scaling-node-js-to-multiple-cores/
---

From <a href="http://groups.google.com/group/nodejs/browse_thread/thread/36146559c089dca0">a thread</a> on multicore leverage in nodejs, <a href="http://blog.feedly.com/">Edwin Khodabakchian</a> talking about <a href="http://feedly.com/">feedly</a>:

> We have an "admin" node process which is started with as input the
> number of "feedly" node processes it should launch and monitor. When
> the admin starts, it spawns multiple "feedly" node processes, padding
> as input the port the feedly node should listen to 9701 for the first
> one,...9710 for the 10th one.
> 
> On the server we have a varnish server which load balances the traffic
> from m.feedly.com to 127.0.0.1:9701 --- 127.0.0.1:9710.
> 
> Each "feedly" node process has a 127.0.0.1:970x/heath endpoint where
> it reports stats about its execution. The admin node monitor each
> feedly node every 2 minutes and makes sure that the feedly node is
> healthy and responding. If not, it kills the child and restart a new
> instance of the feedly node listening to the same port.
> 
> All feedly nodes point to a redis instance for shared memory/session
> management. This allows incoming HTTP requests to be load balances
> across any feedly nodes transparently. In dev/staging, we have varnish
> and all the node processes running on the same box.
> 
> In production, we can scale out by having varnish on one server and
> multiple node cell (where a cell is an admin+10 feedly). This allows
> us to do rolling upgrades, etc. without any interruption to the
> service.
