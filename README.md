# Grafana-Prometheus Envoy Dashboards

Ported from the [Lyft Envoy dashboards](https://github.com/mattklein123/lyft_envoy_dashboards)

These Envoy Grafana dashboards use a Prometheus datasource.

I've tried to use the native Envoy stats endpoint for most of the data, but the timers aren't
currently exposed that way. As a result to get the timing data you need to run a prometheus
statsd exporter locally to your Envoy, with the mapping config from statsd\_exporter.yml and
Enovy pushing stats into that. We can then use the statsd exporter to get the histogram data
and the native stats endpoint for everything else.

The example prometheus.yml uses Consul for service discovery, and does a whole bunch of
relabeling. There's also prometheus-no-consul.yml which doesn't use Consol and relies on our
host naming conventions to work out what labels to add.  

## Histograms vs Summary
The statsd exporter config uses histograms (currently with default bucketing, you can change
this if you need to) rather than summary. This is because the dashboard expects to be able to
aggregate across multiple instances of the same service. You cannot do that with summaries
since e.g. avg(99th %ile) across multiple instances is completely meaningless.

The choices were to either have various tiers of statsd reciever performing the summary at the
different aggregation levels we want (per instance, per source service, per destination service,
and every combination of the above) or to give up and use histograms which do allow aggregation.
The big downside of histograms is that granularity is limited to your buckets. As long as you
configure your buckets sanely for your application this ought to be fine, but be warned that
the largest number you'll see on your response time graphs will be the top bound of the highest
non +Inf bucket!

Obviously if Envoy starts supporting the histograms in the stats output then they absolutely
have to be histograms rather than summaries!

## What isn't there?
* Canary stats. We don't use them yet.
* Cross zone stats. We don't use them yet.
* External Ingress stats. We don't use it that way yet.

## TODO
Other than the obvious "finish this porting excercise" work:
* Remove statsd exporter usage completely once Enovy stats endpoint supports histograms
* Use Prometheus recording rules for some of the more expensive to calculate aggregations
* Work out how to deal with metrics that haven't been initialized yet (e.g. avoiding Success Rate no data when the
response_code_class=5 label doesn't exist yet.)
