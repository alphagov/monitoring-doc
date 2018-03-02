# AlertManager

_The AlertManager handles alerts sent by client applications such as the Prometheus server. It takes care of deduplicating, grouping, and routing them to the correct receiver integration such as email, PagerDuty, or OpsGenie. It also takes care of silencing and inhibition of alerts._ -- [AlertManager](https://prometheus.io/docs/alerting/alertmanager/)

## Running a local  AlertManager

To test AlertManager locally you can run it via the [Docker Compose config](/exporters/exporter-docker-configs.yaml) provided in this repo. In addition to the raw container itself you will need to make two changes to the prometheus config, the addition of an alerting rules file and configuring prometheus
to send events to the alertmanager instance via an `alerting:` stanza in `prometheus.yml`

The rules file, named `alerts.rules` in our example here, contains the rules of which metrics should be monitored and alerted on in the event of the metrics breaching the given thresholds. A simple rule to test with is

```
cat prometheus/alerts.rules
groups:
- name: redis-available
  rules:
  - alert: RedisDown
    expr: redis_up == 0
    for: 1m
    labels:
      severity: "RE Tools"
    annotations:
      summary: Redis Availability alert.
      description: "This is my free form text"
```

This rule will raise an alert if the `redis_up` metric is a literal `0`. This is how the exporter exposes the availability of the backend redis it's
using. To use this rule you will need to plumb it into the prometheus configuration itself. This is done in two parts:

```
# from prometheus.yml
# ... snip ...
rule_files:
  - "/etc/prometheus/alerts.rules"
# ... snip ...
```

This tells prometheus which rules file to load. In the case of our docker-compose example we copy the entire `./prometheus` directory to `/etc/prometheus/` inside the container so you can just add the `alerts.rules` file and it will be placed correctly. The last thing to configure is the link between Prometheus and AlertManager itself.

```
# from prometheus.yml
# ... snip ...
alerting:
  alertmanagers:
    - static_configs:
      - targets: ['alert-manager:9093']
# ... snip ...
```

This uses the docker-compose service name (`alert-manager`) from our config to define where to find it.

Once this is done you should redeploy the stacks and visit the [Prometheus Alerting page](http://127.0.0.1:9090/alerts). You should see a single alert
named RedisDown, hopefully in light green.

In order to test the alert you can kill the redis container and watch it change state to PENDING and then FIRED.

```
docker kill promdocker_redis-server_1
```

Over the next few minutes this should change the status of the alert in both the [Prometheus Alerting page](http://127.0.0.1:9090/alerts)
and the [local AlertManager](http://127.0.0.1:9093/#/alerts).

![Raised Alert](/alertmanager/triggered-alert.png?raw=true "Triggered alert in prometheus and alertmanager")
