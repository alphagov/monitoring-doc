# 4. Cloudwatch Metrics

Date: 2018-06-28

## Status

Pending.

## Context

We would like to gather metrics from our own infrasctructure so that we can drive
alerts when things go wrong. Amazon exposes platform-level metrics via CloudWatch.
Prometheus provides a [cloudwatch_exporter](https://github.com/prometheus/cloudwatch_exporter) which makes these metrics available to prometheus.

We had a spike to see if we could use the Cloudwatch exporter app to get
Cloudwatch metrics and trigger alerts. Whilst estimating costs we had concerns regarding the [high costs](https://aws.amazon.com/cloudwatch/pricing/)
to run it due to the number of metrics requested using the Cloudwatch API.

Every time you hit the /metrics endpoint you will trigger the exporter to retrieve
metrics from the cloudwatch API. Every cloudwatch metric that you request costs
money (regardless of if you ask for 100 metrics using 1 metric per api call or
100 metrics using 100 metrics per api call).

cost = number of metrics on /metrics page x number of scrapes an hour x number of
hours in a year x price of requesting a metric using the API x number of prometheis running across all our environments.

If we try and find a lower bound for the cost by assuming a minimum of 100 metrics
that we would want to retrieve, being scraped once every 10 minutes, 3 proms in
prod, 3 proms in staging and 4 in dev accounts, it would work out roughly:

`100 x 6 x 24 x 365 x $0.00001 x 10 = $525.6 per year`

However if we wish to scrape at the same rate we would a normal target, e.g. 30
seconds that would become roughly $10,500 per year.

100 metrics also appears to be unlikely. Based on asking for just ALB and EBS
metrics as per https://github.com/alphagov/prometheus-aws-configuration-beta/blob/df53643de60ec121f8580d253b889a46148fa81b/terraform/projects/app-ecs-services/config/cloudwatch_exporter/config.yml, we appear to be requesting roughly 4000 metrics
per scrape. If we scraped our current config at a period of once every 10 minutes
we would end up roughly $21,000 a year. If we scraped it every 30 seconds it would
be about $420,000.

By requesting only ALB metrics in the dev accounts, we still produce about 160 API
calls according to the `cloudwatch_requests_total` counter for each scrape.
Somewhat strangely, this only returns about 30 timeseries so we are not sure if
our config is incorrect and if the number of API calls could be reduced to closer
match the number of timeseries.

Note, as dev accounts have lots of resources e.g. volumes, there may be fewer
metrics requested for staging and prod as unlike the dev account we wouldn't be
exporting metrics for other stacks.

We found the cloudwatch exporter app to be very CPU intensive, we had to double our instance size to a m4.2xlarge to get it to work. We were also concerned about having such a CPU intensive task running in the same instance as Prometheus.

There is roughly a 15 minute delay in cloudwatch metrics due to `CloudWatch has been observed to sometimes take minutes for reported values to converge. The default delay_seconds will result in data that is at least 10 minutes old being requested to mitigate this.`.

## Decision

We've proven that we are able to get the metrics from Cloudwatch but due to the
high cost of running the exporter app the team decided not to continue working on
this spike as it was not a viable solution. 

## Consequences

We currently have no alerts for EBS volumes not being attached to the instances.
This is mitigated by the user data script failing when the volume is not attached.
We also don't have metrics for ALBs and other parts of our AWS infrastructure.

We need to explore other solutions to get these metrics that are more 
cost-efficient.
