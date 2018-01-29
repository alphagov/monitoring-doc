# Query examples #

This document is mean to contain useful queries developed during the Reliability Engineering Team investigations or the colaboration with other teams.

See [Prometheus documentation](https://prometheus.io/docs/prometheus/latest/querying/basics/) for more information.

All the text following the pattern `<text>` is mean to be replaced with existing values in real environments.

The name of the metrics can differ from one server to another.

### Is the system running? ###
`up{instance="<instance_tag>"}`

### HTTP status codes rate ###
`sum(http_server_requests_total{code=~"4..", instance="<instance_tag>"}) / sum(http_server_requests_total{instance="<instance_tag>"})`

_Note: 4.. in the code pattern can be replaced with different numbers._

### Average response time ###
`sum(http_server_request_duration_seconds_sum) / sum(http_server_request_duration_seconds_count) * 1000`

### Slowest 5 requests (average) ###
`topk(5, http_server_request_duration_seconds_sum / http_server_request_duration_seconds_count * 1000)`

### More called endpoints ###
`topk(10, http_request_total or http_requests_total or http_server_requests_total)`

### CPU used per minute group by instance ###
`sum(rate(process_cpu_seconds_total{}[1m])) by (instance)`

## Request total per minute by instance ###
`sum(rate(http_requests_total[1m])) by (instance)`
