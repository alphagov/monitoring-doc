# Initial pass at Prometheus expired certificate check

## Introduction

In order to check the status of a remote website or network port you need to use the
[prometheus/blackbox_exporter](https://github.com/prometheus/blackbox_exporter). This
exporter allows you to probe remote endpoints via the HTTP. DNS, TCP or ICMP protocols.
In this example we'll attempt to use it to check if an SSL certificate on a website is valid.

The exporter itself is a simple Go binary and can be run directly from OSX or Linux with
a config file specified. Invoking it in this way allows for interactive testing.

    blackbox_exporter-0.11.0.darwin-amd64/blackbox_exporter \
        --config.file blackbox_exporter-0.11.0.darwin-amd64/blackbox.yml \
        --log.level=debug

You can see a very basic example config here:

    $ cat blackbox_exporter-0.11.0.darwin-amd64/blackbox.yml
    modules:
      http_2xx:
        prober: http
        timeout: 5s
        http:
          method: GET
          preferred_ip_protocol: "ipv4"
          # TODO explain this more.
          tls_config:
            insecure_skip_verify: false

Once you've added a module stanza to the config you can test it by curling the port directly:

    curl "http://127.0.0.1:9115/probe?target=https://www.unixdaemon.net&module=http_2xx&debug=true"

The `module` is the blackbox config stanza to test against, the target is the URL endpoint and the optional
`debug=true` provides a lot more context and debugging information.

    Logs for the probe:
    ts=2018-02-12T15:59:29.862621571Z caller=main.go:116 module=http_2xx target=https://www.unixdaemon.net
    ...snip...
    # HELP probe_success Displays whether or not the probe was a success
    # TYPE probe_success gauge
    probe_success 1
    ...snip...
    Module configuration:
    ...snip...

There are a few things more to investigate. The upstream do not consider the site being up or a certificate being invalid as
a useful difference: https://github.com/prometheus/blackbox_exporter/issues/194 This is something we, and a number of the programs,
do consider to be two different use cases and so we'll need to further investigate and refine our monitoring. It may be possible to
abuse the `probe_ip_protocol` output, which always seems to be `0` on failed requests to help with this.

If we decide to use this exporter, and in some other exporters too, it's important to note the difference between
the `up` and `probe_success` values. Up means that the exporter did its job, it has no bearing on the jobs
actual status. The `probe_success` value reflects if the endpoint and its associated tests are valid. In short
for blackbox probes, you should alert on `probe_success == 0`, not `up == 0`; although you probably want to check for that
too.

For the interactive testing I've been using [BadSSL](https://badssl.com/) to test each of the failure cases.

## Example alert configurion

You can be notified by prometheus of an expiring certificate based on a expression that uses the metric `probe_ssl_earliest_cert_expiry`.
This metric gives you the expiry time for a certificate in epoch format. You can then 

        - alert: SSLCertificatesExpiringIn7Days
          expr: timestamp(probe_ssl_earliest_cert_expiry) < 604800
          for: 1h
          labels:
            severity: "P3"
            risk: "Disruption"
          annotations:
            summary: "SSL certificate expriring in 7 Days"
            description: "The service {{ $labels.job }} has got SSL certificates expriring in 7 days. The URL that the certificate belongs is {{ $labels.instance }}"


Through this alert, you can see how the alerts are configured for certificate expiration. We have to adjust the expression according to the time the notification should be triggered. The metric returns the time value of the expression as an epoch value. When you write the condition you have to take this into consideration.

