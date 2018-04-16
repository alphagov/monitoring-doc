# Monitoring documentation
###### By GDS Reliability Engineering Team

## Project description

This documentation gives an overview of work completed by the GDS Reliability Engineering Team.
This is a living document and is in constant change during the discovery and exploration phases.
This documentation will contain useful resources for teams setting up monitoring tools and for us to support them.

### Quick overview

The solution is based on:
* [Prometheus](https://prometheus.io/)
* [Grafana](https://grafana.com/)

## Index

1. [Documentation](./documentation)
2. [Guidance and best practises](./guidance)
3. [Dashboard templates](./dashboard_templates)
4. [Query examples](./queries)
5. [Useful resources](./resources)
6. [Exporter notes](./exporters)
7. [Alert Manager](./alertmanager)
8. [Diagrams](./diagrams)
9. [Architecture Decision Records](./documentation/architecture/decisions)

## Architecture Decision Records

We will record design decisions for the architecture to ensure we preserve the context of our
choices. These will be written in the format proposed in a
[blog post by Michael Nygard](http://thinkrelevance.com/blog/2011/11/15/documenting-architecture-decisions)

Please see the [decisions directory](decisions/) for a list of all ADRs.

### Tooling

We will use [adr-tools](https://github.com/npryce/adr-tools) to help manage the decisions.

`brew install adr-tools`

`adr new 'Decision to record'`

Please ensure that this tool is used at the **root** of the repository only.
