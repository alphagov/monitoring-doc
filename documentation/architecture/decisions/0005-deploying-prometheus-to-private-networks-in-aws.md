# 1. Model for deploying prometheus to private networks in AWS

Date: 2018-07-26

## Status

Draft

## Context

We are looking to deploy prometheus into private networks alongside
existing infrastructure.  The existing infrastructure will be run by
another team (called "client team" in this document), but our
prometheus will be responsible for gathering metrics from it.

Longer term, we imagine that we will have several prometheis, living
in multiple environments across multiple programmes.

Some of the things we would like to be able to do:

 - maintain prometheus at a single common version, by upgrading
   prometheus across our whole estate
 - access metrics endpoints within private networks, which are not
   generally accessible from outside those networks
 - update configuration without having to rebuild instances
 
There are several ways we might provide a prometheus pattern that runs
in other people's infrastructure:

### Provide an artefact which the client team deploys themselves

The artefact we provide could take several forms:

 - an AMI (amazon machine image) which the client team deploys as an
   EC2 instance
 - a terraform module which the client team includes in their own
   terraform project

### Client team provides IAM access so that we can deploy prometheus ourselves

In this model, the client team would create an IAM Role and allow us
to assume it, so that we can build our own prometheus infrastructure
within their VPC.

### Use VPC Peering to provide access for prometheus to scrape target infrastructure

[VPC Peering][] is a feature which allows you to establish a
networking connection between two VPCs.  Crucially, the VPCs can be in
different AWS accounts.

This means that we could build Prometheus in an account we own, and it
could access client team infrastructure over the peering connection.
It also provides the option for us having a single prometheus scraping
multiple VPCs (via multiple peering connections).

[VPC Peering]: https://docs.aws.amazon.com/AmazonVPC/latest/PeeringGuide/Welcome.html

While VPC peering would be a neat fit in this specific situation, we
have some concerns about it:

 - as RE builds more services that might be provided to client teams,
   and as we extend these services to more client teams, we end up
   with N*M VPC peering connections to maintain
 - the client teams' security analysis becomes much harder when they
   peer to an external black-box VPC
 - if we have a single VPC connecting to multiple client VPCs, that
   provides an indirect link between VPCs which the client had wanted
   to keep separate

## Decision

## Consequences

