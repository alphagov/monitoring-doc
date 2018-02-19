# Importing Pingdom Metrics

## Introduction

We use pingdom as an external verification of our sites availability. As part of our monitoring
and metrics refresh we should be able to incorporate information from it into our dashboards.
This document provides some explanation of using the 
[prometheus-pingdom-exporter](https://github.com/giantswarm/prometheus-pingdom-exporter)
For a basic overview of the exporter you should start
by reading [blackbox-cert-check](./exporters/blackbox-cert-check.md).

## Metrics

The exporter returns metrics for all configured uptime checks. We'd need to add a custom dashboard
to present these in Grafana.

In order to check if the pingdom calls succeeded you should check
the `pingdom_up` metric.

You can view the metrics manually with the curl command

    curl -s 127.0.0.1:8000/metrics

This returns the metrics from the run, including `pingdom_up` to indicate if the last scrape
executed successfully. It does not seem to have a separate `probe_success` metric to verify
the exporter itself.

## Credentials

The exporter requires access to both a user and an API key. While we can currently
add more API keys, which I did during testing, our current pingdom account does not
have any additional user capacity which prevents us from using a locked down, isolated
user for these metrics. We'd need to upgrade our account, or use the RE centralised one
when it exists, to avoid this issue.

The credentials themselves are used on the command line when executing the exporter. We may
want to contribute code to read these from environment variables or even a pre-populated
configuration file.
