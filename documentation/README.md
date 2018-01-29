# Monitoring solution documentation #

## Page index ##
1. [Page index](https://github.com/alphagov/monitoring-doc/documentation#page-index)
2. [Overview](https://github.com/alphagov/monitoring-doc/documentation#overview)
3. [Something about Prometheus](https://github.com/alphagov/monitoring-doc/documentation#something-about-prometheus)
  1. [Architecture](https://github.com/alphagov/monitoring-doc/documentation#architecture)
  2. [Data Model](https://github.com/alphagov/monitoring-doc/documentation#data-model)
4. [Something about Grafana](https://github.com/alphagov/monitoring-doc/documentation#something-about-grafana)
5. [Something about our solution](https://github.com/alphagov/monitoring-doc/documentation#something-about-our-solution)

## Overview ##

The moniroting platform implemented by the GDS Reliability Engineering team is a SaaS platform to allow the different teams in GDS to capture metrics of their applications.

This platform is currently under development.

Having a common platform to capture, process and show metrics results across GDS has the next adventages:
* Standardize the way metrics are managed. Produced, ingested and presented.
* It allows to have a solution add to new projects where previous knowledge can be used.
* It avoid us (as an organization) to solved the same problem multiple times.
* Gives us a common framework and solid ground to continue improving.
* Makes easier to deal with availability, support and resiliancy.

The two products selected to build the monitorizations system are:
* [Prometheus](https://prometheus.io/)
* [Grafana](https://grafana.com/)

## Something about Prometheus ##

### Architecture ###

Prometheus servers scrape (pull) metrics from instrumented jobs. There is no distributed storage. Prometheus servers store all metrics locally. They can run rules over this data and generate new time series, or trigger alerts. Servers also provide an API to query the data. Grafana utilizes this functionality and can be used to build dashboards.

Finally, the available servers to scrape can be defined as a static configuration or service discovery. Service discovery is more common and also recommended.

![alt Prometheus architecture](https://prometheus.io/assets/architecture.svg "Prometheus architecture")

### Data Model ###

Prometheus stores all data as time series. A time series is a stream of timestamped values that belong to the same metric and the same labels. The labels cause the metrics to be multi-dimensional.

For example, if we wish to monitor the total amount of HTTP requests on our API, we could create a metric named `api_http_requests_total`. Now, to make this metric multi-dimensional, we can add labels. Labels are simple key value pairs. For HTTP requests, we can attach a label named method that takes the HTTP method as value. Other possible labels include the endpoint that is called on our API, and the HTTP status returned by the server for that request.

The notation for a metric like that could be the following:

`api_http_requests_total{method="GET", endpoint="/api/users", status="200"}`

## Something about Grafana ##

Grafana is an open source metric analytics & visualization suite. It is most commonly used for visualizing time series data for infrastructure and application analytics but many use it in other domains including industrial sensors, home automation, weather, and process control.


## Something about our solution ##

The solution implementens a metrics monitoring platform based on Prometheus and Grafana, all of them deployed using AWS and GOV.UK PaaS. This is possible using tools like terraform to manage the Infrastructure as a Code and TravisCI to perform automatic deployments.

* Prometheus terraform configuration can be found [here](https://github.com/alphagov/prometheus-aws-configuration).
* Grafana deplyment configurations can be found [here](https://github.com/alphagov/grafana-paas).

00x00 Diagram of the architecture and short description of the picture.


