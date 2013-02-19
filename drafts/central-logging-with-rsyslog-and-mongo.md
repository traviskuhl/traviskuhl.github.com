---
layout: default
title: Logging with Rsyslog, Node.js and MongoDB
class: post
---

{{page.title}}
================================


Logging has been a pain point at [teamcoco.com](http://teamcoco.com) for some time now. After we migrated our stack to [AWS](http://aws.amazon.com), we no longer had a way to get a quick overview of the software health of our stack (AWS's cloudwatch provides a great way to track hardware health). With the new, more traditional, tiered architecture, SSH'ing into multiple boxes to try and track down issues was a huge pain. Worse than the general furstration with tailing logs, it was a huge time suck. So I set out to find a better logging solution.

Our requirements
-----------------
Our requirements were pretty straight forward:

 1. Easy Setup - we use AWS auto scalling, so instances are constantly flowing in and out of rotation. The logging solution had to be easily install and configured by our deployment system.
 2. Light Weight - we try to keep non-rendering resource usage (anything not PHP/nginx) on our front-end boxes as thin as possible. We want as much of the resources on the instance to go toward page rendering. So the logging system had to have a small footprint
 3. Scalable - teamcoco has massive traffic swings. We regularly get large spikes in traffic (@conanobrien has a lot of followers). We needed a solution that gracefully handle the traffic spikes, without needing to add more aggregation servers.
 4. Flexible - we needed something that could handle not only error logging, but nginx access logs, mysql logs, and any custom logs we keep.
 5. MongoDB Storage - we already use mongodb for most of our backend storage, so it made sense to leverage a storage engine we already had running.

After taking a look at a bunch of solutions, including a few SaaS solutions, rsyslog was our choice.
1. It's simple to setup. Our deployment script just has to push our custom config to `/etc/rsyslog.d/` and the instances starts sending logs to our aggregation server
2. So far we haven't had any issue with rsyslog hogging resources. One thing to watch out for is the aggregation server going down. rsyslog saves its message queue in memory and dumps the queue to disk if it's allotted memory fills. So any issues with the aggregation server can put strains on the client instances.
3. Since rsyslog works off a message queue, in general traffic spikes on the client instances are throttled, which protects the aggregation server. We had to mess around with rsyslog's `RateLimit.Interval` and `RateLimit.Burst` settings a bit to try and strike a balance between protecting the server from flooding and prevent the queue on the client from growing to large.
4. We mostly use the rsyslog `imfile` input plugin (which tails a specific file). This allows us to specify a bunch of different log files and adding new log files is as simple as adding a new line to our config.
5. We use the rsyslog `ommongodb` output module, which reads from a TCP and inserts messages into a mongo database

How Our Setup Works
1. We have four main log files on our front-end:
    1. tc-access.log (our general nginx access logs)
    2. tc-access-embed.log (nginx access logs for our video embeds)
    3. tc-error.log (nginx error logs)
    4. tc-php-error.log (php error log)

The rsyslog config on our front-ends looks something like:

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
    input(
        type="imfile"
        Facility="{InstanceName}"
        File="/var/log/nginx/tc-access-embed.log"
        Tag="ngembed"
        StateFile="/var/spool/rsyslog/ngembed"
        Severity="info"
    )
    input(
        type="imfile"
        Facility="{InstanceName}"
        File="/var/log/tc-php-errpr.log"
        Tag="phperror"
        StateFile="/var/spool/rsyslog/phperror"
        Severity="error"
    )

Our deployment system replaces `{InstanceName}`, `{ServicesName}` and `{ServicesPort}` with the correct information during boot-up. We have similar configs on our other layers.

2. The rsyslog config on our aggregation server looks like:
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
This tells rsyslog to listen on port `{ServicesPort}` for incoming messages and write them to `syslog.log` in MongoDB.

3. We have a few different node.js scripts that run various transformations on the raw logs. For example, we have a script called error.js that parses our nginx and PHP error logs to match up requests that error out, with the possible PHP error that caused it. Another script aggregates API endpoint request times. Another aggregates 404 errors and the URLs they occur on. All of the scripts also broadcast the parsed messages to a rabbit-mq stream, which gives us realtime access to the data in our dashboard. Since everything is in MongoDB, we can also use the command line client to run one off queries. We use graphite to graph our aggregated data in our dashboard.

As an added bonus, since we're parsing our nginx access logs, we were also able to add some custom analytics to our CMS for our Editors. Now they're able to see how a peice of content is performing, whwere traffic is coming from, etc, in near-realtime.

A few things we learned:
 * Break apart your logs and assign distinct `Tags` to each type of log. Having everything flow into a single log file/message stream might be simpler on the client side, but it makes efficient parsing the logs a headache. With our access, error, and php error messages in separate streams, our transform scripts can efficiently split up the work of parsing/reporting. Also, spikes in volume for any one type of stream, doesn't drag down the processing of others (ie, traffic spikes, which case an increase in access messages, don't slow down processing of errors)
 * Make sure to log the signal, not the noise (I just finished ready [The Signal and the Noise](http://www.amazon.com/dp/159420411X) by Nate Silver and those terms are stuck in my head). It's tempting to log everything and figure the rest out later. But that can lead to so much useless information (noise), that the valuable information (signal) gets lost. It might take some iteration to figure out what's most important, but it's time well spent.





