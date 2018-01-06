+++
aliases = ["/development/2015/09/04/monitoring-rails-requests-statsd/"]
date    = "2015-09-04T17:03:00-05:00"
tags    = ["ruby", "rails"]
title   = "Monitoring Rails Requests with StatsD"
+++

Monitoring your application is important - full stop. When the response time plummet's you want to know before your
customers do...and certainly before you bother to check your twitter feed.

There are lots of options here, and a lot of them require a ton of upfront investment in terms of time and effort.
However, a simple way to get started is to simply log total request durations and track the number of successful and
failed responses.

This way you can setup alerts when the success rate is below a certain threshold, or when the mean response time goes
from 80ms to 2s.

[StatsD] can really help here, and I'm particularly fond of [DataDog]. IMHO, being able to visualize your request
lifecycle and get notified when things go awry is critical to your application's success.

TL;DR - [monitor your requests]({{< relref "2015-09-monitoring-rails-requests-statsd.md#tldr" >}})

## Setting Up StatsD

I'm going to use the [statsd-instrument] gem written by Shopify. It takes care of handling different backends based on
the environment, provides a clean wrapper around StatsD calls, and ships with a railtie.

To get started, you'll need to add the gem to your Gemfile and add an initializer.

{{< gist pseudomuto aff211b9d08d7f2d00a5 "Gemfile" >}}

{{< gist pseudomuto aff211b9d08d7f2d00a5 "statsd_initializer_01.rb" >}}

As I mentioned, the gem takes care of using different backends per environment.

The special cases are:

* `production` or `staging` - calls are sent via UDP to `ENV["STATSD_ADDR"]`
* `test` - calls are ignored

All other environments will log calls to `Rails.logger`.

For more details, check the README for [statsd-instrument].

## Measuring Request Duration

To measure request duration, we can use middleware. The trick here is to insert it at position 0. That way the entire
request lives and dies within the middleware's `call` method, making the measurement really simple.

In this post I'm going to create `Middleware::StatsDMonitor` and expect Rails to find it in
`middleware/statsd_monitor.rb` (rather than `middleware/stats_d_monitor.rb`).

Fortunately, this isn't the first time this has come up, and there is a solution. Add an acronym for `StatsD` in
`config/initializers/inflections.rb`.

{{< gist pseudomuto aff211b9d08d7f2d00a5 "inflections.rb" >}}

### Test for Request Duration Measurement

Normal controller tests don't load the middleware stack. So for this, we'll need to write an integration test. Add the
following to `test/integration/statsd_request_monitoring_test.rb`:

{{< gist pseudomuto aff211b9d08d7f2d00a5 "test_duration.rb" >}}

Now we can watch it fail so hard by running `bin/rake test:integration`.

### Make It Work

Create the new middleware class in `app/middleware/statsd_monitor.rb`

{{< gist pseudomuto aff211b9d08d7f2d00a5 "app_middleware_statsd_monitor.rb" >}}

Next, add the middleware to the beginning of the stack.

{{< gist pseudomuto aff211b9d08d7f2d00a5 "application.rb" >}}

Finally, we need to go back to our StatsD initializer, and tell it to measure the calls.

{{< gist pseudomuto aff211b9d08d7f2d00a5 "statsd_initializer_02.rb" >}}

### OK, So Far, So Good

Running `bin/rake test:integration` should now pass. Now you can setup alerts for request duration spikes. Or at least
show the average request time on a dashboard somewhere.

## Keeping Counters for Response Codes

Request duration is a good start, but ideally, you'll want to increment a counter everytime a response (that we're
interested in) happens. This way you can setup alerts if `400, 404, 429, 500, or 502` responses spike, or if `200 or
302` responses dip.

### Writing the Tests

Add the following tests to the integration test. Each one simply ensures that a particular counter was incremented.

{{< gist pseudomuto aff211b9d08d7f2d00a5 "test_counters.rb" >}}

### Making Them Pass

To increment the right counter, we're going to use `stats_count_if`. This will increment the count if the supplied block
returns `true`.

{{< gist pseudomuto aff211b9d08d7f2d00a5 "statsd_initializer_03.rb" >}}

## All Together Now
<a href="#" name="tldr"></a>

For those who didn't want to read or just like copy pasta, here are the complete files.

{{< gist pseudomuto aff211b9d08d7f2d00a5 "Gemfile" >}}

{{< gist pseudomuto aff211b9d08d7f2d00a5 "final.rb" >}}

Happy Coding!

[statsd-instrument]: https://github.com/Shopify/statsd-instrument
[StatsD]: https://github.com/etsy/statsd
[DataDog]: https://www.datadoghq.com/
