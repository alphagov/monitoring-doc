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

Some of the things we would like to be able to do are:

 - access metrics endpoints within private networks, which are not
   generally accessible from outside those networks
 - maintain prometheus at a single common version, by upgrading
   prometheus across our whole estate
 - update prometheus configuration without having to rebuild instances
 - allow client teams to provide configuration (say, for alert rules)
 
There are several ways we might provide a prometheus pattern that
allows us to scrape private endpoints:

 - provide an artefact to be deployed by the client team
 - client team provides IAM access to us and we deploy prometheus
   ourselves
 - we build in our own infrastructure and use VPC peering to access
   client team's private networks

### Provide an artefact to be deployed by the client team

The artefact we provide could take several forms:

 - an AMI (amazon machine image) which the client team deploys as an
   EC2 instance
 - a terraform module which the client team includes in their own
   terraform project

This model has the downside that it doesn't allow us to maintain
prometheus at a single common version, because we are at the mercy of
client teams' deploy cadences to ensure things get upgraded.

### Client team provides IAM access so that we can deploy prometheus ourselves

In this model, the client team would create an IAM Role and allow us
to assume it, so that we can build our own prometheus infrastructure
within their VPC.

This would mean that the client team needs to do some work to provide
us with the correct IAM policies so that we can do what we need to do,
without giving us more capability than they feel comfortable with.

The client team would have visibility over what we had built, and
would be able to see it in their own AWS console.  However, they would
likely not have ssh access to the instance itself.

### Use VPC Peering to provide access for prometheus to scrape target infrastructure

[VPC Peering][] is a feature which allows you to establish a
networking connection between two VPCs.  In particular, this is a
peering arrangement which means that the two networks on either side
of the VPC Peering arrangement cannot share the same IP address
ranges.

Crucially for us, the VPCs can be in different AWS accounts.

This means that we could build Prometheus in an account we own, and it
could access client team infrastructure over the peering connection.
Running our own infrastructure in our own account without being
dependent on client teams providing anything to us would make for
smoother operations and deployments for us.

This has a drawback for the client in that it adds extra points of
ingress and egress for traffic in the client networks.  This increases
the attack surface of the network which makes doing a security audit
harder and makes it harder to have confidence in the security of the
system.

There's also a drawback in terms of the combination of connections: as
RE builds more services that might be provided to client teams, and as
we extend these services to more client teams, we end up with N*M VPC
peering connections to maintain.

As RE (and techops more broadly) provides more services, client teams
end up having to consider more VPC Peering connections in their
security analyses, and this doesn't feel like a particularly scalable
way for us to offer services to client teams.

Finally, we believe that VPC Peering is something that has to be
accepted manually on the receiving account console (at least when
peering across AWS accounts), which compounds the scaling problem.

There are two sub cases worth exploring here:

#### A single prometheus VPC peers with multiple client VPCs

In this model, we would build a single prometheus service in a VPC
owned by us, and it would have VPC peering arrangements with multiple
client team VPCs in order to scrape metrics from each of them.

This has the benefit that we can run fewer instances to scrape metrics
from the same number of environments.

This has some drawbacks:

 - the single VPC becomes more privileged, because it has access to
   more environments.  This means that a compromise of the prometheus
   VPC could lead to a compromise of more clients' VPCs.

#### A single prometheus VPC peers with only a single client VPC

In this scenario, we would build a single prometheus in its own VPC
for each client team VPC we offer the service to.

This avoids some of the drawbacks of the previous case, in that the
prometheus doesn't have privileges to access multiple separate VPCs.

[VPC Peering]: https://docs.aws.amazon.com/AmazonVPC/latest/PeeringGuide/Welcome.html
   
## Decision

## Consequences

We haven't yet solved the problem of how client teams provide alert
configuration to prometheus.

For the moment, it is acceptable to define configuration in
cloud-init, but this does not meet our need to update configuration
without rebuilding instances so we will need to revise this in future.
