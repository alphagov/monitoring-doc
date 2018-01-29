# Query examples #


## Page index
1. [Is the system running?](./#q001)
2. [HTTP status codes rate](./#q002)
3. [Average response time](./#q003)
4. [Slowest 5 requests (average)](./#q004)
5. [More called endpoints](./#q005)
6. [CPU used per minute group by instance](./#q006)
7. [Request total per minute by instance](./#q007)

## Overview

This document is mean to contain useful queries developed during the Reliability Engineering Team investigations or the colaboration with other teams.

See [Prometheus documentation](https://prometheus.io/docs/prometheus/latest/querying/basics/) for more information.

All the text following the pattern `<text>` is mean to be replaced with existing values in real environments.

The name of the metrics can differ from one server to another.

### Is the system running?
###### Q001
`up{instance="<instance_tag>"}`

### HTTP status codes rate
###### Q002
`sum(http_server_requests_total{code=~"4..", instance="<instance_tag>"}) / sum(http_server_requests_total{instance="<instance_tag>"})`

_Note: 4.. in the code pattern can be replaced with different numbers._

### Average response time
###### Q003
`sum(http_server_request_duration_seconds_sum) / sum(http_server_request_duration_seconds_count) * 1000`

### Slowest 5 requests (average)
###### Q004
`topk(5, http_server_request_duration_seconds_sum / http_server_request_duration_seconds_count * 1000)`

### More called endpoints
###### Q005
`topk(10, http_request_total or http_requests_total or http_server_requests_total)`

### CPU used per minute group by instance
###### Q006
`sum(rate(process_cpu_seconds_total{}[1m])) by (instance)`

### Request total per minute by instance
###### Q007
`sum(rate(http_requests_total[1m])) by (instance)`
