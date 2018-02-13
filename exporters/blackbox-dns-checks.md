# DNS Checks via the Blackbox exporter

## Introduction

This document contains some example
[prometheus/blackbox_exporter](https://github.com/prometheus/blackbox_exporter) configurations
for checking different aspects of DNS. For a basic overview of the exporter you should start
by reading [blackbox-cert-check](./exporters/blackbox-cert-check.md).

## Checks

### Simple A record check

In this example config we will check that an A record matches a given IPv4 address.

```
modules:
  resolve_a_record:
    prober: dns
    timeout: 5s
    dns:
      query_type: "A"
      query_name: "www.unixdaemon.net"
      preferred_ip_protocol: "ip4"
      validate_answer_rrs:
        fail_if_not_matches_regexp:
        - ".*127.0.0.7"
```

This can be invoked interactively with the curl command.

    $ curl "http://127.0.0.1:9115/probe?target=8.8.8.8&module=resolve_a_record&debug=true"
    # Name server to resolve against           ^^^^^^^  
    # Config module to use                                    ^^^^^^^^^^^^^^^^
    # Optional debug flag                                                      ^^^^^^^^^^^

This returns the metrics from the run, including `probe_success` to indicate if the exporter
executed successfully. It's worth noting when the check fails the metrics do not include any
additional information, which is present in the debug log, as labels to help isolate the failure
type. You'll typically have to rerun queries by hand to track the issue, which is not ideal.
