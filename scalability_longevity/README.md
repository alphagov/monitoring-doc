# Scalability and Longetivity

## Clustering via Thanos

Thanos facilitates running a number of promtheus instances (which themselves are isolated) in a HA environment through its clustering. A thanos "sidecar" runs alongside each prometheus instance. This sidecar is configured to connect with other members in the same cluster. A thanos "query" component (a horizontally scalable API/UI) then connects to the cluster and queries the sidecar instances (which in turn query the prometheus running alongside it).

The sidecar cluster members communicate using a gossip protocol to keep the cluster up to date with current active members. The query plane takes care of deduplicating metric data from each of the underlying prometheus instances by utilising prometheus' "external labels".

To run the example cluster (2 prometheus instances, 2 sidecars and 1 query node):

```
docker-compose -f services.compose.yaml -f thanos-cluster.compose.yaml up -d
```

The thanos query plane is then available on http://localhost:10902.

## Long term metrics storage

There is a requirement to store metrics long term for later review / comparison (or other analysis). The performance of prometheus itself will degrade over time as the size of series grows (not to mention the size of filesystems required to host them) so alternative, external options were reviewed.

### Thanos

The thanos sidecar cluster provides long term storage of metrics data via S3 or GCS. At a configured interval blocks of prometheus metrics are uploaded to the object store (assuming they haven't been added already). Another thanos component, the "store", exposes the capability to the query plane to load metrics data from the object store.

To run the thanos s3 example, update `thanos-s3.compose.yaml` with the s3 settings and then run:

```
docker-compose -f services.compose.yaml -f thanos-s3.compose.yaml up -d
```

### InfluxDB
InfluxDB natively supports Prometheus' remote read and write APIs, making it an attractive candidate for long term storage. Prometheus metrics are sent to the remote write endpoint for long term storage in the database. The read endpoint is queried as part of any request for metrics from the prometheus API and returns any applicable metrics from the backend store (influxdb in this case).

Note: if prometheus is configured with a remote read endpoint, and that endpoint is unavailable, the query interface of the prometheus API itself is unavailable.

To run the influxdb example:

```
docker-compose -f services.compose.yaml -f influx.compose.yaml up -d
```

## InfluxDB storage and Thanos clustering
It is possible to combine the clustering provided by Thanos with a different long term storage solution (like influxDB).

To run the example (2 prometheus instances, 2 thanos sidecars, 1 thanos query node, and 1 influxdb instance):

```
docker-compose -f services.compose.yaml -f influx-thanos.compose.yaml up -d
```
