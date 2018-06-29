# 4. Cloudwatch Metrics

Date: 2018-06-28

## Status

Accepted.

## Context

We wanted to gather metrics from our own infrasctructure so that we can drive alerts when things go wrong. Amazon exposes platform-level metrics via CloudWatch.
Prometheus provides a [cloudwatch_exporter](https://github.com/prometheus/cloudwatch_exporter) which makes these metrics available to prometheus.

We had a spike to see if we could use the Cloudwatch exporter to get Cloudwatch metrics and trigger alerts. 
After getting it to work with a number of metrics we were able to start to estimate costs, from these estimate we had concerns regarding the [high costs](https://aws.amazon.com/cloudwatch/pricing/)
in running it due to the number of metrics requested using the Cloudwatch API.

Each time the `/metrics` endpoint was hit triggers the exporter to retrieve
metrics from the Cloudwatch API. Every Cloudwatch metric that you request costs
money (regardless of if you ask for 100 metrics using 1 metric per api call or
100 metrics using 100 metrics per api call).

cost = `number of metrics on /metrics page` x `number of scrapes an hour` x `number of hours in a year` x `price of requesting a metric using the API` x `number of prometheis running across all our environments`.

Based on a simple assumption of 100 metrics requested, being scraped once every 10 minutes (6 per hour), 3 Prometheis in production, 3 Prometheis in staging and 4 in dev accounts, it would work out:

`100 x 6 x 24 x 365 x $0.00001 x 10 = $525.6 per year`

However if we wish to scrape at the same rate we would a normal target, e.g. 30
seconds that would become roughly $10,500 per year.

100 metrics also appears to be unlikely. Based on asking for just these ALB and EBS
metrics:

```
ApplicationELB/RequestCount
ApplicationELB/TargetResponseTime
ApplicationELB/HTTPCode_ELB_5XX_Count
EBS/VolumeWriteBytes
EBS/VolumeReadBytes
EBS/VolumeReadOps
EBS/VolumeWriteOps
```

we appear to be requesting roughly 4000 metrics per scrape. If we scraped our current config at a period of once every 10 minutes we would end up roughly $21,000 a year. If we scraped it every 30 seconds it would
be about $420,000.

By requesting only ALB metrics in the dev accounts, we still produce about 160 API requests according to the `cloudwatch_requests_total` counter for each scrape. Somewhat strangely, this only returns about 30 timeseries so we are not sure if our config is incorrect and if the number of API calls could be reduced to closer
match the number of timeseries.

Note, as dev accounts have lots of resources e.g. volumes, there may be fewer
metrics requested for staging and prod as unlike the dev account we wouldn't be
exporting metrics for other stacks.

It takes a long time to get a response from the /metrics endpoint as it needs to make many API calls. This causes slow response times for which our prometheus scrape config needs to allow for using the scrape_timeout setting

For speed of development, we still are only running one instance per ECS cluster. There were questions raised about having such as CPU intensive process (that was prone to crashing due to CPU spikes) running on the same instance as prometheus. However we didn't do anything about this as it would require a lot of work to have more than one instance per cluster.

Config race condition was encountered every time with config grabbing for the exporter meaning either a 2nd restart of the task needed or we would need to add a delay to the exporter starting. We did not try and solve this

Exporter task is slow to start up, health check needs longer than usual to pass health checks

We spotted that several targets we are scraping such as prom, alertmanager are set at very short scrape intervals (every 5s), this seems excessive and we can likely change down to every 30secs regardless of this story

We found the Cloudwatch exporter app to be very CPU intensive, we had to double our instance size to a `m4.2xlarge` to get it to work. We were also concerned about having such a CPU intensive task running in the same instance as Prometheus.

There is roughly a 15 minute delay in Cloudwatch metrics due to `CloudWatch has been observed to sometimes take minutes for reported values to converge. The default delay_seconds will result in data that is at least 10 minutes old being requested to mitigate this.` - from [Prometheus CloudWatch exporter README](https://github.com/prometheus/cloudwatch_exporter/blob/master/README.md).

## Decision

We've proven that we are able to get the metrics from Cloudwatch but due to the high cost of running the CloudWatch exporter the team decided not to continue working on this spike as it was not a viable long term solution. 

## Consequences

We will not have alerts for EBS volumes not being attached to the instances, which was a concern as Prometheus would start but no metrics stored.

This has however been fixed in this [commit](https://github.com/alphagov/prometheus-aws-configuration-beta/commit/cd2432045dfd8fc10d7fa1ae34f4dfed63fc9f11), which resolves this by causing the user data script to exit with failure when the volume is not attached. 

No alert is raised but the failure is no longer silent as Prometheus will no longer run without an attached EBS volume.

We also don't have metrics available for ALBs and other parts of our AWS infrastructure.

We need to explore other solutions to get these metrics that are more cost-efficient.
