---
layout: default
title: Logging with Rsyslog, Node.js and MongoDB
class: post
---

{{page.title}}
================================


Centralized logging has been a pain point at [teamcoco.com](http://teamcoco.com) for some time now. After we migrated our stack to [AWS](http://aws.amazon.com), we didn't have a way to get a quick overview of the health of our software (AWS's cloudwatch provides a great way to track hardware health). SSH'ing into multiple boxes to try and track down issues was a huge pain. Worse than the general frustration with tailing logs, it was a huge time suck. So I set out to find a better logging solution.

Our requirements
-----------------

 1. **Easy Setup** - We use AWS auto scaling, so instances are constantly flowing in and out of rotation. The logging solution had to be easily installed and configured by our deployment system.
 2. **Light Weight** - We try to keep non-rendering resource usage (anything not PHP/nginx) on our front-end servers as thin as possible.  So the logging system had to have a small footprint.
 3. **Scalable** - We have massive traffic swings that are usually unpredictable ([@conanobrien](http://twitter.com/conanobrien) has a lot of followers). We needed a solution that could gracefully handle the traffic spikes, without needing to scale the storage servers.
 4. **Flexible** - we needed something that could handle not only error logs, but nginx access logs, mysql logs, and any other custom logs.
 5. **MongoDB Storage** - we already use Mongo for most of our backend storage, so it made sense to leverage a storage engine we already had running.

After taking a look at a bunch of open-source solutions ([Flume](http://flume.apache.org/), [GrayLog2](http://graylog2.org/) and [LogStash](http://www.logstash.net/) were early contenders) and a few SaaS solutions, [rsyslog](http://www.rsyslog.com/) was our choice.

1. It's simple to setup. Our deployment script just has to push our custom config to `/etc/rsyslog.d/` and the instance starts sending logs to our storage server.
2. So far we haven't had any issue with rsyslog hogging resources. One thing to watch out for is memory spikes on clinet instances when the storage server going down. rsyslog saves its message queue in memory and dumps the queue to disk if it's allotted memory fills. So any issues with the storage server can put strains on the client instances.
3. Since rsyslog works off a message queue, in general traffic spikes on the client instances are throttled, which protects the storage server. We had to mess around with rsyslog's `RateLimit.Interval` and `RateLimit.Burst` settings a bit to find a balance between protecting the server from flooding and prevent the queue on the client from growing to large.
4. We use rsyslog's [`imfile`](http://www.rsyslog.com/doc/imfile.html) input plugin (which tails a specific file) on our client instances. This allows us to specify a bunch of different log files and adding new log files is as simple as adding a new line to our config.
5. We use rsyslog's [`ommongodb`](http://www.rsyslog.com/doc/ommongodb.html) output module, which reads from a TCP and inserts messages into a mongo database, on our storage server.

How Our Setup Works
--------------------
Our setup is pretty simple:

1. Client instances write to local log files.
2. rsyslog on the client instances tails the log files and pushes logs to the storage server over TCP.
3. rsyslog on the storage reads the incoming TCP messages and writes them to a Mongo collection (`syslog.log`).
4. Several node.js scripts read from `syslog.log`, parses the raw log messages and inserts them into other collections.

An example rsyslog configuration from a client server:

{% highlight sh %}
    *.*  @@{ServicesIp}:{ServicesPort}
    $ModLoad imfile
    input(
        type="imfile"
        Facility="{InstanceName}"
        File="/var/log/nginx/tc-access.log"
        Tag="ngaccess"
        StateFile="/var/spool/rsyslog/ngaccess"
        Severity="info"
    )
    input(
        type="imfile"
        Facility="{InstanceName}"
        File="/var/log/nginx/tc-error.log"
        Tag="ngerror"
        StateFile="/var/spool/rsyslog/ngerror"
        Severity="error"
    )
{% endhighlight %}

Our deployment system replaces `{InstanceName}`, `{ServicesName}` and `{ServicesPort}` with the correct information during boot-up.

An example rsyslog configuration from our storage server:

{% highlight sh %}
    $ModLoad imtcp
    $InputTCPServerBindRuleset remote
    $InputTCPServerRun {ServicesPort}
    $ModLoad ommongodb
    $RuleSet remote
    *.* action(
        type="ommongodb"
        server="127.0.0.1"
        serverport="27017"
    )
{% endhighlight %}

This tells rsyslog to listen on port `{ServicesPort}` for incoming messages and write them to `syslog.log` in MongoDB.

We have a few different node.js scripts that run various transformations on the raw logs. For example, we have a script called error.js that parses our nginx and PHP error logs to match up requests that error out with the possible PHP error that caused it. Another script aggregates API endpoint request times. Another aggregates 404 errors and the URLs they occur on. All of the scripts broadcast the parsed messages to a [rabbitMQ](http://www.rabbitmq.com/) stream, which gives us realtime access to the data in our dashboard. Since everything is in MongoDB, we can use the `mongo` client to run one off queries.

As an added bonus, since we're parsing our nginx access logs, we are also able to add some custom analytics to our CMS. Now our editors are able to see how content is performing, in near-realtime (the end-to-end flow takes about two minutes from request to display).

A few things we learned
-----------------------

 * **Break apart your logs and assign distinct `Tags` to each type of log.** Having everything flow into a single log file/message stream might be simpler on the client side, but it makes efficiently parsing the logs a headache. With our access, error, and php error messages in separate streams, our parsing scripts can efficiently split up the work. Also, spikes in messages for any one type of stream doesn't drag down the processing of others (ie, traffic spikes, which case an increase in access messages, don't slow down processing of errors)
 * **Make sure to log the signal, not the noise** (I just finished reading [The Signal and the Noise](http://www.amazon.com/dp/159420411X) by Nate Silver and those terms are stuck in my head). It's tempting to log everything and say you'll figure the rest out later. But that can lead to so much useless information (noise), that the valuable information (signal) gets lost. It might take some iteration to figure out what's most important, but it's time well spent.
 * **rsyslog documentation can be confusing** The [documentation](http://www.rsyslog.com/doc/manual.html) is in blog format, which made some things difficult to find. Also checkout out the [wiki](http://wiki.rsyslog.com/index.php/Main_Page) for some additonal examples.
